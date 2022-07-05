---
title:  "[Java] Stream FlatMap - 두 객체 묶기"
excerpt: "Stream FlatMap을 이용하여 두개의 객체 스트림을 하나로 묶기"

tags:
  - [Java, Stream]

toc: true
toc_sticky: true
toc_label: "[Stream FlatMap]"
 
date: 2022-07-05
last_modified_at: 2022-07-05
---

- 참고사이트
  - [블로그](https://doohyun.tistory.com/24?category=592214){:target="_blank"}


### ``1. FlatMap 이란?``

중복된 스트림을 1차원으로 평면화 시키는 메소드 이다. <br>
-> 두 객체의 스트림을 하나로 묶는 것 (SQL의 JOIN과 같다고 볼 수 있다.)


<br>

### ``2. FlatMap 사용하기 ``

```java
List<MemberVo> memberList = Arrays.asList(
	new MemberVo(1, 27, "남두현"), 
	new MemberVo(2, 20, "유형주"),
	new MemberVo(3, 20, "태재영"), 
	new MemberVo(4, 40, "남궁계홍"));

List<BoardDetailVo> boardList = Arrays.asList(
	new BoardDetailVo(1, "람다게시판입니다.", "람다에 관련된 클만 올려주세요."),
	new BoardDetailVo(1, "람다란 무엇이니가?", "함수형 프로그래밍의 꽃?"),
	new BoardDetailVo(2, "물어볼 게 있습니다.", "람다공부는 어떻게 하죠?"));
 
memberList.stream()
	.flatMap(i ->
		boardList.stream()
		.filter(j -> j.getSn() == i.getSn().intValue())
		.map(j -> {
			MemberBoardResultVo result = new MemberBoardResultVo();
			result.setSn(i.getSn());
			result.setName(i.getName());
			result.setContents(j.getContens());
			result.setTitle(j.getTitle());
			return result;
		})
	)
	.forEach(ret -> {
		System.out.println(String.format("[%s]%s:%s", 
		ret.getName(), 
		ret.getTitle(), 
		ret.getContents()));
	});

```

```console
[남두현]람다게시판입니다.:람다에 관련된 클만 올려주세요.
[남두현]람다란 무엇이니가?:함수형 프로그래밍의 꽃?
[유형주]물어볼 게 있습니다.:람다공부는 어떻게 하죠?
```

<br>

### ``3. 로직 설명 ``

1) memberList의 stream이 주가 되어, flatMap 내부에서 boardList의 Stream을 사용하고 있다. <br> 

```java
memberList.stream().flatMap(i -> boardList.stream()....)
```


2) boardList의 stream 내부 파이프라인 중 filter는 memberList 의 각 객체 중, 본인 객체의 sn이 서로 같은 것만 찾으라는 조건을 명시하고 있다. <br> 

```java
filter(j -> j.getSn() == i.getSn().intValue())
```


3) oardList stream의 내부 map에서는 Function 의 역할로 MemberBoardResultVo 를 출력하도록 되어 있다. <br>
람다표현식을 사용하여, MemberVo와 BoardDetailVo 에서 각각 필요한 정보를 주입한다.

```java
map(j -> {
	MemberBoardResultVo result = new MemberBoardResultVo();
	result.setSn(i.getSn());
	result.setName(i.getName());
	result.setContents(j.getContens());
	result.setTitle(j.getTitle());
	return result;
})
```


4) foreach에서는 위의 flatMap으로부터 출력된 컨테이너를 순회하며, 필요한 정보를 출력하고 있다. <br>

```java
forEach(ret -> {
	System.out.println(
		String.format("[%s]%s:%s", 
		ret.getName(), 
		ret.getTitle(), 
		ret.getContents())
	);
});
```
