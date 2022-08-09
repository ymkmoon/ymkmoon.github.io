---
title:  "[AWS] SQS(Simple Queue Service) 사용을 위한 AWS Access Key 발급받기"
excerpt: "AWS SQS 인증을 위한 Access Key 발급"

tags:
  - [AWS, SQS]

toc: true
toc_sticky: true
toc_label: "[AWS SQS #3]"
 
date: 2022-08-03
last_modified_at: 2022-08-03
---

- AWS SQS 활용하기
  1.  [AWS SQS(Simple Queue Service) 란?](https://ymkmoon.github.io/Aws-06-Sqs/)
  2.  [SQS 대기열 생성하기](https://ymkmoon.github.io/Aws-07-Sqs-Create/)
  3.  [AWS Access Key 발급받기](https://ymkmoon.github.io/Aws-08-Access-Key/)
  4.  [SpringBoot 에서 SQS 사용해보기](https://ymkmoon.github.io/Aws-09-Sqs-Message/)

<hr/>

- 참고사이트
  - [AWS Document](https://aws.amazon.com/ko/sqs/){:target="_blank"}


## Access Key 발급받기

### AWS Console 로 이동

[https://ap-northeast-2.console.aws.amazon.com/console/home?region=ap-northeast-2](https://ap-northeast-2.console.aws.amazon.com/console/home?region=ap-northeast-2){:target="_blank"}

![AWS](/assets/image/aws/AWS_SQS_01.PNG)

<hr/>

### IAM 검색 및 선택

![AWS](/assets/image/aws/AWS_SQS_11.PNG)

<hr/>

### 사용자 추가

사이드 메뉴에서 사용자 선택 후 사용자 추가

![AWS](/assets/image/aws/AWS_SQS_12.PNG)

<hr/>

### 사용자 정보와 액세스 유형 선택

![AWS](/assets/image/aws/AWS_SQS_13.PNG)

사용자 이름 칸을 입력하고, AWS 자격 증명 유형 선택 에서는 액세스 키를 선택

<hr/>

### 권한 설정

![AWS](/assets/image/aws/AWS_SQS_14.PNG)

`기존 정책 직접 연결`을 선택하고 정책 필터에 `sqs` 검색 <br>
`AmazonSQSFullAccess` 를 선택 후 다음으로 넘어가자.

<hr/>

### 태그 

그냥 넘어가자

![AWS](/assets/image/aws/AWS_SQS_15.PNG)

<hr/>

### 검토

![AWS](/assets/image/aws/AWS_SQS_16.PNG)

<hr/>

### 완료

![AWS](/assets/image/aws/AWS_SQS_17.PNG)

액세스 키 확인이 가능하다.

