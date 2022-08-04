---
title:  "[React] StrictMode : useEffect 한번만 호출하기"
excerpt: "React useEffect 한번만 호출하기"

tags:
  - [React]

toc: true
toc_sticky: true
toc_label: "[React StrictMode]"
 
date: 2022-05-12
last_modified_at: 2022-05-12
---

### ``React StrictMode``

- 참고사이트
  - [React DOCS](https://ko.reactjs.org/docs/strict-mode.html){:target="_blank"}

<br>

#### `⚡️왜 두번 호출될까?`

바로 `StrictMode` 때문이다. 개발 환경에서 자동으로 활성화 되어 있기 때문에 useEffect 가 두 번 호출되고,
Strict 모드가 뭔지, 어떻게 꺼야하는지 알아보자.

<hr/>

#### `⚡️StrictMode 란?`

`StrictMode` 는 애플리케이션 내의 잠재적인 문제를 알아내기 위한 도구이다.<br>
`Fragment`와 같이 UI 를 렌더링 하지 않으며, 자손들에 대한 부가적인 검사와 경고를 활성화 한다.

> Strict 모드는 개발 모드에서만 활성화 되기 때문에, 프로덕션 빌드에는 영향을 끼치지 않는다.

<br>

StrictMode 의 장점

- 안전하지 않는 생명주기를 사용하는 컴포넌트 발견
- 레거시 문자열 ref 사용에 대한 경고
- 권장되지 않는 findDOMNode 사용에 대한 경고
- 예상치 못한 부작용 검사
- 레거시 context API 검사

<hr/>

#### `StrictMode 끄는 방법`

간단하다. index.js 에서 StrictMode 태그를 주석처리 하면 된다.

🛠 index.js

```js
import React from 'react';
import ReactDOM from 'react-dom/client';
import './index.css';
import App from './App';
import reportWebVitals from './reportWebVitals';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  // <React.StrictMode>
    <App />
  // </React.StrictMode>
);

// If you want to start measuring performance in your app, pass a function
// to log results (for example: reportWebVitals(console.log))
// or send to an analytics endpoint. Learn more: https://bit.ly/CRA-vitals
reportWebVitals();

```

