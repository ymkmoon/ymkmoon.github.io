---
title:  "[Linux] CentOS7 Apache와 Tomcat 연동"
excerpt: "리눅스 환경에 Apache와 Tomcat 연동하기"

tags:
  - [CentOS7, Linux]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #6 Apache Tomcat]"
 
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

## CentOS7 에 Apache와 Tomcat 연동하기
- Apache 와 Tomcat 연동

- 참고사이트
  - [블로그](https://velog.io/@nari120/2-tier-WEB-Apach2.4-Tomcat8-%EC%97%B0%EB%8F%99){:target="_blank"}
  - [블로그](https://hoing.io/archives/178){:target="_blank"}

<br>


### perl 다운로드

  ```console
  cd /usr/local/download
  sudo wget https://www.cpan.org/src/5.0/perl-5.34.0.tar.gz --no-check-certificate
  sudo tar xvfz perl-5.34.0.tar.gz
  
  cd perl-5.34.0
  sudo ./Configure -des -Dprefix=/usr/local/perl
  sudo make
  sudo make install

  perl -v 
  ```

### 커넥터 다운로드
  
  ```console
  cd /usr/local/download

  sudo wget http://apache.mirror.cdnetworks.com/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.48-src.tar.gz
  sudo tar -zxvf tomcat-connectors-1.2.48-src.tar.gz
  sudo mv tomcat-connectors-1.2.48-src /usr/local/tomcat-connectors
  
  cd /usr/local/tomcat-connectors/native
  sudo ./configure --with-apxs=/usr/local/apache2/bin/apxs
  sudo make
  sudo make install

  cd /usr/local/apache2/modules
  ls
  ```

![CentOS7](/assets/image/linux/Centos_connect_apache_tomcat_01.PNG)

잘 설치 되었다면, 위 처럼 mod_jk.so 이 확인 가능하다
<br><br>

### 연동 설정

  ```console
  vi /usr/local/apache/conf/httpd.conf

  ...
  # LoadModule 구문 가장 아래에. 아래 문장을 추가. 약 153번 째 줄.
  LoadModule jk_module modules/mod_jk.so

  ...
  # 현재 문서(httpd.conf) 제일 아래에 아래 문장들을 추가
  <IfModule jk_module>
   	Include conf/mod_jk.conf
  </IfModule>
  ```


  
  - mod_jk.conf 파일 생성

    ```console
    sudo vi /usr/local/apache2/conf/mod_jk.conf


    <IfModule mod_jk.c>
      # workers.properties 파일 위치 지정
      # worker = 들어온 요청을 처리하기 위한 톰캣 인스턴스
      JkWorkersFile conf/workers.properties

      # mod_jk 로그파일 위치 지정
      JkLogFile logs/mod_jk.log

      # 로그 레벨 지정 [debug / error / info]
      JkLogLevel info

      # 로그에 출력되는 시간 포맷을 지정
      # %a :: 요일
      # %b :: 월
      # %d :: 일
      # %H :: 시간
      # %M :: 분
      # %S :: 초
      # %Y :: 연도
      JkLogStampFormat "[%a %b %d %H:%M:%S %Y] "

      # 아파치로부터 들어오는 요청에 대해 URL 패턴별로 workers.properties에
      # 정의한 worker에게 매핑시키는 설정을 지정
      JkMountFile conf/uriworkermap.properties
    </IfModule>
    ```

  - workers.properties 파일 생성

      ```console
      sudo vi /usr/local/apache2/conf/workers.properties


      # list는 mod_jk가 사용할 모든 작업자리스트 기술(콤마로 구분)
      # type은 톰캣과 통신할 프로토콜을 정의(ajp12, ajp13, ajp14, jni, lb) 
      #        ajp13프로토콜이 가장 많이 사용되며, lb는 로드밸런싱에 사용.
      # host는 대상 톰캣의 IP
      # port는 톰캣과 통신할 포트
      worker.list=worker1
      worker.worker1.type=ajp13
      worker.worker1.host=127.0.0.1
      worker.worker1.port=8009
      ```

  - uriworkermap.properties 파일 생성

      ```console
      sudo vi /usr/local/apache2/conf/uriworkermap.propertie

      # /docs 경로가 들어오면 workers.properties에 정의한
      # worker1 톰캣인스턴스로 매핑
      # 설치된 톰캣의 도큐먼트 페이지로 이동
      /*.jsp=worker1
      /*.css=worker1
      /*.svg=worker1
      /*.png=worker1
      /*.ico=worker1
      ```

### Was 설정

  ```console
  sudo vi /usr/local/tomcat9/conf/server.xml


  - 아래는 Default 로 설정된 내역이며 주석으로 되어 있습니다.
  - 이 부분은 그대로 두시고 아래 쪽에 별도로 다시 내용을 추가 하도록 겠습니다.
  <!-- Define an AJP 1.3 Connector on port 8009 -->
  <!--
  <Connector protocol="AJP/1.3"
  address="::1"
  port="8009"
  redirectPort="8443" />
  -->


  <Connector address="0.0.0.0" port="8009" maxHttpHeaderSize="8192" tomcatAuthentication="false"
  maxThreads="150" minSpareThreads="25" maxSpareThreads="75" secretRequired="false"
  enableLookups="false" acceptCount="100" redirectPort="8443"
  connectionTimeout="20000" disableUploadTimeout="true" protocol="AJP/1.3" />
  ```

### 연동 확인
  > Apache와 Tomcat을 재시작 해준다.

  ```console
  systemctl restart apache
  systemctl restart tomcat9
  ```

  > http://IP_ADDRESS/index.jsp 로 접속 했을때, 톰캣화면(고양이)가 뜨면 완료.

![CentOS7](/assets/image/linux/Centos_connect_apache_tomcat_02.PNG)