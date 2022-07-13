---
title:  "[Java] 두 배열 인수의 곱셈 "
excerpt: "두 배열 인수의 곱셈 구하기"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[두 배열 인수의 곱셈]"
 
date: 2022-07-13
last_modified_at: 2022-07-13
---

### ``샘플코드 ``

```java
public static void main(String[] args) {
	int[] A = new int[]{5, 2, 6, 4};
	int[] B = new int[]{-4, 0, 2, -4};
	
	// -20 + 0 + 12 - 16 = -24
	System.out.println("multiplication : "+multiplication(A, B));
	
}

public static int multiplication(int[] a, int[] b) {
    int result = 0;
    
    for (int x=0; x<a.length; x++) {
        result = result + (a[x] * b[x]);
    }
    
    return result;
}
```

<br>


