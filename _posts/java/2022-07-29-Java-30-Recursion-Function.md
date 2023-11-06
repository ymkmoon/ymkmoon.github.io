---
title:  "[Java] 재귀함수란? "
excerpt: "재귀함수의 설명과 사용법"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[재귀함수]"
 
date: 2022-07-29
last_modified_at: 2022-07-29
---

- 참고사이트
  - [나무위키](https://namu.wiki/w/%EC%9E%AC%EA%B7%80%ED%95%A8%EC%88%98){:target="_blank"}
  - [블로그](https://crazykim2.tistory.com/591){:target="_blank"}
  - [블로그](http://melonicedlatte.com/2021/05/10/001900.html){:target="_blank"}



- 자료구조와 알고리즘
  - [재귀함수](https://ymkmoon.github.io/Java-30-Recursion-Function/)
  - [선형탐색](https://ymkmoon.github.io/Java-44-Algorithm-Linear-Search/)
  - [이진탐색](https://ymkmoon.github.io/Java-45-Algorithm-Binary-Search/)



### ``재귀함수란? ``

재귀함수(Recursion)는 정의 단계에서 `자신을 재참조하는 함수`를 의미한다. <br>
함수 내에서 자기 자신을 호출하는 프로세스이다.<br>

좀 더 쉽게 이해하기 위해 아래 나무위키의 예시를 보자.

![AWS](/assets/image/java/Java_Recursion_01.PNG)


재귀함수를 사용하여 HelloWorld 를 반복 출력해보자.

```java
public class PlusFunction {
	public static void main(String[] args)  {
		HelloWorld(5); // 메서드 호출
	}
	
	public static void HelloWorld(int n)
	{
		// n이 0인 경우 return
		if(n == 0)
			return;
		
		System.out.println("HelloWorld"); // HelloWorld 출력
		HelloWorld(n-1);
	}
}
```


위 코드에서 HelloWorld 메소드를 보자. <br>

if문을 이용해 매개변수 n 의 값이 0이면 return; 을 이용해 메소드를 종료하고, <br>
n 의 값이 0이 아니면 자기자신을 다시 호출하여 n-1 값을 매겨변수로 넘겨준다.<br>

위와 같이 자기자신을 재참조하여 호출하는 프로세르를 `재귀함수`라고 부른다.

<hr/>

### ``반복문 vs 재귀함수``

재귀함수를 반복문과 비교했을때 장단점은 아래와 같다. <br>

반복문은 `재귀함수에 비해 속도가 빠르지만, 코드가 복잡해 보일 수 있다.` <br>

재귀함수는 `클린코드 작성이 가능하지만 메모리 사용이 많아 반복문에 비해 속도가 느리다.` <br>

지속적으로 함수를 호출하게 되면서 지역변수, 매개변수, 반환값을 모두 스택에 저장해야 하는데 이러한 과정은 변수의 값만 사용하는 반복문에 비해 메모리를 더 많이 사용하게 되고 결국 성능저하로 이어지게 된다. <br>

비용 문제를 해결하기 위해 나온 방법이 꼬리재귀(tail call recursion) 이고 아래에서 설명하겠다. 

<hr/>

### ``꼬리재귀함수(tail call recursion)``

재귀 함수의 문제점인 자기 자신을 호출 한 뒤 결과를 기다리며 생기는 메모리 낭비를 해결하기 위해 재귀 호출이 끝나는 시점에 아무 일도 하지 않고 바로 결과를 반환하도록 하는 방법으로 `함수의 상태유지 및 추가 연산을 하지 않기에 스택 오버 플로우 해결이 가능`하다. <br>


```java

public static void main(String[] args) {
	System.out.println(factorial(13));
	System.out.println(factorialTail(13, 1));
}

public static int factorial(int n){
    return factorialTail(n, 1);
}

public static int factorialTail(int n, int acc){
	if(n == 1) return acc;
    return factorialTail(n - 1, acc * n);
}
```

차이점을 보면 return 할 때 재귀함수에서 필요한 매개변수를 모두 전달하여, 연산을 없앴고 이러한 사용법을 꼬리 재귀 함수라고 부른다.