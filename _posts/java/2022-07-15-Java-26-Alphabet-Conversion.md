---
title:  "[Java] ASCII 를 이용한 알파벳 대소문자 변환"
excerpt: "알파벳 대문자, 소문자 변환하기"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[알파벳 변환]"
 
date: 2022-07-15
last_modified_at: 2022-07-15
---

- 참고사이트
  - [블로그](https://pridiot.tistory.com/24){:target="_blank"}


### ``샘플코드 ``

```java
public static void main(String[] args) {
	String temp = "aBcdEfgH456";
	conversion(temp);
}

public static String conversion(String s) {
	StringBuffer result = new StringBuffer(); // 최종적으로 출력할 문자열을 저장하는 output 선언

	for (int i = 0; i < s.length(); i++) { 
		// 문자열을 검사하기 위해서 s 의 길이만큼 반복문 실행(한글자씩 int ASCII 에 옮겨서 검사)
		int ASCII = (int) s.charAt(i); // 문자열은 아스키코드(숫자)로 저장되기 때문에 int형으로 변환.
		if ((65 <= ASCII) && (ASCII <= 90)) { 
			// 문자가 65 <= ASCII <= 90인 경우 (대문자인 경우)
			// 32를 더해서 소문자로 변환
			result.append((char)(ASCII + 32)); 
		} else if ((97 <= ASCII) && (ASCII <= 122)) { 
			// 문자가 97 <= ASCII <= 122인 경우 (소문자인 경우)
			// 32를 빼서 대문자로 변환
			result.append((char)(ASCII - 32)); 
		} else { 
			// 그 외의 문자열인 경우
			result.append((char)ASCII);
		}
	}

	System.out.println("변환된 문자열 : " + result);
	
	return String.valueOf(result);
}
```

<br>


