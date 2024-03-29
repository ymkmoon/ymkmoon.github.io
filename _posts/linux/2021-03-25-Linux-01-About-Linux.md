---
title:  "[Linux] 리눅스에 대한 내용과 기본명령어"
excerpt: "리눅스에 대해 알아보자"

tags:
  - [Linux]

toc: true
toc_sticky: true
toc_label: "[리눅스 란]"
 
date: 2021-03-25
last_modified_at: 2021-03-25
---

## 리눅스란 무엇인가
  > Window 와 같이 운영체제의 한 종류이다.

- 장점 : 자유 라이센스(무료), 호환성이 좋다.   
- 단점 : 시스템 장치의 호환성 여부와 인터페이스 환경에서의 편의성  
<br>


## 리눅스 배포판

  > 가장 대중적인 RedHat, Ubuntu, Debian 이 있고, 앞으로 진행 할 예제는 `RedHat`의 `CentOS7` 으로 진행된다.

  
<br>   

### RedHat 과 CentOS 차이는?
- 둘은 동일한 기능과 성능을 제공하지만 RedHat 유료, CentOS 는 무료 이다.
- RedHat은 유료인 만큼 사용할 경우 레드햇사의 기술지원을 받을 수 있다.
  
<br>

### 기본 명령어
#### 서버의 환경정보 확인

```console
OS Bit확인 -> getconf LONG_BIT
OS 계열확인 -> cat /etc/issue
상세 OS확인 -> cat /proc/version
CPU 정보 확인 -> cat /proc/cpuinfo
RAM 용량 확인 ->	cat /proc/meminfo
DISK 용량 확인 -> df -h
```

#### 방화벽

```console
포트확인 -> netstat -na / netstat -nap / netstat -nat
열린 포트 확인 -> netstat -na | grep LISTEN
SELinux 끄기 -> sudo setenforce 0		
iptables 규칙 만드는 순서 -> https://webdir.tistory.com/170
특정 ip / port 허용 및 차단 -> http://blog.plura.io/?p=4519
```

#### 네트워크 활성화

```console
연결상태 확인 -> ping 8.8.8.8
사용가능 네트워크 확인 -> ip addr / ifconfig -a
네트워크 연결 -> centOS 6버젼 : ifup eth0 
                centOS 7버젼 : ifup ens33
현재 네트워크 설정 확인 
    -> /etc/sysconfig/network-scripts/ifcfg-ens33 
    경로의 파일을 열면 확인 가능  
```

```console
BOOTPROTO=none     # 고정 IP 설정을 위해 none으로 설정
. . .
IPV6INIT=no        # IPV6 관련해서는 no로 설정 합니다.
IPV6_AUTOCONF=no
IPV6_DEFROUTE=no
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
. . .
ONBOOT=yes # 부팅/네트워크 재시작 시  자동으로 인터페이스 활성화
IPV6_PRIVACY=no
IPADDR=192.168.122.243              # IP 설정
NETMASK=255.255.255.0               # 서브넷 마스크 설정, prefix=24 설정으로 대체할 수도 있습니다.
GATEWAY=192.168.122.1               # 게이트웨이 설정 
DNS1=8.8.8.8   # 1차 네임서버 설정, /etc/resolv.conf 설정파일에서 정의하셔도 됩니다.
DNS2=8.8.4.4                        # 2차 네임서버 설정
```

  > 해당 설정을 바꾸게 되면 반영을 위해 네트워크 재시작이 필요
  
- systemctl restart network 입력하여 네트워크 재시작.

#### 언어 확인 및 변경

```console
언어셋변경 : localectl set-locale LANG=ko_KR.UTF-8
언어셋확인 : locale
```







