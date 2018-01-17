---
title: Gogs Install in CentOS 7.x
category: Server
excerpt: |
  Gogs Install in CentOS 7.x
feature_text: |
  ## Gogs Install in CentOS 7.x
feature_image: "https://eladnava.com/content/images/2016/04/gogs-4.jpg"
image: "https://eladnava.com/content/images/2016/04/gogs-4.jpg"
---

  새로운 프로젝트에서 형상관리 도입을 위해 gitlab에서 gogs로 전환하여 Cent OS에 설치한 과정을 소개하고자 한다.

### Gogs 설치

#### Git 설치과정

  Git은 Yum에서 이미 설치되어있다.
  하지만 Yum에서 제공된 Git은 최신버전이 아니므로 소스를 받아 재설치하는 것을 권장한다.
  최신버전 확인은 [Github Repository](https://github.com/git/git/releases)에서 확인한다.

  * Git 설치과정

  ```feature_text
  sudo yum remove git
  sudo yum install gettext-devel openssl-devel perl-CPAN perl-devel zlib-devel
  wget https://github.com/git/git/archive/v2.15.1.tar.gz -O git.tar.gz
  tar -zxf git.tar.gz
  cd git-*
  make configure
  ./configure --prefix=/usr/local
  sudo make install
  git --version
  ````


#### MySQL 설치과정

  MySQL은 공식 사이트에서 최신 버전을 확인 후 다운받는다.

  * MySQL 설치

    ```feature_text
    wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
    sudo rpm -ivh mysql57-community-release-el7-9.noarch.rpm
    sudo yum install mysql-server
    ```

  * MySQL 서비스 확인 및 환경설정

  만일 제대로 동작하고 있다면 Active: active (running) 아래와 같은 메시지가 나와야한다.
	또한, MySQL root 계정의 패스워드 변경을 위해 임시 패스워드를 확인한다.
	임시패스워드로 root계정의 새로운 패스워드를 설정해준다.

      sudo systemctl start mysqld
      sudo systemctl status mysqld
      sudo grep 'temporary password' /var/log/mysqld.log
      sudo mysql_secure_installation

  * MySQL 테스트

    ```feature_text
      mysqladmin -u root -p version
      Server version          5.7.20
      Protocol version        10
      Connection              Localhost via UNIX socket
      UNIX socket             /var/lib/mysql/mysql.sock
      Uptime:                 20 hours 58 min 41 sec
      Threads: 3  Questions: 16757  Slow queries: 0  Opens: 423  Flush tables: 1  Open tables: 369  Queries per second avg: 0.221
    ```

#### Golang 설치과정

  Gogs설치를 위해 Golang도 공식 사이트에서 최신 버전을 확인 후 다운받는다. 그전에 데이터베이스를 생성해준다.

  * Gogs 사용을 위한 데이터베이스 생성

    ```feature_text
      mysql -u root -p
      mysql>  create database gogs;
      grant all on gogs.* to 'gogsuser' identified by 'gogsuserS.2017!@';
      flush privileges;
      exit;
    ```

  * Golang 설치 및 환경설정

    ```feature_text
      wget https://storage.googleapis.com/golang/go1.8.3.linux-amd64.tar.gz
      sudo tar -C /usr/local -xvzf go1.8.3.linux-amd64.tar.gz
      sudo vi /etc/profile.d/path.sh
      export PATH=$PATH:/usr/local/go/bin
      source /etc/profile
    ```

#### Gogs 설치과정

  Gogs도 [공식 사이트](https://storage.googleapis.com/golang)에서 최신 버전을 확인 후 다운받는다.

    sudo useradd --system --create-home git
    sudo su git
    cd ~
    wget https://dl.gogs.io/0.11.4/linux_amd64.tar.gz
    tar xvzf linux_amd64.tar.gz
    cd gogs
    ./gogs web --port=1337 <- 구동 후 기본환경설정 및 데이터베이스를 선택해준다.
    스크립트로 Gogs를 구동할 수 있도록 한다.
    cd /etc/init.d
    cp /home/git/gogs/scripts/init/centos/gogs /etc/init.d/gogs
    cd /etc/init.d
    chmod +x gogs
    sudo sed -i -e 's/DAEMON_ARGS="web"/DAEMON_ARGS="web --port 1337"/g' gogs
    sudo service gogs start
