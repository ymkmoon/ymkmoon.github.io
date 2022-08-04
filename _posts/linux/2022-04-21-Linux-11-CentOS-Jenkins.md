---
title:  "[Linux] CentOS7 Jenkins 설치"
excerpt: "리눅스 환경에 Jenkins 설치하기"

tags:
  - [CentOS7, Linux, Jenkins]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #9 Jenkins]"
 
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

## Jenkins 란?
> 젠킨스는 소프트웨어 개발 시 지속적 통합 서비스를 제공하는 툴이다. <br>
 다수의 개발자들이 하나의 프로그램을 개발할 때 버전 충돌을 방지하기 위해 각자 작업한 내용을 공유 영역에 있는 Git등의 저장소에 빈번히 업로드함으로써 지속적 통합이 가능하도록 해 준다.


### Jenkins 설치

  ```console
  sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo --no-check-certificate
  sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
  sudo yum install jenkins
  ```

### 포트 변경

  ```console
  sudo vi /etc/sysconfig/jenkins

  # 위 파일에서 아래내용 변경
  JENKINS_HTTPS_PORT="8080" 
  -> JENKINS_HTTPS_PORT="8089"
  ```

  위 내용으로 변경이 되지 않는다면 해당 설정보다 우선되는 설정을 변경해야 한다. <br>
  아래 파일에서 Environment의 JENKINS_PORT 번호를 변경하자. <br>

  /usr/lib/systemd/system/jenkins.service

### 방화벽 설정

  ```console
  sudo firewall-cmd --permanent --add-port=8089/tcp
  sudo firewall-cmd --reload
  ```

### 실행 및 테스트

  ```console
  systemctl enable jenkins
  systemctl start jenkins
  ```

  > http://[서버ip]:8089 로 접속 가능하며, <br>
  /var/lib/jenkins/secrets/initialAdminPassword 경로에 초기 비밀번호가 존재