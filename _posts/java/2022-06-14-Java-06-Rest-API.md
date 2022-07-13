---
title:  "[Java] Restful API 모범설계"
excerpt: "Restful API 모범설계"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[Restful API]"
 
date: 2022-06-14
last_modified_at: 2022-06-14
---


- 참고사이트
  - [Medium.com](https://medium.com/@mwaysolutions/10-best-practices-for-better-restful-api-33a25a4e92c1){:target="_blank"}

### ``1. 핵심내용 ``

- Rest API 에서 리소스에 대한 작업은 HTTP 메소드를 사용해 정의된다.
- URI 에는 일관된 명명 규칙을 부여하는 것이 좋다.
- 컬렉션은 복수 명사를 사용하고, 단일 항목은 단수 명사를 사용하는 것이 권장된다. 
- URI 에는 언더바(_) 대신 하이푼(-) 을 사용하자.
- URI 에는 동사를 넣지 말자.
- Parameter 는 정렬이나 검색이 필요할 때 사용하자.
- Header 는 인증이나 인증 후 권한 확인등에 사용하자.

<br>

### ``2. 메소드 ``

목적에 맞는 메소드를 사용하자.

- GET : 데이터를 조회 할 때 많이 사용되는 메소드
- POST : 새로운 데이터를 쓸 때(Write) 사용되는 메소드
- PUT : 데이터를 수정 할 때 사용 (수정할 때 전체 Data Column을 전송해야 한다.)
- PATCH : 데이터를 수정 할 때 사용 (수정할 때 일부 Data Column만 전송해도 된다.)
- DELETE : 데이터를 삭제 할 떄 사용되는 메소드

<br>

### ``3. URI 사용에 대한 예시``

#### 언더바 대신 하이푼

- 나쁜예시
  - [GET] /get_bookmark_list 
  - [POST] /create_bookmark
  - [DELETE] /delete_bookmark

- 좋은예시
  - [GET] /bookmark-list
  - [POST] /create-bookmark
  - [DELETE] /delete-bookmark


#### 동사를 넣지 말자

- 나쁜예시
  - [GET] /get-bookmark-list
  - [POST] /create-bookmark
  - [DELETE] /delete-bookmark

- 좋은예시
  - [GET] /bookmarks
  - [POST] /bookmarks
  - [DELETE] /bookmarks

#### 복수 명사 사용

- /car 대신 /cars
- /user 대신 /users
- /product 대신 /products
- /setting 대신 /settings

#### 관계에 대한 하위 리소스 사용

- [GET] /cars/711/drivers/  -> 자동차 711의 드라이버 목록을 반환한다. 
- [GET] /cars/711/drivers/4 -> 자동차 711의 드라이버 #4를 반환한다.

<br>

### ``4. 컬랙션에 대한 필터링, 정렬, 필드 선택 및 페이지 제공``

#### 필터링
모든 필드에 고유한 쿼리 매개변수를 사용하거나 필터링을 위한 쿼리 언어를 사용하자

- [GET] /cars?color=red   -> 빨간색 자동차 목록을 반환
- [GET] /cars?seats<=2    -> 최대 2개의 좌석이 있는 자동차 목록을 반환


#### 정렬
- [GET] /cars?sort=-manufactorer,+model -> 각 필드에 대한 오름차순, 내림차순을 적용하여 반환

#### 필드선택

- [GET] /cars?fields=manufacturer,model,id,color  -> 선택한 필드에 대해서만 반환

#### 페이징

- [GET] /cars?offset=10&limit=5   -> 기본값은 limit=20 및 offset=0 사용