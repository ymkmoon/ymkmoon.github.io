---
title:  "[Java] 병렬 Stream"
excerpt: "병렬 스트림"

tags:
  - [Java, Stream]

toc: true
toc_sticky: true
toc_label: "[병렬 스트림]"
 
date: 2022-07-04
last_modified_at: 2022-07-04
---

- 참고사이트
  - [블로그](https://velog.io/@seungmoo/Java-Stream-%EB%B3%91%EB%A0%AC-%EC%B2%98%EB%A6%AC-2-%EB%B0%98%EB%B3%B5%EB%AC%B8-vs-%EC%88%9C%EC%B0%A8%EC%8A%A4%ED%8A%B8%EB%A6%BC-vs-%EB%B3%91%EB%A0%AC%EC%8A%A4%ED%8A%B8%EB%A6%BC-IntelliJ-Plugin%EC%9C%BC%EB%A1%9C-%EC%84%B1%EB%8A%A5-%EA%B3%84%EC%82%B0%EB%B9%84%EA%B5%90-%ED%95%98%EA%B8%B0){:target="_blank"}


### ``1. 병렬 스트림(Parallel Stream) 이란?``

병렬 스트림이란 각각의 스레드에서 처리할 수 있도록 스트림 요소를 여러 청크로 분할한 스트이다. <br>
병렬 스트림을 이용하면 모든 멀티코어 프로세서가 각각의 청크를 처리하도록 할당할 수 있다. 


<br>

### ``2. 병렬 스트림의 단점``

- 반복 결과로 박싱 된 객체가 만들어 지므로 숫자를 더하려면 언박싱을 해야한다.(단순 for-loop 대비 단점)
- 반복 작업은 병렬로 수행 할 수 있는 독립 단위로 나누기가 어렵다. (반복 계산 iterate 작업 시, 병렬 Task 단위로 독립적으로 쪼개기가 어렵다.)
- 비용이 많이 발생한다. (스트림을 재귀분할하고, 각 서브스트림을 서로 다른 스레드의 리듀싱 연산으로 할당하고, 이들 결과를 하나의 값으로 합쳐야 하기 때문)


위 의 문제들 때문에 비용을 상쇄할 만큼의 큰 데이터들에 대해서만 병렬로 수행 하도록 해야한다.


### ``3. 샘플코드 ``

아래 코드는 순차적으로 List 내부의 Map 에서 특정 데이터를 뽑아 String Array 로 return 해주는 내용이다. 

<br>

```java
private String getString(List<Map<String, Object>> list, String key) {
	return StreamSupport.stream(list.spliterator(), false)
		.map(map -> map)
		.filter(map -> !map.isEmpty())
		.filter(map -> !map.get(key).toString().isBlank())
		.map(map -> map.get(key).toString())
		.collect(Collectors.joining("_"));
}
```

<br>

위 코드에서 StreamSupport.stream()의 두번째 파라미터를 true로 주면 병렬 스트림 처리도 가능하다. (false 일 경우에는 순차 스트림)