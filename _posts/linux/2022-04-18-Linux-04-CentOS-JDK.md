---
title:  "[Linux] CentOS7 Java 설치"
excerpt: "리눅스 환경에 Java 설치하기"

tags:
  - [CentOS7, Linux]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #2 JDK]"
 
date: 2022-04-18
last_modified_at: 2022-04-18
---

- 가상머신을 이용해 윈도우에 리눅스 서버 구축하기
  1.  [가상머신(Virtual machine) 설치](https://ymkmoon.github.io/Linux-02-VMWare/)
  2.  [가상머신에 리눅스 환경 구축하기](https://ymkmoon.github.io/Linux-03-CentOS/)
  3.  [Java 설치](https://ymkmoon.github.io/Linux-04-CentOS-JDK/)
  4.  [SSH 설정](https://ymkmoon.github.io/Linux-05-CentOS-SSH/)
  5.  [Apache 설치](https://ymkmoon.github.io/Linux-06-CentOS-Apache/)
  6.  [Tomcat 설치](https://ymkmoon.github.io/Linux-07-CentOS-Tomcat/)
  7.  [Apache 와 Tomcat 연동](https://ymkmoon.github.io/Linux-08-CentOS-Apache-Tomcat/)
  8.  [FTP 설정](https://ymkmoon.github.io/Linux-09-CentOS-FTP/)
  9.  [Redmine 설치](https://ymkmoon.github.io/Linux-10-CentOS-Redmine/)
  10. [Jenkins 설치](https://ymkmoon.github.io/Linux-11-CentOS-Jenkins/)
  11. [Gradle 설치](https://ymkmoon.github.io/Linux-12-CentOS-Gradle/)
  12. [Jenkins를 이용해 Git Repository 배포하기](https://ymkmoon.github.io/Linux-13-CentOS-Jenkins-Deploy/)

<hr/>

## CentOS7 에 Java 설치하기
<br>


### JDK 설치하기
- 인증 문제로 설치가 안 될 경우 open jdk 8 버전을 설치한다.
  
  ```console
  sudo yum -y install java-1.8.0-openjdk-devel.x86_64
  ```

- wget 명령어 실행이 안된다면 sudo yum install wget 을 입력하려 wget 설치.
  
  ```console
  sudo yum install wget
  ```

<br>

### 환경변수 등록하기
- /usr/bin/java 경로에 심볼릭링크가 걸려있기 때문에 실제 경로를 찾아서 환경변수에 등록해주어야 한다.


  ```console
  readlink -f /usr/bin/java
  # /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-1.el7_9.x86_64/jre/bin/java
  ```
  

- 실제 경로를 찾았으면 /etc/profile을 vi로 열어준 뒤 최하단에 JAVA_HOME, PATH, CLASSPATH를 등록한다.
  
  ```console
  JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-1.el7_9.x86_64
  PATH=$PATH:$JAVA_HOME/bin
  CLASSPATH=$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar

  export JAVA_HOME PATH CLASSPATH
  ```

- 등록한 환경변수 적용

  ```console
  source /etc/profile
  ```


### 환경변수 확인하기
  
  ```console
  echo $JAVA_HOME
  echo $PATH
  echo $CLASSPATH
  ```

### 컴파일 테스트
- jvm 이 정상적으로 작동하는지 간단한 class 를 생성하여 실행해보자.
  
  파일을 생성 후 아래 내용을 작성합니다.
  
  ```console
  vi HelloWorld.java
  ```
  
  ```java
  public class HelloWorld{
   public static void main(String[] args){
        System.out.println("Hello World!!");
   }
  }
  ```

  ```console
  javac HelloWorld.java
  java -cp . Helloworld
  ```

  위 명령어 실행 시 Hello World!! 가 출력됩니다.




