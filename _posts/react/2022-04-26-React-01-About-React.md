---
title:  "[React] 리액트란"
excerpt: "React 에 대해 알아보자"

tags:
  - [React]

toc: true
toc_sticky: true
toc_label: "[React 란]"
 
date: 2022-04-26
last_modified_at: 2022-04-26
---

## ``React 란``
  > 리액트는 자바스크립트 라이브러리의 하나로서 사용자 인터페이스를 만들기 위해 사용된다. 

#### 주요 특징

- Data Flow (단방향 데이터 흐름)
- Component 기반 구조
- Virtual Dom
- Props and State


### ``Data Flow``
React는 데이터의 흐름이 한 방향으로만 흐르는 단방향 데이터 흐름을 가진다.

양방향 데이터 바인딩은 규모가 커질수록 데이터의 흐름을 추적하기가 힘들고 복잡해지는 경향이 있어, 데이터 흐름에서 일어나는 변화를 보다 예측 가능할 수 있도록 단방햘 흐름을 채택.
  
<hr/>

### ``Component 기반 구조``

> Component 는 독립적인 단위의 소프트웨어 모듈을 의미 <br>

html 에서 사용 했던 \<button> 태그, \<div> 태그 등을 독립적인 모듈로 생성해서 사용하고, 재사용성이 높아 코드가 짧아지게 된다. <br>
(Java 에서의 객체지향을 생각하면 될 듯 하다.)

<hr/>

### ``Virtual Dom``

#### Dom 이란
브라우저는 화면을 그리기 위해서 `DOM(Document Object Model)`이라는 개념을 사용한다. DOM은 HTML 파일 내용을 토대로 만들어지는데, JavaScript와 같은 스크립팅 언어로 수정할 수 있도록 만들어진, 웹 페이지의 객체 지향 표현.

> 화면을 다시 그릴때마다 jQuery나 document.getElementById, document.querySelector 등을 사용되는 것이 비산 작업이기 때문에 `Virtual Dom` 이 등장

<hr/>

### ``Props and State``

#### Props 란

> 부모 컴포넌트에서 자식 컴포넌트로 전달해 주는 데이터를 의미. <br>
> 자식 컴포넌트에서 전달받은 props는 변경이 불가능하고 props를 전달해준 최상위 부모 컴포넌트만 props를 변경할 수 있다.

#### State 란

> State는 컴포넌트 내부에서 선언하며 내부에서 값을 변경할 수 있다. <br>
> 동적인 데이터를 다룰 때 사용하며, 클래스형 컴포넌트 에만 사용 할 수 있고, 각각의 state는 독립적이다.




