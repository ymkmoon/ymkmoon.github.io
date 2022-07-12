---
title:  "[Java] 배열의 최대값 찾기 "
excerpt: "배열의 최대값"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[배열의 최대값]"
 
date: 2022-07-12
last_modified_at: 2022-07-12
---

### ``샘플코드 ``

```java
public static void main(String[] args) { 
	// 배열을 생성한다.
	int arr[] = {0, 80, 60, 40, 20, 100, 200, 201, 12, 17}; 
	System.out.println(ArraySort(arr, arr.length)); 
} 

// 크기가 N인 경우 최대값을 가져오는 메서드 선언
// 크기(int n)을 매개변수로 받는 이유는 재귀함수 사용을 위해
// 재귀 함수 란 자기자신을 호출하는 함수를 의미한다.
public static int ArraySort(int[] a, int n) { 
	int x; 
	if(n == 1) 
		return a[0]; 
	else 
		x = ArraySort(a, n - 1); 
	if(x > a[n - 1]) 
		return x; 
	else 
		return a[n - 1]; 
}
```

<br>


