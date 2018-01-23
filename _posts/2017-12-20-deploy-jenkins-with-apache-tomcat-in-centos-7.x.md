---
title: Deploy Jenkins With Apache Tomcat In Cent OS 7.x
category: Server
excerpt: |
  Deploy Jenkins With Apache Tomcat In Cent OS 7.x
feature_text: |
  ## Deploy Jenkins With Apache Tomcat In Cent OS 7.x
feature_image: "https://d1.awsstatic.com/product-marketing/CodeDeploy/Partners/jenkins.069c3fa4bd28d680e13e191fddb65d1e8383161d.png"
image: "https://d1.awsstatic.com/product-marketing/CodeDeploy/Partners/jenkins.069c3fa4bd28d680e13e191fddb65d1e8383161d.png"
---

## 뻘소리..

2018년이 시작되기..전.. 진행하던 프로젝트는 본의아니게 도중에 중단되었지만..
그동안 서버의 환경 구성을 위해 개발 및 배포 프로세스를 구축하던 중 과정을 정리해보자.

서버에 개발환경을 구성하기 위해서는 크게 웹서버, 소스 및 배포관리 툴이 필요했다.

웹서버는 톰캣(Tomcat), 소스관리는 Git으로 호스팅 서비스가 가능한 Gogs, 배포관리는 가장 흔하게 사용될 것 같은 젠킨스(Jenkins)를 선택하였다.

우선 소스관리는 이전에 포스팅한 글을 참고하자.
[Gogs Install in CentOS 7.x](https://dokiman.github.io/server/2017/12/10/Gogs-Install-in-CentOS-7.x/)

### CI(Continuous Integration)

CI(Continuous Integration)가 개발자들 사이에서 오가는 용어라면, 좀더 쉽게 정리하면 소프트웨어 개발 시 지속적 통합 서비스를 제공하는 툴이라고 할 수 있겠다.

CI나 지속적 통합 서비스 제공 툴이나... 직역하면 같긴하다..
어렵게 설명한 것 같다..

젠킨스도 이와 같다.
지속적으로 통합 서비스를 제공하기 위해 만들어진 툴이며, 툴 자체에 있는 Plugin 기능도 활용하면 개발 프로세스를 꽤 괜찮은 방향으로 개선시킬 수 있다.

물론, 사용하는 유저에 따라 입장은 다르겠지만..

시간이 된다면 Jenkins외에 다른 CI도 살펴보자.
[TOP 8 CONTINUOUS INTEGRATION TOOLS](https://www.code-maze.com/top-8-continuous-integration-tools/)

### Install Jenkins

Jenkins 설치를 위해서는 사전에 Java와 Maven설치가 필요했다.
이유는 Project 빌드를 위함이다.
각각 최신 버전을 확인 후 아래와 같이 작업해서 설정해주었다.

  Install Java

      wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-x64.tar.gz"
      mkdir /usr/local/java
      mv jdk-8u151-linux-x64.tar.gz /usr/local/java/
      cd /usr/local/java/
      tar xvzf jdk-8u151-linux-x64.tar.gz

      alternatives --install /usr/bin/java java /usr/local/java/jdk1.8.0_151/bin/java 1
      alternatives --install /usr/bin/jar jar /usr/local/java/jdk1.8.0_151/bin/jar 1
      alternatives --install /usr/bin/javac javac /usr/local/java/jdk1.8.0_151/bin/javac 1
      alternatives --install /usr/bin/javaws javaws /usr/local/java/jdk1.8.0_151/bin/javaws 1
      alternatives --config java
      alternatives --set jar /usr/local/java/jdk1.8.0_151/bin/jar
      alternatives --set javac /usr/local/java/jdk1.8.0_151/bin/javac
      alternatives --set javaws /usr/local/java/jdk1.8.0_151/bin/javaws
      vi /etc/profile
            export JAVA_HOME=/usr/local/java/jdk1.8.0_151
            export JRE_HOME=/usr/local/java/jdk1.8.0_151/jre
            export PATH=$PATH:/usr/local/java/jdk1.8.0_151/bin:/usr/local/jdk1.8.0_151/jre/bin
      . /etc/profile

      java -version
            java version "1.8.0_151"
            Java(TM) SE Runtime Environment (build 1.8.0_151-b12)
            Java HotSpot(TM) 64-Bit Server VM (build 25.151-b12, mixed mode)

  Install Maven

      wget http://www-eu.apache.org/dist/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz
      sudo tar xzf apache-maven-3.5.2-bin.tar.gz
      sudo cp -r ./apache-maven-3.5.2 /usr/local/maven
      vi /etc/profile.d/maven.sh
      source /etc/profile.d//maven.sh
      mvn -version
            Apache Maven 3.5.2 (138edd61fd100ec658bfa2d307c43b76940a5d7d; 2017-10-18T16:58:13+09:00)
            Maven home: /usr/local/maven
            Java version: 1.8.0_151, vendor: Oracle Corporation
            Java home: /usr/local/java/jdk1.8.0_151/jre
            .............................................

Java와 Maven 설치에 이상이 없다면 Jenkins를 설치해준다.

  Install Jenkins

    wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
    rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
    yum install jenkins
    vi /etc/sysconfig/jenkins
    JENKINS_PORT="9090"
    /etc/init.d/jenkins start

    web을 통해 접근한 후, 패스워드 요청시
    vi /var/lib/jenkins/secrets/initialAdminPassword 붙여넣는다.

    웹 사이트가 로딩되기 전 필요한 Plugin은 안내되는 사이트에서 설치해준다.
    추후 설치해도 상관은 없다.

Jenkins 설치가 끝났다면 프로젝트를 띄울 Tomcat을 설치하자.

아, 잠시..
톰캣을 웹서버라고 정의하지는 않는다.
오해가 없도록 하기위해 아래의 글을 참고하자.
[WAS와 웹서버의 차이 – 톰캣과 아파치를 구별하지 못하는 사람을 위해](http://sungbine.github.io/tech/post/2015/02/15/tomcat%EA%B3%BC%20apache%EC%9D%98%20%EC%97%B0%EB%8F%99.html)

그럼 다시 설치과정으로..

  Install Tomcat

      cd ~ && wget http://mirror.apache-kr.org/tomcat/tomcat-8/v8.5.24/bin/apache-tomcat-8.5.24.tar.gz
      mkdir {톰캣을 설치할 경로}
      tar xvf apache-tomcat-8*tar.gz -C {톰캣을 설치할 경로} --strip-components=1
      cd {톰캣을 설치할 경로}
      chgrp -R {톰캣을 실행할 사용자} {톰캣을 설치할 경로}
      chmod -R g+r conf
      chmod g+x conf
      chown -R {톰캣을 실행할 사용자} webapps/ work/ temp/ logs/
      vi context.xml
          …………………
          <Context path="" docBase="/home/nsok_interface_user/interface-tomcat/webapps/ROOT" reloadable="true">
          ………………...
          vi server.xml
          …………………
          <Server port="8006" shutdown="SHUTDOWN">
          <Connector port="8090" protocol="HTTP/1.1"
                         connectionTimeout="20000"
                         connectionUploadTimeout="20000"
                         disableUploadTimeout="true"
                         acceptCount="100"
                         compression="on"
                         maxThreads="512"
                         minSpareThreads="128"
                         tcpNoDelay="true"
                         redirectPort="8453" />
          …………………
      vi tomcat-users.xml
          …………………
          <role rolename="manager-gui"/>
          <role rolename="manager-script"/>
          <user username="tomcatuser" password="tomcatuserS.2017!@" roles="manager-gui,manager-script"/>
          …………………

Tomcat 설치가 끝나면 이제 Jenkins에서 아래와 같은 과정으로 구성해준다.
Get Source From Git -> Build Maven -> Rerun Tomcat

<img src="/assets/post/jenkins-01.png"/>

Jenkins를 로그인한 후에는 위와 같은 화면이 나온다.
처음 로그인 시에는 아이템이 없지만, 생성해놓았기에 2개가 있다.

<img src="/assets/post/jenkins-03.png"/>
<img src="/assets/post/jenkins-04.png"/>

메인화면에서 Jenkins관리 -> Global Tool Configuration에 들어가면 위 화면처럼
Java, Maven Install 경로를 복붙하여 저장해준다.

<img src="/assets/post/jenkins-02.png"/>

위 부분은 각 프로젝트에 대한 SSH 환경설정과 프로젝트별 배포경로를 설정해준다.
덧붙여 설명하자면 한 대의 CentOS 서버에 Jenkins 전용 User, 각 프로젝트별 구동 User가 있다.
<b>서버가 여러대여도 위의 설정은 같다.</b>

Jenkins User에서 각 프로젝트별 구동 User에 접속하기 위해서는 SSH가 필요하다.
키 구성을 위해서는 아래 링크를 참고하자.
[how to setup ssh keys for jenkins to publish via ssh From Stackoverflow](https://stackoverflow.com/questions/37331571/how-to-setup-ssh-keys-for-jenkins-to-publish-via-ssh)

<img src="/assets/post/jenkins-05.png"/>

소스코드관리에서는 Git으로 호스팅서비스하고 있는 Gogs의 프로젝트 Repository 주소를 입력해준다.
이 주소를 통해서 Project 폴더를 통째로 받아온다.

<img src="/assets/post/jenkins-06.png"/>

Build에서는 서버에 설치된 Maven으로 Project 폴더를 clean 해준 다음 재 install을 해준다.

그리고 아래 그림에서는 Jenkins User가 SSH 접속 후
Tomcat Install 경로로 이동하여 우선 서버를 중지시킨후 기존 프로젝트 폴더를 통째로 날려준다.
이후 다시한번 Jenkins User로 SSH 접속 후 Tomcat를 재시작해주는 과정이다.

<img src="/assets/post/jenkins-07.png"/>
<img src="/assets/post/jenkins-08.png"/>

해당 아이템(=배포할 프로젝트)에 설정이 끝나면 Build를 해준다.
Console 결과는 아래와 같다.

<img src="/assets/post/jenkins-09.png"/>

위 그림처럼 Jenkins 설정을 해준다면 에러가 나는 경우가 몇가지 있다.

첫째, SSH 설정<br/>
둘째, SSH Server에 접근 후 Shell 명령 문제<br/>
마지막으로 간혹 Git 주소에 대한 잘못된 접근으로 에러가 나는 경우가 있다.
