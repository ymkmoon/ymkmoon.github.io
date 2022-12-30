---
title:  "[Java] JPA 에서 페이징과 정렬처리하기"
excerpt: "JPA Pageable"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[JPA Pageable]"
 
date: 2022-12-23
last_modified_at: 2022-12-23
---

- 참고사이트
  - [spring document](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories){:target="_blank"}
  - [stackoverflow](https://stackoverflow.com/questions/25008472/pagination-in-spring-data-jpa-limit-and-offset){:target="_blank"}


### 시작히기 전

React & GraphQL & MongoDB 을 이용한 프로젝트 개발을 하고 느낀점이 있다.<br>

서비스 후 시간이 지속되면 데이터가 쌓이게 되고, 그렇다면 모든 데이터를 조회 할 경우 시간이 오래걸리게 된다. <br>
이걸 막기 위해선, 조회하는 쿼리에 offset 과 limit 을 이용해 페이징처리를 해서 제한을 걸어두는 것이다. 트래픽 감소와 더불어 FE 쪽에서도 개발하기 쉬워진다. <br>


FE 에서 페이징 처리를 좀 더 쉽게 하기 위해선 총 3가지가 필요하다. <br>

- 전체 레코드 수(조회 한 데이터의 총 개수)
- 현재 페이지
- 페이지당 노출 개수

이번 챕터에서는 JPA 페이징 처리에 대해 기록을 할 것이며 <br>
내가 참여한 프로젝트에서는 모든 데이터를 생성시간(createdAt)을 이용해 정렬했지만, 이왕 리팩토링 하는 김에 정렬에 대한 요청은 Optional 하게 구현해보도록 하자.


<hr/>


### JPA 페이징 구현

JPA 에서는 페이징처리를 편하게 사용 할 수 있도록 `Pagealbe` 이라는 객체를 제공한다. <br>

> GET /stations?offset=1&limit=1000&sort=stationId,DESC


위와 같은 요청을 받아 페이징, 정렬을 처리 할 수 있고, Pageable 구현 방법은 JPA 의 제공 객체를 그대로 사용해도 되고, 직접 커스텀하여 사용 할 수 있다.

#### 의존성 추가

> Gradle

```gradle
// https://mvnrepository.com/artifact/org.apache.commons/commons-lang3
implementation 'org.apache.commons:commons-lang3:3.12.0'
```


> Maven

```xml
<!-- https://mvnrepository.com/artifact/org.apache.commons/commons-lang3 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
```


#### Pageable 커스텀

JPA 의 객체를 그대로 가지고와 사용해도 문제 없지만, 본인은 직접 제어 할 수 있는걸 좋아하는 편이기에, 아래 파일을 추가해줬다.

최대 limit 에 대한 커스텀은 본인의 판단에 맡기도록 하겠다.


```java
import java.io.Serializable;

import org.apache.commons.lang3.builder.EqualsBuilder;
import org.apache.commons.lang3.builder.HashCodeBuilder;
import org.apache.commons.lang3.builder.ToStringBuilder;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;

public class OffsetBasedPageRequest implements Pageable, Serializable {

    private static final long serialVersionUID = -25822477129613575L;

    private final int limit;
    private final long offset;
    private final Sort sort;

    /**
     * Creates a new {@link OffsetBasedPageRequest} with sort parameters applied.
     *
     * @param offset zero-based offset.
     * @param limit  the size of the elements to be returned.
     * @param sort   can be {@literal null}.
     */
    public OffsetBasedPageRequest(long offset, int limit, Sort sort) {
        if (offset < 0) {
            throw new IllegalArgumentException("Offset index must not be less than zero!");
        }

        if (limit < 1) {
            throw new IllegalArgumentException("Limit must not be less than one!");
        }
        this.limit = limit;
        this.offset = offset;
        this.sort = sort;
    }

    /**
     * Creates a new {@link OffsetBasedPageRequest} with sort parameters applied.
     *
     * @param offset     zero-based offset.
     * @param limit      the size of the elements to be returned.
     * @param direction  the direction of the {@link Sort} to be specified, can be {@literal null}.
     * @param properties the properties to sort by, must not be {@literal null} or empty.
     */
    public OffsetBasedPageRequest(long offset, int limit, Sort.Direction direction, String... properties) {
        this(offset, limit, Sort.by(direction, properties));
    }

    /**
     * Creates a new {@link OffsetBasedPageRequest} with sort parameters applied.
     *
     * @param offset zero-based offset.
     * @param limit  the size of the elements to be returned.
     */
    public OffsetBasedPageRequest(long offset, int limit) {
        this(offset, limit, Sort.unsorted());
    }

    @Override
    public int getPageNumber() {
        return Math.toIntExact(offset / limit);
    }

    @Override
    public int getPageSize() {
        return limit;
    }

    @Override
    public long getOffset() {
        return offset;
    }

    @Override
    public Sort getSort() {
        return sort;
    }

    @Override
    public Pageable next() {
        return new OffsetBasedPageRequest(getOffset() + getPageSize(), getPageSize(), getSort());
    }

    public OffsetBasedPageRequest previous() {
        return hasPrevious() ? new OffsetBasedPageRequest(getOffset() - getPageSize(), getPageSize(), getSort()) : this;
    }


    @Override
    public Pageable previousOrFirst() {
        return hasPrevious() ? previous() : first();
    }

    @Override
    public Pageable first() {
        return new OffsetBasedPageRequest(0, getPageSize(), getSort());
    }

    @Override
    public boolean hasPrevious() {
        return offset > limit;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;

        if (!(o instanceof OffsetBasedPageRequest)) return false;

        OffsetBasedPageRequest that = (OffsetBasedPageRequest) o;

        return new EqualsBuilder()
                .append(limit, that.limit)
                .append(offset, that.offset)
                .append(sort, that.sort)
                .isEquals();
    }

    @Override
    public int hashCode() {
        return new HashCodeBuilder(17, 37)
                .append(limit)
                .append(offset)
                .append(sort)
                .toHashCode();
    }

    @Override
    public String toString() {
        return new ToStringBuilder(this)
                .append("limit", limit)
                .append("offset", offset)
                .append("sort", sort)
                .toString();
    }
}
```


#### Request 에 parameter 추가


> 기존 소스

```java
@GetMapping()
public ResponseEntity<List<StationInformationDto.Response>> getStations() {
    List<StationInformationDto.Response> stations = stationService.getAllStations();
    return new ResponseEntity<>(stations, HttpStatus.OK);
}

```


> 페이징 적용

```java
@GetMapping()
public ResponseEntity<List<StationInformationDto.Response>> getStations(
  @RequestParam(name="offset") @NotNull long offset,
  @RequestParam(name="limit") @NotNull int limit,
  Sort sort) {
    Pageable pageable = new OffsetBasedPageRequest(offset, limit, sort);
      List<StationInformationDto.Response> stations = stationService.getAllStations(pageable);
      return new ResponseEntity<>(stations, HttpStatus.OK);
}

```


기존에는 findAll 을 이용해 모든 데이터를 조회하도록 했지만, 페이징 처리에 필요한 offset, limit 인수를 request 에서 받아 처리하게끔 변경했다. <br>

페이징에 필요한 `offset` 과 `limit` 는 필수적으로 있어야 되지만, 정렬에 필요한 `sort` 는 Optaional 하도록 설정했다. <br>

사실 생각보다 간단하게 구현 가능한 내용이라 따로 뭘 이야기 해야 할 지 모르겠다. 구글 검색만으로도 해결 할 수 있기에. <br>

하지만 아직 본인 프로젝트에 `GET` 요청에 대해 페이지 처리가 없이 findAll 이 실행되고 있다면, 모두 페이징처리를 넣어 혹시 모를 불상사를 막기를 바란다.


<hr/>

### 결과화면

> 정렬하지 않은 결과

![JPA](/assets/image/springboot/Springboot_jpa_03.PNG)

> station ID 값을 DESC 정렬 한 결과

![JPA](/assets/image/springboot/Springboot_jpa_01.PNG)

> station ID 값을 ASC 정렬 한 결과

![JPA](/assets/image/springboot/Springboot_jpa_02.PNG)