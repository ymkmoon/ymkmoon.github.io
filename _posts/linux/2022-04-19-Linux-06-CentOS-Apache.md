---
title:  "[Linux] CentOS7 Apache 설치"
excerpt: "리눅스 환경에 Apche 설치하기"

tags:
  - [CentOS7, Linux]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #4 Apache]"
 
date: 2022-04-19
last_modified_at: 2022-04-19
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

## CentOS7 에 Apache 설치하기
- Apache 를 설치 후 tomcat 과 연동을 진행합니다.

<br>


### Apache 설치하기
  > 필수 툴 설치

  ```console
  sudo yum install gcc gcc-c++ expat-devel.x86_64 pcre-devel
  ```

### pcre 다운로드 및 컴파일
  
  ```console
  sudo mkdir /usr/local/download
  cd /usr/local/download

  sudo wget https://sourceforge.net/projects/pcre/files/pcre/8.44/pcre-8.44.tar.gz --no-check-certificate
  sudo tar zxvf pcre-8.44.tar.gz 
  cd pcre-8.44
  sudo ./configure --prefix=/usr/local
  sudo make
  sudo make install
  ```

### apr-util, apr, httpd 다운로드 및 컴파일
  - 만약 404가 발생한다면 버전이 변경됐을 경우가 있으니, 아래의 페이지에 접속해서 존재하는 버전으로 변경해준다.

  > http://mirror.apache-kr.org/httpd
  <br>
  > http://mirror.apache-kr.org/apr

  ```console
  cd /usr/local/download
  sudo wget http://mirror.apache-kr.org/httpd/httpd-2.4.53.tar.gz
  sudo wget http://mirror.apache-kr.org/apr/apr-1.7.0.tar.gz
  sudo wget http://mirror.apache-kr.org/apr/apr-util-1.6.1.tar.gz

  sudo tar zxvf apr-1.7.0.tar.gz
  sudo tar zxvf apr-util-1.6.1.tar.gz
  sudo tar zxvf httpd-2.4.53.tar.gz

  sudo mv apr-1.7.0 ./httpd-2.4.53/srclib/apr
  sudo mv apr-util-1.6.1 ./httpd-2.4.53/srclib/apr-util

  cd /usr/local/download/httpd-2.4.53
  sudo ./configure --prefix=/usr/local/apache2
  sudo make
  sudo make install
  ```


### httpd.conf의 ServerName 수정

  ```console
  sudo vi /usr/local/apache2/conf/httpd.conf # 파일의 193번째 라인
  ```
  
  ```console
  ServerName localhost:80
  ```

### apache 실행 및 확인
  
  ```console
  cd /usr/local/apache2/bin
  ./apachectl start
  ps -ef|grep httpd
  ```

![CentOS7](/assets/image/linux/Centos_install_apache_01.PNG)



### apache 테스트
  - 내부작동 테스트

    ```console
    cd /usr/local
    sudo wget http://127.0.0.1
    ```

![CentOS7](/assets/image/linux/Centos_install_apache_02.PNG)

    > /usr/local/apache2/htdocs 경로에 위치한 index.html이 다운로드 된다.


  - 외부접근 테스트

    > 방화벽이 실행되고 있고, 80포트에 대한 허가가 없다면 외부에서 접근할 수 없다.

    ```console
    # 방화벽 실행 확인
    sudo firewall-cmd --state
    -> running
    
    # 현재 사용중인 Zone 확인
    sudo firewall-cmd --get-default-zone
    -> public
    
    # 80포트 허용
    sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
    -> success
    
    # 방화벽 재시작
    sudo firewall-cmd --reload
    -> success
    
    # 방화벽 적용 확인
    sudo firewall-cmd --zone=public --list-all
    ```

    > 외부에서 접근이 되는지 테스트해보려면 브라우저에 IP 또는 도메인을 입력하면 된다. 


### httpd service 를 등록 (init.d, systemctl)
  아래중 한가지를 선택해서 진행하면 된다.

  - /etc/init.d (CentOS 6에서 주로 쓰임)

    ```console
    cd /etc/init.d
    cp /usr/local/apache2/bin/apachectl ./httpd
    vi httpd
    ```

    ```console
    맨 위에 다음과 같이 입력한다.

    #!/bin/sh 

    # httpd Startup script for the Apache HTTP Server 
    # chkconfig: 345 85 15 
    # description: Apache is a World Wide Web server. It is used to serve 
    # processname: httpd 
    # pidfile: /usr/local/apache2/logs/httpd.pid

    ...
    ```

    저장 후 서비에스 등록한다

    ```console
    chkconfig --add httpd
    chkconfig --level 345 httpd on
    chkconfig --list
    ```

    ```console
    # service 명령어 테스트
    service httpd stop
    service httpd start
    ```

  - systemctl (CentOS 7에서 주로 쓰임)

    ```console
    cd /usr/lib/systemd/system
    sudo vi apache.service
    ```

    apache.service 파일에 아래와 같이 입력 후 저장.
    
    ```console
    [Unit] 
    Description=apache2 Service 
    After=syslog.target 
    After=network.target 

    [Service] 
    Type=forking 
    ExecStart=/usr/local/apache2/bin/apachectl start 
    ExecStop=/usr/local/apache2/bin/apachectl graceful-stop 
    ExecReload=/usr/local/apache2/bin/apachectl graceful 

    User=root
    Group=root

    [Install] 
    WantedBy=multi-user.target
    ```

    ```console
    # 데몬 재시작
    systemctl daemon-reload

    # linux 재시작 시 apache 자동 실행 등록
    systemctl enable apache
    ```

