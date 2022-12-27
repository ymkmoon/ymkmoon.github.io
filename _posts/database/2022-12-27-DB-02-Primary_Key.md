---
title:  "[DB] PK는 자연키? 대체키? 복합키?"
excerpt: "Primary Key의 종류와 뭘 써야 좋을지에 대한 내용 "

tags:
  - [Database]

toc: true
toc_sticky: true
toc_label: "[Primary Key]"
 
date: 2022-12-27
last_modified_at: 2022-12-27
---

- 참고사이트
  - [블로그](https://velog.io/@park2348190/PK%EB%8A%94-Surrogate-Key-Natural-Key){:target="_blank"}
  - [Stackoverflow](https://stackoverflow.com/questions/9967036/web-application-user-table-primary-key-surrogate-key-vs-username-vs-email-vs-cu/9968767#9968767){:target="_blank"}
  - [agiledata.org](http://www.agiledata.org/essays/keys.html){:target="_blank"}
  - [Stackoverflow](https://stackoverflow.com/questions/9967036/web-application-user-table-primary-key-surrogate-key-vs-username-vs-email-vs-cu){:target="_blank"}



### 시작 하기 전

DB 는 크게 `NoSQL` 과 `RDBMS`, 2가지로 나뉘어진다. <br>


RDBMS는 관계형 데이터베이스 관리 시스템을 의미하고, NoSQL은  RDB 형태의 관계형 데이터베이스가 아닌 다른 형태의 데이터 저장 기술을 의미한다. <br>


RDBMS 를 사용해봤다면, `Primary Key`, `Foreign Key` 등은 들어 봤을테고, PK, FK 를 이용해 DB 스키마의 관계를 지정 할 수 있고 보통은 간단하고 익숙한 `AUTO INCREMENT` 를 많이 사용했을거라고 생각한다. <br>

이렇다 보니 본인도 그냥 습관적으로 `AUTO INCREMENT` 를 사용했고, 이 값은 `unique` 하고 `nullable` 하지 않으니 가장 좋은 방법일거라고 생각했다. 하지만 좀 더 공부를 하다보니 차이점을 다른 방법이 존재하고, 이 내용을 기록하기 위해 포스트를 작성한다.

<hr>


### key 종류

Database 에서 row(=데이터, 레코드) 를 구분 할 수 있도록 `Key(=키)` 를 사용하고, 그 종류는 아래와 같다.

- Composite Key: 두 가지 이상의 컬럼으로 이루어진 키.
- Natural Key: 현실 세계의 속성으로 적용된 키. 이메일, 주민등록번호 등.
- Surrogate Key: 비즈니스적 의미가 없는 키. 정수형 식별자 등.
- Candidate Key: Primary Key가 될 수 있는 unique 한 키의 모음. Composite Key도 해당될 수 있음.
- Primary Key: 해당 엔티티의 식별자 키.
- Alternate Key: Primary Key 대신 사용할 수 있는 레코드의 unique 키.
- Foreign Key: 외부 엔티티의 PK를 가리키는 키.


`Natural Key`, `Surrogate Key`는 간단히 말해서 비즈니스적인 의미, 애플리케이션에서 활용할 여지가 있는 `unique`한 데이터로 이루어진 컬럼인지 차이다. 
<br>


언급했듯이 이메일, 주민등록번호 같은 데이터는 어떤 의미(전자메일 주소, 개인 식별 번호)가 있는 데이터기 때문에 `Natural Key`에 해당하지만 1, 2, 3, ... 으로 증가하는 AUTO INCREMENT 정수형 식별자는 어떤 의미도 없기 때문에 `Surrogate Key`에 해당하는 것이다.

<hr>


### 뭘 사용해야 하지?

데이터는 누적이다. 시간이 지날수록 쌓이고, 그만큼 무거워(?)진다. 

<br>

`이메일(Natural Key)`도 unique not null 하고, `정수식별자(Surrogate Key)`도 unique not null 하다면 불필요한 정수식별자 데이터를 쌓아갈 필요 없이 Surrogate Key 대신 Natural Key 를 사용하는게 더 좋은 방법일까? 
<br>

물론 클라이언트의 요구사항에 따라 다르겠지만, 사용자의 식별자, 즉 개인정보에 해당 할 수 있는 이메일이나 아이다 같은 부분을 노출하고 싶지 않은 경우에는 `Surrogate Key` 를 사용하는게 좋은 방안이 될 수 있다.
<br>


반대로 클아이언트가 이런 정보를 노출하기를 원한다면, 굳이 Surrogate Key 에 대한 데이터를 누적시킬 필요 없이 이메일 같은 `Natural Key` 를 사용하는게 좋을 수 있다.
<br>

중요한 차이는 이메일 주소는 Database 레코드 상으로는 unique 하며 not null 이지만, 현실 세계에서는 `소실` 될 수 있는, stable 하지 않는 `key` 라는 것 이다. 없을거라고 생각하지만, 만에하나 이메일 서비스가 종료 된다면 해당 이메일 주소는 더 이상 유효하지 않기 때문에 추후 사용자가 비밀번호 재설정 등 이메일 주소를 사용하게 될 경우 문제가 발생 할 수 있다.
<br>


즉 레코드를 구별하즌 식별자가 현실 세계의 영향을 받을 수 있다는 점은 이메일 컬럼을 `Primary Key` 로 사용하기 전 충분히 고려해봐야 한다는 의미이다. 물론 정수형 식별자도 분산 데이터베이스 환경에서 문제가 될 수 있다.

<hr>


### unique 컬럼의 인덱스

MySQL 에서는 unique 한 컬럼에 대하여 `UNIQUE INDEX` 를 생성한다. 보통 JPA 에서는 아래 처럼 코드 엔티티를 디자인 하게 되는데 그럴 경우 아래 쿼리가 실행된다.

```java
@Entity(name="admin") 
public class AdminEntity extends BaseTimeEntity {

	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	@Column(name = "id", nullable = false, updatable = false, insertable = false)
	private Long id;
	
  @Column(name = "email", unique = true, nullable = false)
  private String email;

	@Column(name = "nickname", nullable = false)
	private String nickname;
```


```sql
create table admin (id bigint not null, email varchar(255) not null, name varchar(255) not null, primary key (id)) engine=InnoDB
alter table admin add constraint UK_gig8560dcl8rp20ak28x7pjki unique (email)
```


엔티티의 email 필드에 @Column 어노테이션의 unique 속성을 설정했기 때문에 테이블을 생성한 후 unique constraint를 거는 것을 볼 수 있다. MySQL Workbench에서 확인해보면 역시 인덱스가 생성된 것을 볼 수 있다. <br>

그렇기 때문에 약 10000 건의 레코드가 저장된 데이터베이스에서 식별자 컬럼을 기반으로 조회하든 이메일 컬럼을 기반으로 조회하든 같은 실행 계획을 가지는 것을 볼 수 있다

<hr>


### 저장 공간의 차이?

만약 데이터 저장 공간에 민감한 환경이라면 `Surrogate Key(정수 식별자)` 를 제거하고 `Natural Key(이메일)` 을 Primary Key 로 사용해서 불필요한 인덱스를 없애는 것도 가능하겠지만, 그럴 경우가 그렇게 많지는 않을거라고 생각한다. 
<br>

위에서 생성한 테이블에 10,000의 데이터를 집어 넣고 정수식별자와 이메일컬럼을 둘 다 사용하는 경우와 이메일 컬럼만을 사용하는 경우를 비교하면 데이터 `저장공간의 1/3 정도를 차지하는 인덱스 저장 공간이 사라졌기 때문에`, 용량이 확실히 줄어든 걸 확인 할 수 있다. 
<br>


키 자체의 크기 차이(정수, 문자열 자로형)도 [Stackoverflow](https://stackoverflow.com/questions/9967036/web-application-user-table-primary-key-surrogate-key-vs-username-vs-email-vs-cu/9968767#9968767){:target="_blank"} 의 답변으로는 전체 성능에 다이나믹한 영향을 주지 않는다고 한다.

<hr>


### 결론

솔직히 정답은 잘 모르겠다. `unique 인덱스(이메일)`를 사용해서 저장공간을 줄이느냐, 아니면 `정수식별자(Auto Increment)`를  사용해 미세하지만 조금이라도 빠른 성능을 기대하냐. 
<br>


나라면, 일단 unique 인덱스로 개발을 진행하고 나중에 이메일 서비스가 전세계적으로 종료 된다면 그떄 정수식별자를 추가하고 복합키를 이용하는게 더 나은 방향이 아닌가 생각이 들기 때문에 이렇게 진행하지 않을까? 
<br>



의견이 있다면 댓글로 남겨주시면 감사합니다.