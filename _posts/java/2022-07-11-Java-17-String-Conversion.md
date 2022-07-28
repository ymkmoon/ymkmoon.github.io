---
title:  "[Java] String 형변환"
excerpt: "toString() vs String.valueOf()"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[toString() vs String.valueOf()]"
 
date: 2022-07-11
last_modified_at: 2022-07-11
---

- 참고사이트
  - [javadoc](https://docs.oracle.com/javase/8/docs/api/java/lang/String.html#valueOf-java.lang.Object-){:target="_blank"}
  - [stack overflow](https://stackoverflow.com/questions/27465731/string-valueof-vs-object-tostring){:target="_blank"}
  - [Okky](https://okky.kr/article/303538){:target="_blank"}

### ``Object.toString() 과 String.valueOf(Object) 의 차이``

둘 의 차이점은 Object 가 null 일 경우 NPE 가 발생하는지, 아니면 null 을 리턴하는지 이다.

캐스팅을 사용 할 경우에는 null 을 반환 하지만, 데이터가 문자열이 아닐 경우에는 ClassCastException 이 발생한다.

```java
public static void main(String args[]) {  
    String str = null;
    System.out.println((String)str); // null 리턴 
    System.out.println(String.valueOf(str)); // null 리턴
    System.out.println(str.toString()); // NullPointerException
} 
```

<hr/>

### ``뭘 써야 할까?``

블로그를 검색해보니 NPE 를 방지하는 String.valueOf(Object) 사용을 권한다고 한다. <br>


하지만 아래와 같은 의견을 가진 개발자도 존재한다.

요악하자면 구조적으로 null 이라는 데이터가 생기지 않도록 해야 하고, NPE 가 발생 할 경우 추적하여 해결해라. 하지만 여의치 않다면 `String.valueOf(Object)를 사용`하고, `만약 Object 가 무효화(null) 로 되면 안되는 경우에는 Object.toString() 을 사용`하라는 내용이다.


Null 에 대한 내용은 아래 링크를 꼭 읽어보길 바란다. <br>
[Okky](https://okky.kr/article/303538){:target="_blank"}

<hr/>


> Personal opinion
Some developers acquire the (IMO) bad habit of "defending" against nulls. So you see lots of tests for nulls, and treating nulls as special cases. The idea seems to be prevent NPE from happening.

개인적인 의견
일부 개발자들은 널에 대한 "방어"의 나쁜 습관을 얻습니다. 따라서 널에 대한 많은 테스트와 널을 특별한 경우로 취급합니다. 이 아이디어는 NPE가 발생하는 것을 막는 것 같습니다.


> I think this is a bad idea. In particular, I think it is a bad idea if what you do when you find a null is to try to "make good" ... without consideration of why there was a null there.

나는 이것이 나쁜 생각이라고 생각합니다. 특히, 당신이 널을 찾을 때 당신이하는 일이 왜 널이 있었는지 고려하지 않고 "좋은"것을 시도하는 것이 "좋은 생각이라고 생각합니다.

> In general, it is better to avoid the null being there in the first place ... unless the null has a very specific meaning in your application or API design. So, rather than avoiding the NPE with lots of defensive coding, it is better to let the NPE happen, and then track down and fix the source of the unexpected null that triggered the NPE.

일반적으로, 널이 ​​애플리케이션이나 API 설계에서 매우 구체적인 의미를 갖지 않는 한, 널이 처음에 존재하는 것을 피하는 것이 좋습니다. 따라서 방어 코딩이 많은 NPE를 피하는 대신 NPE가 발생하게 한 다음 NPE를 트리거 한 예기치 않은 널의 소스를 추적하고 수정하는 것이 좋습니다.

> So how does this apply here?

> Well, if you think about it, using String.valueOf(obj) could be a way of "making good". That is to be avoided. If it is unexpected for obj to be null in the context, it is better to use obj.toString().

그렇다면 이것이 어떻게 여기에 적용됩니까?

글쎄, 당신이 그것에 대해 생각한다면, string.valueof (obj)를 사용하는 것은 "좋은 일을하는"방법 일 수 있습니다. 그것은 피해야합니다. 컨텍스트에서 OBJ가 무효가되는 것이 예상치 못한 경우 obj.tostring ()을 사용하는 것이 좋습니다.