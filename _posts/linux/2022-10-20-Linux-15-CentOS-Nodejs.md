---
title:  "[Linux] CentOS7 NodeJS 설치"
excerpt: "NodeJS 설치와 no package nodejs available 해결방법"

tags:
  - [CentOS7, Linux, NodeJS]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #15 NodeJS]"
 
date: 2022-10-20
last_modified_at: 2022-10-20
---

### Node & NPM 설치
  
  - Node 설치

    ```console
    yum install -y nodejs
    ```

    만약 노드 설치 시 `no package nodejs available` 라는 문구가 나오며 설치가 제대로 되지 않는 다면 curl 로 yum 저장소에 node 설치가 되어있지 않다는 의미이다. 아래 명령어를 실행하자

    ```console
    yum install -y epel-release
    ```

  - Npm 설치

    Node 를 설치하면 같이 install 된다고 하는데, 나는 npm 설치가 되지 않아 따로 설치했다.

    ```console
    yum install npm
    ```

    만약 인스톨이 되지 않는다면 아래 명령어를 실행 후 인스톨 해보자

    ```console
    yum update
    ```

  
  
