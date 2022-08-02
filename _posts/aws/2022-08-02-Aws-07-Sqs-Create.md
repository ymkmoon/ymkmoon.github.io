---
title:  "[AWS] SQS(Simple Queue Service) 대기열 생성하기"
excerpt: "AWS SQS 의 대기열 생성하기"

tags:
  - [AWS, SQS]

toc: true
toc_sticky: true
toc_label: "[AWS SQS #2]"
 
date: 2022-08-02
last_modified_at: 2022-08-02
---


- 참고사이트
  - [AWS Document](https://aws.amazon.com/ko/sqs/){:target="_blank"}


## SQS 생성하기

### AWS Console 로 이동

[https://ap-northeast-2.console.aws.amazon.com/console/home?region=ap-northeast-2](https://ap-northeast-2.console.aws.amazon.com/console/home?region=ap-northeast-2){:target="_blank"}

![AWS](/assets/image/aws/AWS_SQS_01.PNG)

<hr/>

### SQS 검색

서비스에서 SQS 를 검색 후 선택

![AWS](/assets/image/aws/AWS_SQS_02.PNG)

<hr/>

### 대기열 생성

![AWS](/assets/image/aws/AWS_SQS_03.PNG)

<hr/>

### 대기열 세부 정보 (본인은 표준 선택)

![AWS](/assets/image/aws/AWS_SQS_04.PNG)

- 표준
  - 표쥰 Queue(선입선출) 의 순서를 가능한 보장하려고 하지만, 장담할 수 없다.
  - 메세지 큐에 쌓여있는 메세지를 최소 1회 전송하지만, 가끔 한 개 이상의 메세지 사본이 전송 될 수 있다.
  - 속도 무제한
  - 모든 region 에 존재

- FIFO
  - 선입선출 순서를 무조건 보장한다.
  - 무조건 1회만 전송
  - 속도 300~3000msg/s 제한
  - 특정 region 에 존재

<hr/>

###  대기열 구성 

이미지와 같이 선택하여 진행

![AWS](/assets/image/aws/AWS_SQS_05.PNG)

- 표시 제한 시간: (한 소비자가) 대기열에서 수신한 메시지가 다른 메시지 소비자에게 보이지 않게 되는 시간
- 메시지 보존 기간: Amazon SQS가 삭제되지 않은 메시지를 보관하는 시간
- 전송 지연: 이 대기열에 추가된 각 메시지의 첫 번째 전송에 대한 지연 시간
- 최대 메시지 크기: 이 대기열의 최대 메시지 크기
- 메시지 수신 대기 시간: 폴링이 메시지를 사용할 수 있을 때까지 기다리는 최대 시간
- 콘텐츠 기반 중복 제거: Amazon SQS는 메시지 본문에 기반하여 중복 제거 ID를 자동으로 생성할 수 있음

<hr/>

### 액세스 정책 

이미지와 같이 선택하여 진행

![AWS](/assets/image/aws/AWS_SQS_06.PNG)

이 대기열에 액세스할 수 있는 계정 및 사용자와 허용되는 작업을 정의한다.

- 기본 설정에서 대기열로 메시지를 보낼 수 있는 사람과 대기열에서 메시지를 수신할 수 있는 사람을 구성
- 읽기 전용 JSON 패널에서 대기열에 대한 최종 액세스 정책이 표시
- 기본적으로 대기열 소유자만 메시지를 보내고 받을 수 있음
- 고급 설정에서 JSON 액세스 정책을 직접 수정하면 각 주체(계정, 사용자 또는 역할)가 수행할 수 있는 사용자 지정 작업 세트를 지정할 수 있음

<hr/>

### 리드라이브 허용 정책 

이미지와 같이 선택하여 진행

![AWS](/assets/image/aws/AWS_SQS_07.PNG)

리드라이브 허용 정책은 이 대기열을 배달 못한 편지 대기열로 사용할 수 있는 소스 대기열을 정의한다.

- 리드라이브 허용 정책을 활성화할 때 모든 소스 대기열을 허용 또는 거부하거나 ARN별로 최대 10개의 소스 대기열 목록을 지정할 수 있음
- 소스 대기열은 동일한 AWS 계정이 소유해야 하며 배달 못한 편지 대기열과 동일한 리전에 있어야 함

<hr/>

### 암호화 

이미지와 같이 선택하여 진행

![AWS](/assets/image/aws/AWS_SQS_08.PNG)

서버 측 암호화(SSE)를 활성화하면 Amazon SQS는 이 대기열로 전송되는 모든 메시지를 암호화한다. 메시지는 암호화된 형식으로 저장되고 Amazon SQS는 권한이 있는 소비자에게 전송되는 경우에만 메시지를 해독한다. SSE는 메시지 본문을 암호화하지만 다음 필드는 암호화하지 않음.

- 대기열 메타데이터(대기열 이름 및 속성)
- 메시지 메타데이터(메시지 ID, 타임스탬프 및 속성)
- 대기열별 지표

대기열의 암호화가 활성화된 후 전송된 경우에만 메시지가 암호화되고, Amazon SQS는 백로깅된 메시지를 암호화하지 않는다. 암호화된 메시지는 대기열 암호화가 비활성화된 경우에도 암호화된 상태로 유지한다.

<hr/>

### 배달 못한 편지 대기열 

이미지와 같이 선택하여 진행

메시지를 소비할 수 없는 경우 배달 못한 편지 대기열(DLQ)로 전송할 수 있다. 배달 못한 편지 대기열을 사용하면 문제가 있는 메시지를 격리하여 실패 원인을 확인할 수 있다.

![AWS](/assets/image/aws/AWS_SQS_09.PNG)

대기열을 소스 대기열로 지정하면 DLQ가 자동으로 생성되지 않는다. 먼저 DLQ로 지정할 대기열을 생성해야 하며. DLQ 대기열 유형(표준 또는 FIFO)은 소스 대기열과 일치해야 한다. 동일한 DLQ를 둘 이상의 소스 대기열과 연결할 수 있다. <br>

최대 수신 수 값은 메시지를 DLQ로 전송하는 경우를 결정한다. 메시지의 ReceiveCount가 대기열의 최대 수신 수를 초과하면 Amazon SQS는 메시지를 연결된 DLQ(원래 메시지 ID 포함)로 이동한다. <br>

동일한 AWS 계정을 사용하여 DLQ와 DLQ로 메시지를 전송하는 소스 대기열을 생성해야 합다. 또한 DLQ는 DLQ를 사용하는 소스 대기열과 동일한 리전에 상주해야 한다.

<hr/>

### 태그 

이미지와 같이 선택하여 진행

태그는 AWS 리소스에 지정되는 레이블이며 태그를 사용하여 리소스를 검색 및 필터링하거나 AWS 비용을 추적한다. 자세한 정보는 아래의 링크 참조 <br>

- [태그 관련 참고 및 유념사항](https://docs.aws.amazon.com/ko_kr/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-queue-tags.html){:target="_blank"}

![AWS](/assets/image/aws/AWS_SQS_10.PNG)

<hr/>


### 완료

이제 대기열을 생성하면 완료되며 확인이 가능하다.