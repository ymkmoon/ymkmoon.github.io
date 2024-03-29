---
title:  "[Java] 선형탐색(Linear Search) 이란?"
excerpt: "선형탐색(Linear Search) 이해하기"

tags:
  - [Java, Algorithm]

toc: true
toc_sticky: true
toc_label: "[Java]"
 
date: 2023-10-24
last_modified_at: 2023-10-24
---

- 참고사이트
  - [블로그](https://adjh54.tistory.com/193){:target="_blank"}


- 자료구조와 알고리즘
  - [재귀함수](https://ymkmoon.github.io/Java-30-Recursion-Function/)
  - [선형탐색](https://ymkmoon.github.io/Java-44-Algorithm-Linear-Search/)
  - [이진탐색](https://ymkmoon.github.io/Java-45-Algorithm-Binary-Search/)
  - [완전탐색 #1 정의 및 종류](https://ymkmoon.github.io/Java-46-Algorithm-Exhaustive-Search-01/)
  - [완전탐색 #2 종류별설명](https://ymkmoon.github.io/Java-47-Algorithm-Exhaustive-Search-02/)
  - [완전탐색 #3 예시](https://ymkmoon.github.io/Java-48-Algorithm-Exhaustive-Search-03/)




![Algorithm](/assets/image/java/Java_Algorithm_Linear_Search_01.PNG)


### 선형 탐색(Linear Search) 이란?

배열이나 리스트의 처음부터 끝까지 하나씩 값을 비교하면서 찾는 값을 찾을 때까지 탐색하는 방법

`선형 탐색의 경우 '정렬이 되지 않은 상태'의 배열/리스트에서 값을 찾기 위한 탐색에 사용합니다.` <br/>
`( 이진 탐색과 비교가 되며 정렬이 된 상태에서 사용하는 이진 탐색과 사용되는 방식이 다릅니다)`


<br/>

### 이진 탐색(Binary Search) 이란?

- ‘정렬된 배열’에서 ‘특정 값’을 찾는 알고리즘을 의미
- 해당 탐색 방식은 정렬이 된 배열에서 사용


<br/>

### 선형 탐색 동작 과정

![Algorithm](/assets/image/java/Java_Algorithm_Linear_Search_02.gif)


1. 배열/리스트 를 순회한다.
2. 배열/리스트를 순회하면서 하나씩 값을 비교한다.
3. 원하는 값을 찾은 경우 순회를 멈추고 값을 반환한다.


<br/>

#### 선형 탐색과 이진 탐색의 차이점

- 일반적으로 특정 값을 탐색하고자 할 때 값을 찾는 방법은 '이진 탐색'을 이용하는 것이 '선형 탐색' 보다 더 빠르다.

- 이진 탐색은 정렬 배열에서 원하는 값을 찾는 알고리즘이며, 중간값을 찾아 탐색 범위를 반으로 줄이면서 값을 찾아간다.

- 이에 비해 선형 탐색은 배열 전체를 순회하면서 값을 찾기 때문에 배열의 크기와 상관없이 속도가 일정하게 증가한다.


<br/>

#### 선형 탐색과 순차적 탐색의 차이점

- 선형 탐색과 순차적 탐색은 동일한 의미로 사용된다.

- 선형 탐색의 경우 찾고자 하는 값이 나오지 않으면 종료가 되지만, 순차적 탐색의 경우는 반복을 멈추지 않고 값이 나올 때까지 반복한다.


<br/>

#### 선형 탐색과 이진 탐색의 차이점

|분류|선형 탐색(Linear Search)|이진 탐색(Binary Search)|
|------|------|------|
|정렬여부|정렬되지 않은 배열/리스트|정렬된 배열/리스트|
|탐색속도|느림|빠름|
|탐색범위|처음부터 끝까지|중간값ㅇ르 기준으로 좌/우측 반 중 하나|
|구현방식|for/while 루프 사용|재귀함수 사용|
|시간복잡도|O(n)|O(log n)|


<br/>

### 선형탐색의 사용처

- 데이터의 크기가 작거나 정렬되어 있지 않은 경우에 주로 사용
- 예를 들어, 10개 이하의 원소로 이루어진 리스트에서 값을 찾을 때는 선형 탐색이 효율적
- 하지만 데이터의 크기가 커지면 검색 속도가 급격히 느려지므로, 큰 데이터셋에서는 다른 탐색 알고리즘을 사용하는 것을 추천


<br/>

### 선형 탐색의 성능

💡 시간 복잡도(Time Complexity)란?

- 알고리즘이 실행될 때 필요한 ‘입력 값’과 ‘연산 수행 시간’에 따라 효율적인 알고리즘을 나타내는 척도를 의미합니다.

<br/>

선형 탐색의 경우 시간 복잡도의 '빅오 표기법'을 이용하여 확인 하였을때 선형 시간인 O(n) 으로써 이진 탐색보다는 느리지만, 상대적으로 빠른 속도를 가지고 있다.


![Algorithm](/assets/image/java/Java_Algorithm_Linear_Search_03.PNG)


|표기법|이름|시간복잡도|설명|예시|
|------|------|------|------|------|
|O(1)|상수|상수 시간|입력 크기와 상관없이 일정한 실행 시간을 가진다.|배열에서 원소 하나 찾기|
|O(logn)|로그|로그 시간|입력 크기가 증가함에 따라 실행 시간이 로그함수의 형태로 증가한다|이진 탐색 알고리즘|
|O(n)|선형|선형 시간|입력 크기와 비례하는 실행 시간을 가진다|선형 탐색 알고리즘|
|O(nlogn)|로그 선형|선형 로그 시간|입력 크기가 증가함에 따라 실행 시간이 로그함수와 선형 함수의 곱의 형태로 증가한다|병합 정렬, 힙 정렬 알고리즘|
|O(n^2)|이차|이차 시간|입력 크기의 제곱에 비례하는 실행 시간을 가진다|선택 정렬, 버블 정렬, 퀵 정렬 알고리즘|
|O(2^n)|지수|지수 시간|입력 크기의 지수에 비례하는 실행 시간을 가진다|부분집합|
|O(n!)|계승|팩토리얼 시간|입력 크기의 팩토리얼에 비례하는 실행 시간을 가진다|외판원 문제|

<br/>

`[참고] 선형 탐색의 경우 이진 탐색을 제외 한다면 다른 탐색들과 비교해봐도 속도가 빠름을 확인 할 수 있다.`

<br/>

|알고리즘명|시간복잡도|
|------|------|
|선형 탐색|O(n)|
|이진 탐색|O(log n)|
|보간 탐색|O(log(log n))|
|해시 탐색|O(1)|
|깊이 우선 탐색|O(V + E)|


<br/>

### 선형 탐색의 예시

<br/>

#### 배열 내의 숫자를 찾는 예시

- 해당 코드는 선형탐색의 예시를 보여준다.
- 예시는 파라미터로 배열 'arr'와 값 'x'를 받았을 때, 배열을 순회하면서 요소 중 'x' 에 해당하는 값을 찾았을 경우 반환하며, 찾이 못한 경우는 -1 을 반환한다.


```java
public static int linearSearch(int[] arr, int x) {
    int n = arr.length;
    for (int i = 0; i < n; i++) {
        if (arr[i] == x) {
            return i;
        }
    }
    return -1;
}
```


<br/>

#### 배열 내의 문자열을 찾는 예시

- 해당 코드는 선형탐색의 예시를 보여준다.
- 예시는 파라미터로 배열 'arr'와 값 'target'을 받았을 때, 배열을 순회하면서 요소 중 'target'에 해당하는 값을 찾았을 경우 반환하며, 찾지 못한 경우는 -1을 반환한다.


```java
public static int linearSearch(String[] arr, String target) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i].equals(target)) {
            return i;
        }
    }
    return -1;
}
```