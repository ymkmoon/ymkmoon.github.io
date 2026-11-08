---
title:  "[IoT] MQTT -> Kafka를 이용한 데이터 파이프라인 구축하기 #4"
excerpt: "Kafka Consumer를 활용한 데이터 소비"

categories:
  - Infra

tags:
  - [Kafka, Consumer, Data-Processing]

toc: true
toc_sticky: true
toc_label: "[Kafka 컨슈머 구현]"
 
date: 2026-02-20
last_modified_at: 2026-11-08
---

### IoT 데이터 파이프라인 구축 시리즈

1. [인트로](https://ymkmoon.github.io/IoT-01-Intro/)

2. [EMQX 브로커 설정 및 데이터 수집](https://ymkmoon.github.io/IoT-02-EMQX-Setup/)

3. [EMQX와 Kafka 연동](https://ymkmoon.github.io/IoT-03-EMQX-Kafka-Integration/)

4. [Kafka Consumer를 활용한 데이터 소비(현재글)](https://ymkmoon.github.io/IoT-04-Kafka-Consumer-App/)

---

이번 글에서는 Kafka 토픽(`vehicle-data`)의 메시지를 Spring Boot Consumer에서 수신하고, 비즈니스 로직 처리 후 MongoDB에 저장하는 과정을 단계별로 정리합니다.

> **2026-11 수정:** 처음 글의 Consumer는 예외를 `catch`로 삼키고 있었는데, 실제로 운영해 보니 이 방식은 **MongoDB가 멈춘 동안 들어온 메시지를 모두 버립니다.** 재시도와 실패 토픽(DLT) 구성으로 본문 코드를 바꾸고, 시각 필드와 중복 처리도 함께 보완했습니다.

## 프로젝트 의존성 준비 (Kafka Consumer + MongoDB)

먼저 Consumer 애플리케이션이 Kafka 메시지를 읽고 저장할 수 있도록 핵심 의존성을 구성합니다.

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.kafka:spring-kafka'
    implementation 'org.springframework.boot:spring-boot-starter-data-mongodb'
    implementation 'com.fasterxml.jackson.core:jackson-databind'

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
}
```

`spring-kafka`는 버전을 직접 적지 않고 Spring Boot가 관리하는 버전을 쓰는 편이 안전합니다. 직접 고정하면 Boot를 올릴 때 Kafka 클라이언트와 버전이 어긋날 수 있습니다.

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
    bootstrap-servers: localhost:19092,localhost:19093,localhost:19094
    consumer:
      group-id: vehicle-consumer-group
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    producer:
      # 실패 토픽(DLT)에 기록할 때 쓰는 Producer
      acks: all
```

브로커 주소는 `consumer` 아래가 아니라 `spring.kafka.bootstrap-servers`에 둡니다.  
이번 글에서는 실패 메시지를 DLT로 보내는 Producer와 토픽을 만드는 Admin도 함께 쓰는데, 주소를 `consumer` 아래에만 적으면 이 둘은 기본값인 `localhost:9092`로 접속을 시도합니다.

MongoDB 저장을 위한 설정도 동일 파일에서 함께 관리합니다.

```yaml
spring:
  data:
    mongodb:
      write-uri: mongodb://localhost:27017
      read-uri: mongodb://localhost:27017
      database: iot_vehicle
```

> `write-uri`, `read-uri`는 Spring Boot 표준 키가 아니라 아래 `MongoConfig`에서 직접 읽는 값입니다.

## Kafka Consumer Factory 및 Listener Container 구성

스프링 빈으로 ConsumerFactory와 ListenerContainerFactory를 등록해 런타임 컨슈머 동작을 정의합니다.

```java
@Configuration
public class KafkaConsumerConfig {

    @Value("${spring.kafka.consumer.group-id}")
    private String groupName;

    @Value("${spring.kafka.bootstrap-servers}")
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
    public ConcurrentKafkaListenerContainerFactory<String, String> kafkaListenerContainerFactory(
            DefaultErrorHandler kafkaErrorHandler) {
        ConcurrentKafkaListenerContainerFactory<String, String> factory =
                new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        factory.setConcurrency(3); // 파티션 수에 맞춰 조정
        factory.setCommonErrorHandler(kafkaErrorHandler); // 재시도 · 실패 토픽 (아래 KafkaErrorHandlingConfig)
        return factory;
    }
}
```

> 위 `consumerFactory()`는 yml 값을 그대로 옮기는 코드라, Spring Boot 자동 설정만 써도 같은 동작을 합니다. 설정을 한눈에 보기 위해 직접 등록했습니다.

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

시각은 **두 개로 나눠서** 저장합니다.

| 필드 | 의미 | 채우는 쪽 |
| :--- | :--- | :--- |
| `published_at` | 기기가 메시지를 만든 시각 (payload의 `time`) | 기기 |
| `created_at` | MongoDB에 적재된 시각 | Consumer |

처음에는 `time` 하나를 문자열로 저장했는데, 두 가지 문제가 있었습니다.

- 문자열은 날짜 범위 조회나 정렬이 정확하지 않습니다. `Date` 타입으로 저장해야 `$gte`/`$lt` 조회와 인덱스를 제대로 쓸 수 있습니다.
- Consumer가 멈췄다가 밀린 메시지를 한꺼번에 처리하면 적재 시각이 한 시점에 몰립니다. 기기의 주행 시간 순서로 조회하려면 발행 시각이 따로 있어야 합니다.

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

    @JsonProperty("time")      // payload 의 time 을 받아서
    @Field("published_at")     // published_at(Date) 으로 저장
    private Instant publishedAt;

    @Field("created_at")
    private Instant createdAt;

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

`Instant`는 MongoDB에 `Date` 타입으로 저장됩니다. Spring Boot가 등록하는 `ObjectMapper`는 `"2026-02-20T10:11:12Z"` 같은 ISO-8601 문자열을 `Instant`로 바로 읽습니다.

## 예외 catch 시 메시지 유실 문제

처음 글의 Consumer는 아래처럼 모든 예외를 잡고 아무것도 하지 않았습니다.

```java
// ❌ 처음 글의 코드
@KafkaListener(topics = "vehicle-data", groupId = "vehicle-consumer-group")
public void consume(String message) {
    try {
        // 디코딩 -> 역직렬화 -> 저장
    } catch (Exception e) {
        // 예외 발생 시 로깅 후 재처리 전략 고려
    }
}
```

### 증상

MongoDB가 잠깐 내려간 동안에도 Consumer는 에러 없이 계속 돌았고, MongoDB가 다시 올라온 뒤 확인해 보니 **그 시간대의 데이터가 통째로 비어 있었습니다.**

### 원인

리스너 메서드가 예외 없이 끝나면, 컨테이너는 그 메시지를 **처리 완료로 보고 offset을 확정(commit)** 합니다.  
`catch`가 저장 실패를 삼켰기 때문에 Kafka 입장에서는 정상 처리된 메시지가 되었고, 다음 메시지로 넘어갔습니다. Kafka에 데이터가 남아 있어도 offset이 지나갔으니 다시 읽지 않습니다.

### 해결 (예외는 던지고, 처리는 ErrorHandler에 위임)

리스너에서는 예외를 잡지 않고, 실패를 두 종류로 나눠 컨테이너의 `DefaultErrorHandler`가 처리하게 합니다.

| 실패 종류 | 예시 | 처리 |
| :--- | :--- | :--- |
| 일시 장애 | MongoDB 접속 실패, 타임아웃 | offset을 확정하지 않고 **같은 메시지를 계속 재시도** |
| 메시지 문제 | JSON 형식 오류, Base64 오류 | 몇 번을 다시 해도 실패하므로 **바로 DLT로 보내고** 다음 메시지로 |

메시지 문제를 재시도 대상으로 두면 안 됩니다. 잘못된 메시지 하나가 같은 파티션의 뒤 메시지를 모두 막아 버리기 때문입니다.  
같은 이유로 리스너 안에서 `NullPointerException`처럼 **메시지 내용 때문에 나는 예외**가 생기지 않게 주의해야 합니다. 분류되지 않은 예외는 재시도 대상이 됩니다.

```java
@Configuration
public class KafkaErrorHandlingConfig {

    private static final String DLT_TOPIC = "vehicle-data.DLT";

    // 실패 토픽도 업무 토픽과 같은 파티션 · 복제 수로 미리 만든다
    @Bean
    public NewTopic vehicleDataDltTopic() {
        return TopicBuilder.name(DLT_TOPIC).partitions(3).replicas(3).build();
    }

    @Bean
    public DefaultErrorHandler kafkaErrorHandler(KafkaTemplate<?, ?> kafkaTemplate) {
        // 재시도를 다 써도 실패하면(= 메시지 문제) 원본 메시지를 DLT 로 보낸다
        // 파티션 -1 = 파티션을 지정하지 않음 (DLT 파티션 수가 원본과 달라도 실패하지 않게)
        DeadLetterPublishingRecoverer recoverer = new DeadLetterPublishingRecoverer(kafkaTemplate,
                (record, ex) -> new TopicPartition(DLT_TOPIC, -1));

        // 1초부터 2배씩, 최대 30초 간격으로 재시도. 최대 경과 시간을 두지 않아 일시 장애가 끝날 때까지 기다린다
        ExponentialBackOff backOff = new ExponentialBackOff(1_000L, 2.0);
        backOff.setMaxInterval(30_000L);

        DefaultErrorHandler handler = new DefaultErrorHandler(recoverer, backOff);

        // 다시 해도 같은 결과인 실패 — 재시도 없이 바로 DLT 로
        handler.addNotRetryableExceptions(
                JsonProcessingException.class,   // JSON 형식 오류
                IllegalArgumentException.class); // Base64 디코딩 오류
        return handler;
    }
}
```

포인트:

- 일시 장애일 때 같은 메시지에서 계속 기다리면 그동안 해당 파티션의 처리가 멈춥니다. 의도한 동작입니다. 메시지는 Kafka 보관 기간(기본 7일) 동안 남아 있으므로, MongoDB가 돌아오면 멈춘 위치부터 그대로 이어서 저장합니다.
- DLT로 보낸 메시지에는 원본 토픽 · 파티션 · offset과 예외 메시지가 헤더(`kafka_dlt-*`)로 함께 기록됩니다. 원인을 고친 뒤 다시 넣을 수 있습니다.
- DLT에 기록하는 Producer는 `acks: all`로 둡니다. DLT 기록이 확실히 남아야 원본 offset을 넘겨도 안전합니다.

## KafkaListener로 메시지 소비 및 비즈니스 처리

컨슈머 서비스는 `vehicle-data` 토픽을 구독하고, Base64 디코딩 -> JSON 역직렬화 -> MongoDB 저장 순서로 처리합니다.  
예외는 잡지 않고 그대로 던집니다. 단, 이미 저장된 메시지(중복)만은 예외로 보지 않고 건너뜁니다.

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class VehicleConsumerService {

    private final VehicleWriteRepository writeRepository;
    private final ObjectMapper mapper;

    @KafkaListener(topics = "vehicle-data", groupId = "vehicle-consumer-group")
    public void consume(String message) throws JsonProcessingException {
        String decodedMessage = decode(message);

        // 형식이 틀리면 JsonProcessingException → DLT
        VehicleDataEntity vehicle = mapper.readValue(decodedMessage, VehicleDataEntity.class);
        vehicle.setId(null);
        vehicle.setCreatedAt(Instant.now()); // 적재 시각 (재시도 끝에 저장되면 그때의 시각)

        try {
            // MongoDB 접속 실패는 잡지 않는다 → ErrorHandler 가 재시도
            writeRepository.insert(vehicle);
        } catch (DuplicateKeyException e) {
            // 같은 메시지가 다시 들어온 경우 (QoS 1 · 재시도) — 이미 저장돼 있으니 건너뛴다
            log.info("이미 저장된 메시지 — imei: {}, publishedAt: {}", vehicle.getImei(), vehicle.getPublishedAt());
            return;
        }

        // 로그에서 location 처럼 없을 수도 있는 필드를 바로 꺼내면 NullPointerException → 무한 재시도가 되니 주의
        log.info("IMEI: {}, PublishedAt: {}, Speed: {}",
                vehicle.getImei(), vehicle.getPublishedAt(), vehicle.getSpeed());
    }

    // Connect 의 JsonConverter 가 MQTT 본문을 "Base64 문자열"로 감싸 기록하므로 벗겨낸다
    private String decode(String message) {
        String cleanMessage = message.trim().replaceAll("(^\")|(\"$)", "");
        if (cleanMessage.startsWith("{")) {
            return cleanMessage; // 이미 평문 JSON 이면 그대로
        }
        // Base64 가 아니면 IllegalArgumentException → DLT
        byte[] decodedBytes = Base64.getDecoder().decode(cleanMessage);
        return new String(decodedBytes, StandardCharsets.UTF_8);
    }
}
```

### Base64 디코딩이 필요한 이유

[#3](https://ymkmoon.github.io/IoT-03-EMQX-Kafka-Integration/)의 커넥터는 `value.converter`로 `JsonConverter`를 씁니다.  
MQTT 소스 커넥터가 넘기는 메시지 본문은 byte 배열인데, `JsonConverter`는 byte 배열을 **Base64로 인코딩한 JSON 문자열**로 기록합니다. 그래서 Kafka에는 `"eyJpbWVp..."`처럼 따옴표로 감싼 Base64가 들어오고, Consumer에서 따옴표를 떼고 디코딩해야 원래 JSON이 나옵니다.

> 커넥터의 `value.converter`를 `org.apache.kafka.connect.converters.ByteArrayConverter`로 바꾸면 본문이 그대로 기록되어 디코딩 없이 받을 수 있습니다. 바꿀 때는 커넥터를 재등록한 뒤 `kafka-console-consumer`로 실제 기록 형태를 먼저 확인해주세요. 위 `decode()`는 평문 JSON이 들어와도 그대로 통과시키므로 전환 중에도 동작합니다.

### 중복 저장 방지

QoS 1과 Kafka의 at-least-once 소비에서는 **같은 메시지가 두 번 들어올 수 있습니다.** 예를 들어 저장은 됐는데 offset 확정 직전에 Consumer가 재시작되면 그 메시지를 다시 읽습니다.  
`imei` + `published_at`에 유니크 인덱스를 걸어 두면 두 번째 저장은 `DuplicateKeyException`이 나고, 위 코드에서 건너뜁니다.

```javascript
// mongosh
db.tb_vehicle_operation.createIndex(
  { imei: 1, published_at: 1 },
  { unique: true }
)
```

이 인덱스는 기기별 · 시간 범위 조회에도 그대로 쓰입니다.

쓰기/읽기 분리 템플릿을 통해 저장소 접근 역할을 분리할 수 있습니다.

```java
@Repository
public class VehicleWriteRepository {
    private final MongoTemplate mongoTemplate;

    public VehicleWriteRepository(@Qualifier("mongoWriteTemplate") MongoTemplate mongoTemplate) {
        this.mongoTemplate = mongoTemplate;
    }

    public void insert(VehicleDataEntity vehicleData) {
        mongoTemplate.insert(vehicleData);
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
docker exec -it kafka1 kafka-console-producer \
  --bootstrap-server kafka1:9092 \
  --topic vehicle-data
```

실제 커넥터가 기록하는 형태와 같게 하려면 메시지(JSON)를 Base64로 인코딩해서 입력합니다. 평문 JSON을 그대로 입력해도 처리됩니다.

```bash
echo -n '{"imei":"860000000000001","speed":57,"location":{"lat":37.5665,"lng":126.9780},"time":"2026-02-20T10:11:12Z"}' | base64 -w 0
# macOS 는 base64 (줄바꿈 없이 출력). GNU base64 는 -w 0 이 없으면 76자마다 줄을 바꿔 메시지가 둘로 나뉜다
```

### Consumer 로그 확인

정상 수신 시 주요 필드 로그가 출력됩니다.

```text
IMEI: 860000000000001, PublishedAt: 2026-02-20T10:11:12Z, Speed: 57
```

같은 메시지를 한 번 더 넣으면 중복으로 건너뜁니다.

```text
이미 저장된 메시지 — imei: 860000000000001, publishedAt: 2026-02-20T10:11:12Z
```

### 실패 처리 확인

**메시지 문제 → DLT**

Producer 창에 `not-json` 같은 잘못된 값을 입력하면, 재시도 없이 바로 `vehicle-data.DLT`로 이동하고 다음 메시지는 정상 처리됩니다.

```bash
docker exec -it kafka1 kafka-console-consumer \
  --bootstrap-server kafka1:9092 \
  --topic vehicle-data.DLT \
  --property print.headers=true \
  --from-beginning
# 헤더의 kafka_dlt-exception-message 에서 실패 원인을 확인
```

**일시 장애 → 재시도 후 저장**

MongoDB를 멈춘 상태에서 메시지를 넣고, 다시 켜서 저장되는지 확인합니다(컨테이너 이름은 환경에 맞게 수정).

```bash
docker stop mongodb
# 메시지 발행 → Consumer 로그에 재시도가 1초, 2초, 4초 ... 최대 30초 간격으로 반복
docker start mongodb
# 멈춘 동안의 메시지가 순서대로 저장되면 정상
```

## 운영 시 체크 포인트

아래 항목을 먼저 점검하면 Consumer 안정성을 빠르게 높일 수 있습니다.

- `concurrency`는 토픽 파티션 수에 맞춰 조정 (파티션보다 많으면 남는 스레드는 놉니다)
- DLT에 메시지가 쌓이는지 주기적으로 확인 (쌓였다면 원인을 고친 뒤 원본 토픽으로 다시 발행)
- 일시 장애로 재시도 중인 동안은 Consumer Lag이 늘어나므로, Lag 지표에 알림을 걸어두기
- 스키마(JSON 필드) 변경 시 Consumer 역직렬화 호환성 확인
- 시간 조회 · 정렬은 적재 시각(`created_at`)이 아니라 발행 시각(`published_at`) 기준으로

---

이로써 Kafka에 적재된 IoT 데이터를 Consumer 애플리케이션에서 안정적으로 소비하고, 비즈니스 데이터 저장소로 연결하는 기본 구현이 완성됩니다.
