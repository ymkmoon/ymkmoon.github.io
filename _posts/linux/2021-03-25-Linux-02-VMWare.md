---
title:  "[Linux] 가상머신(Virtual machine) 설치"
excerpt: "윈도우 환경에 리눅스 서버를 구축해보자."

tags:
  - [Linux, VMWare]

toc: true
toc_sticky: true
toc_label: "[가상머신 이란]"
 
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

## 가상머신 이란 무엇인가
  > 물리적 하드웨어를 논리적으로 나누어 사용하는것이다.   


  > 스타크래프트에서 인구수의 맥시멈은 200 이다.   
  프로토스는 다크아칸으로  다른 종족의 일꾼을 흡수해서 각 종족의 유닛과 건물을 지을 수 있게되지만 인구수는 동일하게 200으로 변하지 않는다.  

  > 가상머신은 100이라는 성능(인구수) 안에서 이를 쪼개어 여러 환경(종족) 을 사용 할 수 있게 해준다.

- 사용 하는 이유 : 윈도우 환경에서 가상머신을 이용하면 IOS 환경과 리눅스 환경 설정 연습이 가능하다.
- 장점 : 자유 라이센스(무료), 호환성이 좋다.   
- 단점 : 시스템 장치의 호환성 여부와 인터페이스 환경에서의 편의성  
<br>


### VMWare Worksation 설치하기.
  > https://www.vmware.com/kr.html

- VMWare 공식 홈페이지 접속 후 다운로드 -> 무료 다운로드 -> Workstation Player 를 설치하자.  
<br>   

### Player Install
- 라이센스, 제품 경로 변경 등 개인에 맞게 세팅 후 계속 진행.
- User Experience 설정에서 모두 체크 해제 후 Next   
(사용 통계 정보등을 VMWare 측에 보낼지에 대한 여부등에 대한 내용이 있다.)
- Shortcuts 에서 모두 체크 후 Next   
(바탕화면과 시작메뉴에 바로가기 생성 여부등에 대한 내용이 있다)
  
<br>


> 이번 포스트 에서는 가상머신 설치까지만 진행하고   
다음 포스트에서 본격적으로 가상머신에 Linux 를 구축하겠습니다.


