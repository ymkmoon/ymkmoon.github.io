---
title:  "[Linux] CentOS7 Jenkins Git Repo 배포"
excerpt: "Jenkins 에 Gitgub Repository 배포하기"

tags:
  - [CentOS7, Jenkins, Git]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #11 Jenkins Deploy]"
 
date: 2022-04-21
last_modified_at: 2022-04-21
---

## Jenkins를 통한 프로젝트 배포
> https://github.com/ymkmoon/toyseven <br>
> Spring boot 프로젝트를 jar 형태로 배포 작업을 진행 <br>
> 해당 프로젝트는 gradle 7.2, jdk 1.8 을 사용

<br><br>

> 진행 전에 앞서, 프로젝트의 build.gradle 파일의 최하단에 아래 내용을 작성

```console
bootJar {
	mainClassName = 'com.toyseven.ymk.ToysevenApplication'
}

bootRun {
	String activeProfile = System.properties['spring.profiles.active']
	systemProperty "spring.profiles.active", activeProfile
}
```

<br>



### 플러그인 설치

![CentOS7](/assets/image/linux/Centos_jenkins_01.PNG)

  > http://[서버ip]:port 로 젠킨스 웹 페이지에 접속한다. <br>
  > 사이드메뉴에서 ***[Jenkins 관리]*** -> ***[플러그인 관리]*** 선택 <br>
  > 플러그인에서 ***[Post build task]*** 를 검색하여 설치 합니다.

<br>
<br>

### Global Tool 설정

![CentOS7](/assets/image/linux/Centos_jenkins_02.PNG)

  > 사이드메뉴에서 ***[Jenkins 관리]*** -> ***[Global Tool Configuration]*** 선택 <br>

<br>
<br>


![CentOS7](/assets/image/linux/Centos_jenkins_03.PNG)

  > JDK 부분에 앞서 java 설치 후 환경변수에 등록했던 JAVA_HOME 의 내용을 입력 <br>
  > ***[Add JDK]*** 선택 <br>
  > Name : JAVA_HOME<br>
  > JAVA_HOME : /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-1.el7_9.x86_64 <br>
  > 입력 후 Apply 선택

<br>
<br>

![CentOS7](/assets/image/linux/Centos_jenkins_04.PNG)

  > Gradle 부분에 앞서 설치 후 환경변수에 등록했던 GRADLE_HOME 의 내용을 입력 <br>
  > ***[Add Gradle]*** 선택 <br>
  > Name : GRADLE_HOME<br>
  > GRADLE_HOME : /opt/gradle/gradle-7.2 <br>
  > 입력 후 Apply 선택

  > 마지막으로 save 를 눌러 저장

<br>
<br>

### 프로젝트 등록

![CentOS7](/assets/image/linux/Centos_jenkins_05.PNG)

![CentOS7](/assets/image/linux/Centos_jenkins_06.PNG)

  > Dashboard 에서 ***[새로운 item]*** 선택 <br>
  > ***[item name]*** 을 입력하고, ***[Freestyle project]*** 선택 후 ***[OK]*** <br>

<br><br>

![CentOS7](/assets/image/linux/Centos_jenkins_07.PNG)

  > ***[General]*** 의 설명 란에는 프로젝트에 대한 설명을 기입. <br>
  > 아래의 선택에서는 GitHub project 를 체크 한 뒤 Git Repository url 을 입력 합니다. <br>
  > 설명 : https://github.com/ymkmoon/toyseven <br>
          Springboot-gradle-JPA-JWT-InmemoryDB <br>
  > Project Url : https://github.com/ymkmoon/toyseven/

<br><br>

![CentOS7](/assets/image/linux/Centos_jenkins_08.PNG)
  
  > ***[소스코드 관리]*** 에서는 **[Git]** 을 선택하고, Repository URL 을 입력 <br>
  > ***[Branches to build -> Branch Specifier (blank for'any')]*** 에는 어떤 브랜치를 배포 할 것인지 입력합니다. <br>

  > Repository URL : https://github.com/ymkmoon/toyseven.git <br>
  > Branch Specifier (blank for 'any') : */develop <br>

<br><br>

![CentOS7](/assets/image/linux/Centos_jenkins_09.PNG)

  > ***[빌드유발]*** 에서는 **[Build periodically]** 를 선택 <br>
  > **[Schedule]** 란에 30 10,14 * * * 를 입력  <br>
  > (빌드는 30분에 진행되고 10시 14시 마다 진행됨
			-> 즉 10시 30분, 14시 30분에 주기적으로 빌드진행) <br>

<br><br>

![CentOS7](/assets/image/linux/Centos_jenkins_10.PNG)

![CentOS7](/assets/image/linux/Centos_jenkins_11.PNG)

  > ***[Build]***  에서는 **Invoke Gradle script** 를 선택 <br>
  > Gradle Version : GRADLE_HOME <br>
  > Tasks : bootJar <br>
  > 위 내용을 입력

<br><br>

![CentOS7](/assets/image/linux/Centos_jenkins_12.PNG)

![CentOS7](/assets/image/linux/Centos_jenkins_13.PNG)

  > ***[빌드 후 조치]*** 에서는 **Post build tast** 선택 <br>
  > Log text : BUILD SUCCESS (원하는 로그 메세지를 지정) <br>
  > Script : nohup java -jar -DSpring.profiles.active=dev /var/lib/jenkins/workspace/toyseven/build/libs/toyseven-0.0.1-SNAPSHOT.jar & <br>
  > (프로젝트 deploy 시 생성되는 jar 파일 명을 입력해야 한다.) <br>
  > Run script only if all previous steps were successful : 체크 <br>


<br><br>

### 빌드 및 배포

![CentOS7](/assets/image/linux/Centos_jenkins_14.PNG)


![CentOS7](/assets/image/linux/Centos_jenkins_15.PNG)


  > 성공적으로 빌드가 마무리 되고, 배포가 완료 

<br><br>


![CentOS7](/assets/image/linux/Centos_jenkins_16.PNG)

  > Postman 을 이용하여 성공적으로 api 접근이 가능한 걸 확인 할 수 있다. <br>
  > 만약 api 접근이 안된다면, 임베디드 톰켓이 이용하는 port 번호에 대한 방화벽을 해제해 줄 필요가 있다 <br>
  

<br><br>




