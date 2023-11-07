---
title:  "[Java] 완전탐색(Exhaustive Search) 이란? #3 예시"
excerpt: "완전탐색(Exhaustive Search) 을 문제로 이해하기"

tags:
  - [Java, Algorithm]

toc: true
toc_sticky: true
toc_label: "[Java]"
 
date: 2023-11-07
last_modified_at: 2023-11-07
---

- 참고사이트
  - [블로그](https://adjh54.tistory.com/227){:target="_blank"}


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

- 알고리즘에서 사용되는 기법 중 하나로 `모든 가능한 경우의 수를 탐색하여 `최적의 결과를 찾는 방법`을 의미

`모든 가능성을 고려하기 때문에 항상 최적의 해를 찾을 수 있지만 경우의 수가 매우 많은 경우 시간과 메모리의 부담이 커질 수 있다. 그렇기에 문제의 특성에 따라 다른 탐색 기법을 사용하는 것이 좋다.`

<br/>


### 완전 탐색의 종류


|종류|설명|장점|단점|
|------|------|------|------|
브루트 포스|‘모든 경우의 수를 탐색’하면서 원하는 결과를 얻는 알고리즘을 의미한다|가능한 모든 경우를 다 검사하기 때문에 예상된 결과를 얻을 수 있음|경우의 수가 많을 경우 시간이 오래 걸림|
|비트마스크|‘모든 경우의 수’를 이진수료 표현하고 ‘비트 연산’을 통해 원하는 결과를 빠르게 얻는 알고리즘을 의미|이진수 연산을 이용하여 계산 속도가 빠름	|경우의 수가 많아질수록 메모리 사용량이 늘어남|
|백트래킹|결과를 얻기 위해 진행하는 도중에 ‘막히게 되면’ 그 지점으로 다시 돌아가서 ‘다른 경로를 탐색’하는 방식을 의미, 결국 모든 가능한 경우의 수를 탐색하여 해결책을 찾음|경우의 수를 줄이면서도 모든 경우를 탐색할 수 있음|재귀 함수를 이용하기 때문에 스택 오버플로우가 발생할 가능성 있음|
|순열|‘순열’을 이용하여 모든 경우의 수를 탐색하는 방법이다. 순열은 서로 다른 n개 중에서 r개를 선택하여 나열하는 방법을 의미|경우의 수가 적을 때 사용하면 유용함|경우의 수가 많을 경우 시간이 오래 걸림|
|재귀함수|자기 자신을 호출하여 모든 가능한 경우의 수를 체크하면서 최적의 해답을 얻는 방식을 의미|코드가 간결하며, 이해하기 쉬움|스택 오버플로우가 발생할 가능성이 있음|
|DFS/BFS|`깊이 우선 탐색(DFS: Depth-First Search)` <br> - 루트 노드에서 시작하여 다음 분기로 넘어가기 전에 해당 분기를 완벽하게 탐색하는 방법을 의미 <br> `너비 우선 탐색(Breadth-First Search: BFS)` <br> - 루트 노드에서 시작하여 인접한 노드를 먼저 탐색하는 방법을 의미|미로 찾기 등에 유용함|최악의 경우, 모든 노드를 다 방문해야 하므로 시간이 오래 걸림|

<br/>


### 1. 브루트포스 - 최소 직사각형

💡 문제 설명  
- 명함 지갑을 만드는 회사에서 지갑의 크기를 정하려고 한다. 다양한 모양과 크기의 명함들을 모두 수납할 수 있으면서, 작아서 들고 다니기 편한 지갑을 만들어야 한다. 이러한 요건을 만족하는 지갑을 만들기 위해 디자인팀은 모든 명함의 가로길이와 세로 길이를 조사했다. 아래 표는 4가지 명함의 가로길이와 세로 길이를 나타낸다.
- <하단의 표 1을 참고> 
- 가장 긴 가로 길이와 세로 길이가 각각 80, 70이기 때문에 80(가로) x 70(세로) 크기의 지갑을 만들면 모든 명함들을 수납할 수 있다. 하지만 2번 명함을 가로로 눕혀 수납한다면 80(가로) x 50(세로) 크기의 지갑으로 모든 명함들을 수납할 수 있다. 이때의 지갑 크기는 4000(=80 x 50)이다. 
- 모든 명함의 가로 길이와 세로 길이를 나타내는 2차원 배열 sizes가 매개변수로 주어진다. 모든 명함을 수납할 수 있는 가장 작은 지갑을 만들 때, 지갑의 크기를 return 하도록 solution 함수를 완성해라.


<br/><br/>

💡 제한사항 
- sizes의 길이는 1 이상 10,000 이하
- sizes의 원소는 [w, h] 형식
- w는 명함의 가로 길이를 나타낸다
- h는 명함의 세로 길이를 나타낸다
- w와 h는 1 이상 1,000 이하인 자연수이.


<br/>

> [문제 설멍] 표1

<br/>

|명함번호|가로길이|세로길이|
|------|------|------|
|1|60|50|
|2|30|70|
|3|60|30|
|4|80|40|

<br/>

> 입출력 예

<br/>

|sizes|result|
|------|------|
|[[60, 50], [30, 70], [60, 30], [80, 40]]|4000|
|[[10, 7], [12, 3], [8, 15], [14, 7], [5, 15]]|120|
|[[14, 4], [19, 6], [6, 16], [18, 7], [7, 11]]|133|

<br/>

💡 해당 문제의 요점은 ‘브루트 포스’ 알고리즘을 통해 ‘카드의 사이즈를 구하기 위해서 모든 경우의 수를 다 확인’하여서 결과를 찾는다는 점이다. <br/> 

- [STEP1] 2차원 배열을 1차원 배열로 분해하여 순회한다(* 카드의 모든 경우의 수를 순회) 
- [STEP2] 카드의 가로와 세로의 길이 중 ‘최대값’을 구하고 이전 카드의 너비와 비교하여 ‘최대값’을 구한다. 
- [STEP3] 카드의 가로와 세로의 길이 중 ‘최소값’을 구하고 이전 카드의 높이와 비교하여 ‘최대값’을 구한다. 
- [STEP4] 최종적으로 계산된 너비와 높이값을 구하여 최종 카드 사이즈를 구한다.

<br/>

```java
public static void main(String[] args) {
		int answer = 0;
//	    int[][] size = { {60, 50}, {30, 70}, {60, 30}, {80, 40} };
//	    int[][] size = { {10, 7}, {12, 3}, {8, 15}, {14, 7}, {5, 15} };
	    int[][] size = { {14, 4}, {19, 6}, {6, 16}, {18, 7}, {7, 11} };

	    int width = 0;      // 총 계산된 카드의 너비
	    int height = 0;     // 총 계싼된 카드의 높이

	    // [STEP1] 2차원 배열을 1차원 배열로 순회한다. (* 카드의 모든 경우를 순회 )
	    for (int[] card : size) {

	        int cardItemWidth = card[0];    // n번 명함의 가로 길이
	        int cardItemHeight = card[1];   // n번 명함의 세로 길이

	        // [STEP2] 카드의 기로와 세로의 길이 중 최대 값과 이전 카드 사이즈를 비교하여 최종 너비값을 구한다.
	        width = Math.max(width, Math.max(cardItemWidth, cardItemHeight));

	        // [STEP3] 카드의 가로와 세로의 길이 총 최소 값과 이전에 카드 사이즈를 비교하여 최종 높이값을 구한다.
	        height = Math.max(height, Math.min(cardItemWidth, cardItemHeight));
	    }

	    // [STEP4] 최종적으로 계산된 너비와 높이 값을 곱하여 결과값을 구합니다.
	    answer = width * height;
	    
	    System.out.println("answer : "+answer);
	}
```

<br/>

### 2. 브루트포스 - 모의고사

<br/>

💡 문제 설명- 수포자는 수학을 포기한 사람의 준말이다. <br> 
수포자 삼인방은 모의고사에 수학 문제를 전부 찍으려 한다. <br>
수포자는 1번 문제부터 마지막 문제까지 다음과 같이 찍는다. <br/><br/> 
1번 수포자가 찍는 방식: 1, 2, 3, 4, 5, 1, 2, 3, 4, 5, ... <br/> 
2번 수포자가 찍는 방식: 2, 1, 2, 3, 2, 4, 2, 5, 2, 1, 2, 3, 2, 4, 2, 5, ... <br/> 
3번 수포자가 찍는 방식: 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, 3, 3, 1, 1, 2, 2, 4, 4, 5, 5, ... <br/> 
1번 문제부터 마지막 문제까지의 정답이 순서대로 들은 배열 answers가 주어졌을 때, 가장 많은 문제를 맞힌 사람이 누구인지 배열에 담아 return 하도록 solution 함수를 작성하자. <br/>

<br/><br/>

💡 제한 조건 - 시험은 최대 10,000 문제로 구성되어 있고, 문제의 정답은 1, 2, 3, 4, 5중 하나이다. 가장 높은 점수를 받은 사람이 여럿일 경우, return 하는 값을 오름차순 정렬해라.

<br/>

> 입출력의 예

<br/>

|answers|return|
|------|------|
|[1,2,3,4,5]|[1]|
|[1,3,2,4,2]|[1,2,3]|

<br/>

> 💡 해당 문제의 요점은 ‘브루트 포스’ 알고리즘을 통해 ‘수포자의 패턴’을 통해 결과를 얻기 위해 모든 경우의 수를 다 확인’하여서 결과를 찾는다는 점이다. <br/>

<br/> 

- [STEP1] 수포자들의 문제를 찍는 패턴 정의한다. 
- [STEP2] 문제 배열을 순회하면서 각각의 값을 Counting 한다. (* 모든 경우의 수를 순회) 
- [STEP3] 구성한 리스트의 최대값을 구한다. 
- [STEP4] 최대값인 사용자를 구한다. 
- [STEP5] 최종 배열로 구성.(List to Array[])

<br/>

```java
public static void main(String[] args) {
	// 1번 문제부터 마지막 문제까지의 정답이 순서대로 들어있는 배열 answers
    // int[] answers = new int[]{1, 2, 3, 4, 5};
	int[] answers = new int[]{1, 3, 2, 4, 2};
	
    List<Integer> personScoreList = Arrays.asList(0, 0, 0);

    // [STEP1] 수포자들의 문제를 찍는 패턴 정의
    int[] person1Pattern = new int[]{1, 2, 3, 4, 5};
    int[] person2Pattern = new int[]{2, 1, 2, 3, 2, 4, 2, 5};
    int[] person3Pattern = new int[]{3, 3, 1, 1, 2, 2, 4, 4, 5, 5};

    // [STEP2] 문제 배열을 순회하면서 각각의 값을 Counting (* 모든 경우의 수를 순회)
    for (int i = 0; i < answers.length; i++) {
        if (answers[i] == person1Pattern[i % 5]) personScoreList.set(0, personScoreList.get(0) + 1);
        if (answers[i] == person2Pattern[i % 8]) personScoreList.set(1, personScoreList.get(1) + 1);
        if (answers[i] == person3Pattern[i % 10]) personScoreList.set(2, personScoreList.get(2) + 1);
    }
    // [STEP3] 구성한 리스트의 최대값
    int max = Collections.max(personScoreList);

    // [STEP4] 최대값인 사용자
    List<Integer> maxPersonList = new ArrayList<>();
    for (int i = 0; i < personScoreList.size(); i++) {
        if (max == personScoreList.get(i)) maxPersonList.add(i + 1);
    }

    // [STEP5] 최종 배열로 구성 (List to Array[])
    int[] answer = maxPersonList.stream().mapToInt(i -> i).toArray();
    maxPersonList.stream().mapToInt(i -> i).forEach(i -> System.out.println("answer : "+i));
}
```

<br/>

<br/>

<br/>

<br/>

