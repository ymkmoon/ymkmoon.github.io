---
title:  "[JavaScript] ECMAScript 란?"
excerpt: "ECMAScript 란 무엇인가"

tags:
  - [Javascript]

toc: true
toc_sticky: true
toc_label: "[ECMAScript]"
 
date: 2022-07-18
last_modified_at: 2022-07-18
---

- 참고사이트
  - [위키](https://ko.wikipedia.org/wiki/Ecma_%EC%9D%B8%ED%84%B0%EB%82%B4%EC%85%94%EB%84%90){:target="_blank"}
  - [블로그](https://sumini.dev/til/006-ecmascript/){:target="_blank"}



### ``1. JavaScript 란 ``

서버에 데이터를 보내기 전, 데이터 유효성 검사 등을 처리하는 스크립트 언어이다. <br>
1995년 넷스케이프(Netscape)의 Brendan Eich에 의해 개발된 언어이고, Mocha -> LiveScript -> JavaScript의 네이밍 변천사를 갖고있다. 그리고 사실 우리가 알던 자바스크립트는 ECMAScript + BOM(Browser Object Model) + DOM(Document Object Model)라는 1개의 코어와 2개의 모델로 이루어져있다.

<br>



### ``2. ECMAScript 란 ``

`Ecma` 라는 기관이 만든 script 언어이며, [ECMA-262표준](https://en.wikipedia.org/wiki/ECMAScript){:target="_blank"} 을 따르고 있다.

`Ecma intenational 은 정보와 통신 시스템을 위한 국제적 표준화 기구`이다. <br>
원래 이름은 European Computer Manufacturers Association 이었으나, 기구의 국제적 확장을 반영하여 현재의 이름으로 바뀌었다

<br>



### ``3. ECMAScript 의 히스토리 ``

> ES3 -> ES5 -> ES6(ES2015) -> ES7(ES2016) -> ES8(ES2017)

#### ES3 (1999)
대중적으로 알고있는 그냥 자바스크립트라고 보면 된다. <br>
함수 단위의 스코프, 호이스팅, 클로저, 프로토타입 등… <br>
우리가 익히 알고있는 자바스크립트의 기본적인 특징들을 갖고있다. <br>
대부분의 브라우저에서 지원하며, 
IE8까지 크로스브라우징을 지원하는 환경이라면 ES3을 쓰고 있다고 보면 된다.
<br>


#### ES5 (2009)
ES4는 너무 시대의 흐름을 앞서갔는지 거절되고, 
그 후에 점진적인 개선을 목표로 ES5가 나왔다고 한다. 
아무리 그래도 10년만에 버전업이라니 너무한것 같지만 많은 편리한 기능이 추가되었다. <br>

배열 배열과 관련하여 편리한 메소드들이 다수 생겼다. <br>
forEach, map, reduce, filter, some, every와 같은 순환 메소드들이 생겼다. 
이 메소드들은 개발 시 불필요한 중복 코드를 줄여주어서 가독성은 높이고 버그율은 낮추는 효과가 있다. <br>
객체 객체는 프로퍼티에 대한 설정을 할 수 있게 되었다. <br>
객체를 생성, 수정, 복사하는 표준 메소드 Object.Create(), Object.defineProperty(), 
Object.freeze(), Object.assign() 등 과 getter, setter 등이 추가되었으며, 
Object.keys() 메소드를 이용하면 for in 메소드도 대체할 수 있게 되었다. <br>
strict 모드 문법을 좀 더 깐깐하게 체크하는 모드이다. <br>
너무 자유분방하였던 기존 ES를 안전하고, 
개발자가 인지할 수 있는 범위 안에서 개발할 수 있도록 사용하기 위해 등장했다. <br>
Strict mode - JavaScript | MDN에서 자세한 특징을 확인 할 수 있다.
bind() 메소드 this를 강제로 바인딩 시켜주는 메소드이다. 
좀 더 명확하게 this 스코프를 지정 할 수 있게 되었다.

<br>


#### ES6 (ES2015)
ES6보다 ES2015라고 많이 불리우며, ES6 Harmony라고도 불리운다고 한다. <br>
ES2015에서 다음과 같은 문제점들이 해결되었다. <br>

호이스팅이 사라진 것 같은 효과 <br>
함수 단위 스코프에서 블록 단위 스코프로 변경 <br>
this를 동적으로 바인딩하지 않는 화살표 함수 <br>
모듈화 지원 <br>
콜백 지옥에서 구원해줄 `Promise` <br>
Default, Rest 파라미터 <br>
해체 할당, Spread 연산자 <br>
템플릿 리터럴 <br>
클래스 이 외에도 추가된점이 너무 많아서 이부분에서 사람들이 진입장벽을 느끼는 것 같다. <br>
브라우저(특히 MS 계열)에서 지원해주지 않는 경우가 많아 <br>
바벨(Babel)이라는 트랜스파일러를 써야하는데 이 바벨은 웹브라우저가 아닌 
Node.js 위에서 돌아가고… Node.js를 설치하려면 NPM을 알아야하고… 
또 모듈화를 사용하려면 웹팩(WebPack)같은 모듈 번들러를 알아야하고…
이런 다양한 장벽 때문에 사람들이 ES2015를 쉽게 접근하지 못하는 경향이 있는 것 같다. 

<br>



#### ES7 (ES2016)
제곱 연산자(**) 등장 <br>
Array.includes 배열에 해당 요소가 존재하는지 확인하는 메소드 등장
<br>



#### ES8 (ES2017)
ES2017에서는 Promise 급의 중대한 변화인 async, await등이 발표되었다. <br>

async, await <br>
객체 객체의 좀더 심화된 메소드가 등장했습니다. <br>
Object.keys()에 대응되는 메소드인 Object.values(), Object.keys()와 
Object.values()를 합쳐 놓은 `Object.entries()`, 
Object.getOwnPropertyDescriptor의 
복수 형태인 `Object.getOwnPropertDescriptors()`로써 상속받지 않은 속성들의 설명만 보여줍니다.  <br>
문자열 단순 편의기능이 추가되었습니다.  <br>
문자열 앞부분에 공백을 넣어 자리수를 맞춰주는 String.padStart(), 
문자열 뒷부분에 공백을 넣어 자리수를 맞춰주는 String.padEnd()  <br>
매개변수 마지막에 콤마를 붙이는걸 허용
<br>


#### ES 2018
체 리터럴, 비동기 반복 Promise.prototype.finally및 RegExp에 대한 추가를 위한 스프레드 연산자 및 나머지 매개변수( )가 포함 <br>
스프레드 연산자를 사용하면 아래와 같이 개체 속성을 쉽게 복사할 수 있다.

```javascript
let object = {a: 1, b: 2}

let objectClone = Object.assign({}, object) // before ES2018
let objectClone = {...object} // ES2018 syntax

let otherObject = {c: 3, ...object}
console.log(otherObject) // -> {c: 3, a: 1, b: 2}
```
<br>



#### ES 2019
Array.prototype.flat에 Array.prototype.flatMap대한 변경 사항 이 포함 <br>
Array.sort는 이제 안정적으로 보장된다. 즉, 정렬 우선 순위가 동일한 요소가 정렬된 배열에서 동일한 순서로 나타남. <br>
다른 주목할만한 변화는 소위 `catch 바인딩 이 선택 사항이 된 것`

<br>



#### ES 2020
BigInt은 임의 크기의 정수에 대한 기본 유형, [nullish coalescing operator](https://en.wikipedia.org/wiki/Null_coalescing_operator){:target="_blank"} 및 [globalThis object](https://en.wikipedia.org/wiki/JavaScript_syntax#Declaration_and_assignment){:target="_blank"}
 를 도입 <br>
BigInt는 BigInt생성자 또는 구문을 사용하여 생성되며 10n, 여기서 "n"은 숫자 리터럴 뒤에 배치된다. <br>
BigInt는 정수 이상의 표현 및 조작을 허용하는 Number.MAX_SAFE_INTEGER반면 숫자는 배정밀도 64비트 IEEE 754 값으로 표현된다. <br>
내장 함수는 MathBigInt와 호환되지 않습니다. 예를 들어 BigInt의 지수화는 **대신 연산자를 사용 하여 수행해야한다. <br>
nullish 병합 연산자 , 왼쪽이 또는 ??인 경우 오른쪽 피연산자를 반환한다. . 이는 아래와 같이 모든 "거짓" 값을 반환 하는 연산자와 대조 됨.

```javascript
undefined ?? "string" // -> "string"
null ?? "string" // -> "string"
false ?? "string" // -> false
NaN ?? "string" // -> NaN
```

`Optional chaining`을 사용하면 각 수준에서 AND 검사 없이 개체의 중첩 속성에 액세스할 수 있다. <br>
예를 들어 const zipcode = person?.address?.zipcode 속성이 없는 경우 zipcode가 정의되지 않는ㄴ다.

<br>