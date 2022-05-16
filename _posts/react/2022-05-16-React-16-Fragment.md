---
title:  "[React] Fragment"
excerpt: "Fragment 의 사용이유와 사용법"

tags:
  - [React]

toc: true
toc_sticky: true
toc_label: "[React Fragment]"
 
date: 2022-05-16
last_modified_at: 2022-05-16
---

### ``React Fragment``

- 참고사이트
  - <https://ko.reactjs.org/docs/fragments.html>{:target="_blank"}
  - <https://velog.io/@dolarge/React-Fragment%EB%9E%80>{:target="_blank"}
  - <https://velog.io/@lilyoh/React-Fragments-%EC%82%AC%EC%9A%A9%EC%9D%B4%EC%9C%A0-%EB%B0%8F-%EC%82%AC%EC%9A%A9%EB%B2%95>{:target="_blank"}

<br>

#### `1. Fragment 란?`

블로그를 찾아보니 모두 아래와 같이 설명하고 있었다.

- React v16 에 추가 됐고
- 컴포넌트가 여러 엘리먼트를 return 할 떄, jsx 규칙상 하나의 태그로 묶어서 return 해줘야 한다. 이때 `Fragment` 를 사용하면 dom에 별도의 노드를 추가하지 않고 여러 자식을 그룹화 할 수 있다.

> 그렇다. 보통 의미 없이 단지 묶어서 return 해주기 위해 \<div> 태그 안에 넣어 return 해주던 걸 Fragment 를 사용하면 \<div> 태그를 사용하지 않아도 해결 할 수 있다는 내용이다.

<br>

##### `div 태그를 사용 할 때의 문제점`
1) DOM 크기가 증가해서 로딩 시간이 느려지고, 결국 페이지 속도가 늦어진다.
2) 추가 노드가 생성되어 메모리 사용량이 높아진다.
3) 비용이 늘어난다.
4) 구형 장치에서는 html이 복잡해져 성능 문제가 발생 할 수 있다.
5) 추가 노드의 출처를 디버깅하고 추적하는 것이 어려워 진다.
6) 원하는 레이아웃을 유지하기 어렵다.

<br>

#### `2. Fragment 사용예시`

컴포넌트가 여러 엘리먼트를 반환하는 것은 흔한 패턴이다.

```js
class Table extends React.Component {
  render() {
    return (
      <table>
        <tr>
          <Columns />
        </tr>
      </table>
    );
  }
}
```

렌더링 된 Html이 유효하려면 \<Columms /> 가 여러 <td> 엘리먼트만 반환해야 한다. <br>
\<Columns /> 의 render() 안에 부모 div 자식들을 감싼다면 렌더링 된 HTML은 유효하지 않다.

```js
class Columns extends React.Component {
  render() {
    return (
      <div>
        <td>td 1</td>
        <td>td 2</td>
      </div>
    );
  }
}
```


\<Table />의 출력 결과는 다음과 같다.

```html
<table>
  <tr>
    <div>
      <td>td 1</td>
      <td>td 2</td>
    </div>
  </tr>
</table>
```



Fragments 는 이 문제를 해결해준다.


```js
class Columns extends React.Component {
  render() {
    return (
      <React.Fragment>
        <td>Fragment 1</td>
        <td>Fragment 2</td>
      </React.Fragment>
    );
  }
}
```

올바른 \<Table />의 출력 결과는 아래와 같다.

```html
<table>
  <tr>
    <td>Hello</td>
    <td>World</td>
  </tr>
</table>
```


#### `3. 단축 문법`

\<React.Fragment> \</React.Fragment> 대신 빈태그 <> </> 를 이용한다.

```js
class Columns extends React.Component {
  render() {
    return (
      <>
        <td>Fragment 빈태그 1</td>
        <td>Fragment 빈태그 2</td>
      </>
    );
  }
}
```

#### `4. Fragment key 값 주기`

key 값을 주기 위해서는 빈태그를 사용하면 안되고, Fragment 태그를 직접 명시해줘야 한다.

```js
function Voc(props) {
  return (
    <dl>
      {props.items.map(item => (
        // React는 `key`가 없으면 key warning을 발생한다.
        <React.Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </React.Fragment>
      ))}
    </dl>
  );
}
```