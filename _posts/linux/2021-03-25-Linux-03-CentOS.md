---
title:  "[Linux] 가상머신에 리눅스 환경 구축하기"
excerpt: "가상머신에 CentOS7 설치"

tags:
  - [CentOS7, Linux]

toc: true
toc_sticky: true
toc_label: "[CentOS #1 install]"
 
date: 2021-03-25
last_modified_at: 2021-03-25
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

## CentOS 이미지 파일 다운로드
  > https://www.centos.org/download/

- VMWare에 CentOS7 환경을 만들기 위해서는 ISO 파일이 필요하다.   
위 링크에서 CentOS7 의 ISO 파일을 다운받자.
- 작성자는 x86_64 선택 후 `CentOS-7-x86_64-DVD-2009.iso` 파일 설치
<br>


### VMWare Worksation 시작하기.
![VMWare](/assets/image/linux/vmware_top.PNG)
  > 빨간 네모 부분을 선택하여 가상환경을 생성합니다.

![VMWare](/assets/image/linux/vmware_Centos_install01.PNG)
  > 위에서 설치한 iso 파일을 선택해줍니다.

![VMWare](/assets/image/linux/vmware_Centos_install02.PNG)
  > 가상환경의 이름과 경로를 설정합니다.   
  이후 나오는 창에서는 그냥 Next 를 선택하며 지나갑니다.


- 위 작업이 끝나면 VMWare Workstation 에 방금 설치한 CentOS 환경이 생성됩니다.

  
<br>   

### CentOS 설치하기
- 위에서 설치한 환경을 더블클릭하고 기다리면 아래의 화면이 나옵니다.

![VMWare](/assets/image/linux/vmware_Centos_install03.PNG)
  > 설치 진행을 어떤 언어로 사용할지 선택

![VMWare](/assets/image/linux/vmware_Centos_install04.PNG)
  > 위 이미지에서 보이는 3개만 설정하고 나머지는 그냥 넘어가셔도 좋습니다.   
  파티션은 자동선택 하셔도, 커스텀 하셔도 됩니다. (본인은 자동선택)

![VMWare](/assets/image/linux/vmware_Centos_install05.PNG)
  > Root 계정 생성 대신 일반 계정을 생성하고, 관리자로 사용한다는 부분을 체크.


![VMWare](/assets/image/linux/vmware_Centos_install06.PNG)
  > 위 내용까지 진행하면 정상적으로 CentOS 환경이 구축됩니다.   
  다음 포스트 부터는 프로젝트 배포를 위한 설치 및 설정이 진행됩니다.



