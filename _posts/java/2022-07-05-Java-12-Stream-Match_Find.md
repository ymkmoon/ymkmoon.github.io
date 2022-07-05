---
title:  "[Java] Stream Match & Find - 검색과 매칭"
excerpt: "Stream Match 요소 검색 및 객체 꺼내기"

tags:
  - [Java, Stream]

toc: true
toc_sticky: true
toc_label: "[Stream Match]"
 
date: 2022-07-05
last_modified_at: 2022-07-05
---

- 참고사이트
  - [블로그](https://doohyun.tistory.com/30?category=592214){:target="_blank"}


### ``1. Match 이란?``

특정 속성이 데이터 집합에 있는지, 있다면 해당 객체를 추출할 수 있는 다양한 유틸 메소드이다.

<br>

### ``2. Match 사용하기 ``

#### 기존코드

아래 로직은 특정 숫자 컬렉션에서 5의 배수가 있는지 확인하는 로직이다.

```java
List<Integer> sampleList = Arrays.asList(5,10,12,38);
Boolean isHave = false;
for (Integer number : sampleList) {
	if (number % 5 == 0) {        
		// 5의 배수인 데이터가 있는지 확인
		isHave = true;
		break;
	}
}			
System.out.println("isHave  : " + isHave);
// 출력 : isHave : true
```

<br>

#### anyMatch

- anyMatch : Predicate 가 적어도 한 요소와 일치하는지 확인 

```java
List<Integer> sampleList = Arrays.asList(5,10,12,38);
boolean isHave = sampleList.stream().anyMatch(n -> n % 5 == 0);
System.out.println("isHave  : " + isHave);
```

<br>


#### allMatch & noneMatch

- allMatch : 컬렉션의 모든 요소가 일치하는지 확인
- noneMatch : 컬렉션에서 일치하는 요소가 없는지 확인

```java
List<Integer> sampleList = Arrays.asList(5,10,15,40);

boolean isHave = sampleList.stream().allMatch(n -> n%5 == 0);
boolean doNotHave = sampleList.stream().noneMatch(n -> n % 5 == 0);

System.out.println("isHave : " + isHave);
System.out.println("doNotHave : " + doNotHave);
```

<br>


### ``3. 검색 (findAny, findFirst) 사용하기 ``

`filter(), distinct()` 등 중간연산을 수행 한 다음 `findAny(), findFirst()` 등의 최종연산 메소드로 조건에 맞는 요소를 선택 할 수 있다.


```java
List<Integer> sampleList = Arrays.asList(17,19,28,37,16);
		
sampleList
	.stream()
	.filter(n -> n % 2 == 0)
	.findAny()
	.ifPresent(n -> System.out.println("2의 배소의 요소 찾기 : " + n));
		
sampleList
	.stream()
	.filter(n -> n % 2 == 0)
	.findFirst()
	.ifPresent(n -> System.out.println("2의 배소의 첫번째 요소 찾기 : " + n));
```

```console
2의 배소의 요소 찾기 : 28
2의 배소의 첫번째 요소 찾기 : 28
```

결과값을 보면 동일하다는 걸 알 수 있다. <br>
왜냐하면, 쇼트서킷 검사(보통 Boolean 연산 특정 조건에 의해 검사결과가 정해지면 추 후 연산은 안하는 행위)를 통해 결과를 찾은 뒤 즉시 실행을 하기 때문이다. <br>

결과값이 동일하지만 findAny() 와 findFirst() 가 모두 존재하는 이유는 `공짜로 얻을 수 있는 병렬성 이 후 후처리` 때문이다. <br>
Collection 클래스에서 Stream 추출 시, parallelStream 을 통해 병렬 처리를 쉽게 수행 할 수 있다. 그러나 `병렬처리를 수행 시 컬렉션의 순서가 보장되지 않는다`. <br>

즉, 병렬처리를 통해 데이터 연산을 수행 했다면, 명시적으로 첫 번째 요소를 찾기 위해 `findFirst()` 를 사용해야만 한다. <br>

아래의 코드를 보면 병렬처리를 했음에도, findFirst() 사용 시 첫번째 요소 찾기를 보장해줌을 알 수 있다.

```java
List<Integer> sampleList = Arrays.asList(17,19,28,37,16);
	
sampleList
	.parallelStream()
	.filter(n -> n % 2 == 0)
	.findAny()
	.ifPresent(n -> System.out.println("2의 배소의 요소 찾기 : " + n));
		
sampleList
	.parallelStream()
	.filter(n -> n % 2 == 0)
	.findFirst()
	.ifPresent(n -> System.out.println("2의 배소의 첫번째 요소 찾기 : " + n));
```

<br>


사용하면서 주의할 점은 `결과 값이 항상 있다는 것을 보장하지 않는다는 점`이다.