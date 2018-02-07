---
title: Install ELK Stack In CentOS 7.x
category: Server
excerpt: |
  Install ELK Stack In CentOS 7.x
feature_text: |
  ## Install ELK Stack In CentOS 7.x
feature_image: "https://logz.io/wp-content/uploads/2016/01/what-is-elk2.jpg"
image: "https://logz.io/wp-content/uploads/2016/01/what-is-elk2.jpg"
---

Kibana 기반의 Dashboard 분석을 위해 CentOS에 ELK Stack을 설치하는 과정을 정리해본다.

### Install ELK Stack
  <br/>
  ```
  - Cloud: Google Cloud Platform
  - OS: CentOS 7 (CPU: 2, RAM: 4GB)
  - Install Program: Java, Elasticsearch, Kibana, Nginx, Logstash, Filebeat
  ```

##### Basic Package
  <br/>
  ```
  sudo yum update
  sudo yum groupinstall "Development Tools"
  sudo yum install tcl wget telnet
  ```

##### Java 8
  <br/>
  ```
  wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u162-b12/0da788060d494f5095bf8624735fa2f1/jdk-8u162-linux-x64.tar.gz"
  sudo mkdir /usr/local/java
  sudo mv jdk-8u162-linux-x64.tar.gz /usr/local/java/
  cd /usr/local/java/
  sudo tar xvzf jdk-8u162-linux-x64.tar.gz

  sudo alternatives --install /usr/bin/java java /usr/local/java/jdk1.8.0_162/bin/java 1
  sudo alternatives --install /usr/bin/jar jar /usr/local/java/jdk1.8.0_162/bin/jar 1
  sudo alternatives --install /usr/bin/javac javac /usr/local/java/jdk1.8.0_162/bin/javac 1
  sudo alternatives --install /usr/bin/javaws javaws /usr/local/java/jdk1.8.0_162/bin/javaws 1
  sudo alternatives --config java
      There is 1 program that provides 'java'.

      Selection    Command
      -----------------------------------------------
      *+ 1           /usr/local/java/jdk-9.0.4/bin/java
         2           /usr/local/java/jdk1.8.0_162/bin/java

      Enter to keep the current selection[+], or type selection number: 2

  sudo alternatives --set jar /usr/local/java/jdk1.8.0_162/bin/jar
  sudo alternatives --set javac /usr/local/java/jdk1.8.0_162/bin/javac
  sudo alternatives --set javaws /usr/local/java/jdk1.8.0_162/bin/javaws

  vi /etc/profile
      export JAVA_HOME=/usr/local/java/jdk1.8.0_162
      export JRE_HOME=/usr/local/java/jdk1.8.0_162/jre
      export PATH=$PATH:/usr/local/java/jdk1.8.0_162/bin:/usr/local/jdk1.8.0_162/jre/bin

  . /etc/profile
  java -version
      java version "1.8.0_162"
      Java(TM) SE Runtime Environment (build 1.8.0_162-b12)
      Java HotSpot(TM) 64-Bit Server VM (build 25.162-b12, mixed mode)
  ```

##### Elasticsearch
  <br/>
  [Elastic Search Recent Install Guide](https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html)

  ```
  * Install Elasticsearch
      sudo
       rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
      echo '[elasticsearch-6.x]
          > name=Elasticsearch repository for 6.x packages
          > baseurl=https://artifacts.elastic.co/packages/6.x/yum
          > gpgcheck=1
          > gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
          > enabled=1
          > autorefresh=1
          > type=rpm-md
          > ' | sudo tee /etc/yum.repos.d/elasticsearch.repo
          [elasticsearch-6.x]
          name=Elasticsearch repository for 6.x packages
          baseurl=https://artifacts.elastic.co/packages/6.x/yum
          gpgcheck=1
          gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
          enabled=1
          autorefresh=1
          type=rpm-md    

  * Modify Elasticsearch Network Host
    sudo vi /etc/elasticsearch/elasticsearch.yml
    network.host: private_ip (*.*.*.*)

  * Start and enable the elasticsearch service.
    Config SysV init vs SystemD (Elasticsearch Automatically Start and Stop)
  - init
      sudo chkconfig --add elasticsearch
      sudo -i service elasticsearch start
      sudo -i service elasticsearch stop

  - SystemD
      sudo /bin/systemctl daemon-reload
      sudo /bin/systemctl enable elasticsearch.service
      sudo systemctl start elasticsearch.service
      sudo systemctl stop elasticsearch.service

  * Allow traffic through TCP Port 9200 in Firewall
      sudo firewall-cmd --add-port=9200/tcp
      sudo firewall-cmd --add-port=9200/tcp --permanent

  * Check Service Status & Test
      ps -ef | grep elastic
          elastic+  8447     1 99 06:07 ?        00:00:13 /bin/java -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75   -XX:+UseCMSInitiatingOccupancyOnly -XX:+AlwaysPreTouch -server -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/var/lib/elasticsearch -Des.path.home=/usr/share/elasticsearch -Des.path.conf=/etc/elasticsearch -cp /usr/share/elasticsearch/lib/* org.elasticsearch.bootstrap.Elasticsearch -p /var/run/elasticsearch/elasticsearch.pid --quiet
      curl http://localhost:9200
          {
            "name" : "bEwzZ-b",
            "cluster_name" : "elasticsearch",
            "cluster_uuid" : "gr_GJQuPQzuAjhgifY1OUA",
            "version" : {
              "number" : "6.1.2",
              "build_hash" : "5b1fea5",
              "build_date" : "2018-01-10T02:35:59.208Z",
              "build_snapshot" : false,
              "lucene_version" : "7.1.0",
              "minimum_wire_compatibility_version" : "5.6.0",
              "minimum_index_compatibility_version" : "5.0.0"
            },
            "tagline" : "You Know, for Search"
          }
  ```

##### Logstash
  <br/>
  ```
  * Install Logstash
      sudo vi /etc/yum.repos.d/logstash.repo
          [logstash-6.x]
          name=Elastic repository for 6.x packages
          baseurl=https://artifacts.elastic.co/packages/6.x/yum
          gpgcheck=1
          gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
          enabled=1
          autorefresh=1
          type=rpm-md

      sudo yum install logstash
      sudo systemctl start logstash.service

  * Generate a self-signed SSL certificate by OpenSSL
      sudo vi /etc/pki/tls/openssl.cnf
          [ v3_ca ]
          subjectAltName = IP: 10.160.0.2          

  * Create input, outpint, filter files
      sudo vi /etc/logstash/conf.d/input.conf
          input {
              beats {
                  port => 5044
                  ssl => true
                  ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
                  ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
              }
          }
      sudo vi /etc/logstash/conf.d/output.conf
          output {
               elasticsearch {
                   hosts => ["localhost:9200"]
                   sniffing => true
                   manage_template => false
                   index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
                   document_type => "%{[@metadata][type]}"
               }
          }
      sudo vi /etc/logstash/conf.d/filter.conf
          filter {
              if [type] == "syslog" {
                  grok {
                      match => { "message" => "%{SYSLOGLINE}" }
                  }
                  date {
                      match => [ "timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
                  }
              }
          }

  * Start and enable the logstash service.
    sudo systemctl daemon-reload
    sudo systemctl enable logstash.service
    sudo systemctl start logstash.service

  * Allow traffic through TCP Port 5044 in Firewall
    sudo firewall-cmd --add-port=5044/tcp
    sudo firewall-cmd --add-port=5044/tcp --permanent
  ```

##### Kibana
  <br/>
  ```
  * Install Kibana
    sudo vi /etc/yum.repos.d/kibana.repo
        [kibana-6.x]
        name=Elasticsearch repository for 6.x packages
        baseurl=https://artifacts.elastic.co/packages/6.x/yum
        gpgcheck=1
        gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
        enabled=1
        autorefresh=1
        type=rpm-md

    sudo yum install kibana

  * Start and enable the kibana service.
    sudo systemctl daemon-reload
    sudo systemctl enable kibana.service
    sudo systemctl start kibana.service

  * Allow traffic through TCP Port 5601 in Firewall
    sudo firewall-cmd --add-port=5601/tcp
    sudo firewall-cmd --add-port=5601/tcp --permanent

  * Check Service Status
    ps -ef | grep kibana
      kibana    8970     1  0 06:56 ?        00:00:07 /usr/share/kibana/bin/../node/bin/node --no-warnings /usr/share/kibana/bin/../src/cli -c /etc/kibana/kibana.yml
  ```

### 참고자료

  - [How To Install Elasticsearch, Logstash, and Kibana (ELK Stack) on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-elasticsearch-logstash-and-kibana-elk-stack-on-centos-7)
  - [Configure Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html)
  - [Secure Settings](https://www.elastic.co/guide/en/elasticsearch/reference/current/secure-settings.html)
  - [Loggin Configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/logging.html)
