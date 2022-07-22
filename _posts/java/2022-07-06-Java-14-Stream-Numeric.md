---
title:  "[Java] Stream - 숫자형 스트림"
excerpt: "숫자형 스트림 사용하기"

tags:
  - [Java, Stream]

toc: true
toc_sticky: true
toc_label: "[Stream Numeric]"
 
date: 2022-07-06
last_modified_at: 2022-07-06
---

- 참고사이트
  - [블로그](https://doohyun.tistory.com/36){:target="_blank"}
  - [블로그](https://cornswrold.tistory.com/545){:target="_blank"}


### ``숫자형 스트림 이란?``

Stream은 숫자들에 대한 스트림을 지원하며 이 기본형 특화 스트림들은 `sum, max, min 등` 자주 사용하는 리듀싱 메소드를 제공한다. <br>
숫자 스트림을 사용하기 위해서는 map 의 파생 메소드인 `mapToInt, mapToDouble, mapToLong` 등의 메소드를 사용해야 한다.

<br>

### ``reduce() 와의 차이``

메뉴에 있는 음식의 칼로리 합계를 구한다고 했을때 `reduce()` 로 구현하면 아래와 같을 수 있다.

메뉴는 음식들이 담긴 목록이며 <br>
음식은 Entity 또는 Table 라고 생각하면 되고, <br>
칼로리는 Field 또는 Cloumn 이라고 생각하면 된다.

#### reduce()

```java
int calories = menu.stream()
  .map(Dish::getCalories)
  .reduce(0, Integer::sum);
```

위 코드에는 박싱 비용이 숨겨져 있다. 내부적으로 합계를 계산하기 전에 `Integer`를 기본형으로 언박싱 해야한다.


#### 숫자형스트림

Stream API 는 박싱 비용을 피할 수 있도록 `IntStream, DoubleStream, LongStream` 을 제공한다.

각각의 인터페이스는 sum, max, min, average 등 자주 사용하는 숫자 관련 리듀싱 연산 수행 메소드를 제공한다.


##### 숫자형 스트림으로 매핑

```java
int calories = menu.stream() // Stream<Dish> 반환
  .mapToInt(Dish::getCalories) // IntStream 반환
  .sum();
```

##### 객체 스트림으로 복원

```java
// 객체 스트림으로 복원
IntStream intstream = menu.stream().mapToInt(Dish::getCalories); // 스트림을 숫자 스트림으로 변환
Stream<Integer> stream = intstream.boxed(); // 숫자 스트림을 스트림으로 변환
```

##### OptionalInt

스트림에 요소가 없는 경우를 고려해서 OptionalInt 를 이용 할 수 있다.

```java
OptionalInt maxCalories = menu.stream().mapToInt(Dish::getCalories).max();
int max = maxCalories.orElse(1);
```

<br>

### ``range() 와 rangeClosed() ``

JAVA8 에서는 숫자형스트림에서 특정 범위안의 숫자 집합을 출력해주는 정적메소드인 range() 와 rangeClosed() 를 제공한다. 두 메소드 모두 범위에 대한 인수를 받으며, 차이는 시작값과 종료값이 결과에 포함하는 여부이다. (range 의 경우 결과에 포함되지 않음)


```java
// 1 ~ 100 사이의 짝수를 생성한다.
IntStream evenNumbers = IntStream.rangeClosed(1, 100).filter(n -> n % 2 == 0);
System.out.println(evenNumbers.count());
```


```java
// 숫자 Stream 을 이용하여 피타고라스의 수를 구하는 로직
IntStream.rangeClosed(1, 100)
			.boxed()
			.flatMap(a -> IntStream.rangeClosed(a, 100)
					.mapToObj(b -> new int[] {a, b, (int)Math.sqrt(a*a + b*b)}))
			.filter(v -> v[2]%1 == 0)
			.forEach(v -> System.out.println(v[0] + " " + v[1] + " " + v[2]));
```

flatMap 을 통해 두 숫자 스크림을 엮었다. 첫번째 범위 에서는 Stream 연산의 결과를 객체로 받기 위해 `boxed 메소드`를 사용했는데, boxed() 를 사용하지 않으면 `IntStream` 은 오직 중간연산 결과로 숫자 타입의 스트림만 생성 가능하기 때문이다.

<br>

### ``스트림 만들기``

컬렉션이나 배열은 아래와 같이 데이터를 만들 수 있다.

```java
String[] dataArray = new String[]{"남두현", "윤석진", "성지윤", "오진명"};
List<String> dataList = Arrays.asList(dataArray);
```


스트림은 아래 두 방법으로 만들 수 있다.

```java
String[] dataArray = new String[]{"남두현", "윤석진", "성지윤", "오진명"};
Stream<String> ofStream = Stream.of("남두현", "윤석진", "성지윤", "오진명");
Stream<String> arrayStream = Arrays.stream(dataArray);
```

<br>

### ``Unbounded Stream (언바운드 스트림)``

고정되지 않은 크기(무한한)의 스트림 만드는 방법은 아래와 같다.

```java
// iterate 는 Function type 람다를 사용 
Stream.iterate(0, n -> n + 2).forEach(System.out::println);
// generate 는 Supplier type 메소드 레퍼런스 사용
Stream.generate(Math::random).forEach(System.out::println);
```

사용 되는 iterate() 와 generate() 는 보통 limit 로 제한을 두어 사용 한다. <br>
제한을 두지 않은 스트림을 `Unbounded Stream (언바운드 스트림)` 이라 하며, 특정 로직에서 무한한 결과 때문에 원하는 값을 얻을 수 없을수도 있다.

