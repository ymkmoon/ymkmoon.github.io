---
title:  "[React] Typescript DragAndDrop(DND) 사용하기"
excerpt: "Fragment 의 사용이유와 사용법"

tags:
  - [React]

toc: true
toc_sticky: true
toc_label: "[React DND]"
 
date: 2023-11-14
last_modified_at: 2023-11-146
---

### ``React DND``

- 참고사이트
    - [Medium.com](https://medium.com/litslink/react-dnd-in-examples-ce509b25839d){:target="_blank"}
    - [블로그](https://dsuumb.tistory.com/12){:target="_blank"}

<br>

### `DND 란?`

간단하다. 드래그와 드랍을 통해 특정 창의 위치를 변경한다는 의미이다. <br>

<br>

React 환경에서 DND 기능을 제공해주는 모듈은 대표적으로 3개가 존재한다.

<br>

- react-draggable
> 드래그로 아이템 간의 순서를 변경하는 기능보다, 아이템의 위치를 옮기능 기능에 강점이 있다.


- react-dnd
> DND 기능을 사용 할 수 있게 해주는 hook 을 제공해주는 모듈 이다. 


- react-beautiful-dnd

> react-dnd 에서 좀 더 확장된 느낌으로 UI/UX 퍼포먼스가 좋은 동작이 미리 정의되어 있다. 이처럼 편리한 기능을 제공하기 때문에 `react-dnd 보다 용량이 약 2배 많다.`

<br>

학습 후 우리가 얻게 될 결과물은 아래와 같다.

![React](/assets/image/react/React_DND_01.gif)

<br/>


### 라이브러리 다운로드

```
npm i react-dnd react-dnd-html5-backend
```

<br/>


### 프로젝트 구조




<br/>


### CSS 추가

일단 각 컴포넌트에서 사용 될 css 를 추가하자.

```css
.dnd-container {
    display: flex;
    flex-direction: row;
    justify-content: space-around;
}

.dnd-column {
    height: 600px;
    width: 200px;
    margin: 20px;
    border-radius: 10px;
    box-shadow: 1px 1px 6px rgba(0, 0, 0, 0.5);
    justify-content: center;
    flex-wrap: wrap;
}

.dnd-first-column {
    background-color: #f5ffea;
}

.dnd-second-column {
    background-color: #fffbf5;
}

.dnd-movable-item {
    border-radius: 10px;
    background-color: #fff3f3;
    height: 100px;
    width: 170px;
    margin: 10px auto;
    display: flex;
    justify-content: center;
    align-items: center;
    box-shadow: 0px 0px 3px rgba(0, 0, 0, 0.5);
}
```


<br/>
