---
title:  "[Linux] CentOS7 FTP 설정"
excerpt: "리눅스 환경에 FTP 설정하기"

tags:
  - [CentOS7, FTP]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #7 FTP]"
 
date: 2021-03-25
last_modified_at: 2021-03-25
---


### 1. FTP 란?
> 파일 전송 프로토콜(File Transfer Protocol)의 약자 이며, 
<br>
> 파일 전송 프로토콜은 TCP/IP 프로토콜을 가지고 서버와 클라이언트 사이의 파일 전송을 하기 위한 프로토콜이다. 파일 전송 프로토콜은 TCP/IP 프로토콜 테이블의 응용 계층에 속한다.
<br>

### 2. FTP 설치
  - FTP 설치 확인

    ```console
    yum list installed | grep vsftpd
    ```
  
  - 설치가 안되어 있다면 설치

    ```console
    yum -y install vsftpd
    ```
  

### 3. FTP 서버 설정
  - vsftpd.conf 파일 수정

    ```console
    sudo vi /etc/vsftpd/vsftpd.conf

    anonymous_enable=YES -> NO 로 수정 
    chroot_list_enable=YES -> 주석처리 
    chroot_list_file=/etc/vsftpd/chroot_list -> 내용추가 
    allow_writeable_chroot=YES -> 내용추가
    use_localtime=YES -> 내용추가
    listen_ipv6=YES -> NO 로 수정
    pasv_enable=YES -> 내용추가
    pasv_min_port=10090 -> 내용추가
    pasv_max_port=10010 -> 내용추가
    listen=NO -> YES 로 수정

    ```

- chroot_list 파일 생성

  ```console
  sudo vi /etc/vsftpd/chroot_list

  # chroot_list 파일에서 허용 할 계정을 입력 후 저장한다.
  ```

### 4. 방화벽 설정
  
  ```console
  firewall-cmd --permanent --add-service=ftp
  firewall-cmd --permanent --add-port=21/tcp
  firewall-cmd --reload
  ```

### 5. 접속 거부 리스트 수정

  ```console
  sudo vi /etc/vsftpd/ftpusers
  sudo vi /etc/vsftpd/user_list

  # 위 2개의 파일에서 내가 사용할 계정을 삭제 후 저장하고 종료
  ```

### 6. FTP 재시작

  ```console
  systemctl enable vsftpd
  systemctl restart vsftpd
  ```

### 7. FTP 접속하기

  > FTP 툴을 설치하여 접속이 잘 되는지 테스트 해본다.
  <br>
  > (본인은 FileZilla 툴을 설치하여 사용)
  
![VMWare](/assets/image/linux/Centos_setting_ftp_01.PNG)

### 8. SELINUX 설정
  - 명령어 중 P 옵션은 영구설정을 의미하고, 1은 on, 0은 off 를 의미한다.

    ```console
    # FTP 서버가 홈 디렉토리에만 접근 가능하도록 하는 옵션
    setsebool -P ftp_home_dir 1
    # 위 명령어 실패 시 setsebool -P ftpd_full_access=1
    
    # ftpd가 Passive 모드 사용을 허가하는 옵션
    setsebool -P ftpd_use_passive_mode 1
    ```


### 9. FTP 서버 관련 파일
  > FTP 서버 관련 파일

  - /etc/vsftpd/vsftpd.conf
    - vsftpd 설정파일 (소스 설치시 /etc/vsftpd.conf)

  - /usr/sbin/vsftpd
    - vsftpd 실행파일 (소스 설치시 /usr/local/sbin/vsftpd)

  - /etc/pam.d/vsftpd
    - vsftpd PAM 인증파일 (소스 설치시에는 생성되지 않으므로 파일을 생성해주어야 한다.)

  - /etc/vsftpd.ftpusers 
    - 로그인 불가능한 유저 파일 (소스 설치시에는 생성되지 않으므로 파일을 생성해주어야 한다.)

  - /etc/vsftpd.user_list
    - 유저의 로그인 허용여부 결정  (소스 설치시에는 생성되지 않으므로 파일을 생성해주어야 한다.)

  - /etc/xinetd.d/vsftpd
    - xinetd 서비스 파일

<br><br>

  > /etc/vsftpd.conf 관련 설정

  ```console
  1) 익명 접속 설정 옵션

  - anonymous_enable = yes         익명로그인허용여부
  - anon_upload_enalbe = no         익명의 사용자 업로드 허용여부
  - anon_mkdir_write_enable = no 쓰기 권한이 있을 시 새로운 디렉토리생성 권한 허용여부
  - deny_email_enable = no         /etc/vsftpd/baned-emails 파일에 존재하는 anonymous@와 같은 형태의 로그인 불허
  - banned_email_file = /etc/vsftpd/baned-emails  로그인을 불허하고자 하는 이메일을 작성해준다.
  - secure_email_list_enable         익명 로그인 유효한 이메일 적용여부 설정 ( 적용시 /etc/vsftpd.email_passwords 파일 참조)
  - email_password_file = /etc/vsftpd.email_passwords 로그인 가능한 이메일 패스워드를 저장하고 있는 파일
  - anon_other_write_enable = no       파일 업로드및 디렉토리 생성 권한 설정
  - anon_world_reasable_only = yes anonymous 권한의 읽기전용 파일 다운로드 허용
  - no_anon_password = no         익명 접속시 password 묻지 않고 로그인 허용
  - anon_max_rate = 0          다운로드 최대 전송율 지정으로 단위는 bps
  - anon_umask = 077
  - anon_root = 경로          익명 접속시 사용할 홈디렉토리 설정
  - ftp_username = ftp          익명 ftp에 사용될 사용자명 지정으로 기본값은 ftp 이다.
  ```

  ```console
  2) 실명 로그인 접속 설정 옵션

  - local_enable = no  /etc/passwd 파일에 명시되어 있는 계정사용 (기본값 no로 익명연결로 허용되어 있다.)
  - wirte_enable = no  데이터 업로드 허용
  - local_umask = 022  
  - dirlist_enable = yes LIST 명령의 허용여부 설정
  - dirmessage_enable = no 사용자가 새로운 디렉토리로 이동하였을 경우 해당 디렉토리에 있는 메세지 파일을 보여준다.(파일명 .message)
  - message_file = .message dirmessage_enable = yes 일 경우 적용되는 옵션
  - download_enable = yes 다운로드 허용여부
  - force_dot_files = no 점으로 시작되는 파일과 디렉토리를 출력되도록 설정
  - guest_enable = no   실제 계정에는 없는 가상계정으로 로그인 허용 (가상계정_패스워드 = /etc/vsftpd_login.db) guest_username과 연동된다.
  - guest_username = ftp guest_enable = yes 일 경우 사용
  - text_userdb_names = no  디렉토리및 파일 목록에서 사용자와 그룹의 ID가 숫자 대신에 텍스트 이름으로 보여지도록 설정 (보안상 활성화금지)
  - userlist_enable = no  userlist_file 명시된 계정들로만 로그인 허용
  - userlist_deny = /etc/vsftpd.user_list     userlist_enable 의 옵션과는 반대로 접속 거부할 계정 명시
  - userlist_file = /etc/vsftpd/user_list       userlist_enable = yes 일 경우 로그인 가능한 유저 목록을 명시
  - file_open_mode = 0666 파일 업로드시 퍼미션 지정
  - listen = no  standalone 동작 설정
  - listen_port = 21  
  - user_config_dir = /etc/vsftpd_user_conf    가상유저마다 각기 다른 vsftpd.conf 설정을 적용한다.
  - virtual_use_local_privs = no       가상사용자 권한설정
  - deny_file   업/다운로드를 금지할 파일형태를 지정한다. (예: deny_file={*.exe,*.pdf,*.mp3})
  - hide_file   숨김디렉토리/파일 지정한다. (예: hide_file={*.exe,*.conf,*.mpeg})
                단, 파일 위치를 알고있다면 직접 다운로드할 수 있다.
  ```

  ```console
  3) 접속제어 설정 옵션

  - one_process_model = no 익명 접속자에게 하나의 프로세스가 작동되도록 한다.
  - max_clients = 100  vsftpd 서버에 접속할 수 있는 최대 클라이언트수 지정
  - max_per_ip = 0  호스트로 접속할 때 최대 접속수 지정 (0값은 무제한을 의미)
  - local_max_rate = 0  파일 전송 최대 속도를 제한하는 옵션
  - idle_session_timeout = 300 세션종료 설정 기본값 300초
  - ftpd_banner = Welcome to FTP server  
  - tcp_wrappers  /etc/allow 와 /etc/deny 파일에서 vsftpd 서버에 접근할 수 있는 IP 지정
  - accept_timeout = 60        수동모드에서 서버에 접속할때까지의 시간 설정
  ```

  ```console
  4) 보안 관련 설정 옵션

  - chroot_local_user = no 로컬경로에 대한 chroot를 적용할 것인지에 대한 설정
  - chroot_list_enable = no 사용자가 로그인시 chroot를 적용할 것인지에 대한 설정
  - chroot_list_file = /etc/vsftpd/chroot-list     chroot를 적용할 사용자계정을 설정
  - chown_uploads = no  익명으로 업로드된 파일에 대해서 지정된 사용자의 소유권으로 설정
  - chown_username = root chown_uploads 에 대한 사용자 지정
  - nopriv_user = nobody 서버를 일반권한으로 작동
  - async_abor_enable = no 파일전송시 취소하였을 경우 취소되지 않는 경우 해결하는데 사용하지만 보안상  no로 설정한다.
  - ascii_download_enable/ ascii_upload_enable = no    파일전송을 ascii 모드로 작동할지 설정
  - ls_recurse_enable = no  ls -R 명령 사용여부 설정 (DoS공격의 취약성으로 사용에 주의한다.)
  - hide_ids = no  디렉토리 목록에서 uid와 gid를 보여주지 않고 모두 ftp로 표시하도록 한다.
  - chmod_enable = yes  실명 접속에서만 사용가능한 명령
  - pam_service_name = ftp PAM을 이용하여 사용자 인증을 할 때 인증파일을 지정하는 옵션
  - check_shell = yes  사용자 로그인시 /etc/shell 파일에서 체크할지 설정
  ```

  ```console
  5) 데이터 연결 관련 설정

  - pasv_enable = yes        수동데이터 연결모드로 지원할지 설정
  - pasv_promiscuous = yes       수동보안체크 기능을 off한다.
  - pasv_max_port / pasv_min_port    수동모드 포트 50000 ~ 60000 포트지정 (기본값은 0)
  - ftp_data_port = 20
  - data_connection_timeout = 300
  - connect_from_port_20 = no  
  - port_enable = yes  
  - connect_timeout = 60
  ```

  ```console
  6) 로그 관련 설정 옵션

  - xferlog_enable = no  파일 송/수신 결과를 xferlog_file 옵션으로 지정된 파일에 기록한다. (/var/log/vsftpd.log)
  - xferlog_file = /var/log/vsftpd.log
  - xferlog_std_format = no 파일 송/수신 로그를 xferlog로 저장할것인지 설정
  - log_tp_protocol = no ftp 요청 및 응답에 관련된 메세지를 기록할것인지의 설정으로 오류분석에 유용하다.
  - syslog_enable = no  /var/log/vsftpd.log에 저장되지않고 /var/log/syslog에 저장되도록한다.
  - dual_log_enable = no /var/log/xferlog 파일과 /var/log/vsftpd.log 파일에 파일전송 기록이 저장된다.
  - vsftpd_log_file = /var/log/vsftpd.log    vsftpd 데몬에 의한 로그메세지를 기록할 파일명을 지정
  ```

