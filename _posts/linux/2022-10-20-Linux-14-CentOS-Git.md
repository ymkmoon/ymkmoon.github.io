---
title:  "[Linux] CentOS7 Git 설치 및 Repository clone 하기"
excerpt: "리눅스 환경에 깃 설치와 레파지토리 복제하기"

tags:
  - [CentOS7, Linux, Git]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #14 Git]"
 
date: 2022-10-20
last_modified_at: 2022-10-20
---

### Git 설치
  
  - Git 설치

    ```console
    yum -y install git
    ```

  - Git 초기 설정

    ```console
    git config --global user.name "사용자명"
    git config --global user.email "메일주소"
    ```

    설정 내용은 사용자 홈 폴더의 .gitconfig 파일에 저장된다.
    직접 파일 수정을 통해서 변경 할 수 있다.
    (--global 옵션을 제거해서 git 저장소마다 다른 설정도 가능)

  - Clone
    ```console
    cd /usr/workspace ## 클론 할 위치로 이동
    git clone 깃허브URL
    ```
  
  - 브랜치 변경
    ```conole
    git checkout 변경할브랜치이름
    ```
  
