---
title:  "VMWare 프리징현상 해결하기(동작안함, 자꾸죽음)"
excerpt: "VMWare (VMWorkStation) 와 WSL 충돌문제 해결, Docker Desktop 설치 시 주의사항"

tags:
  - [VMWare, WSL]

toc: true
toc_sticky: true
toc_label: "[VMWare 프리징]"
 
date: 2022-10-25
last_modified_at: 2022-10-25
---

### VMWare 프리징

> 충동 해결 후에도 WSL2 를 계속 사용해야 한다면 다른 방법을 찾으셔야 합니다.


기존 VMWare 를 잘 사용하고 있었는데, Docker Desktop 을 설치하며 WSL2 을 함께 설치했더니 그 이후부터 VMWare 가 프리징이 걸리며 멈추는 현상이 생겼다.

알아보니 windows 10 에서 Docker 와 VMWare 의 동시사용에 대해 많은 이슈가 보였다.

구글링을 하며 해결방법을 알아보니, `windows 기능 켜기/끄기` 를 열어 Htper-V 를 체크해제 후 재부팅 하라고 하는데 해결되지 않아 헤메던 중 해결 방법을 발견했다.



> cmd 창을 관리자 권한으로 실행 후 아래 명령어를 입력한다.


- dism.exe /Online /Disable-Feature:Microsoft-Hyper-V
- bcdedit /set hypervisorlaunchtype off


처음 명령어를 입력 후 재부팅을 묻는다면 두번째 명령어까지 실행 후 재부팅을 해야하기 때문에 N 을 입력한다.

두 명령어를 실행 후 재부팅을 하게 되면 VMWare 의 프리징 현상이 해결된다.

