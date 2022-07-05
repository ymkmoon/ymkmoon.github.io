---
title:  "[Java] Stream Reduce - 가공과 연산"
excerpt: "Stream Match 요소 검색 및 객체 꺼내기"

tags:
  - [Java, Stream]

toc: true
toc_sticky: true
toc_label: "[Stream Reduce]"
 
date: 2022-07-06
last_modified_at: 2022-07-06
---

- 참고사이트
  - [블로그](https://doohyun.tistory.com/35?category=592214){:target="_blank"}


### ``1. reduce() 란?``

Stream의 데이터를 변환하지 않고, 더하거나 빼는 등의 연산을 수행하여 하나의 값으로 만들어주는 메소드

<br>

### ``2. reduce() 사용하기``

#### 기존코드

```java
List<Integer> numberList = Arrays.asList(5, 10, 3, 6, 92);
int sum = 0;
for (Integer number : numberList) {
	if (number % 2 == 0) {
		sum += number;
	}
}
System.out.println("총 합 : " + sum);
// 총 합 : 108
```

#### reduce 적용

```java
List<Integer> numberList = Arrays.asList(5, 10, 3, 6, 92);
System.out.println("총 합 : " + 
	numberList
		.stream()
		.filter(n -> n % 2 == 0)
		.reduce(0, (a, b) -> a + b) // 초기값을 0으로 설정
		// .reduce((a, b) -> a + b) // 초기값을 설정 X (결과값이 Optional 객체)
	);
```

`.reduce(0, (a, b) -> a + b)` 에서  0 은 초기 값을 의미하며, 초기값이 없어도 연산을 수행 할 수 있지만 결과값은 `Optional 객체`로 나타나게 된다.

<br>

#### map-reduce 패턴

객체 필드의 합을 구해보자.

```java
@Getter
@AllArgsConstructor
public static class TestVo {
	private String name;
	private Integer value;
}

List<TestVo> numberList = Arrays.asList(
	new TestVo("Doohyun Nam",7), 
	new TestVo("Ford Mill",9), 
	new TestVo("Hwang",2));

numberList.stream()
	.map(value -> value.getValue())
	.reduce((v1, v2) -> v1 + v2)
	.ifPresent(System.out::println);
```

특정 필드를 꺼내기 위해 map() 을 이용하였고, stream 의 reduce 를 통해 합(sum) 을 구할 수 있었다. 위와 같이 map() 과 reduce() 를 이용하여 데이터를 가공하는 패턴을 map-reduce 패턴이라고 하며, 웹 검색 엔진등에서 많이 사용 된다.
<br>

그 이유는 `Stream API 에서 쉽게 볼 수 있는 병렬성 이라는 특징 덕분`이다. <br>

예를 들어 특정 컬렉션의 합을 병렬적으로 구하기 위해서는 `임계영역간 스레드의 경쟁 비용이 발생`하기 마련이지만, reduce 내부의 내부반복은 개발자에게는 `추상화 되어 제공`된다. <br>

병렬스트림의 리듀싱 결과를 일반 스트림과 동일하게 얻기 위해서는 연산이 `어떤 순서로 연산되어도 결과가 보장`되는 구조이며, `람다 내부 인스턴스는 변경되어서는 안되는 구조` 이어야만 한다. <br>
-> 즉 각 트랜잭션 중 병렬구조로 처리가 가능한 부분만 사용이 가능하다.