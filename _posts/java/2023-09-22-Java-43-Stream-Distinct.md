---
title:  "[Java] 배열에서 특정 Key 의 중복제거"
excerpt: "Stream Filter 와 Predicate 를 이용한 특정 Key 의 중복제거"

tags:
  - [Java, Stream, distinct]

toc: true
toc_sticky: true
toc_label: "[Java]"
 
date: 2023-09-23
last_modified_at: 2023-09-23
---

- 참고사이트
  - [stackoverflow](https://stackoverflow.com/questions/47387475/how-predicate-maintain-state-in-java-8/47392827#47392827){:target="_blank"}
  - [블로그](https://circlee7.medium.com/java8-stream-%ED%8A%B9%EC%A0%95-key-%EC%A4%91%EB%B3%B5%EC%A0%9C%EA%B1%B0-7b10dcede37){:target="_blank"}
  - [블로그](https://jj-yi.tistory.com/60){:target="_blank"}



### Predicate 란?

argument를 받아 boolean 값을 반환하는 함수형 인터페이스 이다.

predicate interface는 T에대한 조건에 대해서 true / false를 반환하는 Functional Interface입니다.
Predicat`<T>`로 사용되고 여기서 T는 파라미터이자 조건이다.
T가 true 또는 false를 return하도록 작성하면 된다.


### Stream 특정 Key 중복제거

Java 에서 배열의 중복제거 하는 방법은 몹시 간단하다. <br>
Stream 의 Distinct 를 이용하면 된다.


```java
List<String> names = Arrays.asList("Jang", "Hyan", "UYjh", "IIuy", "KiM", "Jang");
names.stream()
    .distinct() // 중복 제거
    .forEach(System.out::println);
```

출력값은 아래와 같이 중복요소인 "Jang" 은 하나의 값만 가지게 된다.

```
"Jang", "Hyan", "UYjh", "IIuy", "KiM"
```

<br>

이렇게 단순 배열에서의 중복제거는 몹시 간단하다. <br>


> 하지만 List 안에 Map 이 있거나 Class 가 들어가 있다면?


구현이야 어떻게든 가능하다. Stream 의 map을 이용해  특정 값만 남겨 둔 상태에서 중복제거를 할수도 있고 여러 방법이 존재하겠지만, 우리는 항상 더 나은 방법을 고민해야 한다. <br>

본인은 구글링을 통해 더 나은 방법을 알게 됐고 이를 공유한다.


```java

public static void main(String args[]) {
    List<Map<String, String>> cigarettes = new ArrayList<>();

    Map<String, String> malboro = new HashMap<>();
    malboro.put("name", "말보로");
    malboro.put("englishName", "Malboro");

    Map<String, String> raison = new HashMap<>();
    malboro.put("name", "레종");
    malboro.put("englishName", "raison");

    Map<String, String> malboro2 = new HashMap<>();
    malboro.put("name", "말보로");
    malboro.put("englishName", "Malboro");


    cigarettes.add(malboro);
    cigarettes.add(raison);
    cigarettes.add(malboro2);

    // Stream 의 map을 이용해  특정 값만 남겨 둔 상태에서 중복제거
    ListMap<String, String> distinctVer1 = cigarettes.stream().map(cigarette -> cigarette.get("name")).distinct();

    // Predicate 을 이용해 중복 제거하는 방법
    ListMap<String, String> distinctVer2 = cigarettes.stream().filter(cigarette -> distinctByKey(cigarette.get("name")));

}



public static <T> Predicate<T> distinctByKey( Function<? super T, Object> keyExtractor) {
    Map<Object, Boolean> map = new ConcurrentHashMap<>();
    return t -> map.putIfAbsent(keyExtractor.apply(t), Boolean.TRUE) == null;  
}
```



Predicate 를 리턴하는 distinctByKey 라는 메소드는 메소드의 지역변수로 HashMap을 생성하고 리턴하는 Predicate 에서는 Map을 참조하고 있다.(javascript의 closure 같다.) <br>

