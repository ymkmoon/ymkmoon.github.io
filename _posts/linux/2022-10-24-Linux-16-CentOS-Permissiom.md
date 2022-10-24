---
title:  "[Linux] CentOS7 계정에 sudo 권한 부여하기"
excerpt: "CentOS7 sudo 권한 설정"

tags:
  - [CentOS7, Linux]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #16 Sudo]"
 
date: 2022-10-24
last_modified_at: 2022-10-24
---

### Sudo 권한 추가하기

처음 OS 를 설치하면서 생성 한 계정으로 sudo 명령어를 사용하면 아래와 같은 메세지가 보일 수 있다.

```
계정 is not in the sudoers file. This incident will be reported.
```

![CentOS7](/assets/image/linux/Centos_Permission_01.PNG)

해결 하기 위해서는, root 계정으로 접속하여 해당 계정에 권한을 부여해야 하며 방법은 아래와 같다.


```console
su -                  # root 계정 접속
chmod +w /etc/sudoers # sudoers 파일의 수정 권한 부여
vi /etc/sudoers       # 계정에 권한을 부여하기 위해 sudoers 파일 편집


계정  ALL=(=ALL)  ALL # 원하는 계정에 권한을 부여한다.

chmod -w /etc/sudoers # sudoers 파일의 수정 권한 회수
```



![CentOS7](/assets/image/linux/Centos_Permission_02.PNG)


위 내용을 추가하고 저장한다.

이제 성공적으로 sudo 명령어 사용이 가능하다.