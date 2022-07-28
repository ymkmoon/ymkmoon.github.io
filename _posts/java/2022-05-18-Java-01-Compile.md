---
title:  "[Java] 성능 개선"
excerpt: "Java Complie 속도 향상시키기"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[Java 성능 개선]"
 
date: 2022-05-18
last_modified_at: 2022-05-18
---

### ``String 보다는 StringBuffer 가 빠르다.``

String 은 불변성을 가지기 때문에 `변하지 않는 문자열을 자주 읽어들이는 경우에는 String 을 사용` 하고, `문자열 추가, 수정, 삭제 등의 연산이 발생하는 경우에는 힙 메모리에 많은 Garbage 가 생성되어 성능에 영향을 끼치게 되니 StringBuffer, StringBuilder` 를 사용하자.

```java
String str = "Hello";
str = str + "World";
```

코드상으로는 `str` 이라는 변수 하나이지만, 실제로는 "Hello" 에 "World" 라는 
새로운 스트링 객체가 생성 되었기 때문에, 두개를 카피한 후 원래 스트링 객체의 메모리가 해제(Grabage Collection) 되면서 새로운 `str="HelloWorld"` 가 생성된다.



StringBuffer 를 사용하면, 하나의 객체에 추가가 되는 것 이므로, 새로운 객체가 생성되지 않아 결국 Garbage Collection 을 할 필요가 없어진다.

```java
StringBuffer sb = new StringBuffer();
sb.append("ccccHello");
sb.append("World");
```


#### StringBuffer 와 StringBuilder

StringBuffer 와 StringBuilder 의 가장 큰 차이점은 동기화의 유무이다.

`StringBuffer` 는 동기화 키워드를 지원하여 멀티쓰레드 환경에서 안전성을 가지고 있다(thread-safe). 참고로 `String` 도 불변성을 가지기 때문에 멀티쓰레드 환경에서 안정성을 가지고 있다(thread-safe).


반대로 `StringBuilder` 는 동기화를 지원하지 않기 때문에 멀티쓰레드 환경에서 사용하는 것은 적합하지 않지만, 동기화를 고려하지 않는 만큼 단일쓰레드 에서는 `StringBuffer` 보다 더 뛰어난 성능을 보인다.


잘 모르겠다면 그냥 `StringBuffer` 를 사용하면 된다. 


##### Multi Thread (멀티쓰레드)

운영체제(OS)에서 실행 중인 하나의 어플리케이션, 즉 작업 관리자에서 프로세스에 올라와 있는 어플리케이션 하나를 하나의 프로세스라고 부른다. 만약 크롬을 2개 띄웠다면 두개의 프로세스가 생성 된 것이다.


`멀티태스킹` 이란 두 가지 이상의 작업을 동시에 처리하는 것을 의미한다.
예를 들어 문서작업을 하는 동시에 음악을 듣는 것, 메신저에서 채팅 기능을 제공 하면서 동시에 파일 업로드 기능을 수행 할 수 있는 것이다. 이처럼 한 프로세스에서 멀티 태스킹이 가능한 이유는 `Multi Thread` 덕분이다.

<hr/>

### ``연산``

#### 증감연산자 사용하기

```java
int i = 1;
i = i + 1; 
```

```java
int i = 1;
i++;
```

위와 같은 코드가 있을때는 i++; 을 사용한다.


#### += 사용하기

```java
String str = "Hello";
str = str + "World";
```

```java
String str = "Hello";
str += "World";
```

<hr/>

### ``변수의 범위와 타입``

범위를 따지면 지역 메소드가 스택 영역여 저장되므로 더 빠르고,
타입을 따지면 int와 참조변수가 가장 빠르다.

<hr/>

### ``Object 재사용 ``

```java
StringBuffer sb=new StringBuffer();
sb.append("Hello");
sb=null;
sb=new StringBuffer();
sb.append("World");
```

```
StringBuffer sb=new StringBuffer();
sb.append("Hello");
sb.setLength(0);
sb.append("World");
```

위와 같이 사용할 경우 두번의 초기화 과정을 거치게 된다.
아래와 같이 각 클래스에서 지원해주는 메소드를 이용하여 Object를 재사용 할 수 있다.

<hr/>

### ``반복문``

반복문을 중첩해서 사용 할 경우, 내부 반복문의 횟수보다 외부 반복문의 횟수를 최소화 하자.
