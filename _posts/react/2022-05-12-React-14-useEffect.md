---
title:  "[React] Hooks : useEffect()"
excerpt: "React 기술용어"

tags:
  - [React]

toc: true
toc_sticky: true
toc_label: "[React Hooks : useEffect()]"
 
date: 2022-05-12
last_modified_at: 2022-05-12
---

### ``React useEffect``

- 참고사이트
  - <https://ko.reactjs.org/docs/hooks-overview.html>
  - <https://cocoon1787.tistory.com/796>
  - <https://ingg.dev/hook-work/>
  - <https://devbirdfeet.tistory.com/52>

<br>

#### `⚡️side effects 란?`

React 컴포넌트가 화면에 1차로 렌더링된 이후에 비동기로 처리되어야 하는 부수적인 효과들을 흔히 Side Effect라고 한다.
예를들어 데이터를 가져오려고 외부 API를 호출할 때, 일단 화면에 렌더링할 수 있는 것은 1차로 먼저 렌더링하고 실제 데이터는 비동기로 가져오는 것이 권장된다. 왜 먼저 렌더링하냐면 연동된 API가 응답이 늦거나 없을 때 데미지(답답함)을 최소화 시켜 사용자 경험 측면에서 유리하기 때문이다.

`→ 한마디로 요구되어지는 이펙트 이외에 다른 이펙트가 발생하는 현상이라고 생각하면 된다.`

`→ Hook은 이 side effect 를 수행하는역할을 한다. side effect 를 줄여 그냥 effect 라고 한다. 그래서 훅의 이름은 useEffect 가 된다.`


<br>

#### `⚡️React Hooks 란?`

Hooks 는 리액트 v16.8 에 새로 도입된 기능이다. 함수형태의 컴포넌트에서 사용되는 몇가지 기술을 Hook이라고 부른다.

함수형 컴포넌트에서도 상태 관리를 할 수 있는 useState, 그리고 렌더링 직후 작업을 설정하는 useEffect 등의 기능 등을 제공한다.
 
`그렇다면 왜 굳이 잘 쓰고 있던 클래스형 컴포넌트를 함수형 컴포넌트로 바꿔야할까?`

react를 배우는 데에 있어서 클래스는 큰 진입장벽이었다. 코드의 재사용성과 코드 구성을 어렵게 만들고, this의 사용이나 이벤트 핸들러의 등록 등 기본적인 JS 문법 사항을 알아야 다룰 수 있기 때문이다. 또한 클래스는 잘 축소되지 않고, reloading을 깨지기 쉽고 신뢰하기 어렵게 만든다. 따라서 react의 최신 기술들이 클래스형 컴포넌트에 효과적으로 적용되지 않았다.

- 클래스의 문법이 어렵다.

- 축소가 어렵다.

- reloading의 신뢰성이 떨어진다.

- 최신 기술의 적용이 효과적이지 않다.

`→ 이러한 클래스의 단점들을 함수형 컴포넌트로 커버할 수 있다. 하지만 클래스 컴포넌트의 장점인 state 사용이나 life cycle을 직접 다루는 등의 기능을 사용하지 못한다. 이를 해결하기 위해 Hook이 등장했다.`

<br>

#### `⚡️useEffect() 란?`

useEffect() 함수는 React component가 렌더링 될 때마다 특정 작업(Sied effect)을 실행할 수 있도록 하는 리액트 Hook 이다. 

여기서 Side effect는 component가 렌더링 된 이후에 비동기로 처리되어야 하는 부수적인 효과들을 뜻한다. 이러한 기능으로 인해 함수형 컴포넌트에서도 클래스형 컴포넌트에서 사용했던 생명주기 메서드를 사용할 수 있게 되었다.

<br>

#### `⚡️useEffect() 사용하기`

기본 형태

```js
useEffect(function, deps)
```

- `function : 실행하고자 하는 함수`
- `deps :  배열 형태. function을 실행시킬 조건.
deps에 특정값을 넣게 되면 컴포넌트가 mount 될 때, 지정한 값이 업데이트될 때 useEffect를 실행한다.`

<br>

1) Component가 Mount 되었을 때(나타날 때)

```js
useEffect(() => {
  console.log("렌더링 될때마다 실행");
});
```

deps 부분을 생략 한다면 해당 컴포넌트가 렌더링 될 때마다 `useEffect()`가 실행 된다. 만약 맨 처음 렌더링 될 때 한번만 실행하고 싶다면 deps 위치에 빈 배열을 넣어준다.

```js
useEffect(() => {
  console.log("맨 처음 렌더링될 때 한 번만 실행");
},[]);
```

<br>

2) Component가 Update 되었을 때(props, state 변경)

```js
useEffect(() => {
  console.log(name);
  console.log("name이라는 값이 업데이트 될 때만 실행");
},[name]);
```

특정 값이 업데이트 될 때만 실행하고 싶을 때는 deps 위치의 배열 안에 실행 조건을 넣어준다. 업데이트 뿐만 아니라 마운트 될 때도 실행되므로 업데이트 될 때만 실행시키길 원한다면 아래와 같이 사용한다.

```js
const mounted = useRef(false);
useEffect(() => {
  if (!mounted.current) {
    mounted.current = true;
  } else {
    console.log(name);
    console.log("업데이트 될 때마다 실행");
  }
}, [name]);
```


<br>

3) Component가 Unmount 되었을때(사라질때) & update 되기 직전에

```js
useEffect(() => {
  console.log("컴포넌트 나타남");
  console.log(name);
  return () => {
    console.log("cleanUp 함수");
  };
});
```

useEffect는 함수를 반환 할 수 있는데 이 함수를 cleanup 이라고 한다.

Unmount 될 때만 cleamup 함수를 실행시키고 싶다면 deps 에 빈 배열을,

특정값이 update 되기 직전에 cleanup 함수를 실행시키고 싶다면 deps 에 해당 값을 넣어주면 된다.
