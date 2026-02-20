---
title:  "[IoT] 01. EMQX 브로커 설정 및 데이터 수집"
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

1. [인트로](https://ymkmoon.github.io/Aws-02-Cognito/)

2. [EMQX 브로커 설정 및 데이터 수집(현재글)](https://ymkmoon.github.io/Aws-03-Cognito-OAuth2/)

3. [EMQX와 Kafka 연동](https://ymkmoon.github.io/Aws-04-Cognito-React/)

4. [Kafka Consumer를 활용한 데이터 소비](https://ymkmoon.github.io/Aws-04-Cognito-React/)

---

## EMQX 란

EMQX는 IoT(사물인터넷) 및 AI 애플리케이션을 위해 설계된 세계에서 가장 확장성이 뛰어나고 신뢰할 수 있는 오픈 소스 MQTT 메시징 플랫폼


### MQTT 란

MQTT(Message Queuing Telemetry Transport)는 저전력, 낮은 대역폭, 불안정한 네트워크 환경에서도 안정적인 통신을 지원하는 경량 IoT 및 M2M 전용 메시징 프로토콜입니다. Publish-Subscribe(발행-구독) 모델을 사용하여 Publisher(데이터 전송)와 Subscriber(데이터 수신) 사이의 브로커(Broker)가 메시지를 중계하며, 1:N 통신에 매우 유용하다.



