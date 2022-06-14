---
title:  "[Java] Optional 사용예시"
excerpt: "Optional 사용 예시"

tags:
  - [Java, Optional]

toc: true
toc_sticky: true
toc_label: "[Optional]"
 
date: 2022-06-14
last_modified_at: 2022-06-14
---


- 참고사이트
  - [Oracle Docs java8](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html){:target="_blank"}
  - [Oracle Docs java9](https://docs.oracle.com/javase/9/docs/api/java/util/Optional.html){:target="_blank"}


### ``1. Optional 이란? ``

A container object which may or may not contain a non-null value. If a value is present, isPresent() will return true and get() will return the value.
Additional methods that depend on the presence or absence of a contained value are provided, such as orElse() (return a default value if value not present) and ifPresent() (execute a block of code if the value is present).

This is a value-based class; use of identity-sensitive operations (including reference equality (==), identity hash code, or synchronization) on instances of Optional may have unpredictable results and should be avoided.

`Oracle` 에서는 위와 같이 설명하고 있다.


=> 간단하게 Optional 을 사용하여 NPE 를 미리 예방 할 수 있고, 각종 메소드를 이용하여 분기처리가 가능하다.

NPE(NullPointException)은 대게 런타임중에 발생하기 때문에 예측하기 까다롭기 마련인데, Optional 을 사용하면 좀 더 클린하게 코드를 작성 할 수 있고 직접 NPE 를 유발 할 수 있는 null 을 다루지 않아도 된다.

<br>

### ``2. Optional 사용 예시 ``

우리는 text 에 뭐가 담겨있는지 알 수 없기 때문에 이 전까지는 직접 null 체크를 했지만,
Optional 을 사용하면 isPresent() 함수를 이용해 아래와 같이 사용 가능하며, 익숙해지면 코드를 훨씬 클린하게 작성 할 수 있게된다.

```java
// Optional 사용 전
String text = getText();
int length;
if (text != null) {
  length = text.length();
} else {
  length = 0;
}
```

```java
// Optional 사용 
String text = getText();
Optional<String> maybeText = Optional.ofNullable(text);
int length;
if (maybeText.isPresent()) {
  length = maybeText.get().length();
} else {
  length = 0;
}
```

```java
// Optional 사용 
String text = getText();
int length = Optional.ofNullable(text).map(String::length).orElse(0);
```

<br>

### ``3. Optional 의 사용목적 ``

Optional 사용할 때 주의 할 점이 있다.
<br>

API Note:
Optional is primarily intended for use as a method return type where there is a clear need to represent "no result," and where using null is likely to cause errors. A variable whose type is Optional should never itself be null; it should always point to an Optional instance.


API 참고:
Optional은 주로 "결과 없음"을 나타낼 필요가 있고 null을 사용하면 오류가 발생할 가능성이 있는 메서드 반환 유형으로 사용하기 위한 것입니다. 유형이 Optional인 변수 자체는 null이 아니어야 하며 항상 Optional 인스턴스를 가리켜야 합니다.


특정 메소드에 대한 비용이 비싸기 때문에 아래와 같이 주의 할 사항이 있다.

- isPresent() - get() 대신 orElse(), orElseGet(), orElseThrow() 를 사용하자.
- orElse(new ...) 대신 orElseGet(() -> new ....) 를 사용하자.
- 단지 값을 얻을 목적이라면 Optional 대신 null 비교
- Optional 대신 비어있는 컬렉션 반환
- Optional 을 필드로 사용 금지
- Optional 을 생성자나 매개변수로 사용 금지
- Optional을 컬렉션의 원소로 사용 금지
- of() 와 ofNullable() 차이점 파악
