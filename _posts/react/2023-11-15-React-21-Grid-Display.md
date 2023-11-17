---
title:  "[React] Grid Layout 사용하기 (Typescript) #2"
excerpt: "Grid Layout 으로 원하는 아이템 숨기기 "

tags:
  - [React]

toc: true
toc_sticky: true
toc_label: "[React Grid Layout]"
 
date: 2023-11-15
last_modified_at: 2023-11-15
---

### ``React Grid Layout``

- 참고사이트
    - [Grid Layout Example](https://react-grid-layout.github.io/react-grid-layout/examples/0-showcase.html){:target="_blank"}
    - [Grid Layout Git](https://github.com/react-grid-layout/react-grid-layout){:target="_blank"}
    - [블로그](https://velog.io/@deli-ght/loglog-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8-1.-react-grid-layout){:target="_blank"}
    - [블로그](https://velog.io/@dlruddms5619/React-Grid-Layout-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0-Grid-draggable-resizable){:target="_blank"}

<br>

### `시작하기 전에`

프로젝트를 진행하며, 대시보드를 위젯 형태로 꾸며야 하는 상황이 발생했다. <br/>
<br/>

드래그와 드랍을 이용해 위치를 조절하고, 추가적으로 위젯의 크기를 조절 해야되기 때문에, 검색도 해보고 챗GPT 한테도 물어보고 얻은 결과는 DND 와 Resizing 에 대한 기능뿐 이었다. <br/>
<br/>

DND 와 Resizing 을 직접 만져보고 더 깊숙하게 들어가던 중 `react-grid-layout` 모듈을 발견했다. <br/>
<br/>
일주일만 일찍 발견했으면.... 좋았겠지만 이번 포스트는 해당 모듈을 이용하는 방법을 알아보려고 한다.

<br/>


### Grid Layout 이란?

`react-grid-layout`은 리액트를 위한 그리드 레이아웃 시스템이다. 반응성이 뛰어나고 breakpoint를 지원하며 breakpoint layout은 사용자가 제공하거나 자동으로 생성될 수 있다. 드래그와 크기 조정이 가능하며 static 위젯을 설정해서 특정 위젯의 위치를 고정할 수도 있다.

<br/>


#### layout

GridLayout 컴포넌트에는 layout이라는 props가 있다. 해당 데이터는 GridLayout에 들어갈 개별 요소들의 성질을 각각 객체에 담아 배열 형식으로 만든 것이다. <br/>

```ts
const layout = [
  { i: "a", x: 0, y: 0, w: 1, h: 2, static: true },
  { i: "b", x: 1, y: 0, w: 3, h: 2, minW: 2, maxW: 4 },
  { i: "c", x: 4, y: 0, w: 1, h: 2 },
];
```


| key  | value type | Description | require |
|----------|---------|-----------|-----------|
|i|string|각 요소의 id값과 일치하는 식별자값|✅|
|x|number|요소가 어느 x 지점에서 시작할 것인지|✅|
|y|number|요소가 어느 y 지점에서 시작할 것인지|✅|
|w|number|요소가 몇칸의 넓이를 차지할 것인지|✅|
|h|number|요소가 몇칸의 높이를 차지할 것인지|✅|
|minW|number|요소가 최소 넓이 몇칸을 유지할 수 있는지||
|maxW|number|요소가 최대 넓이 몇칸을 유지할 수 있는지||
|minH|number|요소가 최소 높이 몇칸을 유지할 수 있는지||
|maxH|number|요소가 최대 높이 몇칸을 유지할 수 있는지||
|static|boolean|크기와 위치가 고정됨||
|isDraggable|boolean|드래그 가능 여부||
|isResizable|boolean|크기 조절 가능 여부||
|resizeHandles|array <'s', 'w' , 'e' , 'n' , 'sw' , 'nw' , 'se' , 'ne'>	|사이즈 조절 핸들 위치 배열||



> static이 false인 경우 isDraggable : false , isResizable : false 로 설정값과 상관없이 덮어쓰기 된다.

<br/>




### 모듈 설치

```
npm install react-grid-layout 
npm install @types/react-grid-layout
npm install react-resizable
```

<br/>


### css 

최상이 파일에서 css 파일을 import 한다.

> App.tsx

```ts
import "/node_modules/react-grid-layout/css/styles.css";
import "/node_modules/react-resizable/css/styles.css";
```

<br/>


### Dashboard 생성

> Dashboard.tsx

```ts
import { FC } from "react";
import { Responsive, WidthProvider } from "react-grid-layout";

const ResponsiveGridLayout = WidthProvider(Responsive);

const Dashboard: FC = () => {

    const LAYOUTS = {
        items: [
            { i: "a", x: 0, y: 0, w: 1, h: 1, minW: 0, minH: 0 },
            { i: "b", x: 0, y: 1, w: 1, h: 1, minW: 0, minH: 0 },
            { i: "c", x: 0, y: 2, w: 1, h: 1, minW: 1, minH: 1 },
            { i: "d", x: 0, y: 3, w: 1, h: 1, minW: 1, minH: 1 },
            { i: "e", x: 0, y: 4, w: 1, h: 1, minW: 1, minH: 1 },
        ]
    };

    return (
        <ResponsiveGridLayout
            className="layout"
            layouts={LAYOUTS}
            breakpoints={{ items: 1000 }}
            cols={{ items: 4 }}
        >
            {LAYOUTS.items.map((item) => (
                <div key={item.i} className="dnd-movable-item" {...item}>
                    <h1>{item.i}</h1>
                </div>
            ))}
        </ResponsiveGridLayout>
    );
};

export default Dashboard;
```

<br/>


### 결과화면

<img src="https://ymkmoon.github.io/assets/image/react/React_Grid_Layout_01.gif" width="100%" height="100%"/>