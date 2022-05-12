---
title:  "[React] 기술용어"
excerpt: "React 기술용어"

tags:
  - [React]

toc: true
toc_sticky: true
toc_label: "[React 기술 용어]"
 
date: 2022-05-09
last_modified_at: 2022-05-09
---

### ``React 기술 용어``

- 참고 사이트
  - https://ko.reactjs.org/docs/glossary.html


#### `Bundler (번들러)`

분리 된 JavaScript와 CSS 묘둘 코드를 브라우저에 최적화된 여러 개의 파일로 결합한다.
React 에서 널리 사용되는 번들러에는 Webpack 와 Browserify가 있다.

<br>

#### `JSX`

JSX는 JavaScript의 확정 문법 이다.
JSX는 템플릿 언어와 비슷해 보이지만 JavaScript의 강력한 기능들을 모두 사용 할 수 있다.

<br>


#### `Element(엘리먼트)`

React 엘리먼트는 React 애플리케이션을 구성하는 블록이다.
엘리먼트는 화면에 보이는 것들을 기술하며, React 엘리먼트는 변경되지 않는다.

```js
const element = <h1>Hello, world</h1>;
```

일반적으로 엘리먼트는 직접 사용되지 않고 컴포넌트로 부터 반환됩니다.

<br>

#### `Component(컴포넌트)`
	
React 컴포넌트는 페이지에 렌더링 할 React 엘리먼트를 반환하는 작고 재사용 가능 한 코드조각이다.
컴포넌트는 기능별로 나눌 수 있으며, 다른 컴포넌트 안에서 사용 할 수 있다.
컴포넌트의 이름은 항상 대문자로 시작해야 한다. 

<span style="background-color: #F0E68C">\<Wrapper/></span> (o) 
<span style="background-color: #F0E68C">\<wrapper/></span> (x)

<br>

#### `props`

props 는 컴포넌트 입력 값 이며, 부모 컴포넌트로 부터 자식 컴포넌트로 전달 된 데이터 이다.
props 는 `읽기 전용`이며, 어떤 방식으로든 수정 해서는 안된다.
사용자의 입력 또는 네트워크 응답에 반응하여 어떤 값을 수정해야 한다면 state를 이용하자.

<br>

#### `props.children`
모든 컴포넌트에서 props.children 을 사용 할 수 있고, props.children 은 컴포넌트의 여는 태그와 닫는 태그 사이의 내용을 포함한다.

```js
<Welcome>Hello world!</Welcome>
```

Hello world! 문자열은 Welcome 컴포넌트의 props.children으로 사용할 수 있다.

```js
function Welcome(props) {
  return <p>{props.children}</p>;
}
```

Class로 정의된 컴포넌트에서는 this.props.children을 사용합니다.

```js
class Welcome extends React.Component {
  render() {
    return <p>{this.props.children}</p>;
  }
}
```

<br>

#### `state`

컴포넌트와 관련 된 일부 데이터가 시간에 따라 변경 될 경우 state 가 필요.
예를 들어, Checkbox 컴포넌트는 isChecked state가 필요할 수 있으며, NewsFeed 컴포넌트는 fetchedPosts를 컴포넌트의 state를 통해 계속 주시하려고 할 수 있다.

`state` 와 `props` 의 가장 중요한 차이점은 `props` 는 부모 컴포넌트로부터 전달받지만, `state` 는 컴포넌트에서 관리 된다. 컴포넌트는 `props`를 변경 할 수 없지만, `state` 는 변경 할 수 있다.

데이터가 변경되는 각 특정한 부분에 대해, 해당 상태(state)를 "소유" 하는 컴포넌트는 하나만 존재해야 한다. 서로 다른 두 컴포넌트의 상태를 동기화 하려고 해선 안되고, 공통 상태를 두 컴포넌트의 공통 조상으로 끌어 올리고 해당 데이터를 두 컴포넌트에 props 로 전달하자.

<br>

#### `key`

"key"는 엘리먼트의 배열을 만들 때 포함해야 하는 특별한 문자열 이다. key는 React가 어떤 항목을 변경, 추가 혹은 삭제 할 지 식벽해야 하는 것을 돕는다. 엘리먼트들을 안정적으로 식별 할 수 있도록 배열 내의 엘리먼트에 key를 제공해야 한다.`(같은 배열에 포함 된 다른 요소 사이에서만 고유값을 가지면 되고, 전체 애플리케이션 또는 단일 컴포넌트 전체에서 고유값을 가질 필요는 없다.)`

Math.random() 같은 값을 key 로 사용하면 안된다. React가 항목 추가, 제거 또는 재정렬 할 시기를 결정 할 수 있도록 key 는 다시 렌더링 하는 과정동안 "안정적으로 식별 가능"해야 하기 때문이다. 

이상적으로, Key는 post.id 와 같이 데이터에서 사용되는 유일하고 안정적인 식별자를 사용 하는 것이 좋다.

