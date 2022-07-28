---
title:  "[Linux] CentOS7 Tomcat 설치"
excerpt: "리눅스 환경에 Tomcat 설치하기"

tags:
  - [CentOS7, Linux]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #5 Tomcat]"
 
date: 2022-04-19
last_modified_at: 2022-04-19
---

## CentOS7 에 Tomcat 설치하기
- Tomcat 을 설치 후 apache 와 연동을 진행합니다.

<br>


### Tomcat 설치하기
  > https://tomcat.apache.org/ 에서 원하는 버젼의 tar.gz 링크주소 복사 후 작성 

  ```console
  cd /usr/local/download
  sudo wget https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.62/bin/apache-tomcat-9.0.62.tar.gz --no-check-certificate
  ```

### 압축 해제 및 폴더 이동

  ```console
  cd /usr/local/download
  sudo tar xvfz apache-tomcat-9.0.62.tar.gz
  sudo mv apache-tomcat-9.0.62 /usr/local/tomcat9
  ```

### Tomcat 설정

  ```console
  sudo vi /usr/local/tomcat9/conf/server.xml

  # 에서 아래의 내용을 찾아 URIEncoding="UTF-8"을 추가
		
  <Connector port="8080" protocol="HTTP/1.1"
              connectionTimeout="20000"
              redirectPort="8443"
              URIEncoding="UTF-8" />
  ```

### 방화벽 설정

  ```console
  firewall-cmd --permanent --add-port=8080/tcp
  firewall-cmd --reload
  ```

### 환경변수 설정
  > 환경변수 CATALINA_HOME 을 추가하고 CLASSPATH 의 변수에도 아래 내용을 추가해준다.

  ```console
  sudo vi /etc/profile

  JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-1.el7_9.x86_64
  CATALINA_HOME=/usr/local/tomcat9 # 추가
  PATH=$PATH:$JAVA_HOME/bin
  CLASSPATH=$JAVA_HOME/jre/lib:$JAVA_HOME/lib/tools.jar:$CATALINA_HOME/lib-jsp-api.jar:$CATALINA_HOME/lib/servlet-api.jar # 내용 추가

  source /etc/profile
  ```

### Tomcat 실행

  ```console
  sudo /usr/local/tomcat9/bin/startup.sh
  ```

![CentOS7](/assets/image/linux/Centos_install_tomcat_01.PNG)

  ```console
  # 톰캣 프로세스 확인
  ps -ef|grep tomcat9

  # 테스트
  sudo wget http://localhost:8080/
  # 위 명령어 입력시 index.html이 다운로드 되면 tomcat이 정상적으로 실행되는 것
  ```

### 서비스 등록
  - tomcat9.service 파일 생성 후 아래 내용을 입력 후 저장한다.

  ```console
  vi /usr/lib/systemd/system/tomcat9.service
  ```

  ```console
  [Unit]
  Description=Apache Tomcat Web Application Container
  After=network.target syslog.target

  [Service]
  Type=forking
  Environment="JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-1.el7_9.x86_64"
  Environment="CATALINA_HOME=/usr/local/tomcat9"
  Environment="CATALINA_BASE=/usr/local/tomcat9"
  Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"
  Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom"
  ExecStart=/usr/local/tomcat9/bin/startup.sh
  ExecStop=/usr/local/tomcat9/bin/shutdown.sh
  User=root
  Group=root

  [Install]
  WantedBy=mult-user.target
  ```

  ```console
  systemctl daemon-reload
  systemctl enable tomcat9
  systemctl start tomcat9

  sudo systemctl enable tomcat9.service # 부팅 시 자동 실행
  systemctl list-unit-files --type service |grep tomcat9 # 등록된 서비스 등록
  ```

### Tomcat 유저 설정

  ```console
  sudo vi /usr/local/tomcat9/conf/tomcat-users.xml
  ```
  
  > 위 파일에서 'tomcat-users' 태그 안에 추가

  ```console
  <role rolename="manager"/>
  <role rolename="manager-gui"/>
  <role rolename="manager-status"/>
  <role rolename="manager-script"/>
  <role rolename="manager-jmx"/>
  <role rolename="admin"/>
  <user name="아이디" password="패스워드" roles="admin,manager,manager-gui,manager-script,manager-jmx,manager-status" />
  ```

### 외부접근 허용 설정
  - 관리파일을 별도로 생성한다.

  ```console
  sudo vi /usr/local/tomcat9/conf/Catalina/localhost/manager.xml
  ```
  
  ```console
  <Context privileged="true" antiResourceLocking="false" docBase="${catalina.home}/webapps/manager">
    <Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="^.*$" />
  </Context>
  ```

  - http://아이피:8080/manager  로 접속하면 아래와 같은 페이지가 노출된다.

![CentOS7](/assets/image/linux/Centos_install_tomcat_02.PNG)