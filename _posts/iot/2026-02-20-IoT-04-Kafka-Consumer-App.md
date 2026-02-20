---
title:  "[IoT] MQTT -> Kafka를 이용한 데이터 파이프라인 구축하기 #4"
excerpt: "Kafka Consumer를 활용한 데이터 소비"

tags:
  - [Kafka, Consumer, Data-Processing]

toc: true
toc_sticky: true
toc_label: "[Kafka 컨슈머 구현]"
 
date: 2026-02-20
last_modified_at: 2026-02-20
---

### IoT 데이터 파이프라인 구축 시리즈

1. [인트로](https://ymkmoon.github.io/IoT-01_Intro/)

2. [EMQX 브로커 설정 및 데이터 수집](https://ymkmoon.github.io/IoT-02-EMQX-Setup/)

3. [EMQX와 Kafka 연동](https://ymkmoon.github.io/IoT-03-EMQX-Kafka-Integration/)

4. [Kafka Consumer를 활용한 데이터 소비(현재글)](https://ymkmoon.github.io/IoT-04-Kafka-Consumer-App/)

---

이번 글에서는 Kafka 토픽(`vehicle-data`)의 메시지를 Spring Boot Consumer에서 수신하고, 비즈니스 로직 처리 후 MongoDB에 저장하는 과정을 단계별로 정리합니다.

## 프로젝트 의존성 준비 (Kafka Consumer + MongoDB)

먼저 Consumer 애플리케이션이 Kafka 메시지를 읽고 저장할 수 있도록 핵심 의존성을 구성합니다.

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation group: 'org.springframework.kafka', name: 'spring-kafka', version: '3.0.12'
    implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
    implementation 'com.fasterxml.jackson.core:jackson-databind'

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
}
```

## Spring Profile 및 Kafka Consumer 환경 설정

`application.yml`에서 기본 프로파일을 `dev`로 지정하고, `application-dev.yml`에서 Kafka 브로커/컨슈머 그룹/역직렬화 설정을 정의합니다.

```yaml
# application.yml
spring:
  profiles:
    active: dev
```

```yaml
# application-dev.yml
spring:
  kafka:
    consumer:
      group-id: vehicle-consumer-group
      bootstrap-servers: localhost:19092,localhost:19093,localhost:19094
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

MongoDB 저장을 위한 설정도 동일 파일에서 함께 관리합니다.

```yaml
spring:
  data:
    mongodb:
      write-uri: mongodb://localhost:27017
      read-uri: mongodb://localhost:27017
      database: iot_vehicle
```

## Kafka Consumer Factory 및 Listener Container 구성

스프링 빈으로 ConsumerFactory와 ListenerContainerFactory를 등록해 런타임 컨슈머 동작을 정의합니다.

```java
@Configuration
public class KafkaConsumerConfig {

    @Value("${spring.kafka.consumer.group-id}")
    private String groupName;

    @Value("${spring.kafka.consumer.bootstrap-servers}")
    private String serverName;

    @Value("${spring.kafka.consumer.auto-offset-reset}")
    private String autoOffsetReset;

    @Bean
    public ConsumerFactory<String, String> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, serverName);
        props.put(ConsumerConfig.GROUP_ID_CONFIG, groupName);
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, autoOffsetReset);
        return new DefaultKafkaConsumerFactory<>(props);
    }

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(3); // 파티션 수에 맞춰 조정
        return factory;
    }
}
```

## 수신 메시지 스키마(JSON)와 도메인 모델 정의

Kafka에서 수신하는 payload는 차량 텔레메트리 JSON이며, `VehicleDataEntity`로 매핑합니다.

```json
{
  "imei": "860000000000001",
  "speed": 57,
  "location": {
    "lat": 37.5665,
    "lng": 126.9780
  },
  "time": "2026-02-20T10:11:12Z"
}
```

```java
@Document(collection = "tb_vehicle_operation")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class VehicleDataEntity {
    @Id
    private String id;
    private String imei;
    private int speed;
    private Location location;
    private String time;

    @Getter
    @Setter
    @NoArgsConstructor
    @AllArgsConstructor
    public static class Location {
        private double lat;
        private double lng;
    }
}
```

## KafkaListener로 메시지 소비 및 비즈니스 처리

컨슈머 서비스는 `vehicle-data` 토픽을 구독하고, Base64 디코딩 -> JSON 역직렬화 -> MongoDB 저장 순서로 처리합니다.

```java
@Service
@RequiredArgsConstructor
public class SampleCousumerService {

    private final SampleWriteRepository writeRepository;
    private final ObjectMapper mapper;

    @KafkaListener(topics = "vehicle-data", groupId = "vehicle-consumer-group")
    public void consume(String message) {
        try {
            String cleanMessage = message.replaceAll("(^\")|(\"$)", "");
            byte[] decodedBytes = Base64.getDecoder().decode(cleanMessage);
            String decodedMessage = new String(decodedBytes, StandardCharsets.UTF_8);

            VehicleDataEntity vehicle = mapper.readValue(decodedMessage, VehicleDataEntity.class);
            vehicle.setId(null); // 항상 insert
            writeRepository.save(vehicle);
        } catch (Exception e) {
            // 예외 발생 시 로깅 후 재처리 전략 고려
        }
    }
}
```

쓰기/읽기 분리 템플릿을 통해 저장소 접근 역할을 분리할 수 있습니다.

```java
@Repository
public class SampleWriteRepository {
    private final MongoTemplate mongoTemplate;

    public SampleWriteRepository(@Qualifier("mongoWriteTemplate") MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
    }

    public void save(VehicleDataEntity vehicleData) {
        mongoTemplate.save(vehicleData);
    }
}
```

## MongoDB 연결 설정 (Write/Read 분리)

`MongoTemplate`을 write/read로 분리하면 추후 레플리카셋 또는 읽기 확장 구조로 자연스럽게 확장할 수 있습니다.

```java
@Configuration
public class MongoConfig {

    @Value("${spring.data.mongodb.write-uri}")
    private String writeUri;

    @Value("${spring.data.mongodb.read-uri}")
    private String readUri;

    @Value("${spring.data.mongodb.database}")
    private String databaseName;

    @Bean(name = "mongoWriteTemplate")
    @Primary
    public MongoTemplate mongoWriteTemplate() {
        MongoClient client = MongoClients.create(writeUri);
        return new MongoTemplate(client, databaseName);
    }

    @Bean(name = "mongoReadTemplate")
    public MongoTemplate mongoReadTemplate() {
        MongoClient client = MongoClients.create(readUri);
        return new MongoTemplate(client, databaseName);
    }
}
```

## 실행 및 메시지 소비 테스트

### 애플리케이션 실행

```bash
./gradlew bootRun
```

### Docker 기반 Kafka 브로커가 준비되어 있다면 토픽으로 테스트 메시지 발행

아래는 예시 명령입니다(브로커 컨테이너/네트워크 이름은 환경에 맞게 수정).

```bash
docker exec -it kafka-1 kafka-console-producer \
  --broker-list localhost:9092 \
  --topic vehicle-data
```

발행할 메시지(JSON)를 Base64로 인코딩해서 입력합니다.

```bash
echo '{"imei":"860000000000001","speed":57,"location":{"lat":37.5665,"lng":126.9780},"time":"2026-02-20T10:11:12Z"}' | base64
```

### Consumer 로그 확인

정상 수신 시 스레드/메시지/주요 필드 로그가 출력됩니다.

```text
Thread: org.springframework.kafka.KafkaListenerEndpointContainer#0-0-C-1 || message: eyJpbWVpIjoiODYwMDAwMDAwMDAwMDAxIiwic3BlZWQiOjU3LCJsb2NhdGlvbiI6eyJsYXQiOjM3LjU2NjUsImxuZyI6MTI2Ljk3ODB9LCJ0aW1lIjoiMjAyNi0wMi0yMFQxMDoxMToxMloifQ==
IMEI: 860000000000001, Time: 2026-02-20T10:11:12Z, Lat: 37.5665, Lng: 126.978, Speed: 57
```

## 운영 시 체크 포인트

아래 항목을 먼저 점검하면 Consumer 안정성을 빠르게 높일 수 있습니다.

```yaml
# 권장 점검 항목(예시)
consumer:
  retry-policy: "DLQ 또는 재시도 토픽 적용"
  offset-strategy: "enable-auto-commit/manual-ack 정책 명확화"
  idempotency: "중복 소비 대비 키(imei+time 등) 설계"
  observability: "처리량/지연/실패율 지표 수집"
```

- `concurrency`는 토픽 파티션 수에 맞춰 조정
- 실패 메시지는 DLQ(Dead Letter Queue)로 분리해 운영 안정성 확보
- 스키마(JSON 필드) 변경 시 Consumer 역직렬화 호환성 확인
- MongoDB 인덱스(예: `imei`, `time`)로 조회 성능 사전 확보

---

이로써 Kafka에 적재된 IoT 데이터를 Consumer 애플리케이션에서 안정적으로 소비하고, 비즈니스 데이터 저장소로 연결하는 기본 구현이 완성됩니다.
