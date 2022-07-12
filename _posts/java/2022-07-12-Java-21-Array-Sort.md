---
title:  "[Java] 숫자형 배열 오름차순 내림차순 정렬하기 "
excerpt: "배열 정렬하기"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[배열 정렬]"
 
date: 2022-07-12
last_modified_at: 2022-07-12
---

### ``샘플코드 ``

```java
public static void main(String[] args) { 
	int[] A = new int[]{3, 2, -2, 5, -3};
	
	// 양수
	// positivies : [5, 3, 2]
	int[] positivies = IntStream.of(A).boxed().sorted(Comparator.reverseOrder())
	    .filter(o -> o >= 0)
	    .mapToInt(i -> i)
	    .toArray();
	
	// 음수
	// negatives : [-2, -3]
	int[] negatives = IntStream.of(A).boxed().sorted(Comparator.reverseOrder())
	    .filter(o -> o < 0)
	    .mapToInt(i -> i)
	    .toArray();
	
	// 내림차순
	// descOrder : [5, 3, 2, -2, -3]
	int[] descOrder = IntStream.of(A).boxed().sorted(Comparator.reverseOrder())
		    .mapToInt(i -> i)
		    .toArray();
	
	// 오름차순
	// ascOrder : [-3, -2, 2, 3, 5]
	int[] ascOrder = IntStream.of(A).boxed().sorted(Comparator.naturalOrder())
		    .mapToInt(i -> i)
		    .toArray();
	    
	for(int value : positivies) {
		if(IntStream.of(negatives).anyMatch(o -> o == -value)) {
			System.out.println("value : "+value);
		}
	}
} 
```

<br>


