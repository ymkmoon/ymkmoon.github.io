---
title:  "[Java] Consumer 예제"
excerpt: "var와 lambda를 이용한 Consumer 예제"

tags:
  - [Java, Lambda, Consumer]

toc: true
toc_sticky: true
toc_label: "[Consumer]"
 
date: 2022-06-14
last_modified_at: 2022-06-14
---


- 참고사이트
  - [블로그](https://codechacha.com/ko/java8-consumer-example/){:target="_blank"}
  - [블로그](https://kudl.tistory.com/entry/JAVA-var-%ED%82%A4%EC%9B%8C%EB%93%9C){:target="_blank"}


### ``1. Consumer ``

```java
public static void main(String[] args) {		
		// Example 1 : Consumer
		Consumer<String> testFoo = (@NotBlank var testString) -> {
			System.out.println("s : "+testString);
			return;
		};

    testFoo.accept("1"); // "s : 1" 이 출력된다.
}
```

위 코드에서 testString 타입을 var 로 사용 한 이유는 @NotBlank 어노테이션을 사용 하기 위해서이다.

<br>

### ``2. Primitive Consumer``

```java
public static void main(String[] args) {		
		// Example 2 : Primitive Consumer
		// IntConsumer 외에 DoubleConsumer, LongConsumer가 있다.
		IntConsumer intConsumer = n -> { 
			System.out.println(n * 100); 
		};
		intConsumer.accept(5); // 5 * 100 인 "500" 이 출력된다.
		intConsumer.accept(10); // 10 * 100 인 "1000" 이 출력된다.
}
```

<br>

### ``3. List.forEach()``

```java
public static void main(String[] args) {		
		// Example 3 : List.forEach()
		Consumer<String> consumer = s -> System.out.println(s.toUpperCase());
    List<String> fruits = Arrays.asList("apple", "kiwi", "orange");
    fruits.forEach(consumer); 
}
```

<br>

### ``4. andThen()``

```java
public static void main(String[] args) {		
    // Example 4 : andThen()
		List<String> fruits = Arrays.asList("apple", "kiwi", "orange");
    Consumer<List<String>> addNumber = list -> {
        for (int i = 0; i < list.size(); i++) {
            list.set(i, (i + 1) + ". " + list.get(i));
        }
    };

    Consumer<List<String>> printList = list -> {
        for (String fruit : list) {
            System.out.println(fruit);
        }
    };

    addNumber.andThen(printList).accept(fruits); // fruits 의 요소들이 인덱스와 함께 줄바꿈되어 출력된다.
}
```

```java
// 실제로 andThen 은 아래와 같이 구성되어 있다.
public interface Consumer<T> {
  ...
  default Consumer<T> andThen(Consumer<? super T> after) {
      Objects.requireNonNull(after);
      return (T t) -> { accept(t); after.accept(t); };
  }
}
```
<br>
