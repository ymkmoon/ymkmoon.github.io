---
title:  "[IoT] MQTT -> Kafka를 이용한 데이터 파이프라인 구축하기 #3"
excerpt: "EMQX와 Kafka 연동"

categories:
  - Infra

tags:
  - [IoT, EMQX, Kafka]

toc: true
toc_sticky: true
toc_label: "[EMQX-Kafka 통합 설정]"
 
date: 2026-02-20
last_modified_at: 2026-11-08
---

### IoT 데이터 파이프라인 구축 시리즈

1. [인트로](https://ymkmoon.github.io/IoT-01-Intro/)

2. [EMQX 브로커 설정 및 데이터 수집](https://ymkmoon.github.io/IoT-02-EMQX-Setup/)

3. [EMQX와 Kafka 연동(현재글)](https://ymkmoon.github.io/IoT-03-EMQX-Kafka-Integration/)

4. [Kafka Consumer를 활용한 데이터 소비](https://ymkmoon.github.io/IoT-04-Kafka-Consumer-App/)

---

## Kafka Connect 커스텀 이미지 빌드

MQTT Source Connector 플러그인을 포함한 Kafka Connect 이미지를 먼저 준비합니다.

```dockerfile
# Dockerfile
FROM confluentinc/cp-kafka-connect:7.7.4

ARG CAMEL_PLUGIN_PATH="/home/appuser/camel-kafka-connectors"
ARG CAMEL_REPOSITORY_BASE_URL="https://repo.maven.apache.org/maven2/org/apache/camel/kafkaconnector"
ARG CAMEL_PACKAGE_VERSION="4.11.0"

RUN mkdir -p "${CAMEL_PLUGIN_PATH}" \
  && wget -qO- "${CAMEL_REPOSITORY_BASE_URL}/camel-mqtt-source-kafka-connector/${CAMEL_PACKAGE_VERSION}/camel-mqtt-source-kafka-connector-${CAMEL_PACKAGE_VERSION}-package.tar.gz" | tar -C "${CAMEL_PLUGIN_PATH}" -xz

ENV CONNECT_PLUGIN_PATH="/usr/share/java,${CAMEL_PLUGIN_PATH}"
```

### 이미지 빌드

```bash
docker build -t custom-kafka-connect:7.7.4 .
```

---

## Kafka 브로커 클러스터 기본 구성 확인

`docker-compose.yml`에서 브로커 3대, 내부/외부 리스너, 복제 설정을 확인합니다.

> 이 글은 ZooKeeper 기반 구성(cp-kafka 7.7)입니다. Kafka 4.0부터는 ZooKeeper 지원이 제거되어 KRaft 모드만 쓸 수 있으니, 새로 구성한다면 KRaft를 검토해주세요.

```yaml
# docker-compose.yml (Kafka broker 핵심)
kafka1:
  image: confluentinc/cp-kafka:7.7.4
  environment:
    KAFKA_BROKER_ID: 1
    KAFKA_ZOOKEEPER_CONNECT: zookeeper:${ZOOKEEPER_PORT}
    KAFKA_LISTENERS: INTERNAL://0.0.0.0:${KAFKA_INTERNAL_PORT},EXTERNAL://0.0.0.0:${KAFKA_EXTERNAL_PORT}
    KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka1:${KAFKA_INTERNAL_PORT},EXTERNAL://${KAFKA1_ADVERTISED_HOST}:${KAFKA1_HOST_PORT}
    KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: INTERNAL:PLAINTEXT,EXTERNAL:PLAINTEXT
    KAFKA_INTER_BROKER_LISTENER_NAME: INTERNAL
    KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: ${KAFKA_REPLICATION_FACTOR}

kafka2:
  image: confluentinc/cp-kafka:7.7.4
  environment:
    KAFKA_BROKER_ID: 2
    KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka2:${KAFKA_INTERNAL_PORT},EXTERNAL://${KAFKA2_ADVERTISED_HOST}:${KAFKA2_HOST_PORT}
    KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: ${KAFKA_REPLICATION_FACTOR}
    # (나머지 리스너 설정은 kafka1과 동일)

kafka3:
  image: confluentinc/cp-kafka:7.7.4
  environment:
    KAFKA_BROKER_ID: 3
    KAFKA_ADVERTISED_LISTENERS: INTERNAL://kafka3:${KAFKA_INTERNAL_PORT},EXTERNAL://${KAFKA3_ADVERTISED_HOST}:${KAFKA3_HOST_PORT}
    KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: ${KAFKA_REPLICATION_FACTOR}
    # (나머지 리스너 설정은 kafka1과 동일)
```

### 브로커 최적화 포인트

- `INTERNAL/EXTERNAL` : 리스너 분리로 클러스터 내부 통신과 외부 접근을 명확히 분리
- `KAFKA_INTER_BROKER_LISTENER_NAME=INTERNAL` : 브로커 간 트래픽 안정화
- `KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR` : 멀티 브로커 수에 맞춰 설정해 장애 복구성 강화

> 이후 명령어는 `KAFKA_INTERNAL_PORT=9092` 기준입니다. 컨테이너 안에서는 INTERNAL 리스너(`kafka1:9092`)로, 호스트에서는 EXTERNAL 리스너로 접속합니다.

---

## 토픽 선생성(권장) 및 클러스터 기동

`topic-creator` 컨테이너를 통해 `vehicle-data` 토픽을 사전 생성합니다.

> 토픽이 없는 상태에서 커넥터가 먼저 메시지를 쓰면, 브로커가 토픽을 **기본값으로 자동 생성**합니다. 기본값(`num.partitions`, `default.replication.factor`)을 따로 지정하지 않았다면 파티션 1개 · 복제 1개 토픽이 만들어져, 브로커 3대를 두고도 분산과 복제가 되지 않습니다. 한 번 만들어진 토픽의 복제 계수는 쉽게 바꿀 수 없으니 꼭 토픽을 미리 생성해주세요.


### Docker Compose 를 이용한 토픽 생성

```yaml
# docker-compose.yml (topic-creator)
topic-creator:
  image: confluentinc/cp-kafka:7.7.4
  depends_on:
    kafka1:
      condition: service_healthy
    kafka2:
      condition: service_healthy
    kafka3:
      condition: service_healthy
  command:
    - bash
    - -c
    - |
      echo "Waiting for Kafka brokers to be ready..."
      cub kafka-ready -b kafka1:${KAFKA_INTERNAL_PORT} 3 30
      echo "Creating vehicle-data topic..."
      kafka-topics --create --if-not-exists --topic vehicle-data --bootstrap-server kafka1:${KAFKA_INTERNAL_PORT} --partitions ${KAFKA_REPLICATION_PARTITIONS} --replication-factor ${KAFKA_REPLICATION_FACTOR}
      echo "Topic creation completed."
```

### 수동 토픽 생성

```bash
kafka-topics --create \
  --topic 토픽이름 \
  --bootstrap-server Kafka 브로커 접속 주소 \
  --partitions 3 \
  --replication-factor 3

docker exec -it kafka1 kafka-topics --create \
  --bootstrap-server kafka1:9092,kafka2:9092,kafka3:9092 \
  --replication-factor 3 \
  --partitions 3 \
  --topic vehicle-data
```


### 클러스터 실행

```bash
# 로컬 예시
docker compose -p kafka-cluster --env-file kafka.local.env down --volumes --remove-orphans
docker compose -p kafka-cluster --env-file kafka.local.env up -d
```

---

## Kafka Connect 분산 모드 구성

Connect 노드를 3대로 구성하고, 동일한 `CONNECT_GROUP_ID`로 분산 실행합니다.

```yaml
# docker-compose.yml (Kafka Connect 핵심)
kafka-connect1:
  image: custom-kafka-connect:7.7.4
  environment:
    CONNECT_BOOTSTRAP_SERVERS: "kafka1:${KAFKA_INTERNAL_PORT},kafka2:${KAFKA_INTERNAL_PORT},kafka3:${KAFKA_INTERNAL_PORT}"
    CONNECT_REST_ADVERTISED_HOST_NAME: "kafka-connect1"
    CONNECT_REST_PORT: ${KAFKA_CONNECT_INTERNAL_PORT}
    CONNECT_GROUP_ID: "vehicle-connect-group"
    CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: ${KAFKA_REPLICATION_FACTOR}
    CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: ${KAFKA_REPLICATION_FACTOR}
    CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: ${KAFKA_REPLICATION_FACTOR}
    CONNECT_KEY_CONVERTER: "org.apache.kafka.connect.storage.StringConverter"
    CONNECT_VALUE_CONVERTER: "org.apache.kafka.connect.storage.StringConverter"
    CONNECT_VALUE_CONVERTER_SCHEMAS_ENABLE: "false"
  volumes:
    - ./mqtt-connector.json:/etc/kafka-connect/mqtt-connector.json
    - ./kafka-init:/kafka-init
```

여기서 분산 모드의 이점은 **처리량이 아니라 장애 대비**입니다.  
MQTT 소스 커넥터는 EMQX에 구독자 하나로 붙기 때문에 일을 나눌 수 없고, task는 1개만 돕니다. Connect 노드 1대가 죽으면 같은 그룹의 다른 노드가 그 task를 이어받습니다.

---

## MQTT Source Connector 설정

EMQX(MQTT 브로커)에서 구독한 메시지를 Kafka `vehicle-data` 토픽으로 보냅니다.

> 아래 파일은 `curl`로 그대로 보낼 파일이라 주석을 넣지 않았습니다. JSON은 `//` 주석을 허용하지 않아서, 주석이 있으면 등록 요청이 실패합니다. 각 항목 설명은 아래 표를 참고해주세요.

```json
{
  "name": "mqtt-source-connector",
  "config": {
    "connector.class": "CONNECTOR CLASS",
    "tasks.max": "1",
    "topics": "vehicle-data", // Kafka 브로커에 메시지가 기록될 대상 토픽명
    "camel.kamelet.mqtt-source.brokerUrl": "tcp://localhost:1883", // MQTT 브로커 접속 URL (프로토콜 + 호스트 + 포트)
    "camel.kamelet.mqtt-source.clientId": "mqtt-source", // MQTT 클라이언트 ID. 브로커에서 고유하게 식별하는 ID
    "camel.kamelet.mqtt-source.topic": "/mqtt/test/#", // MQTT 브로커에서 구독할 토픽 (와일드카드 # 지원)
    "camel.kamelet.mqtt-source.username": null, // MQTT 브로커 접속에 필요한 사용자명 (없으면 null)
    "camel.kamelet.mqtt-source.password": null, // MQTT 브로커 접속에 필요한 비밀번호 (없으면 null)
    "camel.kamelet.mqtt-source.qos": "1", // MQTT QoS 레벨 (0: At most once, 1: At least once, 2: Exactly once)
    "camel.kamelet.mqtt-source.clean.session.enabled": true, // true면 연결 재시작 시 이전 세션 상태를 유지하지 않음
    "camel.kamelet.mqtt-source.connect.timeout.seconds": 30, // MQTT 브로커 연결 시도 타임아웃(초)
    "camel.kamelet.mqtt-source.keepalive.interval.seconds": 60, // MQTT keep-alive 간격(초)
    "camel.kamelet.max.retry.time.ms": 30000, // 재시도 최대 대기 시간(ms)
    "camel.aggregation.size": 10, // 메시지를 모아서 처리할 개수(배치 크기)
    "camel.aggregation.timeout": 500, // 메시지 집계 대기 시간(ms)
    "camel.beans.aggregate": null, // 커스텀 Aggregation 전략 Bean 지정 가능
    "camel.error.handler": "default", // 오류 처리 전략 (default, deadletter 등)
    "camel.error.handler.max.redeliveries": 0, // 에러 발생 시 재시도 최대 횟수
    "camel.error.handler.redelivery.delay": 1000, // 재시도 간 대기 시간(ms)
    "camel.idempotency.enabled": false, // 중복 메시지 방지를 위한 멱등성 활성화 여부
    "camel.idempotency.expression.header": null, // 멱등성 키를 추출할 Kafka 메시지 헤더
    "camel.idempotency.expression.type": "body", // 멱등성 키를 메시지 본문에서 추출할지 여부
    "camel.idempotency.repository.type": "memory", // 멱등성 저장소 타입 (memory / kafka)
    "camel.idempotency.memory.dimension": 100, // memory 저장소 크기 제한
    "camel.idempotency.kafka.bootstrap.servers": "localhost:9092", // Kafka 멱등성 저장소를 사용할 경우 연결할 Kafka 클러스터 주소
    "camel.idempotency.kafka.topic": "kafka_idempotent_repository", // 멱등성 저장소용 Kafka 토픽명
    "camel.idempotency.kafka.max.cache.size": 1000, // Kafka 멱등성 저장소 캐시 최대 크기
    "camel.idempotency.kafka.poll.duration.ms": 100, // Kafka 멱등성 저장소 poll 주기(ms)
    "camel.map.headers": true, // 메시지 헤더를 매핑할지 여부
    "camel.map.properties": true, // 메시지 프로퍼티를 매핑할지 여부
    "camel.remove.headers.pattern": "", // 제거할 헤더 패턴 지정
    "camel.source.camelMessageHeaderKey": null, // Kafka 메시지 키를 Camel 메시지 헤더에서 추출
    "camel.source.component": null, // 사용할 Camel 컴포넌트 이름 (예: mqtt5)
    "camel.source.contentLogLevel": "OFF", // 메시지 콘텐츠 로그 레벨 (OFF, INFO, DEBUG)
    "camel.source.marshal": null, // 메시지를 직렬화할 포맷 지정 (예: json, avro)
    "camel.source.unmarshal": null, // 메시지를 역직렬화할 포맷 지정 (예: json, avro)
    "camel.source.maxBatchPollSize": 1000, // Kafka Connect가 한 번에 poll할 최대 메시지 개수
    "camel.source.maxNotCommittedRecords": 1024, // 커밋되지 않은 최대 레코드 수 제한
    "camel.source.maxPollDuration": 1000, // poll 대기 시간(ms)
    "camel.source.pollingConsumerBlockTimeout": 0, // polling 시 block timeout (0이면 무제한 대기)
    "camel.source.pollingConsumerBlockWhenFull": true, // 큐가 가득 찼을 때 block 여부
    "camel.source.pollingConsumerQueueSize": 1000, // polling consumer 내부 큐 크기
    "camel.source.url": null, // Camel source용 URL (특정 컴포넌트 기반일 때만 사용)
    "key.converter": "org.apache.kafka.connect.storage.StringConverter", // Kafka 메시지 key 직렬화 방식
    "key.converter.schemas.enable": false, // key에 스키마 포함 여부
    "value.converter": "org.apache.kafka.connect.json.JsonConverter", // Kafka 메시지 value 직렬화 방식
    "value.converter.schemas.enable": false // value에 스키마 포함 여부
  }
}

```

| 항목 | 설명 |
| :--- | :--- |
| `connector.class` | Camel MQTT Source 커넥터 클래스 |
| `tasks.max` | 반드시 `1` (아래 설명 참고) |
| `topics` | Kafka 브로커에 메시지가 기록될 대상 토픽명 |
| `brokerUrl` | MQTT 브로커 접속 URL (프로토콜 + 호스트 + 포트) |
| `clientId` | EMQX가 이 커넥터를 식별하는 ID. 지속 세션을 쓰려면 고정값이어야 함 |
| `topic` | EMQX에서 구독할 MQTT 토픽 (와일드카드 `#` 지원) |
| `qos` | `1` = At least once. 유실 대신 중복을 허용하고 중복은 Consumer에서 처리 |
| `clean.session.enabled` | `false` = 지속 세션. 커넥터가 재시작되는 동안의 메시지를 EMQX가 보관 |
| `connect.timeout.seconds` / `keepalive.interval.seconds` | 연결 시도 타임아웃 / keep-alive 간격(초) |
| `max.retry.time.ms` | 연결 재시도 최대 대기 시간(ms) |
| `key.converter` / `value.converter` | Kafka에 기록할 key / value 직렬화 방식 |

Camel 커넥터에는 이 밖에도 aggregation, idempotency 관련 옵션이 많지만, 기본값으로 두어도 동작합니다. 위 항목만 상황에 맞게 조정하면 충분합니다.

실무 팁:

- **`brokerUrl`에 `localhost`를 쓰면 안 됩니다.** 커넥터는 Connect 컨테이너 안에서 돌기 때문에 `localhost`는 Connect 컨테이너 자신을 가리킵니다. 같은 Docker 네트워크라면 `nginx-mqtt` 같은 서비스 이름을, 아니라면 호스트 주소를 넣어주세요.
- **`clean.session.enabled`는 `false`로 둡니다.** `true`면 커넥터가 재시작되거나 재배포되는 동안 발행된 메시지는 받을 세션이 없어 그대로 사라집니다. [#2](https://ymkmoon.github.io/IoT-02-EMQX-Setup/)의 지속 세션 조건(고정 Client ID + QoS 1 이상)이 커넥터에도 똑같이 적용됩니다.
- **`tasks.max`를 늘리면 메시지가 중복됩니다.** task마다 같은 MQTT 토픽을 따로 구독하므로 같은 메시지가 task 수만큼 Kafka에 기록됩니다. Client ID가 고정이라면 task끼리 서로의 연결을 끊어 버리기도 합니다.
- **MQTT 토픽은 발행하는 쪽과 정확히 맞춰야 합니다.** 발행 토픽을 바꾸고 커넥터 `topic`을 그대로 두면 에러 없이 조용히 수집이 멈춥니다. 토픽을 바꿀 때는 양쪽을 함께 바꿔주세요.
- `value.converter`가 `JsonConverter`이면 MQTT 본문(byte 배열)이 Base64 문자열로 기록됩니다. Consumer에서 이를 어떻게 처리하는지는 [#4](https://ymkmoon.github.io/IoT-04-Kafka-Consumer-App/)에서 다룹니다.

---

## 커넥터 등록

`POST /connectors`는 처음 한 번만 성공하고, 같은 이름의 커넥터가 있으면 `409 Conflict`로 실패합니다.  
설정을 바꿀 때마다 삭제 후 재등록하게 되므로, **`PUT /connectors/{name}/config`**를 쓰는 편이 좋습니다. 없으면 만들고 있으면 설정을 바꾸기 때문에, 몇 번을 실행해도 결과가 같습니다.

PUT은 `name`을 경로로 받고 본문에는 `config` 객체만 보냅니다. 아래 예시는 `jq`로 `config` 부분만 꺼내 보냅니다(`jq`가 없다면 `config` 내용만 담은 파일을 따로 두고 `--data @파일명`으로 보내도 됩니다).

```bash
# 최초 등록 (한 번만)
curl -X POST -H "Content-Type: application/json" http://localhost:28083/connectors --data @mqtt-connector.json

# 등록 + 변경 (권장 — 여러 번 실행해도 안전)
jq .config mqtt-connector.json | \
  curl -X PUT -H "Content-Type: application/json" \
  http://localhost:28083/connectors/mqtt-source-connector/config --data @-
```

---

## 동작 검증 (토픽/메시지/분산 처리)

브로커와 커넥터가 정상 연동되면, 토픽/메시지 적재 상태를 순서대로 검증합니다.

```bash
# 토픽 목록/상세 확인
docker exec -it kafka1 kafka-topics --list --bootstrap-server kafka1:9092
docker exec -it kafka1 kafka-topics --describe --topic vehicle-data --bootstrap-server kafka1:9092

# 메시지 소비 확인(파티션 출력)
docker exec -it kafka1 kafka-console-consumer --bootstrap-server kafka1:9092 --topic vehicle-data --property print.partition=true --from-beginning

# 파티션 오프셋 확인(분산 적재 체크)
docker exec -it kafka1 kafka-get-offsets --bootstrap-server kafka1:9092 --topic vehicle-data

# 커넥터 상태 확인
curl http://localhost:28083/connectors/mqtt-source-connector/status

## 커넥터와 실패한 task 재시작 (task 번호는 0부터 시작)
curl -X POST "http://localhost:28083/connectors/mqtt-source-connector/restart?includeTasks=true&onlyFailed=true"

## 커넥터 삭제
curl -X DELETE http://localhost:28083/connectors/mqtt-source-connector
```

> task를 하나만 재시작하려면 `POST /connectors/{name}/tasks/{taskId}/restart`를 씁니다. `tasks.max=1`이면 task는 `0`번 하나뿐이라 `tasks/1`은 404가 납니다.

---

## 운영 시 체크리스트

- EMQX 접속 주소와 Kafka 외부 리스너 주소가 서로 통신 가능한 네트워크인지 확인
- 토픽 복제 계수(`replication-factor`)와 브로커 수 불일치 여부 점검
- Connect 내부 토픽과 업무 토픽 모두 보존 정책/파티션 수를 운영 목적에 맞게 사전 설계
- 커넥터를 재시작하는 동안 발행한 메시지가 Kafka에 들어오는지(지속 세션) 확인
- 장애 테스트(브로커 1대/Connect 1대 다운)로 복구 시간과 데이터 연속성 검증
