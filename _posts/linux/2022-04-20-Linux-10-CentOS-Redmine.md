---
title:  "CentOS7 Redmine 설치"
excerpt: "리눅스 환경에 Redmin 설치하기"

tags:
  - [CentOS7, Redmine]

toc: true
toc_sticky: true
toc_label: "[CentOS7 #8 Redmine]"
 
date: 2021-03-25
last_modified_at: 2021-03-25
---


## Redmine 이란?
> 레드마인은 오픈 소스 프로그램으로 웹 기반의 프로젝트 관리와 버그 추적 기능을 제공하는 도구이다. 화면기반의 프로젝트 관리에 도움이 되도록 달력과 간트 차트를 제공하고 일정관리 기능을 제공한다. 또한 레드마인은 통합된 프로젝트관리 기능과 이슈추적, 여러가지 형상 관리 기능을 제공한다


### 1. Redmine 용 Maria DB 설치

  ```
  sudo yum -y install mariadb-server mariadb  # MariaDB 설치
  sudo systemctl start mariadb.service        # 서비스 시작
  sudo systemctl enable mariadb.service       # 재부팅 시 자동실행 설정

  sudo mysql_secure_installation              # root 비밀번호 설정
  
  mysql -u root -p                            # mariadb 접속

  MariaDB [(none)]> CREATE DATABASE redmine CHARACTER SET utf8 COLLATE utf8_general_ci; 
  MariaDB [(none)]> CREATE USER 'redmine'@'%' IDENTIFIED BY '1234!';
  MariaDB [(none)]> GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'%';
  ```

  - 첫번째 물음에 VALIDATE PASSWORD PLUGIN을 사용할지 물어봅니다.

    > 개발용/테스트 장비에서 mysql server를 설치할때는 비밀번호가 길면 귀찮으니, 아무키나 입력해서 플러그인을 비활성화 하는 것도 좋지만, production용 라이브 장비라면 Y 또는 Yes를 입력해서 안전한 비밀번호를 사용하는게 좋습니다.
  
  - root 비밀번호를 물으면, 원하는 root 비밀번호를 동일하게 두번 입력해 줍니다.

    > 앞서 VALIDATE PASSWORD PLUGIN을 활성화 했다면 대소문자, 숫자, 특수문자 1개 이상을 섞어서 8자 이상의 비밀번호를 만들어야합니다.

  - Anonymous User를 삭제할지 물으면 Y를 입력해서 삭제해 줍니다.

  - 외부에서 root 계정으로 접근을 허용할지 묻습니다.

    > 각자 환경에 따라 다르겠지만, 대부분 외부에서 root 계정으로 DB에 접근할 일은 많지 않을 것으로 예상되기 때문에 Y를 입력해서 접근하지 못하도록 합니다. 혹여 필요하시다면 N으로 외부 접근을 허용하시면 됩니다.
  
  - Test DB를 삭제할지 물으면 필요 없으니 Y를 입력해서 삭제해 줍니다.

  - 마지막으로 privileges table을 reload 할지 물으면 Y를 입력해서 reload 해줍니다.


### 2. Redmine 의존성 설치

  ```
  sudo yum -y install libxslt-devel libyaml-devel libxml2-devel gdbm-devel libffi-devel pcre-devel git memcached-devel valgrind-devel mysql-devel 
  sudo yum -y install gcc g++ cpp gcc-c++
  sudo yum -y install openssl-devel readline-devel zlib-devel curl-devel 
  sudo yum -y install mariadb-devel
  sudo yum -y install ImageMagick ImageMagick-devel

  ```

### 3. Ruby 설치
  > 아래 페이지에서 권장환경과 호환 버전을 꼭 확인 할 것.<br>
  > https://www.fastruby.io/blog/ruby/rails/versions/compatibility-table.html <br>
  > https://rubygems.org/gems/rails/versions/6.0.4

  ```
  cd /usr/local/download

  # 루비 설치 시 고려해야 할 사항은 rails 와 호환 버전 유무가 중요합니다.
  # 설치하려는 Ruby, Rails 버전 호환성을 미리 확인 후 진행 하시기 바랍니다.
  # 호환 문제 발생시 결국 해결 가능했지만 까다로운 문제이니 꼭 확인하세요.
  sudo wget https://cache.ruby-lang.org/pub/ruby/2.6/ruby-2.6.5.tar.gz 
  sudo tar xvf ruby-2.6.5.tar.gz 

  sudo mv ruby-2.6.5 /usr/local/ruby-2.6.5 

  cd /usr/local/ruby-2.6.5 
  sudo ./configure 
  sudo make 
  sudo make install 
  ruby -v
  ```

### 4. ruby-gems 설치

  ```
  cd /usr/local/download
  sudo wget https://rubygems.org/rubygems/rubygems-3.1.2.tgz 
	sudo tar xvf rubygems-3.1.2.tgz 

  sudo mv rubygems-3.1.2 /usr/local/rubygems-3.1.2 

  /usr/local/bin/ruby setup.rb
  ```

### 5. bundler, chef ruby-shadow 설치
  > bundler(번들러)는 Ruby 어플리케이션을 위한 일관된 환경을 관리

  ```
  sudo gem install bundler chef ruby-shadow
  ```

  > 위 내용에서 권한 등의 문제가 발생 시 아래 내용을 따라해주세요.<br>
  > git clone repository 가 404 일 경우 fork 떠 놨으니 제 레포에서 찾으시면 됩니다.

  ```
  # ruby-build 플러그인 설치
  sudo yum groupinstall development
  sudo yum install git

  # rbenv 레파지토리 클론을 받고 환경 변수 등록
  git clone https://github.com/sstephenson/rbenv.git ~/.rbenv 
  echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile

  # bash shell 재시작
  source ~/.bash_profile 

  # ruby-build 플러그인 설치
  git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
  sudo ~/.rbenv/plugins/ruby-build/install.sh


  # rbenv 를 이용하여 루비 설치
  rbenv install 2.6.5
  rbenv rehash		
  rbenv global 2.6.5

  # rbenv 로 설치 된 루비 버전 확인
  rbenv versions
  ruby -v
  env | grep PATH

  # 변수 설정
  vi /home/ymk/.bash_profile

  # bash_profile 파일의 export PATH 바로 아래 내용 추가
  eval "$(rbenv init -)"

  # 수정 사항 반영
  source ~/.bash_profile

  # bundler 설치
  gem install bundler ruby-shadow
  gem install ruby-openid
  gem install mysql2
  ```

### 6. rails 설치
  > Ruby 2.6.5 버전을 설치 하였고, Rails 6.0.X 버전의 권장 환경이 Ruby 2.6.X 이기에 6.0.4 버전을 설치한다. <br>
  > https://www.fastruby.io/blog/ruby/rails/versions/compatibility-table.html <br>
  > https://rubygems.org/gems/rails/versions/6.0.4

  ```
  gem install rails -v 6.0.4
  ```

### 7. Redmine 설치

  ```
  cd /usr/local/download
  sudo wget https://www.redmine.org/releases/redmine-4.1.0.tar.gz --no-check-certificate
  sudo tar xvfz redmine-4.1.0.tar.gz
  sudo mv redmine-4.1.0 /usr/local/redmine
  ```

### 8. Redmine DB 연동

  ```
  cd /usr/local/redmine/config
  cp database.yml.example config/database.yml 

  vi config/database.yml # 파일을 열고 production 값을 아래의 내용으로 수정한다.  
  production: 
    adapter: mysql2 
    database: redmine 
    host: localhost 
    username: redmine 
    password: "1234!" 
    encoding: utf8

  ```

### 9. 구동에 필요한 패키지 설치

  ```
  bundle install --without development test --path vendor/bundle
  bundle install --without development test 

  bundle exec rake generate_secret_token # 쿠키 암호화를 위한 시크릿 토큰 생성

  RAILS_ENV=production bundle exec rake db:migrate # DB 생성

  bundle exec rake redmine:load_default_data RAILS_ENV=production REDMINE_LANG=ko # 기본언어를 한국어로 설정
  ```

### 10. Redmine 실행

  ```
  sudo firewall-cmd --permanent --zone=public --add-port=3000/tcp # 방화벽 해제
  sudo firewall-cmd --reload 

  cd /usr/local/redmine
  bundle exec rails server webrick -e production -b 0.0.0.0 &	
  ```

  > http://[서버ip]:3000 로 Redmine 접속 가능


### 11. Apache Passenger 를 사용하여 아파치와 연동

  ```
  sudo yum -y install httpd libcurl-devel httpd-devel apr-devel apr-util-devel

  
  # gem 을 통해 passenger 설치
  gem install passenger

  # passenger 을 통해 apache module 설치
  # 기존에 설치한 apache 가 존재 할 경우
  passenger-install-apache2-module --apxs2-path "apxs 경로"
  passenger-install-apache2-module --apxs2-path "/usr/local/apache2/bin/apxs"

  # 기존에 apache를 설치하지 않았을 경우
  assenger-install-apache2-module
  

  # httpd.conf 파일에 아래 내용 추가
  sudo vi /usr/local/apache2/conf/httpd.conf

  LoadModule passenger_module /home/ymk/.rbenv/versions/2.6.5/lib/ruby/gems/2.6.0/gems/passenger-6.0.13/buildout/apache2/mod_passenger.so
  <IfModule mod_passenger.c>
    PassengerRoot /home/ymk/.rbenv/versions/2.6.5/lib/ruby/gems/2.6.0/gems/passenger-6.0.13
    PassengerDefaultRuby /home/ymk/.rbenv/versions/2.6.5/bin/ruby
    Include conf/redmine.conf
  </IfModule>


  # redmine.conf 파일 생성 후 아래내용 추가
  sudo vi /usr/local/apache2/conf/redmine.conf

  Listen 3000
  <VirtualHost *:3000>
  # ServerName redmine
  DocumentRoot /usr/local/redmine/public
  ErrorLog logs/redmine_error_log
    <Directory /usr/local/redmine/public>
      AllowOverride all
      Options -Multiviews
      Require all granted
    </Directory>
  </VirtualHost>


  # apache 재시작
  systemctl restart apache
  systemctl status apache
  ```


![VMWare](/assets/image/linux/Centos_redmine_01.PNG)


> http://[서버ip]:3000 로 Redmine 접속 가능하며, 초기 계정은 admin / admin 입니다.

