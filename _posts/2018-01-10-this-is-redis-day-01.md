---
title: This is redis Day-01
category: Server
excerpt: |
  This is Redis Book Review
feature_text: |
  ## This is redis Day-01
feature_image: "https://kingbbode.github.io/images/2016/2016_12_04_Spring_Boot_Redis/redis.png"
image: "https://kingbbode.github.io/images/2016/2016_12_04_Spring_Boot_Redis/redis.png"
---

---
## This is redis Day-01
---

## Introduction

  * NoSQL: 빅데이터를 처리하기위한 데이터저장소, 데이터 증가량을 측정하기 불가능하거나 서비스 요청량의 증가를 예측하기 어려운 상황에서 적합
  * Scale-Up: 단일서버의 성능을 증가시켜서 더 많은 요청을 처리하는 방법, 서비스의 중단이나 추가비용이 발생함
  * Scale-Out: 동일한 사양의 새로운 서버를 추가하는 방법
  * MySQL 성능증가를 위한 스케일 업
    - CPU 코어를 4개부터 증가시켜서 성능 테스트를 한 결과 16코어를 넘어간 순간부터 성능증가가 미미함
    - MySQL Connection 또한 16개가 넘어가는 순간부터 성능 증가가 미미함
  * Redis: 모든 데이터를 메모리에 저장하고 조회, 인메모리 데이터베이스 솔루션, 다양한 자료구조의 형태를 가짐
    - 영속성 지원
    - 읽기 성능 증대를 위한 서버 측 복제 지원
    - <b>쓰기 성능 증대를 위한 클라이언트 측 샤딩 지원</b>

      ```text
      샤딩과 파티셔닝의 차이점
      - 샤딩: 주로 키값(해쉬값 또는 특정 컬럼값)을 이용하여 테이블들의 데이터 자체를 나눠서 분산저장하거나, 특정 분류기준을 가지고(저장 데이터 종류-유저, 일반데이터 등) 테이블을 분류하여 저장하는 것을 말한다.
      - 파티셔닝: 수평파티셔닝과 수직파티셔닝이 있는데, 수평파티셔닝은 샤딩 중 값을 기준으로 나누는 방식과 동일하고, 수직파티셔닝은 모든 컬럼들 중 특정 컬럼들을 쪼개서 따로 저장하는 형태를 의미한다.
      ```

      샤딩관련 그림 및 참고사이트

      <a href="http://mongodb.citsoft.net/?page_id=225#comment-91922">샤딩 시스템 개요</a>
      <img src="http://mongodb.citsoft.net/wp-content/uploads/pic4-1.png">

    - 문자열, 리스트, 해시, 셋, 정렬된 셋과 같은 다양한 데이터형을 지원함

    - 레디스와 맴캐시드의 차이

      <a href="http://nosql-database.org">NoSQL List</a>
      ```text
      - 맴캐시드: 고성능 분산 메모리 객체 캐싱 시스템, 동적 웹 서비스의 DB부하를 경감시키는 것이 목적, Not NoSQL
      - 레디스: 향상된 키-값 저장소, 데이터 구조 서버로 활용
      ```

## Install Redis on Linux

  ```text
  설치환경: CentOS 7.x
  구조: 3대의 물리서버, 1대는 Master, 2대는 Slave
  기타: 장애대응을 위해 Redis의 Sentinel을 활용하여 총 3대에 모두 설치한다.
  ```
  * Redis 설치 및 환경설정

    ```text
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


  * Redis 상태체크 및 테스트

    ```text
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
    ```

  * Redis-Cli Info

    ```text
    # Server: 실행 중인 레디스 서버의 실행 정보
    .......
    # Clients: 서버와 연결된 클라이언트들의 통계 정보
    .......
    # Memory: 메모리 사용 정보 및 메모리 통계 정보
    * _human이 붙은 경우 사용자가 읽기 쉽게 메가바이트 또는 기가바이트 단위로 표현
    - used_memory: 레디스 서버가 사용하는 메모리의 양
    - used_memory_peak: 레디스 서버가 최대로 사용했던 메모리 크기
    - mem_fragment_ratio: 레디스 서버에 저장된 데이터 중에 연속되지 않은 메모리 공간에 저장되어있는 비율
    .......
    # Persistence: 영구 저장소와 관련된 저장 정보
    .......
    # Stats: 명령 수행과 저장된 키에 대한 통계 정보
    .......
    # Replication: 현재 동작 중인 복제 정보
    .......
    # CPU: CPU 사용률 통계 정보
    .......
    # Keyspace: 데이터베이스별로 저장된 키 정보
    .......
    ```

## Redis Basic Command

  * 문자열 명령: 문자열을 다루는 데 사용하는 명령의 집합

    ```text
    * set: 주어진 키에 값을 저장, 하나의 키와 하나의 값을 저장
    * get: 주어진 키로 값을 조회
    * append: 주어진 키가 존재하면 함께 입력된 값을 이미 저장되어 있는 제일 마지막 값의 뒤에 추가, 단 문자열 값만 유효
    * incr: 저장된 데이터의 값을 1씩 증가시킴
    * decr: 저장된 데이터의 값을 1씩 감소시킴
    ```

  * 리스트 명령: 리스트 데이터를 다루기 위한 명령의 집합, 논리적으로 링크드 리스트의 구현체

    ```text
    * lpush: 지정된 리스트의 맨 앞쪽에 입력된 요소를 저장
    * lrange: 지정된 리스트의 시작인덱스부터 종료인덱스 범위의 요소를 조회
    ```

  * 셋 명령: 셋 데이터를 다루는 명령의 집합, 순서가 보장되지 않으며 중복을 허용하지 않음

    ```text
    * sadd: 지정된 셋에 입력된 값을 저장
    * smembers: 지정된 셋에 저장된 모든 값의 목록을 조회함
    ```

  * 정렬된 셋 명령: 정렬된 셋 데이터를 다루는 명령의 집합, 저장된 요소에 가중치를 부여하여 작은 값부터 큰 값으로 정렬을 제공

    ```text
    * zadd: 정렬된 셋에 가중치와 값으로 이루어진 데이터를 저장
    * zrange: 정렬된 셋의 시작인덱스부터 종료인덱스 범위에 해당하는 값들을 가중치 오름차순으로 조회
    ```

  * 해시 명령: 해시 데이터를 다루는 명령의 집합, 키와 값이 쌍으로 이루어진 데이터를 저장하는 자료구조

    ```text
    * hset: 지정된 해시에 요청한 필드와 값을 저장, 요청한 필드가 존재할 때는 저장된 값에 업데이트
    * hget: 지정된 해시에 저장된 필드의 값을 조회
    * hgetail: 지정된 키에 저장된 모든 필드와 값을 조회
    ```

## Redis Performance Monitor

  * redis-benchmark
