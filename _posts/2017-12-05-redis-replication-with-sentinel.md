---
title: Redis Repliction with Sentinel
category: Server
excerpt: |
  Redis Repliction with Sentinel
feature_text: |
  ## Redis Repliction with Sentinel
feature_image: "https://kingbbode.github.io/images/2016/2016_12_04_Spring_Boot_Redis/redis.png"
image: "https://kingbbode.github.io/images/2016/2016_12_04_Spring_Boot_Redis/redis.png"
---

  새로운 프로젝트 진행을 위해 3대의 각 Linux (Cent OS 7.x)서버에 Redis를 설치하고, Replication 및 Sentinel을 적용해야했다.

  크게 과정은 Redis를 설치하고, 설치한 Redis에 상태체크 및 테스트 이후 Replication과 Sentinel을 적용한다.

  본 문서에서는 3대의 서버에 1대는 Master, 2대는 Slave역할로 Redis-Server(7000)를 구성하였다.

  또한, 장애대응을 위해 Redis의 Sentinel을 활용하여 총 3대의 서버에 Redis-Sentinel-Server(8000)를 설치하고 Slave-Master-Slave 구조를 갖도록 구성하였다.

### Redis 설치

#### Redis 다운로드 및 빌드

Redis를 다운로드 받을때 yum install redis를 사용해도 좋지만, 최신버전이 아닌 것으로 나오기 때문에 tar.gz를 받아 별도로 빌드하여 설치한다.

    wget -c http://download.redis.io/redis-stable.tar.gz
    tar xvzf redis-stable.tar.gz
    cd redis-stable && make
    make test && sudo make install
    => make test는 코드의 안정성 여부를 확인하기 위해 진행해주는 것이 좋다.
    만약 오류가 날 경우 성공할 때까지 다시 진행해도 좋고 make install만 해주어도 상관없다.
    \o/ All tests passed without errors!
    => make test가 정상적으로 끝났을 때
    INSTALL install
    => make install을 정상적으로 끝났을 때


#### Redis 설치 및 실행

Redis를 설치 시에 설치경로, 로그파일경로, 데이터경로, 실행파일경로를 지정하는 부분이 있다.
별도로 지정해도 상관없고, 그대로 사용해도 상관없다.


  * 환경설정 정보

    ```feature_text
    redis_port: 7000
    redis_install_path: {default}/redis_{redis_port}.conf
    redis_log_path: {default}
    redis_data_path: {default}
    redis_executable_path: /usr/local/bin/redis-server

    sudo ./utils/install_server.sh
        Welcome to the redis service installer
        This script will help you easily set up a running redis server
        Please select the redis port for this instance: [6379] 7000
        Please select the redis config file name [/etc/redis/7000.conf] /etc/redis/redis_7000.conf
        Please select the redis log file name [/var/log/redis_7000.log] /var/log/redis_7000.log
        Please select the data directory for this instance [/var/lib/redis/7000] /var/lib/redis/7000
        Please select the redis executable path [] /usr/local/bin/redis-server
        Selected config:
        Port       	              : 7000
        Config file	: /etc/redis/redis_7000.conf
        Log file   	: /var/log/redis_7000.log
        Data dir   	: /var/lib/redis/7000
        Executable 	: /usr/local/bin/redis-server
        Cli Executable      : /usr/local/bin/redis-cli
    ```


#### Redis 상태체크 및 테스트

  현재까지는 각 서버 모두 Master, Slave 구분이 없기 때문에 아래와 같이 테스트해주고, 끝나면 Redis를 종료해준다.

    /etc/init.d/redis_7000 status
    Redis is running (30516)
    tail –n 16 /var/log/redis_7000.log
    445:M 28 Nov 01:06:15.413 # Server initialized
    ………………………………………………………….
    445:M 28 Nov 01:06:15.413 * Ready to accept connections
    ps aux | grep redis
    root       445  .. /usr/local/bin/redis-server 127.0.0.1:7000
    redis-cli -p 7000
          127.0.0.1:7000> ping
          PONG
          127.0.0.1:7000> set nsok good
          OK
          127.0.0.1:7000> get nsok
          "good"
          service redis_7000 stop
          Stopping …
          Redis stopped
          /etc/init.d/redis_7000 status
          cat: /var/run/redis_7000.pid: No such file or directory
          Redis is running ()


#### Redis 복제 (Replication)

  Redis 복제를 위해서는 Master, Slave로 나눠서 각각 환경설정을 해주어야 한다.

  * Master Config 설정

        sudo vi /etc/redis/redis_7000.conf
        bind 0.0.0.0
        => 사용할 IP를 지정하는 부분으로 #처리하거나 0.0.0.0 or 서버 ip를 지정하면 된다.
        이때 bind하는 ip에 따라 설정부분에 차이가 있음을 인지하는 것이 중요하다.
        requirepass nsokaicontrol
        => 사용할 패스워드 지정
        masterauth nsokaicontrol
        => requirepass와 동일한 패스워드
        # repl-ping-slave-period 10
        # repl-timeout 60
        => replication 관련 설정으로 주석해제 후 값은 원하는 대로 바꿔준다.
        마스터 서버도 replication을 구성하면 추후 slave로 변경될 수 있기 때문에 미리 해제해준다

  * Slave Config 설정

        sudo vi /etc/redis/redis_7000.conf
        bind 0.0.0.0
        => 사용할 IP를 지정하는 부분으로 #처리하거나 0.0.0.0 or 서버 ip를 지정하면 된다.
        이때 bind하는 ip에 따라 설정부분에 차이가 있음을 인지하는 것이 중요하다.
        requirepass nsokaicontrol
        => 사용할 패스워드 지정
        masterauth nsokaicontrol
        => requirepass와 동일한 패스워드
        # repl-ping-slave-period 10
        # repl-timeout 60
        => replication 관련 설정으로 주석해제 후 값은 원하는 대로 바꿔준다.
        마스터 서버도 replication을 구성하면 추후 slave로 변경될 수 있기 때문에 미리 해제해준다.
        slaveof 10.160.0.6 7000
        => slave에 해당되는 서버의 환경설정 파일에 수정해준다.

  * redis_7000 설정 (각 3개의 서버 모두 동일하게 변경해준다.)

        sudo vi /etc/init.d/redis_7000
            REDISPWD=”nsokaicontrol”
            $CLIEXEC -p $REDISPORT -a $REDISPWD shutdown


#### Redis 복제 상태체크 및 테스트

  설정이 끝났으면, Master, Slave, Slave순으로 다시  Redis를 실행시켜준다.

  * Master 구동여부, 정보확인

  Master서버는 로그에서 다른 Slave 서버와 Sync되었는지 확인해준다.
  또한 기존에 테스트했었던 읽기, 쓰기를 다시 해본 후 Replication 정보를 확인한다.

    sudo /etc/init.d/redis_7000 start
          Starting Redis server...
          /etc/init.d/redis_7000 status
          tail –n 16 /var/log/redis_7000.log
          => slave 서버까지 모두 구동 후 아래와 같은 로그가 확인된다면 replication 이 완료된 것이다.
          761:M.. * Slave 10.160.0.7:7000 asks for synchronization
          ……………………………………………………………………………………………………………………………………………………………..
          761:M..* Synchronization with slave 10.160.0.7:7000 succeeded
          761:M.. * Slave 10.160.0.8:7000 asks for synchronization
          ……………………………………………………………………………………………………………………………………………………………..
          761:M..* Synchronization with slave 10.160.0.8:7000 succeeded
          ps aux | grep redis
          root    761  …… /usr/local/bin/redis-server 0.0.0.0:7000
          redis-cli -p 7000 -a nsokaicontrol
                127.0.0.1:7000> ping
                PONG
                127.0.0.1:7000> set nsok good
                OK
                127.0.0.1:7000> get nsok
                "good"
                127.0.0.1:7000> info replication
                #Replication
                role:master
                connected_slaves:2
                slave0:ip=10.160.0.7,port=7000,state=online,offset=378,lag=1
                slave1:ip=10.160.0.8,port=7000,state=online,offset=364,lag=1
                master_replid:2a4803e7e009fd7f0052d9d91d0f6fd48c1e84ec
                master_replid2:0000000000000000000000000000000000000000
                master_repl_offset:378
                second_repl_offset:-1
                repl_backlog_active:1
                repl_backlog_size:1048576
                repl_backlog_first_byte_offset:1
                repl_backlog_histlen:378

  * Slave 구동여부, 정보확인

  각 Slave서버에서는 로그에서 Master 서버와 Sync되었는지 확인해준다.
  또한 기존에 테스트했었던 읽기, 쓰기를 다시 해본 후 Replication 정보를 확인한다.
  이때, Slave 서버에서는 쓰기가 안되는 것이 정상이다.

      sudo /etc/init.d/redis_7000 start
      /etc/init.d/redis_7000 status
      tail –n 16 /var/log/redis_7000.log
      30780:S 28 Nov 01:40:49.439 * Connecting to MASTER 10.160.0.6:700030780:S 28 Nov 01:40:49.439 * MASTER <-> SLAVE sync started
      ………………………………………………………………………………………………………………………………………………………………..
      30780:S 28 Nov 01:40:49.487 * MASTER <-> SLAVE sync: receiving 191 bytes from master
      30780:S 28 Nov 01:40:49.487 * MASTER <-> SLAVE sync: Flushing old data
      30780:S 28 Nov 01:40:49.487 * MASTER <-> SLAVE sync: Loading DB in memory
      30780:S 28 Nov 01:40:49.488 * MASTER <-> SLAVE sync: Finished with success
      ps aux | grep redis
      redis-cli -p 7000 -a mypassword
            127.0.0.1:7000> ping
            PONG
            127.0.0.1:7000> set nsok good
            (error) READONLY You can't write against a read only slave.
            127.0.0.1:7000> get nsok
            "good"
            127.0.0.1:7000> info replication
            #Replication
            role:slave
            master_host:10.160.0.6
            master_port:7000
            master_link_status:up
            master_last_io_seconds_ago:3
            master_sync_in_progress:0
            slave_repl_offset:882
            slave_priority:100
            slave_read_only:1
            connected_slaves:0
            master_replid:2a4803e7e009fd7f0052d9d91d0f6fd48c1e84ec
            master_replid2:0000000000000000000000000000000000000000
            master_repl_offset:882
            second_repl_offset:-1
            repl_backlog_active:1
            repl_backlog_size:1048576
            repl_backlog_first_byte_offset:1
            repl_backlog_histlen:882

#### Redis 센티넬(Sentinel)

  센티넬은 Master, Slave의 구분이 없으므로 아래와 같이 동일하게 파일을 만들어주고 설정해주면 된다.

    sudo cp ./redis-stable/sentinel.conf /etc/redis/sentinel_8000.conf
    sudo vi /etc/redis/sentinel_8000.conf
          bind 10.160.0.6
          => sentinel에 사용할 IP를 지정하는 부분으로 #처리하거나 0.0.0.0 or 서버 ip를 지정하면 된다.
          이때 bind하는 ip에 따라 설정부분에 차이가 있음을 인지하는 것이 중요하다.
                        #이나 0.0.0.0으로 처리할 경우, protected-mode no 로 바꿔주어야 한다.
          port 8000
          => 사용할 포트 지정
          sentinel 관련 설정, 값은 원하는 대로 바꿔주거나 추가해준다.
          sentinel monitor mymaster 10.160.0.6 7000 2
          => mymaster는 마스터 이름으로 전부 통일해서 바꿔줘도 괜찮다. 여기서는 default로 사용하였다.
          port 뒤에 입력되는 숫자는 sentinel을 구성하는데 중요한 정보이다.
          quorum으로 failover를 진행하기위해 필요한 sdown의 수로 알아두면 된다.
          sdown과 관련한 상세정보
          sentinel down-after-milliseconds mymaster 30000
          sentinel parallel-syncs mymaster 1
          sentinel failover-timeout mymaster 180000
          sentinel auth-pass mymaster nsokaicontrol
          daemonize yes
          pidfile /var/run/sentinel_8000.pid
          logfile /var/log/sentinel_8000.log
          => 위 정보는 sentinel.conf 파일에 없기 때문에 추가해준다


    sudo cp /etc/init.d/redis_7000 /etc/init.d/sentinel_8000
    sudo vi /etc/init.d/sentinel_8000

          EXEC=/usr/local/bin/redis-sentinel
          PIDFILE=/var/run/sentinel_8000.pid
          CONF="/etc/redis/sentinel_8000.conf"
          SENTINELHOST="10.160.0.2"
          => sentinel의 bind를 private ip로 했을경우 추가해준다.
          SENTINELPORT="8000"
          $REDISPWD="nsokaicontrol”
          ,...........
          ### END INIT INFO
          => 위 부분은 삭제해준다.
          $CLIEXEC -h $SENTINELHOST -p $SENTINELPORT shutdown

### Redis 센티넬(Sentinel), 상태체크 및 테스트

  설정이 끝났으면, Master, Slave, Slave순으로 Sentinel을 실행시켜준다.
	아래정보와 같이 로그가 확인되면 정상적으로 failover를 테스트 할 수 있다.

  * Master 및 Slave Sentinel 정보

      ```
      sudo /etc/init.d/sentinel_8000 start
      Starting Sentinel server...
      /etc/init.d/sentinel_8000 status
      Sentinel is running (1250)
      tail –n 16 /var/log/sentinel_8000.log
      1250:X 28 Nov 02:32:27.777 # +monitor master mymaster 10.160.0.6 7000 quorum 2
      1250:X 28 Nov 02:32:27.779 * +slave slave 10.160.0.7:7000 10.160.0.7 7000 @ mymaster 10.160.0.6 7000
      1250:X 28 Nov 02:32:27.781 * +slave slave 10.160.0.8:7000 10.160.0.8 7000 @ mymaster 10.160.0.6 7000
      1250:X 28 Nov 02:33:09.357 * +sentinel sentinel 56b5c3ddb013e973b58bda9b5ee831a8c4bfac74 10.160.0.7 8000 @ mymaster 10.160.0.6 7000
      1250:X 28 Nov 02:33:53.868 * +sentinel sentinel 46257646104aadc8ed42ecca5e5c5aa8b9754fbc 10.160.0.8 8000 @ mymaster 10.160.0.6 7000
      ps aux | grep redis
      root …. 02:32   0:00 /usr/local/bin/redis-sentinel 10.160.0.6:8000 [sentinel]
      redis-cli -h 10.160.0.2 -p 8000
            10.160.0.6:8000> info sentinel
            #Sentinel
            sentinel_masters:1
            sentinel_tilt:0
            sentinel_running_scripts:0
            sentinel_scripts_queue_length:0
            sentinel_simulate_failure_flags:0
            master0:name=mymaster,status=ok,address=10.160.0.6:7000,slaves=2,sentinels=3
      ```

  * Failover 테스트 정보

  Failover 테스트를 위하여 Master 서버의 Redis를 강제로 다운시킨다.
	로그를 확인하여 Slave에서 Master 서버로 변경된 서버로 접속하여 제대로
	설정되었는지 확인해준다.

    sudo /etc/init.d/redis_7000 stop
    tail -f /var/log/sentinel_8000.log
          1250:X 28 Nov 02:39:26.112 # +sdown master mymaster 10.160.0.6 70001250:X 28 Nov 02:39:26.197 # +new-epoch 1
          1250:X 28 Nov 02:39:26.199 # +vote-for-leader 56b5c3ddb013e973b58bda9b5ee831a8c4bfac74 1
          1250:X 28 Nov 02:39:26.719 # +config-update-from sentinel 56b5c3ddb013e973b58bda9b5ee831a8c4bfac74 10.160.0.7 8000 @ mymaster 10.160.0.6 7000
          1250:X 28 Nov 02:39:26.719 # +switch-master mymaster 10.160.0.6 7000 10.160.0.7 7000
          1250:X 28 Nov 02:39:26.719 * +slave slave 10.160.0.8:7000 10.160.0.8 7000 @ mymaster 10.160.0.7 7000
          1250:X 28 Nov 02:39:26.719 * +slave slave 10.160.0.6:7000 10.160.0.6 7000 @ mymaster 10.160.0.7 7000
          1250:X 28 Nov 02:39:56.732 # +sdown slave 10.160.0.6:7000 10.160.0.6 7000 @ mymaster 10.160.0.7 7000
    redis-cli -p 7000 -a nsokaicontrol (slave에서 master 서버로 변경된 정보 확인)
          127.0.0.1:7000> set nsok new
          OK
          127.0.0.1:7000> get nsok
          "new"
          127.0.0.1:7000> ping
          PONG
          127.0.0.1:7000> info replication
          #Replication
          role:master
          connected_slaves:1
          slave0:ip=10.160.0.8,port=7000,state=online,offset=870245,lag=1
          master_replid:f2e58cd5763691a88e76a7345f8937a57d68f317
          master_replid2:2a4803e7e009fd7f0052d9d91d0f6fd48c1e84ec
          master_repl_offset:870379
          second_repl_offset:72465
          repl_backlog_active:1
          repl_backlog_size:1048576
          repl_backlog_first_byte_offset:1
          repl_backlog_histlen:870379

#### Redis 삭제


  ```
  sudo rm -rf /etc/redis/
  sudo rm -rf /var/log/redis_* /etc/init.d/redis_7000 /usr/local/bin/redis*
  sudo rm -rf /var/lib/redis/7000
  ```

#### 참고사이트  

  * Basic
    - [Install and Configure Redis on CentOS 7](https://www.linode.com/docs/databases/redis/install-and-configure-redis-on-centos-7)
    - [How to install Redis 4 on Centos 6 / 7, Ubuntu 16 and Debian 8 Install Redis](https://www.hugeserver.com/kb/install-redis-4-centos-ubuntu-debian/)

  * Clustering Tutorial & Architecture
    - [Redis Tutorial Point](https://www.tutorialspoint.com/redis/index.htm)
    - [Redis Architecture Overview](http://www.redisgate.com/redis/configuration/redis_overview.php)
    - [Redis cluster tutorial](https://redis.io/topics/cluster-tutorial)

  * Clustering & Environment Setup
    - [How To Configure a Redis Cluster on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-configure-a-redis-cluster-on-centos-7)
    - [How To Configure a Redis Cluster on CentOS 7](https://rmohan.com/?p=6879)
    - [Cent OS Redis Cluster 구성 (Ruby)](http://binshuuuu.tistory.com/30)
    - [Redis Cluster 구성 (Java)](http://firstboos.tistory.com/entry/Redis-Cluster-%EA%B5%AC%EC%84%B1)
    - [Redis Cluster 설치 및 구성 java Source](https://www.mynotes.kr/redis-cluster-%EC%84%A4%EC%B9%98-%EB%B0%8F-%EA%B5%AC%EC%84%B1-java-source/)
    - [Redis 설치 및 Cluster 구성](https://brunch.co.kr/@daniellim/31)
    - [Redis cluster. Quick overview](https://ilyabylich.svbtle.com/redis-cluster-quick-overview)
    - [Redis Clustering - Setup on Centos 6 and Advanced Failover Testing](http://ericgreer.info/guides/linux/2014/05/17/redis-clustering-setup-on-centos-6.html)

  * Clustering & Environment Setup
    - [Redis Sentinel Documentation](https://redis.io/topics/sentinel)
    - [Redis 설치부터 Sentinel까지](http://crystalcube.co.kr/175)
    - [Redis, Sentinel 고가용성(HA) 설정과 운용방법, Python Client example](http://bryan.wiki/244)
