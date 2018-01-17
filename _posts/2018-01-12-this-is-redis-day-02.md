---
title: This is redis Day-02
category: Server
excerpt: |
  This is Redis Book Review
feature_text: |
  ## This is redis Day-02
feature_image: "https://kingbbode.github.io/images/2016/2016_12_04_Spring_Boot_Redis/redis.png"
image: "https://kingbbode.github.io/images/2016/2016_12_04_Spring_Boot_Redis/redis.png"
---

### NOSQL

  * NoSQL: 마틴 파울러의 "NoSQL: 빅데이터 세상으로 떠나는 간결한 안내서"에 의한 정의
    - 대용량 웹 서비스를 위하여 만들어진 데이터 저장소
    - 관계형 데이터 모델을 지양하며 대량의 분산된 데이터를 저장하고 조회하는데 특화된 저장소
    - 스키마 없이 사용가능하거나 느슨한 스미카를 제공하는 저장소


  * CAP정리: 컴퓨터 과학 분야에서 분산 컴퓨터 시스템을 설명하는데 사용되는 이론
    - 일관성: 동시성 또는 동일성, '다중 클라이언트에서 같은 시간에 조회하는 데이터는 항상 동일한 데이터임을 보증하는 것
    - 가용성: 모든 클라이언트의 읽기와 쓰기 요청에 대하여 항상 응답이 가능해야 함을 보증하는 것
    - 네트워크 분할 허용성: 지역적으로 분할된 네트워크 환경에서 동작하는 시스템에서 두 지역 간의 네트워크가 단절되거나 네트워크 데이터의 유실이 일어나더라도 각 지역 내의 시스템은 정상적으로 동작해야 함을 의미


  * 데이터 구조에 따른 분류
    - 키-값 모델 NoSQL: 키 하나로 데이터 하나를 저장하고 조회할 수 있는 단일 키-값 구조를 가짐
    - 문서 모델 NoSQL: 하나의 키(like URL, JSON key)에 하나의 구조화된 문서를 저장하고 조회함
    - 컬럼 모델 NoSQL: 하나의 키에 여러 개의 컬럼 이름과 컬럼 값의 쌍으로 이루어진 데이터를 저장하고 조회함
    - 그래프 모델: 노드와 관계를 사용하여 데이터를 저장하고 조회함
