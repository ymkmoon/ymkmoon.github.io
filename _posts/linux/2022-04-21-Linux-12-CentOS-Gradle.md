---
title:  "[Linux] CentOS7 Gradle 설치"
excerpt: "리눅스 환경에 Gradle 설치하기"

tags:
  - [CentOS7, Linux, Gradle]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #10 Gradle]"
 
date: 2022-04-21
last_modified_at: 2022-04-21
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

## Gradle 이란?
> Gradle은 그루비를 이용한 빌드 자동화 시스템이다. <br>
Groovy와 유사한 도메인 언어를 채용하였으며, 현재 안드로이드 앱을 만드는데 필요한 안드로이드 스튜디오의 공식 빌드 시스템이기도 하다. Java, C/C++, 파이썬 등과 같은 여러 가지 언어를 지원한다.


### Gradle 설치
  > gradle 은 빌드 도구이며, 저는 추후 jenkins 를 이용해 배포 할 프로젝트의 버전과 동일한 버전을 다운로드 하겠습니다.

  ```console  
  cd /usr/local/download
  sudo wget https://services.gradle.org/distributions/gradle-7.2-bin.zip
  sudo unzip -d /opt/gradle gradle-7.2-bin.zip
  ```

### 환경 변수 설정

  ```console
  vi /etc/profile.d/gradle.sh

  # gradle.sh 파일에 아래 내용을 작성 후 저장한다.
  export GRADLE_HOME=/opt/gradle/gradle-7.2
  export PATH=${GRADLE_HOME}/bin:${PATH}

  sudo chmod +x /etc/profile.d/gradle.sh # chmod 를 사용하여 스크립트 실행
  source /etc/profile.d/gradle.sh # 환경변수 로드
  ```

### Gradle 설치 확인

  ```console
  gradle -v # 명령어 실행 시 아래와 같이 출력된다.

  Welcome to Gradle 7.2!

  Here are the highlights of this release:
  - Toolchain support for Scala
  - More cache hits when Java source files have platform-specific line endings
  - More resilient remote HTTP build cache behavior

  For more details see https://docs.gradle.org/7.2/release-notes.html


  ------------------------------------------------------------
  Gradle 7.2
  ------------------------------------------------------------

  Build time:   2021-08-17 09:59:03 UTC
  Revision:     a773786b58bb28710e3dc96c4d1a7063628952ad

  Kotlin:       1.5.21
  Groovy:       3.0.8
  Ant:          Apache Ant(TM) version 1.10.9 compiled on September 27 2020
  JVM:          1.8.0_322 (Red Hat, Inc. 25.322-b06)
  OS:           Linux 3.10.0-1062.el7.x86_64 amd64

  ```

