---
title:  "[Java] Setter? Builer? 뭘 사용해야 되지?"
excerpt: "Builder Pattern"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[Builder Pattern]"
 
date: 2022-07-26
last_modified_at: 2022-07-26
---

- 참고사이트
  - [Okky](https://okky.kr/article/396206){:target="_blank"}
  - [블로그](https://effortguy.tistory.com/15){:target="_blank"}
  - [Hashcode](https://hashcode.co.kr/questions/887/%EC%9E%90%EB%B0%94%EC%97%90%EC%84%9C-builder%EB%A5%BC-%EC%93%B0%EB%8A%94-%EC%9D%B4%EC%9C%A0%EB%8A%94-%EB%AD%94%EA%B0%80%EC%9A%94){:target="_blank"}
  - [stack overflow](https://stackoverflow.com/questions/22416239/setters-and-not-or-or-vs-builder-patterns){:target="_blank"}


### ``Builder Pattern 이란?``

`텔리스코핑 생성자 패턴의 안정성`과 `자바빈즈 패턴의 가독성`을 결합한 패턴이다. <br>
즉, 생성자를 통해 빌더 객체를 얻은 후 선택 매개변수는 setter 메소드들로 설정하고 build 메소드를 호출하는 점층적 생성자 패턴의 안정성 + 자바빈즈 패턴의 가독성을 겸비한 패턴 <br>


`점층적 생성자 패턴 (Telescoping Constructor Pattern)` : 생성자를 필수 매개변수 1개만 받는 생성자, 필수 매개변수 1개와 선택 매개변수 1개를 받는 생성자, 선택 매개변수 2개를 받는 생성자 등에 형태로 매개변수 개수만큼 생성자를 늘리는 방식 <br>

`자바빈즈 패턴 (JavaBeans Pattern)` : 매개변수가 없는 생성자로 객체를 만든 후 세터(setter) 메소드들을 호출해 원하는 매개변수의 값을 설정하는 방식 <br>


<hr/>


### ``꼭 써야하나?``

`Telescoping Constructor Pattern`은 매개변수의 수가 증가하면 클라이언트 코드 작성이 힘들고 코드의 가독성이 떨어지는 단점이 있고, <br> 
`JavaBeans Pattern`은객체가 일관된 상태를 유지하지 못할 수 있기 때문에, 스레드에서 안전성을 유지 하려면 프로그래머의 추가적인 노력이 필요하다. <br>


Builder Pattern 의 장단점은 아래와 같다.

- 장점
  - 클라이언트 코드는 작성이 쉽고, 가독성이 좋다.
  - 생성자처럼 빌더는 자신의 매개변수에 불변 규칙(invariants)을 적용할 수 있고 build 메소드는 그런 불변 규칙을 검사할 수 있다.
  - 여러 개의 가변인자(varargs) 매개변수를 가질 수 있다.
  - 유연성이 좋다. 하나의 빌더는 여러 개의 객체를 생성하는데 사용될 수 있으며, 이러한 과정 중에 빌더의 매개변수는 다양하게 조정될 수 있다.
  - 매개변수들의 값이 설정된 빌더는 훌륭한 추상 팩토리를 만든다. 즉, 클라이언트 코드에서는 그런 빌더를 메소드로 전달하여 그 메소드에서 하나 이상의 객체를 생성하게 할 수 있다.

- 단점
  - 어떤 객체를 생성하려면 우선 그것의 빌더를 생성해야 한다. 성능이 매우 중요한 상황에서는 문제가 될 수 있다.
  - 빌더 패턴은 텔리스코핑 패턴보다 코드가 길어지므로, 매개변수가 많을 때(4개 이상)만 사용하는 것이 좋다. 그러나 향후에 매개 변수가 추가될 수 있다는 것을 염두에 두자


<br>

텔리스코핑 생성자 패턴과 자바빈즈 패턴을 사용하다 보면 알 수 있겠지만 가독성과 객체불변을 지키는 건 몹시 중요하다. 
<br>

가독성이 떨어지면 분석하기 어렵고, 객체가 여기저기 다른 메소드를 거치면서 필드 값이 변경된다면.. 추적하기 매우 번거롭기 때문이다. <br>

물론 builder 에 대한 부분을 코딩으로 하면 지저분해 보일 수 있지만, lombok 을 이용한다면 좀 더 클린하게 코드를 작성 할 수 있으며 Entity 와 DTO 변환 시에도 매우 유용하다.

<hr/>


### ``샘플코드``

> Entity Class

```java
@Getter
@NoArgsConstructor
@Proxy(lazy = false)
@Entity(name="voc_category")
public class VocCategoryEntity implements Serializable {
	/**
	 * 
	 */
	private static final long serialVersionUID = -5049789187171798678L;
	
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "id", nullable = false, updatable = false, insertable = false)
	private Long id;
	
	
	@Column(name="name", nullable = false, updatable = false)
	private String name;
	@Column(name="display_name", nullable = false, updatable = false)
	private String displayName;
	
	public VocCategoryEntity(Long id) {
		this.id = id;
	}

	@Builder
	public VocCategoryEntity(Long id, String name, String displayName) {
		this.id = id;
		this.name = name;
		this.displayName = displayName;
	}
	
	public VocCategoryDto.Response toVocCategoryResponse() {
		return VocCategoryDto.Response.builder()
				.id(id)
				.name(name)
				.displayName(displayName)
				.build();
	}
	
	
}
```

<br>

> DTO Class

```java
public class VocCategoryDto {	
	@Builder
	@Getter
	@AllArgsConstructor
	public static class Response {
		private Long id;
		private String name;
		private String displayName;
	}
	
}
```

<br>

> Service

```java
public List<VocCategoryDto.Response> getVocCategory() {
	List<VocCategoryEntity> category = vocCategoryRepository.findAll();
	return category.stream().map(VocCategoryEntity::toVocCategoryResponse).collect(toList());
}
```
<br>

위 코드를 보면 Entity 에서 DTO 로 Builder 해주는 메소드가 존재하고, stream 을 이용하여 Entity 를 DTO 로 변환해주는 부분이다. 매개변수가 3개 이기 떄문에 Builder Pattern 을 이용하는 것 보다, Setter 메소드를 이용하는게 좋을 수 있다. 하지만 실무에서는 기획이 변경되고 언제라도 필드가 추가 될 수도 삭제 될 수도 있다는 점을 생각하자. <br>

<br>

위 코드를 확인하고 싶다면 [여기](https://github.com/ymkmoon/toyseven){:target="_blank"}ㄴ에서 확인 가능하다.