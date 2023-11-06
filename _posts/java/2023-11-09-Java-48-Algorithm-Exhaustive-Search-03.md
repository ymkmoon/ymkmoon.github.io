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


### 깊이 우선 탐색(DFS: Depth-First Search) 

💡 깊이 우선 탐색(DFS: Depth-First Search) 이란?

- 루트 노드에서 시작하여 다음 분기로 넘어가기 전에 해당 분기를 완벽하게 탐색하는 방법을 의미
- 해당 방식은 자료구조의 ‘스택’을 이용하여서 구현할 수 있다
- 연결된 노드를 따라서 계속 방문을 한 후에 더 이상 연결된 노드들을 없을 때 그 전 노드로 되돌아가고 다시 연결된 노드를 따라서 탐색을 한다.

<br/>

|분류|깊이우선탐색(DFS)|너비우선탐색(BFS)|
|------|------|------|
|자료구조|스택|큐|
|탐색방식|깊이 우선|너비 우선|
|모든 경로 탐색|O|X|
|최단 경로 탐색|X|O|
|재귀 구현 가능|O|X|
|반복문 구현 가능|X|O|
|메모리 사용량|적음|많음|

<br/>


![Algorithm](/assets/image/java/Java_Algorithm_Exhaustive_Search_04.PNG)


위와 같은 그래프가 존재한다고 하고, 1번 노드부터 DFS탐색을 해보자.

- 1번 노드 방문처리 후 출력 (탐색 순서 : 1)
- 1번 노드에 인접한 노드 2번, 3번, 8번 중 하나를 방문해야 하는데 오름차순으로 방문하기로 가정
- 2번 노드 방문처리 후 출력 (탐색 순서 : 1 -> 2)
- 2번 노드에 인접한 노드 6번, 8번 중 오름차순 기준이니까 6번 노드로 이동
- 6번 노드 방문처리 후 출력 (탐색 순서 : 1 -> 2 -> 6)
- 6번 노드에 인접한 노드는 2번 노드뿐인데 이미 방문을 하였으므로 더 이상 방문처리는 하지 않고 6번 - 노드를 호출한 노드(부모 노드)로 다시 돌아간다. (여기서는 2번 노드)
- 2번 노드에 인접한 노드 중 8번 노드가 남아있으므로 8번 노드로 이동
- 8번 노드 방문처리 후 출력 (탐색 순서 : 1 -> 2 -> 6 -> 8)
- 8번 노드에 인접한 노드는 1번과 2번 노드인데 둘 다 모두 방문처리가 되어있으므로 더 이상 방문은 하지 않고 8번을 호출한 노드(부모 노드)인 2번 노드로 이동
- 2번 노드에 인접한 노드(자식 노드) 6번, 8번이 모두 방문 처리된 상태이므로 2번을 호출한 노드(부모 노드)인 1번 노드로 이동
- 이제 1번 노드에 인접한 노드 중 방문하지 않은 노드 3번으로 이동
- 3번 노드 방문처리 후 출력 (탐색 순서 : 1 -> 2 -> 6 -> 8 -> 3)
- 3번 노드에 연결된 노드는 5번 노드뿐이므로 5번 노드 방문 처리 후 출력 (탐색 순서 : 1 -> 2 -> 6 -> 8 -> 3 -> 5)
- 5번 노드에 연결된 노드 4번, 7번 중 오름차순 기준이니까 4번 먼저 방문처리 후 출력 
- 마지막으로 남은 7번 노드 방문 처리 후 출력하면 더 이상 방문할 노드가 없으므로 종료

<br/>

이렇게 되면 최종적으로 탐색한 순서는 다음과 같다. <br>
탐색 순서(재귀) : 1 -> 2 -> 6 -> 8 -> 3 -> 5 -> 4 -> 7 <br>
저는 오름차순 기준으로 해서 위와 같은 탐색 순서가 나왔지만, 기준이나 구현 방법에 따라서 탐색 순서는 바뀔 수 있다. <br>
DFS를 코드로 구현하는 방법은 재귀로 구현하는 방법과 Stack자료구조를 사용해서 구현하는 방법 2가지가 있다. <br>
위의 설명 예시는 재귀 형태로 구현했을 때의 탐색 순서이다. <br>
만약에 Stack자료구조로 구현을 한다면 연결된 자식 노드를 오름차순으로 Stack에 집어넣고 하나씩 꺼내서 다시 연결된 노드를 찾아서 Stack에 넣는 식으로 구현을 할 수 있다. <br>
이러한 형태로 탐색은 진행하면 순서는 다음과 같다. <br>
탐색 순서(Stack자료구조) : 1 -> 8 -> 3 -> 5 -> 7 -> 4 -> 2 -> 6 <br>
실제 코드로 구현해보면 스택 자료구조를 사용한 것보다는 재귀로 구현하는 게 코드가 더 짧고 간결하기 때문에 일반적으로 재귀로 많이 구현을 한다. <br>

<br/>

#### 재귀(Recursion) 형태로 구현

```java
public class Main {

	// 방문처리에 사용 할 배열선언
	static boolean[] vistied = new boolean[9];
	
	// 그림예시 그래프의 연결상태를 2차원 배열로 표현
	// 인덱스가 각각의 노드번호가 될 수 있게 0번인덱스는 아무것도 없는 상태라고 가정
	static int[][] graph = {{}, {2,3,8}, {1,6,8}, {1,5}, {5,7}, {3,4,7}, {2}, {4,5}, {1,2}};
	
	public static void main(String[] args) {
		dfs(1);
	}
	
	static void dfs(int nodeIndex) {
		// 방문 처리
		vistied[nodeIndex] = true;
		
		// 방문 노드 출력
		System.out.print(nodeIndex + " -> ");
		
		// 방문한 노드에 인접한 노드 찾기
		for (int node : graph[nodeIndex]) {
			// 인접한 노드가 방문한 적이 없다면 DFS 수행
			if(!vistied[node]) {
				dfs(node);
			}
		}
	}
}
```

<br/>

#### Stack 을 이용한 구현

```java
public class Main {

	// 방문처리에 사용 할 배열선언
	static boolean[] vistied = new boolean[9];
	
	// 그림예시 그래프의 연결상태를 2차원 배열로 표현
	// 인덱스가 각각의 노드번호가 될 수 있게 0번인덱스는 아무것도 없는 상태라고 가정
	static int[][] graph = {{}, {2,3,8}, {1,6,8}, {1,5}, {5,7}, {3,4,7}, {2}, {4,5}, {1,2}};
	
	// DFS 사용 할 스택
	static Stack<Integer> stack = new Stack<>();
	
	public static void main(String[] args) {
		
		// 시작 노드를 스택에 넣는다
		stack.push(1);
		// 시작 노드 방문처리
		vistied[1] = true;
		
		// 스택이 비어있지 않으면 계속 반복
		while(!stack.isEmpty()) {
			
			// 스택에서 하나를 꺼낸다.
			int nodeIndex = stack.pop();
			
			// 방문 노드 출력
			System.out.print(nodeIndex + " -> ");
			
			// 꺼낸 노드와 인접한 노드 찾기
			for (int LinkedNode : graph[nodeIndex]) {
				// 인접한 노드를 방문하지 않았을 경우에 스택에 넣고 방문처리 
				if(!vistied[LinkedNode]) {
					stack.push(LinkedNode);
					vistied[LinkedNode] = true;
				}
			}
		}
	}
}
```




### 너비 우선 탐색(Breadth-First Search: BFS)

💡 너비 우선 탐색(Breadth-First Search: BFS) 이란?

- 루트 노드에서 시작하여 인접한 노드를 먼저 탐색하는 방법을 의미
- 해당 방식은 자료구조의 ‘큐’를 이용하여서 구현할 수 있다
- 기본적으로 그래프 탐색에 사용되며, 가까운 노드부터 우선적으로 탐색하는 알고리즘

<br/>

|분류|깊이우선탐색(DFS)|너비우선탐색(BFS)|
|------|------|------|
|자료구조|스택|큐|
|탐색방식|깊이 우선|너비 우선|
|모든 경로 탐색|O|X|
|최단 경로 탐색|X|O|
|재귀 구현 가능|O|X|
|반복문 구현 가능|X|O|
|메모리 사용량|적음|많음|

<br/>


![Algorithm](/assets/image/java/Java_Algorithm_Exhaustive_Search_04.PNG)


위와 같은 그래프가 존재하고 노드의 탐색은 1번부터 시작한다고 가정

- 큐에 1번 노드를 넣고 방문 처리 (여기서 방문처리라는 것은 내가 해당 노드에 방문했음을 기록하는 것)
-  1번 노드와 가까운 노드를 큐에 넣고 방문 처리합니다. (2번, 3번 8번 노드) (어떻게 넣어도 상관 없다. 오름차순으로 넣는다.)
-  큐에서 노드를 하나 꺼낸다.
-  연결된 노드가 없으면 3번으로 다시 돌아간다.
-  연결된 노드가 있고 방문하지 않았으면 큐에 넣고 방문처리 후 3번으로 돌아간다.
-  연결된 노드가 있지만 방문을 이미 한 경우에는 3번으로 돌아간다.
-  3~6을 큐가 빌 때까지 반복하며 큐가 비었으면 종료한다.
- 이러한 순서로 BFS를 진행하면 위의 그래프에서 탐색 순서는 다음과 같다. <br/> 1 -> 2 -> 3 -> 8 -> 6 -> 5 -> 4 -> 7
- BFS알고리즘은 각 노드 간의 간선의 길이가 동일할 경우 최단거리를 구하는 알고리즘으로도 사용할 수 있다.

#### BFS 구현

```java
public class Main {

	public static void main(String[] args) {
		
		// 그래프를 2차원 배열로 표현한다.
		// 배열의 인덱스를 노드와 매칭시켜서 사용하기 위해 인덱스 0은 아무것도 저장하지 않는다.
		// 1번인덱스는 1번노드를 뜻하고 노드의 배열의 값은 연결된 노드들이다.
        // {}, {1번노드와 연결된 노드}, {2번노드와 연결된 노드}, {3번노드와 연결된 노드}, {4번노드와 연결된 노드}, ...
		int[][] graph = {{}, {2,3,8}, {1,6,8}, {1,5}, {5,7}, {3,4,7}, {2}, {4,5}, {1,2}};
		
		// 방문처리를 위한 boolean배열 선언
		boolean[] visited = new boolean[9];
		
		System.out.println(bfs(1, graph, visited));
		//출력 내용 : 1 -> 2 -> 3 -> 8 -> 6 -> 5 -> 4 -> 7 -> 
	}
	
	static String bfs(int start, int[][] graph, boolean[] visited) {
		// 탐색 순서를 출력하기 위한 용도
		StringBuilder sb = new StringBuilder();
		// BFS에 사용할 큐를 생성
		Queue<Integer> q = new LinkedList<Integer>();
		 
		// 큐에 BFS를 시작 할 노드 번호를 넣는다
		q.offer(start);
		
		// 시작노드 방문처리
		visited[start] = true;
		
		// 큐가 빌 때까지 반복
		while(!q.isEmpty()) {
			int nodeIndex = q.poll();
			sb.append(nodeIndex + " -> ");
			//큐에서 꺼낸 노드와 연결된 노드들 체크
			for(int i=0; i<graph[nodeIndex].length; i++) {
				int temp = graph[nodeIndex][i];
				// 방문하지 않았으면 방문처리 후 큐에 넣기
				if(!visited[temp]) {
					visited[temp] = true;
					q.offer(temp);
				}
			}
		}
		// 탐색순서 리턴
		return sb.toString() ;
	}

    
}
```