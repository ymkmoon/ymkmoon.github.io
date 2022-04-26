---
title:  "CentOS7 Jenkins 설치"
excerpt: "리눅스 환경에 Jenkins 설치하기"

tags:
  - [Linux, CentOS7, VMWare, Jenkins]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #9 Jenkins]"
 
date: 2021-03-25
last_modified_at: 2021-03-25
---


## Jenkins 란?
> 젠킨스는 소프트웨어 개발 시 지속적 통합 서비스를 제공하는 툴이다. <br>
 다수의 개발자들이 하나의 프로그램을 개발할 때 버전 충돌을 방지하기 위해 각자 작업한 내용을 공유 영역에 있는 Git등의 저장소에 빈번히 업로드함으로써 지속적 통합이 가능하도록 해 준다.


### 1. Jenkins 설치

  ```
  sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo --no-check-certificate
  sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
  sudo yum install jenkins
  ```

### 2. 포트 변경

  ```
  sudo vi /etc/sysconfig/jenkins

  # 위 파일에서 아래내용 변경
  JENKINS_HTTPS_PORT="8080" 
  -> JENKINS_HTTPS_PORT="8089"
  ```

### 3. 방화벽 설정

  ```
  sudo firewall-cmd --permanent --add-port=8089/tcp
  sudo firewall-cmd --reload
  ```

### 4. 실행 및 테스트

  ```
  systemctl enable jenkins
  systemctl start jenkins
  ```

  > http://[서버ip]:8089 로 접속 가능하며, <br>
  /var/lib/jenkins/secrets/initialAdminPassword 경로에 초기 비밀번호가 존재