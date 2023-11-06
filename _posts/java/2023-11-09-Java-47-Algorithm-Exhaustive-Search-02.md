---
title:  "[Java/알고리즘] 완전탐색(Exhaustive Search) 이란? #1 정의 및 종류"
excerpt: "완전탐색(Exhaustive Search) 정의 및 종류"

tags:
  - [Java, Algorithm]

toc: true
toc_sticky: true
toc_label: "[Java]"
 
date: 2023-11-06
last_modified_at: 2023-11-06
---

- 참고사이트
  - [블로그](https://adjh54.tistory.com/196){:target="_blank"}


- 자료구조와 알고리즘
  - [재귀함수](https://ymkmoon.github.io/Java-30-Recursion-Function/)
  - [선형탐색](https://ymkmoon.github.io/Java-44-Algorithm-Linear-Search/)
  - [이진탐색](https://ymkmoon.github.io/Java-45-Algorithm-Binary-Search/)
  - [완전탐색 #1 정의 및 종류](https://ymkmoon.github.io/Java-46-Algorithm-Exhaustive-Search-01/)
  - [완전탐색 #2 종류별설명](https://ymkmoon.github.io/Java-47-Algorithm-Exhaustive-Search-02/)
  - [완전탐색 #3 예시](https://ymkmoon.github.io/Java-48-Algorithm-Exhaustive-Search-03/)



![Algorithm](/assets/image/java/Java_Algorithm_Exhaustive_Search_01.PNG)


<br/>

### 완전 탐색(Exhaustive Search) 이란?

- 알고리즘에서 사용되는 기법 중 하나로 ‘모든 가능한 경우의 수를 탐색’하여 ‘최적의 결과를 찾는 방법’을 의미

`모든 가능성을 고려하기 때문에 항상 최적의 해를 찾을 수 있지만 경우의 수가 매우 많은 경우 시간과 메모리의 부담이 커질 수 있다. 그렇기에 문제의 특성에 따라 다른 탐색 기법을 사용하는 것이 좋다.`

<br/>


### 브루트 포스(Brute-Force)

💡 브루트 포스(Brute-Force) 이란?

- ‘모든 경우를 다 탐색’하면서 결과를 얻는 알고리즘을 의미

- 해당 알고리즘은 경우의 수가 작을 때 사용하는 것이 일반적이다. 대표적인 예시로는 자물쇠의 비밀번호를 알 수 없는 경우 모든 경우를 대입하여서 비밀번호를 찾아가는 경우가 있다.

<br/>

💡 브루트 포스(Brute-Force)의 의미

- 영어로 부르트 포스는 "난폭한 힘, 폭력"이라는 뜻을 의미하며 무식하지만 확실한 방식을 의미한다. 이는 수행하는데 오래 걸리는 데다 자원이 많이 소요되지만 이론적으로 가능한 수를 모두 검색하기에 예상된 결과를 얻을 수 있다.


💡 완전 탐색에서 브루트 포스란?

- 모든 경우의 수를 탐색하여 최적의 결과를 찾아가는 완전탐색에서 브루트 포스는 모든 경우를 다 검사하여 원하는 값을 탐색하는 방법이다.


💡 브루트 포스를 이용한 공격방식이란?

- 해커들은 이 알고리즘을 이용하여 다양한 공격을 시도한다. 예를 들어, 로그인 페이지에서 사용자 이름과 비밀번호를 입력하여 로그인하는 경우 해커들은 브루트 포스 알고리즘을 이용하여 가능한 모든 비밀번호를 시도하면서 로그인을 시도한다. 이러한 공격을 방지하기 위해서는 강력한 비밀번호를 사용하고 로그인 시도 횟수를 제안하는 등의 보안 조치를 취해야 한다.


<br/>


#### 브루토 포스 사용 예시

<br/>

- 배열탐색

💡 배열에서 특정 값을 찾는 문제에서 브루트 포스 알고리즘은 배열을 모두 탐색하여 값을 찾는 방식으로 문제를 해결합니다.

```java
public static int findIndex(int[] arr, int target) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == target) {
            return i;
        }
    }
    return -1;
}
```

<br/>

- 문자열 비교

💡 문자열 비교 문제에서 브루트 포스 알고리즘은 가능한 모든 문자열 쌍을 비교하여 최소값 또는 최대값을 찾는 방식으로 문제를 해결합니다.

```java
public static int stringCompare(String s1, String s2) {
    int n1 = s1.length(), n2 = s2.length();
    int min = Math.min(n1, n2);

    for (int i = 0; i < min; i++) {
        if (s1.charAt(i) != s2.charAt(i)) {
            return s1.charAt(i) - s2.charAt(i);
        }
    }

    if (n1 == n2) {
        return 0;
    } else if (n1 > n2) {
        return s1.charAt(min);
    } else {
        return s2.charAt(min);
    }
}
```

<br/>

### 비트마스크

💡 비트마스크(bitmask)란?

- ‘이진수’를 ‘비트 연산’을 통해 경우의 수를 줄여가며 탐색하는 방식을 의미

- 비트 마스크를 사용하면 하나의 변수에 여러 개의 상태 정보를 저장할 수 있으며 이를 통해 복잡한 조건문을 간단하게 처리할 수 있다. 이 방법은 비트 연산을 사용하기 때문에 빠르게 계산할 수 있어서, 경우의 수가 많은 경우에 유용하다.

<br/>

💡 이진수란?

- 0과 1로 이루어진 수의 체계를 의미합니다.


💡 이진법이란?

- 이진수를 사용하여 숫자를 표현하는 방법을 의미합니다.


💡 완전 탐색에서 비트마스크란?

- 모든 경우의 수를 탐색하여 최적의 결과를 찾아가는 완전탐색에서 비트마스크는 모든 경우의 수를 이진수로 표현하여 빠르게 계산을 해 나아가는 방식입니다.


<br/>

#### 비트마스크의 비트연산

|연산자|설명|
|------|------|
|&|두 비트 모두 1일 때 1을 반환, 그 외에는 0을 반환|
|\||두 비트 중 하나라도 1일 때 1을 반환, 둘 다 0일 때 0을 반환|
|^|두 비트가 다를 때 1을 반환, 같을 때 0을 반환|
|~|비트를 반전시켜 반환|
|<<|비트를 왼쪽으로 이동|
|>>|비트를 오른쪽으로 이동|

<br/>

- 비트마스크의 이진수 연산의 예시

```java
int a = 10; // 1010
int b = 12; // 1100

int result1 = a & b; // 1000
int result2 = a | b; // 1110
int result3 = a ^ b; // 0110
int result4 = a << 1; // 10100
int result5 = a >> 1; // 0101
```

<br/>

#### 비트마스크 사용 예시

- 권한관리

💡 보통 권한은 4가지 이상으로 구분되어 있는 경우가 많습니다. 이때 각 권한을 비트로 표현하여 하나의 정수값으로 나타내면 매우 간편해진다.

```java
public static final int PERMISSION_READ = 1 << 0; // 0001
public static final int PERMISSION_WRITE = 1 << 1; // 0010
public static final int PERMISSION_DELETE = 1 << 2; // 0100
public static final int PERMISSION_EXECUTE = 1 << 3; // 1000

int userPermission = PERMISSION_READ | PERMISSION_WRITE; // 0011
int groupPermission = PERMISSION_READ | PERMISSION_EXECUTE; // 1001

boolean hasReadPermission = (userPermission & PERMISSION_READ) != 0; // true
boolean hasDeletePermission = (groupPermission & PERMISSION_DELETE) != 0; // false
```

<br/>

- 집합관리

💡 집합을 비트로 표현하여 메모리를 절약할 수 있습니다. 예를 들어, 0부터 31까지의 정수 중에서 3, 5, 7번째 원소를 포함하는 집합을 나타내면 다음과 같이 표현할 수 있습니다.

```java
int set = (1 << 3) | (1 << 5) | (1 << 7); // 0010 1010 1000

boolean hasElement5 = (set & (1 << 5)) != 0; // true
boolean hasElement6 = (set & (1 << 6)) != 0; // false
```

<br/>

- 상태 플래그 관리

💡 여러 상태를 하나의 정수 값으로 나타내어 관리할 수 있습니다. 예를 들어, 주어진 수가 2의 거듭제곱인지 여부를 판단할 때 다음과 같은 방법을 사용할 수 있습니다.



```java
public static final int FLAG_POWER_OF_TWO = 1 << 0; // 0001
public static final int FLAG_NEGATIVE = 1 << 1; // 0010

int number = 8; // 2의 거듭제곱
int flags = 0;

if ((number & (number - 1)) == 0) { // 2의 거듭제곱인 경우
    flags |= FLAG_POWER_OF_TWO;
}

if (number < 0) { // 음수인 경우
    flags |= FLAG_NEGATIVE;
}

if ((flags & FLAG_POWER_OF_TWO) != 0) {
    System.out.println(number + " is power of two");
}

if ((flags & FLAG_NEGATIVE) != 0) {
    System.out.println(number + " is negative");
}
```

<br/>



### 백트래킹(Backtracking)


💡 백트래킹(Backtracking)

- 해결책으로 가는 도중에 막히게 되면 그 지점으로 다시 돌아가서 다른 경로를 탐색하는 방식을 의미하나다. 이를 통해 모든 가능한 경우의 수를 탐색하여 해결책을 찾는다.
- 해당 알고리즘을 사용할때 주로 재귀함수를 사용하여 구현하며 스택을 이용하여서 사용하는 경우도 있다.
- 재귀함수를 이용하는 경우 해결책을 찾기 위해 생성, 검사, 선택, 이동, 되돌아가기 등 과정이 재귀적으로 수행된다.
- 스택을 이용하는 경우 이전 상태로 되돌아가기 위해 스택에 현재 상태를 저장하고, 되돌아갈 때 스택에서 상태를 꺼내오는 형태로 수행한다.


<br/>

💡 완전 탐색에서 백트래킹이란?

- 모든 경우의 수를 탐색하여 최적의 결과를 찾아가는 완전탐색에서 백트래킹은 해결책을 찾아가는 도중에 막히게 되면 다시 돌아가서 다른 경로로 탐색을 하여 결국 모든 가능한 경우의 수를 탐색하여 해결책을 찾아가는 방식으로 사용됩니다.

<br/>
