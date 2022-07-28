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
  - [블로그](https://kdhyo98.tistory.com/40){:target="_blank"}


### ``Optional 이란? ``

A container object which may or may not contain a non-null value. If a value is present, isPresent() will return true and get() will return the value.
Additional methods that depend on the presence or absence of a contained value are provided, such as orElse() (return a default value if value not present) and ifPresent() (execute a block of code if the value is present).

This is a value-based class; use of identity-sensitive operations (including reference equality (==), identity hash code, or synchronization) on instances of Optional may have unpredictable results and should be avoided.

`Oracle` 에서는 위와 같이 설명하고 있다.


=> 간단하게 Optional 을 사용하여 NPE 를 미리 예방 할 수 있고, 각종 메소드를 이용하여 분기처리가 가능하다.

NPE(NullPointException)은 대게 런타임중에 발생하기 때문에 예측하기 까다롭기 마련인데, Optional 을 사용하면 좀 더 클린하게 코드를 작성 할 수 있고 직접 NPE 를 유발 할 수 있는 null 을 다루지 않아도 된다.

<hr/>

### ``Optional 사용 예시 ``

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

<hr/>

### ``Optional 의 사용목적 ``

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

<hr/>

### ``of 와 ofNullable ``

객체를 Optional 로 감싸기 위해서는 Optional 에서 제공하는 `of` 와 `ofNullable` 를 사용 해야한다.

`of` 는 인자로서 null 값이 허용되지 않는다는 의미이고, <br>
`ofNullable` 은 null 을 허용 한다는 의미이다.

```java
String str = null;
Optional ofNullableStr = Optional.ofNullable(str); // NPE 발생 X
Optional ofStr = Optional.of(str); // NPE 발생
```

of 를 사용하면 null 데이터에 대해 NPE 가 발생하고, 에러를 직관적으로 보기 힘들기 때문에 난 아래와 같이 사용한다.

```java
String str = null;
Optional ofNullableStr = Optional.ofNullable(str)
    .orElseThrow(() -> new BusinessException("str 값이 존재하지 않습니다.", ErrorCode.UNAUTHORIZED))
```

위와 같이 사용하면 보다 직관적인 예외를 확인 할 수 있고, 응답할 때 커스텀 된 에러메세지를 응답하기 수월해진다.

<hr/>

### ``orElse 와 orElseGet ``

```java
public T orElse(T other)

public T orElseGet(Supplier<? extends T> other)
```

```java
public T orElse(T other) {
    return value != null ? value : other;
}

public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}
```

`orElse` 는 `T의 모든 매개 변수`를 사용하고, <br>
`orElseGet` 은 t 유형의 개체를 반환하는 `Supplier 유형의 인터페이스`를 허용한다.

따라서 두개의 차이점은 <br>
`orElse()` : T 클래스를 인수로 받는다.<br>
`orElseGet()` : T 클래스를 상속받은 하위 클래스를 return 해주는 Supplier 함수 인터페이스를 받는다.

> `Supplier` 은 함수적 인터페이스로 get 을 호출하여 결과를 리턴하는 역할을 한다.

<hr/>

#### orElse()

```java
// orElse 
public static void main(String[] args) {
  String name = Optional.of("baeldung")
        .orElse(getRandomName());
  System.out.println("name : "+name);
}


public static String getRandomName() {
  System.out.println("getRandomName() method - start");	    		    
  System.out.println("getRandomName() method - end");
  return "temp";
}

// 위 메인메소드를 실행 시 콘솔에는 아래와 같이 출력된다.
getRandomName() method - start
getRandomName() method - end
name : baeldung
```

위 출력 된 콘솔메세지에서 알 수 있듯이<br>
`orElse()` 는 메소드를 인수로 받지 않고 바로, 값을 인수로 받는다. 

결국, `orElse()` 메소드 인수를 할당하기 위해 getRandomName() 메소드가 실행된 후 해당 결과 값을 `orElse()` 메소드 인수로 할당하기 때문에 Optional의 값과 상관 없이 메소드가 실행되게 되는 것이다.

<hr/>

#### orElseGet()

```java
// orElse 
public static void main(String[] args) {
  String name = Optional.of("baeldung")
        .orElseGet(getRandomName());
  System.out.println("name : "+name);
}


public static String getRandomName() {
  System.out.println("getRandomName() method - start");	    		    
  System.out.println("getRandomName() method - end");
  return "temp";
}

// 위 메인메소드를 실행 시 콘솔에는 아래와 같이 출력된다.
name : baeldung
```

위 출력 된 콘솔메세지에서 알 수 있듯이<br>
`orElseGet()` 에서는 getRandomName() 메소드를 호출하지 않는다.

그 이유는 `인수로 전달 된 Supplier 메소드 경우 Optional의 값이 없을 때만 get()을 통해 실행되기 때문` 이다.

<hr/>

#### 결론

`orElseGet()`
- null일 경우 메소드를 실행해야 할 때 

`orElse()`
- null일 때 값을 넘겨야 할 때