---
title:  "[Linux] CentOS7 SSH 설정"
excerpt: "리눅스 환경에 SSH 설정하기"

tags:
  - [CentOS7, SSH]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #3 SSH]"
 
date: 2022-04-18
last_modified_at: 2022-04-18
---

## CentOS7 에 SSH 설정하기
- 앞으로의 진행은 직접 서버에 로그인 하지 않고, ssh 를 이용하여 진행하겠습니다.

<br>


### SSH 란?
> 시큐어 셸(Secure SHell, SSH)은 네트워크 상의 다른 컴퓨터에 로그인하거나 원격 시스템에서 명령을 실행하고 다른 시스템으로 파일을 복사할 수 있도록 해 주는 응용 프로그램 또는 그 프로토콜을 가리킨다

<br>

### SSH 설정 변경
  - ssh config 파일을 연다.

    ```console
    sudo vi /etc/ssh/sshd_config
    ```

  - port 번호를 설정한다. (본인은 17라인에 주석처리 되어 있는 내용을 주석해제함.)
    
    ```console
    #Port 22
    Port22
    ```

  - Root Login 을 막는다. (본인은 38라인에서 주석처리 되어 있는 내용을 변경.)
    
    ```console
    PermitRootLogin yes
    PermitRootLogin no
    ```
  

### 방화벽 추가하기
  - 22번 포트를 열고, 반영을 위해 firewall 를 재기동.
    
    ```console
    sudo firewall-cmd --permanent --zone=public --add-port=22/tcp
    sudo firewall-cmd --reload
    ```
  

### ping 에 응답하지 않도록 설정하기

  ```console
  sudo vi /etc/sysctl.conf
  ```

  ```console
  net.ipv4.icmp_echo_ignore_all=1
  ```
  위 내용을 작성 후 저장한다.

### 재부팅 후 적용하기

  ```console
  systemctl restart sshd
  shutdown -r now
  ```

### SSH 로 접속하기
  - ssh 툴을 설치하여 CLI 환경에 접속한다. (본인은 poderosa 이용)

  ![CentOS7](/assets/image/linux/Centos_setting_ssh_01.PNG)
  ![CentOS7](/assets/image/linux/Centos_setting_ssh_02.PNG)

  

    






