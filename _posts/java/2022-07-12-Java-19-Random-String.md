---
title:  "[Java] 랜덤 알파벳+숫자 생성하기 "
excerpt: "랜덤 String 생성"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[랜덤 String 생성]"
 
date: 2022-07-12
last_modified_at: 2022-07-12
---

### ``샘플코드 ``

```java
public static void main(String[] args) {
	int leftLimit = 48; // numeral '0'
	int rightLimit = 122; // letter 'z'
	int targetStringLength = 13;
	Random random = new Random();

	String generatedString = random.ints(leftLimit,rightLimit + 1)
	  .filter(i -> (i <= 57 || i >= 65) && (i <= 90 || i >= 97))
	  .limit(targetStringLength)
	  .collect(StringBuilder::new, StringBuilder::appendCodePoint, StringBuilder::append)
	  .toString();

	System.out.println(generatedString);

}
```

<br>


