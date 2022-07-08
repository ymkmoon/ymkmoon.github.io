---
title:  "[Java] StringBuffer 와 StringBuilder 중 뭘 써야 할까?"
excerpt: "String vs StringBuffer vs StringBuilder"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[StringBuffer vs StringBuilde]"
 
date: 2022-07-07
last_modified_at: 2022-07-07
---

- 참고사이트
  - [블로그](https://unabated.tistory.com/entry/%EC%99%9C-%EC%9E%90%EB%B0%94%EC%97%90%EC%84%9C-static%EC%9D%98-%EC%82%AC%EC%9A%A9%EC%9D%84-%EC%A7%80%EC%96%91%ED%95%B4%EC%95%BC-%ED%95%98%EB%8A%94%EA%B0%80){:target="_blank"}

### ``1. String vs StringBuffer & StringBuilder ``

String 과 StringBuffer, StringBuilder 와의 차이점은 `불변성` 이다. <br>

<br>

### ``2. StringBuffer vs StringBuilder``

둘의 가장 큰 차이점은 `동기화 유무` 이다. <br>

`StringBuffer`는 동기화 키워드를 지원하여 `멀티쓰레드 환경에서 안전`하다. <br>

`StringBuilder` 는 동기화를 지원하지 않기 때문에 멀티쓰레드 환경에서의 사용은 적합하지 않지만, 단일쓰레드에서의 성능은 StringBuffer 보다 뛰어나다. 

<br>


### ``3. 그럼 뭘 써야할까?``

- String
  - 문자열 연산이 적고 멀티쓰레드 환경인 경우

- StringBuffer
  - 문자열 연산이 많고 멀티쓰레드 환경인 경우

- StringBuilder
  - 문자열 연산이 많고 단일쓰레드 환경인 경우


위 환경에 맞춰서 사용하면 되지만, 잘 모르겠다면 멀티쓰레드 환경에서도 안정성이 보장 된 `StringBuffer` 를 사용하면 된다.