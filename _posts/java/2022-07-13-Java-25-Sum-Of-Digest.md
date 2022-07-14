---
title:  "[Java] 자연수 자릿수의 합"
excerpt: "자연수 자릿수의 합 구하기"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[자연수 자릿수의 합]"
 
date: 2022-07-12
last_modified_at: 2022-07-12
---

### ``샘플코드 ``

```java
public static void main(String[] args) {
	int A = 2345;
	
    // 14
	System.out.println("sum : "+sum(A));
	
}

public static int sum(int n) {
    int result = 0;
    String strArray[] = Integer.toString(n).split("");
    
    for (int x=0; x<strArray.length; x++) {
        String str = strArray[x];
        int num = Integer.parseInt(str);
        result += num;
    }
            
    return result;
}
```

<br>


