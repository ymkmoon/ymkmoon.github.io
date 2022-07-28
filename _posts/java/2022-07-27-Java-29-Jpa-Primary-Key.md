---
title:  "[Java] JPA 기본키 매핑 전략과 IDENTITY의 주의점"
excerpt: "JPA 기본키 매핑 전략과 IDENTITY 의 주의점"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[JPA]"
 
date: 2022-07-27
last_modified_at: 2022-07-27
---

- 참고사이트
  - [블로그](https://coding-start.tistory.com/71){:target="_blank"}
  - [블로그](https://knight76.tistory.com/entry/auto-increment-%EC%9D%B4%EC%8A%88){:target="_blank"}
  - [블로그](https://www.letmecompile.com/mysql-innodb-auto-increment-%EC%84%B1%EB%8A%A5-%EC%B5%9C%EC%A0%81%ED%99%94/){:target="_blank"}
  - [블로그](https://heekim0719.tistory.com/411){:target="_blank"}
  - [블로그](https://whitepro.tistory.com/281){:target="_blank"}




### ``JPA 기본키 매핑 전략``

JPA 에서 기본키 매핑 전략은 아래와 같다.

- 직접할당 : 기본키를 애플리케이션에서 직접 Entity 클래스의 @Id 필드에 set 해준다.
- 자동생성 : 대리키 사용방식(IDENTITY, SEQUENCE, TABLE, ...)

자동생성 전략이 다양한 이유는 데이터베이스 벤더바다 지원하는 방식이 다르기 떄문이다. <br>
위 방식에서 `IDENTITY, SEQUENCE` 는 데이터베이스 벤더에 의존적이다. 하지만 `TABLE` 방식은 키 생성용 테이블을 따로 하나 만들어두고,
마치 시퀀스 처럼 사용하는 방법이기 때문에 벤더에 의존하지 않는다. (각각 장단점이 존재) <br>

IDENTITY 전략을 사용하면 MySQL 의 AUTO INCREMENT 처럼 사용


#### IDENTITY

`IDENTITY`는 기본 키 생성을 데이터베이스에 위임하는 전략이며 주로 `MySQL, PostgreSQL` 등2에서 사용한다.<br>

```java
/*
 * 유니크키 설정 및 nullable,length 등의 속성은 모두 auto DDL을 사용했을 때만 유효한 설정이다.
 * 즉, 테이블을 직접 생성한다면 적용되지 않는다. 하지만 테이블과 객체간의 관계표현에 있어 해당 설정들을 해놓으면
 * 엔티티 클래스만 봐도 테이블의 구조가 파악되기에 가독성을 위해서라도 설정을 해놓는 것이 좋다.
 */
@Entity
@Table(name = "MEMBER"
        ,uniqueConstraints = {
        @UniqueConstraint(
                name = "NAME_AGE_UNIQUE",
                columnNames = {"NAME","AGE"} //uniqueConstraints는 auto DDL 속성을 사용할때만 유효한 설정이다.
        )
})
@Getter
@Setter
@ToString
public class Member {
    @Id
    @Column(name = "ID")
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long id;
 
    /*
     * not null
     * varchar2(10) -> 기본값 255;
     */
    @Column(name = "NAME",nullable=false,length=10)
    private String username;
 
    private Integer age;
 
    /*
     * EnumType의 기본값 설정은 정수이다.
     */
    @Enumerated(EnumType.STRING)
    @Column(name="ROLE_TYPE",nullable=false,length=20)
    private RoleType roleType;
 
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;
 
    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;
 
    @Lob
    private String description;
}
 

```

IDENTITY 전략의 단점이라고 할 수 있는 특징이 있다. 그것은 데이터베이스에 값을 저장하고 나서야 기본 키 값을 구할 수 있다. <br>
여기서 기본 키 값을 구하는 것이 무슨 상관이냐? 라고 말할 수 있지만 영속성 컨텍스트에 1차캐시(엔티티의 영속상태화)를 하기 위해서는 구분자로 기본키(@Id) 필드를 이용한다. <br>
즉, 영속성 컨텍스트에 캐싱을 하기위한 primary key 값을 가져오기 위하여 테이블을 추가로 조회하게 된다. <br>
그래서 다른 전략과는 다른 행동중 하나가 persist()호출을 하자마자 지연쓰기를 하는 것이 아니라, primary key값을 가져오기 위하여 바로 flush를 호출하게 된다. <br>


##### IDENTITY 의 주의점

IDENTITY 의 주의점은 AUTO INCREMENT 는 Transaction 을 사용해도 rollback 되지 않는다. <br>
이게 무슨말이냐면, Insert 쿼리를 실패해도 Table 의 AUTO INCREMENT 컬럼의 값은 1이 증가된다. <br>

텅 비어있는 사원 테이블이 존재하고, 테이블의 id 값은 AUTO INCREMENT 로 설정되어 있다. <br>
JAP 로직을 이용하여 사원 테이블에 2번에 save 를 실행했지만, 첫번째 save만 성공하고 두번째 save는 실패했다. <br>
여기서 세번째 save를 실행했고 성공적으로 save 되었을 때, ID 컬럼의 값은 무엇일까? <br>
<br>

대부분 2라고 생각하겠지만, 실제 DB에 입력 된 값은 3이 입력되어있다. <br>

왜냐하면 Transaction 으로 인해 로직 자체는 rollback 되었지만, 이미 테이블에 적용 된 AUTO INCREMENT 값은 증가 했기 때문이다. <br>
`AUTO INCREMENT 는 Transaction 이 적용되지 않는다. 따라서 rollback 이 적용되지 않는다.` <br>
<br>

그럼 어떻게 해결해야 할까? 

AUTO INCREMENT 값을 신뢰해서는 안된다. 만약 AUTO INCREMENT 값을 중요하게 사용한다면, innodb_autoinc_lock_mode을 잘 알아야한다. <br>

- innodb_autoinc_lock_mode
1. 여러 트랜잭션이 insert 시도 - 먼저 들어온 트랜잭션A, 나중에 들어온 트랜잭션B
2. 트랜잭션A는 먼저 들어와 insert 진행
3. 트랜잭션B는 auto_increment를 잠궈 대기
4. 트랜잭션A의 연속적인 pk 값을 받아온 뒤, 트랜잭션B 진행

여기에도 또 단계가 있다. 락이 언제까지 유지되는지.

traditional mode - 0
- auto_increment 컬럼이 있는 테이블에 insert하는 모든 문장에 테이블 수준의 auto inc 잠금 수해
- insert 구문 수행까지 락 유지

<br>

consecutive mode - 1
- insert되는 레코드 건수 예측 가능하면 lock을 걸지 않고 뮤텍스로 처리(아주 짧은 시간 잠금, auto_inc 가져오는 순간 해제)
- 예측이 불가능할 경우 auto lock을 건다

<br>

interleaved mode - 2
- auto increment lock을 걸지 않고, 항상 뮤텍스를 이용한다.

여기서 잠깐! 

뮤텍스 : 한 개 리소스에 대하여 Lock/Unlock 매커니즘을 사용하는 기술 (ex. 한칸짜리 전용 열쇠 화장실)
세마포어 : 프로세스들이 여러개의 리소스에 접근할 수 있는 카운터 수를 표현하는 기술 (ex. 여러 칸 있는 화장실)


#### SEQUENCE

`데이터베이스의 시퀀스 오브젝트를 이용`하여 유일한 값을 순서대로 생성한다. <br>
이 전략은 주로 Oracle,PostgreSQL,DB2,H2 등의 데이터베이스에서 사용할 수 있다. <br>

```java
/*
 * 유니크키 설정 및 nullable,length 등의 속성은 모두 auto DDL을 사용했을 때만 유효한 설정이다.
 * 즉, 테이블을 직접 생성한다면 적용되지 않는다. 하지만 테이블과 객체간의 관계표현에 있어 해당 설정들을 해놓으면
 * 엔티티 클래스만 봐도 테이블의 구조가 파악되기에 가독성을 위해서라도 설정을 해놓는 것이 좋다.
 */
/*
 * sequence table
 * CREATE TABLE MY_SEQUENCE(
 *    sequence_name varchar2(255) PRIMARY KEY,
 *    next_val number(22,0)
 * )
 */
@Entity
@SequenceGenerator(
        name="BOARD_SEQ_GENERATOR",
        sequenceName="BOARD_SEQ",
        initialValue=1,allocationsSize=1
)
@Table(name = "MEMBER"
        ,uniqueConstraints = {
        @UniqueConstraint(
                name = "NAME_AGE_UNIQUE",
                columnNames = {"NAME","AGE"} //uniqueConstraints는 auto DDL 속성을 사용할때만 유효한 설정이다.
        )
})
@Getter
@Setter
@ToString
public class Member {
    @Id
    @Column(name = "ID")
    @GeneratedValue(strategy=GenerationType.SEQUENCE
            ,generator="BOARD_SEQ_GENERATOR"
    )
    private Long id;
 
    /*
     * not null
     * varchar2(10) -> 기본값 255;
     */
    @Column(name = "NAME",nullable=false,length=10)
    private String username;
 
    private Integer age;
 
    /*
     * EnumType의 기본값 설정은 정수이다.
     */
    @Enumerated(EnumType.STRING)
    @Column(name="ROLE_TYPE",nullable=false,length=20)
    private RoleType roleType;
 
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;
 
    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;
 
    @Lob
    private String description;
}
```

IDENTITY와는 다른 설정이 있다면 시퀀스생성기 어노테이션이다.  <br>
여기서 name속성은 실제 @Id필드에서 참조할 이름이라고 생각하면 되고, sequenceName은 실제 데이터베이스에 생성되는 시퀀스 오브젝트 이름이다.  <br>
그리고 시퀀스 초기값과 allocationSize라는 속성이 있다. <br>
여기서 allocationSize란 실제 데이터베이스에서 가져오는 시퀀스의 한번 호출에 증가하는 값의 크기이다. 
이것은 성능 최적화와 관련된 속성이므로 마지막에 따로 설명한다. <br>

SEQUENCE 전략은 em.persist()를 호출할 때 먼저 데이터베이스 시퀀스를 사용해서 식별자를 조회한다.(실제 엔티티에 할당할 primary key값) <br>
그리고 조회한 식별자를 엔티티에 할당한 후에 엔티티를 영속성 컨텍스트에 저장한다. 이후 커밋이 일어나게 되면 실제 데이터베이스에 INSERT되게 된다.<br>


#### TABLE

TABLE 전략은 `키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 사용할 컬럼을 만들어 데이터베이스 시퀀스를 흉내내는 전략`이다. <br>
이것은 벤더에 의존적이지 않은 전략이다. <br>
이 전략을 사용할 때 auto DDL설정을 했다면 상관없지만 나중에 프로덕환경에서의 데이터베이스 설계에서 꼭 시퀀스 테이블에 생성이 선행되어야한다. <br>


```console
CREATE TABLE APP_SEQUENCE(
  sequence_name varchar2(255) primary key,
  next_val number(22,0)
)
```


```java
/*
 * 유니크키 설정 및 nullable,length 등의 속성은 모두 auto DDL을 사용했을 때만 유효한 설정이다.
 * 즉, 테이블을 직접 생성한다면 적용되지 않는다. 하지만 테이블과 객체간의 관계표현에 있어 해당 설정들을 해놓으면
 * 엔티티 클래스만 봐도 테이블의 구조가 파악되기에 가독성을 위해서라도 설정을 해놓는 것이 좋다.
 */
/*
 * sequence table
 * CREATE TABLE MY_SEQUENCE(
 *    sequence_name varchar2(255) PRIMARY KEY,
 *    next_val number(22,0)
 * )
 */
@Entity
@Table(name = "MEMBER"
        ,uniqueConstraints = {
        @UniqueConstraint(
                name = "NAME_AGE_UNIQUE",
                columnNames = {"NAME","AGE"} //uniqueConstraints는 auto DDL 속성을 사용할때만 유효한 설정이다.
        )
})
public class Member {
    @Id
    @Column(name = "ID")
    @GeneratedValue(strategy=GenerationType.TABLE, generator = "MEMBER_SEQ_GENERATOR")
    @TableGenerator(
            name="MEMBER_SEQ_GENERATOR",
            table="MY_SEQUENCE", //시퀀스 생성용 테이블 이름
            pkColumnName="sequence_name", //MY_SEQUENCE 테이블에 생성할 필드이름(시퀀스네임)
            pkColumnValue="MEMBER_SEQ", //SEQ_NAME이라고 지은 칼럼명에 들어가는 값.(키로 사용할 값)
            allocationSize=50
    )
    private Long id;
 
    /*
     * varchar2(10) -> 기본값 255;
     */
    @Column(name = "NAME",nullable=false,length=10)
    private String username;
 
    private Integer age;
 
    /*
     * EnumType의 기본값 설정은 정수이다.
     */
    @Enumerated(EnumType.STRING)
    @Column(name="ROLE_TYPE",nullable=false,length=20)
    private RoleType roleType;
 
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;
 
    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;
 
    @Lob
    private String description;
}
 
```


<br>

### ``allocationSize를 이용한 성능 최적화?``

영속성 컨텍스트에 엔티티를 저장하기 위해 식별자를 구하는 과정을 설명하면, <br>
<br>
1)식별자를 구하려고 데이터베이스 시퀀스를 조회한다.<br>
->select board_seq.nextval from dual<br>
<br>

그렇다면 만약 increment by 1이라는 설정이라면? 엔티티를 하나하나 저장할 때마다 데이터베이스에 엑세스하여 시퀀스 값을 가져와야한다.(allocationSize=1일때)<br>
<br>
여기에서 allocationSize를 이용하여 성능 최적화를 할 수 있다. <br>
allocationSize값을 적절히 크기를 키워 설정한 값만큼 한 번에 시퀀스 값을 증가시키고 나서 <br>
그만큼 메모리에서 기억해 시퀀스 값 자체를 메모리에서 할당하는 것이다. <br>
예를 들어 allocationSize 값이 50이면 시퀀스를 한번 가져올때 마다 한번에 50 씩 증가된 값을 받아온다. <br>
그말은 처음 시퀀스 값(50)을 받아오면 1~50까지는 메모리에서 엔티티에 식별자를 할당한다. 그리고 51이 되면 시퀀스 값(100)을 한번더 가져와 
51~100까지를 메모리에서 다시 할당해준다. 이말은 쉽게 말하자면 allocationSize=1일 때보다 시퀀스 값을 가져오기 위해 데이터베이스에 엑세스하는 횟수를 49번을
줄인 것이다. <br>
insert 성능이 크게 중요하지 않은 애플리케이션은 상관없지만 그 반대는 이 성능최적화 전략을 사용해보는 것도 좋은 방안일 듯하다. 하지만 제대로 사용하지 못하면 아무리 좋은 것도 독이 되니 잘 설계해서 쓰는 것이 좋을 것 같다.


