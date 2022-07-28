---
title:  "[Java] 암호화 복호화 예제코드"
excerpt: "암호화 복호화 예제코드"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[암호화 복호화 예제코드]"
 
date: 2022-06-24
last_modified_at: 2022-06-24
---

- 참고사이트
  - [블로그](https://coding-start.tistory.com/211){:target="_blank"}



### ``참고``

복호화 때 사용 할 값을 쿠키에서 가지고 오면, 브라우저에서 값을 인코딩하여 저정하기 때문에, decAES 함수 실행 전 에 복호화 할 값을 디코딩 해야한다.

<br>

### ``예제코드 ``

```java

import java.net.URLDecoder;
import java.nio.charset.StandardCharsets;
import java.security.Key;
import java.util.Base64;

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;

import lombok.experimental.UtilityClass;

@UtilityClass
public class EncryptionUtil {

	public Key getAESKey() thorws Exception { 
        String key = "ymkymkymkymk";
        byte[] keyBytes = new byte[16];
        byte[] b = key.getBytes(StandardCharsets.UTF_8);
 
        int len = b.length;
        if (len > keyBytes.length) {
           len = keyBytes.length;
        }
 
        System.arraycopy(b, 0, keyBytes, 0, len);
        Key keySpec = new SecretKeySpec(keyBytes, "AES");
 
        return keySpec;
    }
 
    // 암호화
    public String encAES(String str) thorws Exception {
        Key keySpec = getAESKey();
        String iv = "0987654321654321";
        Cipher c = Cipher.getInstance("AES/CBC/PKCS5Padding");
        c.init(Cipher.ENCRYPT_MODE, keySpec, new IvParameterSpec(iv.getBytes(StandardCharsets.UTF_8)));
        byte[] encrypted = c.doFinal(str.getBytes(StandardCharsets.UTF_8));
        return new String(Base64.getEncoder().encode(encrypted));
    }
 
    // 복호화
    public String decAES(String enStr) throws Exception thorws Exception {
        Key keySpec = getAESKey();
        String iv = "0987654321654321";
        Cipher c = Cipher.getInstance("AES/CBC/PKCS5Padding");
        c.init(Cipher.DECRYPT_MODE, keySpec, new IvParameterSpec(iv.getBytes(StandardCharsets.UTF_8)));
        byte[] byteStr = Base64.getDecoder().decode(URLDecoder.decode(enStr, "UTF-8").getBytes(StandardCharsets.UTF_8));
        return new String(c.doFinal(byteStr), "UTF-8");
    }
    
}

```

<br>

