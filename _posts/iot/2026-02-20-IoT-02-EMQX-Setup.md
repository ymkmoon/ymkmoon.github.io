---
title:  "[IoT] MQTT -> Kafka를 이용한 데이터 파이프라인 구축하기 #2"
excerpt: "EMQX 브로커 설정 및 데이터 수집"

categories:
  - Infra

tags:
  - [IoT, MQTT, EMQX]

toc: true
toc_sticky: true
toc_label: "[EMQX 설정]"
 
date: 2026-02-20
last_modified_at: 2026-11-08
---

### IoT 데이터 파이프라인 구축 시리즈

1. [인트로](https://ymkmoon.github.io/IoT-01-Intro/)

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

```
# env.local

# 클러스터 노드끼리 서로를 인증하는 값 (세 노드가 같아야 한다)
EMQX_COOKIE=change-me-cluster-cookie

# emqx1 포트 (호스트에서 직접 붙어볼 때 사용)
EMQX1_MQTT_PORT=1883
EMQX1_DASHBOARD_PORT=8080

# emqx2 포트
EMQX2_MQTT_PORT=1884
EMQX2_DASHBOARD_PORT=8081

# emqx3 포트
EMQX3_MQTT_PORT=1885
EMQX3_DASHBOARD_PORT=8082

# nginx 포트 (기기는 이 포트로 접속한다)
NGINX_PORT=1886
```

---

## EMQX 3노드 + Nginx 구성 확인

`docker-compose.yml`에서 EMQX 3개 노드와 Nginx(로드밸런서)를 함께 정의합니다.

```yaml
x-emqx-env: &emqx-env
  EMQX_CLUSTER__DISCOVERY_STRATEGY: static
  EMQX_CLUSTER__STATIC__SEEDS: "[emqx@emqx1.mqtt,emqx@emqx2.mqtt,emqx@emqx3.mqtt]"
  EMQX_NODE__COOKIE: ${EMQX_COOKIE}
  EMQX_LISTENERS__TCP__DEFAULT__BIND: "0.0.0.0:1883"

services:
  emqx1:
    image: emqx/emqx:5.7.0
    container_name: emqx1
    environment:
      <<: *emqx-env
      EMQX_NODE_NAME: emqx@emqx1.mqtt
    ports:
      - "${EMQX1_MQTT_PORT}:1883"
      - "${EMQX1_DASHBOARD_PORT}:18083"
    networks:
      mqtt:
        aliases: [emqx1.mqtt]

  emqx2:
    image: emqx/emqx:5.7.0
    container_name: emqx2
    environment:
      <<: *emqx-env
      EMQX_NODE_NAME: emqx@emqx2.mqtt
    ports:
      - "${EMQX2_MQTT_PORT}:1883"
      - "${EMQX2_DASHBOARD_PORT}:18083"
    networks:
      mqtt:
        aliases: [emqx2.mqtt]

  emqx3:
    image: emqx/emqx:5.7.0
    container_name: emqx3
    environment:
      <<: *emqx-env
      EMQX_NODE_NAME: emqx@emqx3.mqtt
    ports:
      - "${EMQX3_MQTT_PORT}:1883"
      - "${EMQX3_DASHBOARD_PORT}:18083"
    networks:
      mqtt:
        aliases: [emqx3.mqtt]

  nginx:
    image: nginx:1.27
    container_name: nginx-mqtt
    depends_on: [emqx1, emqx2, emqx3]
    environment:
      - NGINX_PORT=${NGINX_PORT}
    ports:
      - "${NGINX_PORT}:${NGINX_PORT}"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/stream.conf.template:/etc/nginx/templates/stream/stream.conf.template:ro
    networks:
      - mqtt

networks:
  mqtt:
```

포인트:

- 환경변수 이름은 **EMQX 5.x 형식**(`EMQX_` + 설정 경로, 점은 `__`)이어야 합니다. 예전 글에서 쓰던 `EMQX_LISTENER__TCP__EXTERNAL`, `EMQX_ALLOW_ANONYMOUS`는 4.x 형식이라 5.7에서는 적용되지 않습니다.
- `EMQX_NODE_NAME`의 `@` 뒤 호스트(`emqx1.mqtt`)는 다른 노드가 실제로 찾아갈 수 있는 이름이어야 합니다. 네트워크 `aliases`가 없으면 시드 목록의 노드를 찾지 못해 **노드 3개가 각자 따로 뜹니다.**
- 세 노드의 `EMQX_NODE__COOKIE`가 다르면 서로 클러스터로 묶이지 않습니다.

> EMQX 5.x는 인증기(authenticator)를 하나도 설정하지 않으면 익명 접속을 허용합니다. 로컬 테스트에는 편하지만, 외부에 열리는 환경이라면 대시보드의 **Access Control → Authentication**에서 인증을 반드시 설정해주세요.

---

## Nginx MQTT 로드밸런싱 설정

Nginx Stream 설정으로 MQTT 접속을 EMQX 노드에 분산합니다.

MQTT는 HTTP가 아닌 TCP라서 `stream` 블록에 넣어야 합니다. 그런데 Nginx 이미지 기본 설정은 `conf.d/*.conf`를 **`http` 블록 안에서** 읽기 때문에, upstream 설정을 `conf.d`에 그냥 넣으면 동작하지 않습니다.  
그래서 `nginx.conf`를 직접 마운트해 최상위에 `stream` 블록을 둡니다.

```nginx
# nginx/nginx.conf
worker_processes auto;

events {
    worker_connections 1024;
}

stream {
    log_format stream '$remote_addr [$time_local] $upstream_addr';
    access_log /dev/stdout stream;

    include /etc/nginx/conf.d/stream/*.conf;
}
```

```nginx
# nginx/stream.conf.template
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

Nginx 설정 파일은 원래 환경변수를 읽지 못합니다. 공식 이미지는 `/etc/nginx/templates/` 아래의 `*.template` 파일만 기동 시 환경변수로 치환해 `/etc/nginx/conf.d/`에 같은 하위 경로로 써 줍니다.  
위 구성에서는 `templates/stream/stream.conf.template` → `conf.d/stream/stream.conf`가 되고, `nginx.conf`의 `include`가 이 파일을 읽습니다.

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

> 토픽은 `/`로 시작하지 않는 것을 권장합니다. `/mqtt/test`처럼 쓰면 첫 레벨이 빈 문자열인 토픽이 되어, 다른 시스템과 토픽 규칙을 맞출 때 헷갈리기 쉽습니다.

### 구독 (발행된 메세지 읽기)

```bash
# subscribe
mosquitto_sub -h 127.0.0.1 -p 1886 -t mqtt/test/# -v

"C:\Program Files (x86)\mosquitto\mosquitto_sub.exe" -h MQTT 브로커 접속 URL -p 포트 -t MQTT 브로커에서 구독할 토픽 -v

"C:\Program Files (x86)\mosquitto\mosquitto_sub.exe" -h <broker-host> -p 1886 -t mqtt/test/# -v
```

### 발행 (메세지 발행)

```bash
# publish
mosquitto_pub -h 127.0.0.1 -p 1886 -t mqtt/test/vehicle1 -m "{\"imei\":\"1234\",\"speed\":80,\"time\":\"2026-02-20T10:00:00Z\"}"

"C:\Program Files (x86)\mosquitto\mosquitto_pub.exe" -h MQTT 브로커 접속 URL -p 포트 -t MQTT 브로커에서 구독할 토픽 -m 메세지

# 1초 간격으로 1000건 발행 (QoS 1)
"C:\Program Files (x86)\mosquitto\mosquitto_pub.exe" -h <broker-host> -p 1886 -t mqtt/test/vehicle1 -m "{\"imei\":\"0002\",\"speed\":111,\"time\":\"2026-02-20T10:00:00Z\"}" --repeat 1000 --repeat-delay 1 -q 1
```

```json
{
  "imei": "1234",
  "speed": 80,
  "time": "2026-02-20T10:00:00Z"
}
```

---

## 지속 세션(cleanSession = false) 확인

구독자가 잠깐 끊긴 동안 발행된 메시지를 다시 붙었을 때 받으려면 **구독자 쪽**에 지속 세션이 있어야 합니다.  
발행자에 `-c`를 붙여도 구독자가 받는 메시지에는 영향이 없습니다. 브로커가 메시지를 쌓아 두는 곳은 구독자의 세션이기 때문입니다.

조건은 세 가지입니다.

| 조건 | 옵션 | 이유 |
| :--- | :--- | :--- |
| 고정 Client ID | `-i sub_client_1` | 다시 붙었을 때 같은 세션을 찾기 위해 |
| 지속 세션 | `-c` (cleanSession = false) | 끊긴 동안의 메시지를 브로커가 보관 |
| QoS 1 이상 | `-q 1` (구독·발행 모두) | QoS 0 메시지는 보관 대상이 아님 |

```bash
# 1) 지속 세션으로 한 번 구독한 뒤 Ctrl+C 로 끊는다
mosquitto_sub -h 127.0.0.1 -p 1886 -t mqtt/test/# -v -q 1 -i sub_client_1 -c

# 2) 끊긴 상태에서 발행
mosquitto_pub -h 127.0.0.1 -p 1886 -t mqtt/test/vehicle1 -m "{\"imei\":\"1234\",\"speed\":80}" -q 1

# 3) 같은 Client ID 로 다시 구독하면 2) 의 메시지가 바로 들어온다
mosquitto_sub -h 127.0.0.1 -p 1886 -t mqtt/test/# -v -q 1 -i sub_client_1 -c
```

다음 편의 Kafka 커넥터도 EMQX 입장에서는 하나의 구독자입니다. 같은 원리로 커넥터가 재시작되는 동안의 메시지를 지키려면 커넥터 쪽 세션 설정이 중요합니다.

---

## 클러스터 상태 점검

각 노드 상태와 클러스터 구성 노드를 확인합니다.

```bash
docker exec emqx1 /opt/emqx/bin/emqx ctl status
docker exec emqx2 /opt/emqx/bin/emqx ctl status
docker exec emqx3 /opt/emqx/bin/emqx ctl status
```

```bash
docker exec emqx1 /opt/emqx/bin/emqx ctl cluster status
# running_nodes에 emqx1/emqx2/emqx3가 모두 보여야 정상
# 자기 자신만 보인다면 aliases 또는 cookie 설정을 확인
```

```bash
docker logs -f nginx-mqtt
# 접속이 서로 다른 EMQX 백엔드($upstream_addr)로 분산되는지 확인
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
