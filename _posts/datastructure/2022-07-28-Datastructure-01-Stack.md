---
title:  "[자료구조] 스택? 큐? "
excerpt: "스택, 큐에 대한 설명"

tags:
  - [자료구조]

toc: true
toc_sticky: true
toc_label: "[자료구조]"
 
date: 2022-07-28
last_modified_at: 2022-07-28
---

- 참고사이트
  - [위키백과 스택](https://ko.wikipedia.org/wiki/%EC%8A%A4%ED%83%9D){:target="_blank"}
  - [위키백과 큐](https://ko.wikipedia.org/wiki/%ED%81%90_(%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0)){:target="_blank"}
  - [블로그](https://mygumi.tistory.com/357){:target="_blank"}
  - [블로그](https://ljtaek2.tistory.com/129?category=897337){:target="_blank"}
  - [블로그](https://hidelookit.tistory.com/157){:target="_blank"}


### ``스택? 큐?``

스택(Stack)은 나중에 넣은 데이터가 먼저 나오는 형태 (후입선출) <br>
큐(Queue)는 먼저 넣은 데이터가 먼저 나오는 형태 (선입선출)

<hr/>

### ``스택(Stack) 이란``

객체들의 집합소로써, 데이터를 기록하는 구조라고 보면 된다. <br>
하지만 접근 방법에 제한을 두고 있다. <br>

스택은 한 쪽 끝에서만 자료를 넣거나 뺄 수 있는 `선형 구조(LIFO - Last In First Out)`으로 되어 있다. `자료를 넣는 것을 '밀어넣는다' 하여 푸쉬(push)`라고 하고 반대로 `넣어둔 자료를 꺼내는 것을 팝(pop)`이라고 하는데, 이때 꺼내지는 자료는 가장 최근에 푸쉬한 자료부터 나오게 된다. 이처럼 나중에 넣은 값이 먼저 나오는 것을 `LIFO 구조`라고 한다. <br>

![Stack](/assets/image/datastructure/DataStructure_Stack_01.PNG)


아래가 막혀있고, 위가 뚫린 형태로 차곡차곡 쌓는 구조이다. <br>
이러한 구조이기에, 마지막에 삽입한 요소는 가장 처음으로 삭제 할 수 있다. <br>

정해진 크기에 무언가를 계속 넣다가 받아들일수 있는 크기를 초과해서 흘러넘치는 걸 `스택오버플로우` 라고 이야기하며, 재귀함수 사용 시 발생할 수 있다.

<hr/>

### ``큐(Queue) 란``

스택의 반대 개념을 갖는다. <br>

먼저 집어 넣은 데이터가 먼저 나오는 FIFO(First In First Out)구조로 저장하는 형식을 말한다. 영어 단어 queue는 표를 사러 일렬로 늘어선 사람들로 이루어진 줄을 말하기도 하며, 먼저 줄을 선 사람이 먼저 나갈 수 있는 상황을 연상하면 된다. <br>

![Queue](/assets/image/datastructure/DataStructure_Queue_01.PNG)

- 이미지출처 : https://ko.wikipedia.org/wiki/%ED%81%90_(%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0)#/media/%ED%8C%8C%EC%9D%BC:Data_Queue.svg)


위와 같은 구조이기 때문에 마지막에 삽입한 요소를 삭제하기 위해서는 앞에 있는 요소들이 전부 삭제되어야 한다. <br>

주로 순서를 보장하기 위한 처리가 필요할 떄 사용된다. <br>
큐에 저장된 요소들은 순서대로 존재하고, 가장 앞에 있을수록 오래 기다리고 있다는 걸 의미한다. <br>
