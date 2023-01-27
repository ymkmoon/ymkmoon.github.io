---
title:  "[Java] Querydsl 이란? 검색 기능과 페이징"
excerpt: "JPA 에서 Querydsl을 이용한 검색 기능 구현과 Pagealbe을 이용한 페이징처리"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[Querydsl]"
 
date: 2023-01-20
last_modified_at: 2023-01-26
---

- 참고사이트
  - [Querydsl Document](http://querydsl.com/static/querydsl/latest/reference/html/ch02.html#jpa_integration){:target="_blank"}
  - [Querydsl Github](https://github.com/querydsl/querydsl){:target="_blank"}
  - [우아한형제들 블로그](https://velog.io/@youngerjesus/%EC%9A%B0%EC%95%84%ED%95%9C-%ED%98%95%EC%A0%9C%EB%93%A4%EC%9D%98-Querydsl-%ED%99%9C%EC%9A%A9%EB%B2%95#1-extends--implements-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EC%95%8A%EA%B8%B0){:target="_blank"}
  - [Github Sample Repository](https://github.com/DolphaGo/spring-querydsl-example){:target="_blank"}
  - [블로그](https://dico.me/java/articles/290/ko){:target="_blank"}

### Querydsl 이란?

Querydsl은 DB 에서 CRUD 작업을 할 때 SQL 쿼리 대신 Java 코드로 작성할 수 있도록 도와주는 라이브러리 이다. <br>

JPA 를 사용하면, 기본적인 CRUD 에 대해서는 아주 훌륭하지만 검색에 따라 애로사항이 발생 할 수 있다. <br>

```
게시판 검색 기능에 대해 클라이언트의 요구사항이 있다고 생각해보자. 

"저는 날짜로도 검색 가능하고 제목으로도 검색 가능했으면 좋겠어요. 그리고 작성자도 검색 가능하게 추가해주세요!" 
```

위 요구사항을 해결하기 위해선 이러한 방법들이 존재 할 것이다.

- 날짜, 제목, 작성자로 검색 가능한 각각의 엔드포인트 생성
- 하나의 엔드포인트에서 조건값을 받아 핸들링
- ...


첫 번째 방법을 이용하게 된다면

```
"FE 개발자님! 검색 조건에 따라서 엔드포인트가 달라져야 되니 프론트 에서 날짜는 여기, 제목은 여기, 작성자는 여기로 요청 주시면 돼요!"
```

각 테이블마다 엔드포인트는 셀 수 없이 늘어나게 될 것이고, 결국 FE, BE 둘 다 재사용성이 낮아지게 된다.<br>

하지만 하나의 엔드포인트에서 조건값을 받아 핸들링 한다면? <br>
기존 조회 기능을 Querydsl 을 사용하도록 리팩토링하거나, 이미 서비스 중이라면 `하나의 엔드포인트 만을 추가함으로 해결 할 수 있다`. <br>


해당 챕터를 진행하기 위해서는 JPA 를 선행학습 해보는 걸 추천한다.<br>

[JPA란?](https://ymkmoon.github.io/Java-31-Jpa/){:target="_blank"}

<hr/>


### 의존성 추가

> build.gradle

```gradle
plugins {
  id 'com.ewerk.gradle.plugins.querydsl' version '1.0.10' // Querydsl
}

dependencies {
  implementation "com.querydsl:querydsl-jpa" // Querydsl
  implementation "com.querydsl:querydsl-apt" // Querydsl
}
```



### QEntity 파일 생성

JPA에서 사용 하는 엔티티를 Querydsl 에서 사용가능 하도록 QEntity 파일 생성

> build.gradle

```gradle
def querydslDir = "src/main/generated/queryDsl" // Querydsl

querydsl {
    jpa = true // Querydsl
    querydslSourcesDir = querydslDir // Querydsl
}
sourceSets {
    main.java.srcDir querydslDir // Querydsl
}
configurations {
    querydsl.extendsFrom compileClasspath // Querydsl
}
compileQuerydsl {
    options.annotationProcessorPath = configurations.querydsl // Querydsl
}
```

> gradle build

![Querydsl](/assets/image/java/Querydsl_01.PNG)

빌드하고나면 기존에 @Entity 어노테이션이 작성되어 있던 클래스 들이 `src/main/generated/queryDsl` 경로에 생성된다.

![Querydsl](/assets/image/java/Querydsl_02.PNG)

<hr/>


### QEntity 파일 경로 설정

QEntity 파일들은 내가 설정 한 `src/main/generated/queryDsl` 경로에 생성되기 때문에, 해당 경로에 대해 설정을 추가해야 한다. <br>


이클립스에서 프로젝를 선택 후 "Alt + Enter" 를 누르면 Properties 창이 표시된다.

![Querydsl](/assets/image/java/Querydsl_04.PNG)


Java Build Path -> Source 를 선택하고 "ADD Folder" 를 선택한다.

![Querydsl](/assets/image/java/Querydsl_05.PNG)


`src/main/generated/queryDsl` 경로를 체크 한 뒤 OK 를 눌러 창을 종료하고 Apply 한다.

![Querydsl](/assets/image/java/Querydsl_06.PNG)

<hr/>


### Config 파일 생성

JPAQueryFactory : Querydsl 생성 후 EntityManager 를 통해 쿼리 결과를 반환할 수 있도록 해준다.

> QuerydslConfig.java

```java
import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import com.querydsl.jpa.impl.JPAQueryFactory;

@Configuration
public class QuerydslConfig {
	
	@PersistenceContext
	private EntityManager entityManager;
	
	@Bean 
	JPAQueryFactory jpaQueryFactory() {
		return new JPAQueryFactory(entityManager);
	}

}
```

<hr/>


### DTO 추가

메인 모델 속성이 외부에 노출되면 보안 문제가 발생할 수 있다. 그러니 Repose 데이터를 반환 할 때 사용 할 DTO 를 추가해주자.

- @Builder : JPA 사용 시 Entity -> DTO 변환을 위한 어노테이션
- @QueryProjection : gradle build 를 통해 QDto 파일을 생성


> VocQuestionDto.java

```java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class VocQuestionDto {

  @Getter
  public static class Response {
    private Long id;
    private String category;
    private String title;
    private String content;
    private String email;
    private String username;
    private String stationId;
    private int needReply;
    private boolean active;
    
    @JsonDeserialize(using = CustomLocalDateTimeDeserializer.class) 
    @JsonSerialize(using = CustomLocalDateTimeSerializer.class) 
    private LocalDateTime createdAt;
    
    @JsonDeserialize(using = CustomLocalDateTimeDeserializer.class) 
    @JsonSerialize(using = CustomLocalDateTimeSerializer.class) 
    private LocalDateTime updatedAt;

    @Builder
    @QueryProjection
    public Response(Long id, String category, String title, String content, String email, String username,
        String stationId, int needReply, LocalDateTime createdAt, LocalDateTime updatedAt, boolean active) {
      this.id = id;
      this.category = category;
      this.title = title;
      this.content = content;
      this.email = email;
      this.username = username;
      this.stationId = stationId;
      this.needReply = needReply;
      this.createdAt = createdAt;
      this.updatedAt = updatedAt;
      this.active = active;
    }
  }

}
```

@QueryProjection 을 추가 후 gradle 을 다시 빌드하면 QDto 파일이 생성된 걸 확인 할 수 있다.

![Querydsl](/assets/image/java/Querydsl_03.PNG)

<hr/>


### 검색 조건 추가

검색 기능을 위해 해당 테이블에서 사용 할 조건 검색에 대한 값을 추가하자.

> VocQuestionSearchCondition.java

```java
@NoArgsConstructor
@AllArgsConstructor
@Setter
@Getter
public class VocQuestionSearchCondition {
	
  private String categoryId;
  private String title;
  private String username;
  private String email;
  private String stationId;
}
```

<hr/>


### 컨트롤러 추가

> VocController.java

```java
@GetMapping("/v2")
public ResponseEntity<Page<VocQuestionDto.Response>> getVocQuestionsVer2(
    VocQuestionSearchCondition condition,
    @RequestParam(name="offset") @NotNull long offset,
    @RequestParam(name="limit") @NotNull int limit,
    Sort sort) {
  Pageable pageable = new OffsetBasedPageRequest(offset, limit, sort);
  return new ResponseEntity<>(vocQuestionService.getVocQuestionsSearchable(pageable, condition), HttpStatus.OK);
}
```

파라미터에서 offset, limit, sort 는 페이징과 정렬 처리를 위한 값이며, <br>
VocQuestionSearchCondition 은 검색을 위한 내용이다. <br>

OffsetBasedPageRequest 에 대한 코드는 github 에서 확인 가능하다.


[github](https://github.com/ymkmoon/toyseven/blob/develop/src/main/java/com/toyseven/ymk/common/OffsetBasedPageRequest.java){:target="_blank"}

<hr/>


### 서비스 추가

> VocQuestionServiceImpl.java

```java
private final VocQuestionRepositoryCustom customRepository;

@Override
public Page<VocQuestionDto.Response> getVocQuestionsSearchable(Pageable pageable, VocQuestionSearchCondition condition) {
  return customRepository.searchVocQuestion(condition, pageable);
}
```

<hr/>


### Custom Repository 추가

위 까지의 내용은 준비 단계 였고, 사실 아래 내용이 실제 Querydsl 을 사용하는 부분이다.

> VocQuestionRepositoryCustom.java

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageImpl;
import org.springframework.data.domain.Pageable;
import org.springframework.stereotype.Repository;
import org.springframework.util.StringUtils;

import com.querydsl.core.QueryResults;
import com.querydsl.core.types.dsl.BooleanExpression;
import com.querydsl.jpa.impl.JPAQueryFactory;
import com.toyseven.ymk.common.dto.QVocQuestionDto_Response;
import com.toyseven.ymk.common.dto.VocQuestionDto;
import com.toyseven.ymk.common.model.entity.QVocQuestionEntity;
import com.toyseven.ymk.common.search.VocQuestionSearchCondition;

import lombok.RequiredArgsConstructor;

@Repository
@RequiredArgsConstructor
public class VocQuestionRepositoryCustom {
	
	private final JPAQueryFactory queryFactory; // 쿼리 결과를 반환 할 수 있도록 Bean 에서 가지고 온다.
	private final QVocQuestionEntity question = QVocQuestionEntity.vocQuestionEntity; // QEntity 클래스에서 엔티티를 가지고온다.
	
	public Page<VocQuestionDto.Response> searchVocQuestion(final VocQuestionSearchCondition condition, final Pageable pageable) {
    QueryResults<VocQuestionDto.Response> result = 
      queryFactory.select(
        // 조회 한 데이터를 DTO 로 변환시켜주는 부분이다.
        new QVocQuestionDto_Response(
            question.id,
            question.category.displayName,
            question.title,
            question.content,
            
            question.email,
            question.username,
            question.stationId.stationId,
            question.needReply,
            
            question.createdAt,
            question.updatedAt,
            question.active
        )
      )
      .from(question)
      .where(
        // 조건절에 해당하는 부분이다.
        stationIdEq(condition.getStationId()),
        titleEq(condition.getTitle()),
        usernameEq(condition.getUsername()),
        emailEq(condition.getEmail()),
        categoryIdEq(condition.getCategoryId())
      )
      // 데이터는 누적이고, 전체 검색은 매우 위험하다.
      // 결과의 row 제한을 추가하는 부분이다.
      // 결과적으로 페이징 처리와 같다고 보면 된다.
      .offset(pageable.getOffset())
      .limit(pageable.getPageSize())
      .fetchResults();
      
    List<VocQuestionDto.Response> content = result.getResults();
//  long total = result.getTotal();
//  return new PageImpl<>(content, pageable, total);
    
    return new PageImpl<>(content);
  }

  private BooleanExpression categoryIdEq(final String categoryId) { 
    return StringUtils.hasText(categoryId) ? question.category.displayName.eq(categoryId) : null;
  }

  private BooleanExpression titleEq(final String title) { 
		return StringUtils.hasText(title) ? question.title.eq(title) : null;
	}

  private BooleanExpression usernameEq(final String username) { 
		return StringUtils.hasText(username) ? question.username.eq(username) : null;
	}

  private BooleanExpression emailEq(final String email) { 
		return StringUtils.hasText(email) ? question.email.eq(email) : null;
	}

  private BooleanExpression stationIdEq(final String stationId) { 
    return StringUtils.hasText(stationId) ? question.stationId.stationId.eq(stationId) : null;
  }

}

```

<hr/>


### 검색 테스트

위까지 진행 후 서버를 올려보면 무리없이 작동할 것 이라고 생각하지만, 만약 안된다면 아래내용을 참고하여 @EnableJpaRepositories 어노테이션을 추가하시면 될거라고 생각합니다.

> ToysevenApplication.java

```java
import org.springframework.data.jpa.repository.config.EnableJpaAuditing;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

@SpringBootApplication
@EnableJpaAuditing
@EnableJpaRepositories // 추가
public class ToysevenApplication {

	public static void main(String[] args) {
		SpringApplication application = new SpringApplication(ToysevenApplication.class);
		application.addListeners(new ApplicationPidFileWriter());
		application.run(args);
	}

}
```


### 정렬 기능 추가

정렬에 대한 기능추가를 위해 유틸클래스를 추가한다. 

> QuerydslUtil.java

```java
import com.querydsl.core.types.Order;
import com.querydsl.core.types.OrderSpecifier;
import com.querydsl.core.types.Path;
import com.querydsl.core.types.dsl.Expressions;

import lombok.experimental.UtilityClass;

@UtilityClass
public class QuerydslUtil {

	@SuppressWarnings({ "rawtypes", "unchecked" })
	public static OrderSpecifier<?> getSortedColumn(Order order, Path<?> parent, String fieldName) {
        Path<Object> fieldPath = Expressions.path(Object.class, parent, fieldName);
        return new OrderSpecifier(order, fieldPath);
    }
}
```


추가 된 유틸클래스를 이용하여 기능 구현

> VocQuestionRepositoryCustom.java

```java
public Page<VocQuestionDto.Response> searchVocQuestion(final VocQuestionSearchCondition condition, final Pageable pageable) {
	
	List<OrderSpecifier<?>> orders = getAllOrderSpecifiers(pageable);
	
	QueryResults<VocQuestionDto.Response> result = queryFactory
			.select(
				new QVocQuestionDto_Response(
						question.id,
						question.category.displayName,
						question.title,
						question.content,
						
						question.email,
						question.username,
	    				question.stationId.stationId,
	    				question.needReply,
	    				
	    				question.createdAt,
	    				question.updatedAt,
	    				question.active
				)
			)
			.from(question)
			.where(
				stationIdEq(condition.getStationId()),
				titleEq(condition.getTitle()),
				usernameEq(condition.getUsername()),
				emailEq(condition.getEmail()),
				categoryIdEq(condition.getCategoryId())
			)
			.offset(pageable.getOffset())
			.limit(pageable.getPageSize())
			.orderBy(orders.stream().toArray(OrderSpecifier[]::new))
			.fetchResults();
    
	List<VocQuestionDto.Response> content = result.getResults();
	
	return new PageImpl<>(content);
}

private List<OrderSpecifier<?>> getAllOrderSpecifiers(Pageable pageable) {
    List<OrderSpecifier<?>> orders = new ArrayList<>();
    OrderSpecifier<?> updatedAt = QuerydslUtil.getSortedColumn(Order.DESC, question, "updatedAt");
    orders.add(updatedAt);
    return orders;
}
```



### 마치며

Querydsl 이 특정 부분에선 간편하지만 만능은 아니다. <br>

예를 들면 검색부분에서 확실히 간편하게 구현 가능하다고 생각 된다. <br>
<br>
일반적인 방법으로 JPA Repository Interface 을 이용한다면, findById() 나 save() 등 과 같은 기본적인 인터페이스가 아니라면 가독성이 떨어진다. 코드를 작성하는 당사자는 가이드대로 메소드명만 만들면 되니, 편할 지 몰라도 추가적인 조건이 둘 이상이 되면 메소드 명을 보고 목적이 직관적으로 파악되지 않는다.

```java
List<Board> findByIdOrUsernameAndActive
```

그리고 통계와 같은 복잡한 쿼리를 위해서는 SQLMapper(MyBatis) 방식이 필요 할 수 있다. <br>
<br>
무조건 하나의 방식을 고집하기 보다는 상황에 맞게 사용 할 줄 알아야 되고, 그걸 위해 모두 미리 공부하여 경험을 해보는 게 중요하다고 생각한다. 내가 언제 어디서 어떤 기술을 사용 할 지 알 수 없기 때문에 <br>
<br>



모든소스는 아래 github 을 클릭하여 확인 가능 합니다. <br>

[github](https://github.com/ymkmoon/toyseven){:target="_blank"}
<br>
