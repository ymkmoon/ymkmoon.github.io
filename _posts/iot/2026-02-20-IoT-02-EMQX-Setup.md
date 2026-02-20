---
title:  "[IoT] 02. EMQX 브로커 설정 및 데이터 수집"
excerpt: "IoT 기기 연결을 위한 EMQX Broker 설치 및 기본 설정 방법"

tags:
  - [IoT, MQTT, EMQX]

toc: true
toc_sticky: true
toc_label: "[EMQX 설정]"
 
date: 2026-02-20
last_modified_at: 2026-02-20
---

### IoT 데이터 파이프라인 구축 시리즈

1. [인트로](https://ymkmoon.github.io/IoT-01_Intro/)

2. [EMQX 브로커 설정 및 데이터 수집(현재글)](https://ymkmoon.github.io/IoT-02-EMQX-Setup/)

3. [EMQX와 Kafka 연동](https://ymkmoon.github.io/IoT-03-EMQX-Kafka-Integration/)

4. [Kafka Consumer를 활용한 데이터 소비](https://ymkmoon.github.io/IoT-04-Kafka-Consumer-App/)

---

## EMQX 란

EMQX는 IoT(사물인터넷) 및 AI 애플리케이션을 위해 설계된 세계에서 가장 확장성이 뛰어나고 신뢰할 수 있는 오픈 소스 MQTT 메시징 플랫폼이다.


### MQTT 란

MQTT(Message Queuing Telemetry Transport)는 저전력, 낮은 대역폭, 불안정한 네트워크 환경에서도 안정적인 통신을 지원하는 경량 IoT 및 M2M 전용 메시징 프로토콜이다.

Publish-Subscribe(발행-구독) 모델을 사용하여 Publisher(데이터 전송)와 Subscriber(데이터 수신) 사이의 브로커(Broker)가 메시지를 중계하며, 1:N 통신에 매우 유용하다.


---

## 환경 변수 구성

먼저 클러스터 공통 설정과 노드별 포트를 정의한다.  
아래는 `env.local` 기준이니 참고 부탁드립니다.

```env
# 공통 설정
EMQX_ALLOW_ANONYMOUS=true

# 클러스터 설정 (모든 노드 공통)
EMQX_CLUSTER_DISCOVERY_STRATEGY=static
EMQX_CLUSTER_STATIC_SEEDS=emqx@emqx1.mqtt,emqx@emqx2.mqtt,emqx@emqx3.mqtt

# emqx1 포트
EMQX1_MQTT_PORT=1883
EMQX1_DASHBOARD_PORT=8080

# emqx2 포트
EMQX2_MQTT_PORT=1884
EMQX2_DASHBOARD_PORT=8081

# emqx3 포트
EMQX3_MQTT_PORT=1885
EMQX3_DASHBOARD_PORT=8082

# nginx 포트
NGINX_PORT=1886
NGINX_STAGE=local
```

---

## EMQX 3노드 + Nginx 구성 확인

`docker-compose.yml`에서 EMQX 3개 노드와 Nginx(로드밸런서)를 함께 정의합니다.

```yaml
services:
  emqx1:
    image: emqx/emqx:5.7.0
    container_name: emqx1
    environment:
      EMQX_NODE_NAME: emqx@emqx1.mqtt
      EMQX_CLUSTER__DISCOVERY_STRATEGY: ${EMQX_CLUSTER_DISCOVERY_STRATEGY}
      EMQX_CLUSTER__STATIC__SEEDS: ${EMQX_CLUSTER_STATIC_SEEDS}
      EMQX_ALLOW_ANONYMOUS: ${EMQX_ALLOW_ANONYMOUS}
      EMQX_LISTENER__TCP__EXTERNAL: 1883
    ports:
      - "${EMQX1_MQTT_PORT}:1883"
      - "${EMQX1_DASHBOARD_PORT}:18083"

  emqx2:
    image: emqx/emqx:5.7.0
    container_name: emqx2
    environment:
      EMQX_NODE_NAME: emqx@emqx2.mqtt
      EMQX_CLUSTER__DISCOVERY_STRATEGY: ${EMQX_CLUSTER_DISCOVERY_STRATEGY}
      EMQX_CLUSTER__STATIC__SEEDS: ${EMQX_CLUSTER_STATIC_SEEDS}
      EMQX_ALLOW_ANONYMOUS: ${EMQX_ALLOW_ANONYMOUS}
      EMQX_LISTENER__TCP__EXTERNAL: 1883
    ports:
      - "${EMQX2_MQTT_PORT}:1883"
      - "${EMQX2_DASHBOARD_PORT}:18083"

  emqx3:
    image: emqx/emqx:5.7.0
    container_name: emqx3
    environment:
      EMQX_NODE_NAME: emqx@emqx3.mqtt
      EMQX_CLUSTER__DISCOVERY_STRATEGY: ${EMQX_CLUSTER_DISCOVERY_STRATEGY}
      EMQX_CLUSTER__STATIC__SEEDS: ${EMQX_CLUSTER_STATIC_SEEDS}
      EMQX_ALLOW_ANONYMOUS: ${EMQX_ALLOW_ANONYMOUS}
      EMQX_LISTENER__TCP__EXTERNAL: 1883
    ports:
      - "${EMQX3_MQTT_PORT}:1883"
      - "${EMQX3_DASHBOARD_PORT}:18083"

  nginx:
    image: nginx:1.27
    container_name: nginx-mqtt
    environment:
      - NGINX_STAGE=${NGINX_STAGE}
      - NGINX_PORT=${NGINX_PORT}
    ports:
      - "${NGINX_PORT}:${NGINX_PORT}"
```

---

## Nginx MQTT 로드밸런싱 설정

Nginx Stream 설정으로 MQTT 접속을 EMQX 노드에 분산합니다.

```nginx
upstream emqx_backend {
    least_conn;
    server emqx1.mqtt:1883 max_fails=3 fail_timeout=30s;
    server emqx2.mqtt:1883 max_fails=3 fail_timeout=30s;
    server emqx3.mqtt:1883 max_fails=3 fail_timeout=30s;
}

server {
    listen ${NGINX_PORT};
    proxy_pass emqx_backend;
}
```

---

## 클러스터 기동

실행 전 기존 컨테이너/볼륨을 내린 뒤, 환경 파일 기준으로 재기동합니다.

```bash
# 로컬 환경
docker compose -p emqx-cluster --env-file env.local down --volumes --remove-orphans
docker compose -p emqx-cluster --env-file env.local up -d
```

---

## MQTT 메시지 발행/구독 테스트

먼저 Subscriber를 열고, Publisher로 JSON 메시지를 발행해 데이터 수집을 확인합니다.

### 구독 (발행된 메세지 읽기)

```bash
# subscribe
mosquitto_sub -h 127.0.0.1 -p 1886 -t /mqtt/test/# -v

"C:\Program Files (x86)\mosquitto\mosquitto_sub.exe" -h MQTT 브로커 접속 URL -p 포트 -t MQTT 브로커에서 구독할 토픽 -v

"C:\Program Files (x86)\mosquitto\mosquitto_sub.exe" -h 192.168.1.16 -p 1886 -t /mqtt/test/# -v
```

### 발행 (메세지 발행)

```bash
# publish
mosquitto_pub -h 127.0.0.1 -p 1886 -t /mqtt/test/vehicle1 -m "{\"imei\":\"1234\",\"speed\":80,\"time\":\"2026-02-20 10:00:00\"}"

"C:\Program Files (x86)\mosquitto\mosquitto_pub.exe" -h MQTT 브로커 접속 URL -p 포트 -t MQTT 브로커에서 구독할 토픽 -m 메세지

# cleanSession = true
"C:\Program Files (x86)\mosquitto\mosquitto_pub.exe" -h 192.168.1.16 -p 1886 -t /mqtt/test/vehicle1 -m "{\"imei\":\"1234\",\"speed\":80,\"time\":\"2025-08-27 10:00:00\"}"

# cleanSession = false
"C:\Program Files (x86)\mosquitto\mosquitto_pub.exe" -h 192.168.1.16 -p 1886 -t /mqtt/test/vehicle1 -m "{\"imei\":\"0002\",\"speed\":111,\"time\":\"2025-09-01 10:00:00\"}" --repeat 1000 --repeat-delay 1 -q 1 -i vehicle_client_1 -c
```

```json
{
  "imei": "1234",
  "speed": 80,
  "time": "2026-02-20 10:00:00"
}
```

---

## 클러스터 상태 점검

각 노드 상태와 클러스터 구성 노드를 확인합니다.

```bash
docker exec emqx1 /opt/emqx/bin/emqx_ctl status
docker exec emqx2 /opt/emqx/bin/emqx_ctl status
docker exec emqx3 /opt/emqx/bin/emqx_ctl status
```

```bash
docker exec emqx1 /opt/emqx/bin/emqx_ctl cluster status
# running_nodes에 emqx1/emqx2/emqx3가 모두 보여야 정상
```

```bash
docker exec nginx-mqtt tail -f /var/log/nginx/access/stream_access.log
# 요청이 서로 다른 EMQX 백엔드 IP로 분산되는지 확인
```

---

## EMQX 대시보드 접속 및 초기 계정

아래 URL로 각 노드 대시보드에 접속할 수 있습니다.

```text
EMQX1: http://localhost:8080
EMQX2: http://localhost:8081
EMQX3: http://localhost:8082
```

EMQX 기본 관리자 계정은 아래 값으로 시작합니다.

```text
username: admin
password: public
```

최초 로그인 후 보안을 위해 비밀번호를 즉시 변경하는 것을 권장합니다.

