---
title:  "[Java] 암호화 복호화란?"
excerpt: "암호화와 복호화시 사용 알고리즘"

tags:
  - [Java]

toc: true
toc_sticky: true
toc_label: "[암호화, 복호화란?]"
 
date: 2022-06-24
last_modified_at: 2022-06-24
---


- 참고사이트
  - [블로그](https://sieunlim.tistory.com/16){:target="_blank"}

### ``요약 ``

- 암호는 단방향, 양방향으로 나뉜다.
- 단방향 : 재사용 X (ex.사용자 비밀번호) 
- 양방향 : 재사용 O (ex.주소,이메일)
- 단방향 암호는 bcrypt가 제일 보편화 되어있다.
- 양방향 암호는 AES 와 RSA 등을 쓴다 (선택사항)
- 암호의 알고리즘을 직접 구현할 수 있다.
- jasypt 는 간편하고 단방향,양방향 모두 지원하지만 사용하는 암호 알고리즘의 보안이 약하다 <br>
DIGEST ALGORITHMS: [MD2, MD5, SHA, SHA-256, SHA-384, SHA-512] <br>
PBE ALGORITHMS: [PBEWITHMD5ANDDES, PBEWITHMD5ANDTRIPLEDES, PBEWITHSHA1ANDDESEDE, PBEWITHSHA1ANDRC2_40


<br>

### ``1. 암호란? ``

`암호(Cryptography)` : 평문을 암호문으로 변환 하거나 암호문을 평문으로 변환하는 기술

`평문(Plaintext)` : 해독 가능한 형태의 텍스트 <br>
ex) ("123456")

`암호문(Ciphertext)` : 해독 불가능한 형태의 텍스트 <br>
ex) ("aBD#fefa$fae!") 

`암호화(Encryption)` : 평문을 암호문으로 변환하는 과정 <br>
ex) 평문("123456") → 암호화 → 암호문("aBD#fefa$fae!")

`복호화(Decryption)` : 암호문을 평문으로 변환하는 과정 <br>
ex) 암호문("aBD#fefa$fae!!") → 복호화 → 평문("123456")


<br>

### ``2. 암호화의 종류 ``

#### `단방향 암호화` 

`단방향 암호화` : 암호화 후 복호화 할 수 없다.

ex) 사용자 비밀번호 - 사용자가 입력한 비밀번호를 암호화 하여 DB에 저장이 될 때, 모든 접근자(개발자포함)는 암호화 된 코드를 다시 평문으로 볼 수 없다. <br>
또한, 해킹이 되어도 복호화가 매우 힘들다. (예외 - RainbowTable) 

- 참고사이트
  - [유튜브](https://www.youtube.com/watch?v=TeIVhioUAXs){:target="_blank"}

<br>

#### `양방향 암호화`

`양방향 암호화` : 암호화와 복호화 모두 가능하다

ex) 사용자 주소, 이메일, 번호 또는 전자서명 같이 재사용성이 있는 정보는 암호화 복호화 모두 이루어져야한다

양방향 암호화에는 두가지 종류가 있다.

##### 대칭형(비밀키 암호) 

대칭형(비밀키 암호) : 암호화 할때 사용한 key == 복호화 할때 사용하는 key <br>
주로 데이터 통신 암호화에 많이 쓰인다 <br>

- 참고사이트 
  - [유튜브 1](https://www.youtube.com/watch?v=cs69fRxAzTQ){:target="_blank"}
  - [유튜브 2](https://www.youtube.com/watch?v=at9dA1Q4-M8){:target="_blank"}

##### 비대칭형(공개키 암호)

비대칭형(공개키 암호) : 암호화 할때 사용한 key != 복호화 할때 사용하는 key <br>
주로 인증에 많이 쓰인다 <br>

- 참고사이트 
  - [유튜브](https://www.youtube.com/watch?v=Qc1OqFv8b5Y){:target="_blank"}

<br>

### ``3. 암호 알고리즘 ``

#### 단방향 알고리즘
알고리즘은 다양하다 그 중 가장 많이 쓰이는 해시함수 알고리즘이 적용된 것만 가져옴
해시 함수사용의 중요성 

- 참고사이트 
  - [해시 함수사용의 중요성](http://ppss.kr/archives/16823){:target="_blank"}

##### SHA

`SHA는 대표적인 해시함수의 시초` since 1993 <br>

- 참고사이트
  - [SHA 적용코드](http://blog.kindler.io/java-encrypt/){:target="_blank"}


#### SHA를 보안해 좀더 쉽게 사용가능한 알고리즘 <br>

##### PBKDF2 

`해시 함수의 컨테이너인 PBKDF2`는 솔트를 적용한 후 해시 함수의 반복 횟수를 임의로 선택할 수 있다. <br>
PBKDF2는 아주 가볍고 구현하기 쉬우며, SHA와 같이 검증된 해시 함수만을 사용한다.(Django 에서 사용) <br>
PBKDF2 적용코드 링크삭제됨 <br>

##### bcrypt 

`bcrypt는 애초부터 패스워드 저장을 목적으로 설계`되었다. <br>
bcrypt는 보안에 집착하기로 유명한 OpenBSD에서 기본 암호 인증 메커니즘으로 사용되고 있고   미래에 PBKDF2보다 더 경쟁력이 있다고 여겨진다. <br>
입력값을 72 byte로 해야 해서 조금 귀찮다 하지만 가장많이쓰인다. <br>

- 참고사이트
  - [bcrypt 적용코드 1](http://vip00112.tistory.com/39){:target="_blank"}
  - [bcrypt 적용코드 2](http://kingle1024.tistory.com/11){:target="_blank"}

##### scrypt 
`scrypt는 상대적으로 최신 알고리즘`이며 나머지 둘보다 더 경쟁력 있는 것으로 평가되나, 아직 덜 확산되어 있다. <br>
scrypt는 다이제스트를 생성할 때 메모리 오버헤드를 갖도록 설계되어, `억지 기법 공격(brute-force attack)`을 시도할 때 병렬화 처리가 매우 어렵다. <br>
따라서 PBKDF2보다 안전하다고 평가되며 미래에 bcrypt에 비해 더 경쟁력이 있다고 여겨진다. <br>
scrypt 적용코드 아직/ PBKDF2의 강화판. 굼벵이 같은 해싱 속도로 유명하다 <br>

- 참고사이트 
  - [암호학적 해시 함수 장단점 & 보안점](http://d2.naver.com/helloworld/318732){:target="_blank"}
  - [해킹공격 & 팁](http://www.codeok.net/%ED%8C%A8%EC%8A%A4%EC%9B%8C%EB%93%9C%20%EB%B3%B4%EC%95%88%EC%9D%98%20%EA%B8%B0%EC%88%A0){:target="_blank"} <br>

#### 알고리즘 (양방향)

##### 대칭형(비밀키 암호) 알고리즘
AES (현재 가장 보편적으로 쓰이는 암호화 방식은 현 미국 표준 방식인 AES. <br> 128~256비트 키를 적용할수 있어 보안성이 뛰어나며 공개된 알고리즘이라 누구나 사용할 수 있다.) <br>

- 참고사이트 
  - [AES 적용 코드](http://blog.kindler.io/java-encrypt/){:target="_blank"}

##### 비대칭형(공개키 암호)알고리즘
RSA (공개키 암호시스템의 하나로 암호화뿐만 아니라 전자서명이 가능한 최초의 알고리즘) <br>

- 참고사이트
  - [RSA 적용 코드](https://www.holaxprogramming.com/2017/06/12/encryption-with-rsa/){:target="_blank"}
  - [대칭/비대칭형 비교](https://sungjk.github.io/2016/09/30/Security.html){:target="_blank"}
  - [암호종류 위키](https://namu.wiki/w/%EC%95%94%ED%98%B8%20%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98){:target="_blank"}
  - [암호화 개념참고](http://blog.kindler.io/java-encrypt/){:target="_blank"}

