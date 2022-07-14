---
title:  "[Java] 배열의 평균값 구하기"
excerpt: "배열의 평균값 구하기"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[배열의 평균값]"
 
date: 2022-07-13
last_modified_at: 2022-07-13
---

### ``샘플코드 ``

```java
public static void main(String[] args) {
    int[] A = new int[]{5, 2, 6, 4}; 
    int[] B = new int[]{-4, 0, 2, -4};
	
    System.out.println("average : "+average(A));
    System.out.println("average : "+average(B));
    
    System.out.println("average : "+averageUseStream(A));
    System.out.println("average : "+averageUseStream(B));
}

public static double average(int[] arr) {
    double result = 0;
    
    for (int x=0; x<arr.length; x++) {
        int num = arr[x];
        result += num;
    }
    result = result / arr.length;
    
    return result;
}

public static double averageUseStream(int[] arr) {
	return Arrays.stream(arr).average().getAsDouble();
}
```

<br>


