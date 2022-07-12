---
title:  "[Java] 16진수 변환하기 "
excerpt: "16진수를 10진수로 변환하기"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[16진수 변환]"
 
date: 2022-07-12
last_modified_at: 2022-07-12
---

### ``샘플코드 ``

```java
public static void main(String[] args) throws IOException {
	parseByte();
}

public static double parseByte() {
	String hex;
	// for Test
	hex = "29a"; // 666
//		hex = "3e7"; // 999
//		hex = "FFFFFD05"; // -763
//		hex = "FFFFFFFFF1"; // -15
	
	System.out.println("hex : "+hex);
	System.out.println("hexToDec(hex) : "+hexToDec(hex));
	
	return Double.parseDouble(String.valueOf(hexToDec(hex)));
}

/**
 * @param hex
 * @return
 */
public static Number hexToDec(String hex) {

	if (hex == null) {
		throw new NullPointerException("hexToDec: hex String is null.");
	}

	// You may want to do something different with the empty string.
	if (hex.equals("")) {
		return Byte.valueOf("0");
	}

	// If you want to pad "FFF" to "0FFF" do it here.

	hex = hex.toUpperCase();

	// Check if high bit is set.
	boolean isNegative = hex.startsWith("8") || hex.startsWith("9") || hex.startsWith("A") || hex.startsWith("B")
			|| hex.startsWith("C") || hex.startsWith("D") || hex.startsWith("E") || hex.startsWith("F");

	BigInteger temp;

	if (isNegative) {
		// Negative number
		temp = new BigInteger(hex, 16);
		BigInteger subtrahend = BigInteger.ONE.shiftLeft(hex.length() * 4);
		temp = temp.subtract(subtrahend);
	} else {
		// Positive number
		temp = new BigInteger(hex, 16);
	}

	// Cut BigInteger down to size.
	if (hex.length() <= 2) {
		return (Byte) temp.byteValue();
	}
	if (hex.length() <= 4) {
		return (Short) temp.shortValue();
	}
	if (hex.length() <= 8) {
		return (Integer) temp.intValue();
	}
	if (hex.length() <= 16) {
		return (Long) temp.longValue();
	}

	return temp;
}
```

<br>


