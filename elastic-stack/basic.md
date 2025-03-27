# Elastic 기본 개념과 이론

## 목차

- [1. 기본](#1-기본)
  - [분산형 검색 및 분석 엔진](#분산형-검색-및-분석-엔진)
  - [핵심 용어](#핵심-용어)

- [2. Document 구조와 데이터 모델](#2-Document-구조와-데이터-모델)
  - [Document](#Document)
  - [Mapping](#Mapping)

- [3. 검색과 쿼리의 개념](#3-검색과-쿼리의-개념)
  - [Inverted Index(역색인) 구조](#Inverted-Index(역색인)-구조)
  - [Full-text Search(전문 검색)](#Full-text-Search(전문-검색))
  - [Query DSL](#Query-DSL)
  - [분석과 토큰화](#분석과-토큰화)

- [4. Elasticsearch 데이터 저장 및 검색 프로세스](#4-Elasticsearch-데이터-저장-및-검색-프로세스)
  - [데이터 저장 과정](#데이터-저장-과정)
  - [데이터 검색 과정](#데이터-검색-과정)

- [5. Elastic Stack 구성 요소](#5-Elastic-Stack-구성-요소)
  - [Kibana](#Kibana)
  - [Logstash](#Logstash)
  - [Beats](#Beats)

- [6. 분산 시스템 이론](#6-분산-시스템-이론)
  - [분산 아키텍처](#분산-아키텍처)
  - [CAP 이론](#CAP-이론)

- [7. 성능 최적화](#7-성능-최적화)
  - [Index 설계](#Index-설계)
  - [Query 최적화](#Query-최적화)

---

## 1. 기본

### 분산형 검색 및 분석 엔진

- Apache Lucene을 기반으로 구축된 분산형 RESTful 검색 및 분석 엔진
- JSON 기반 document를 저장하고 실시간 검색 및 분석 가능
- 대량의 데이터를 처리하기 위해 수평적 확장 가능

---

### 핵심 용어

- **Cluster**: 여러 node의 집합
  - 같은 cluster에 속한 node는 데이터를 공유하고 부하를 분산
  - cluster의 이름을 기반으로 같은 cluster인지 판별

- **Node**: elasticsearch의 단일 서버 인스턴스
  - master node, data node, coordinating node 등의 역할이 있음
  - node들은 서로 데이터를 주고 받으며 분산 저장 구조를 형성

- **Index**: document들의 논리적 컨테이너
  - 하나의 Index는 여러개의 document를 포함

- **Shard**: index 데이터를 나누어 저장하는 단위(수평적 확장 가능)
  - 데이터 양이 많을 경우 sharding을 통해 분산 처리
  - primary shard, replica shard가 있음

---

## 2. Document 구조와 데이터 모델

### Document

- elasticsearch는 json 형식의 문서를 저장
- 각 문서는 unique한 id를 가지며 필드와 값으로 구성
- schemaless하지만, mapping을 통해 schema 정의 가능

---

### Mapping

- document의 필드와 데이터 타입을 정의
- 동적, 명시적 mapping이 있음
- 필드 타입: text, keyword, date, numeric, boolean, geo 등

---

## 3. 검색과 쿼리의 개념

### Inverted Index(역색인) 구조

- 데이터를 저장할 때 미리 색인을 생성하여 빠르게 검색 가능
- 검색어를 키워드 단위로 분리하여 인덱싱(토큰화)

---

### Full-text Search(전문 검색)

- 검색어를 자연어로 입력해도 유사도를 고려하여 검색 가능
- elastic은 TF-IDF 및 BM25 알고리즘을 활용하여 검색 결과를 랭킹

---

### Query DSL

- json 기반의 강력한 query language를 제공
- 다양한 query 유형:
  - match query: 전문 검색
  - term query: 정확한 값 검색
  - range query: 범위 검색
  - bool query: 여러 query 조합

---

### 분석과 토큰화

- 텍스트를 검색 가능한 토큰으로 변환하는 과정
- analyzer는 tokenizer와 filter로 구성
- 언어별 분석기를 제공(영어, 한국어 등)

---

## 4. Elasticsearch 데이터 저장 및 검색 프로세스

### 데이터 저장 과정

1. json 데이터 입력(POST 요청)
2. inverted index(역색인) 생성(검색 속도 향상)
3. shard에 분산 저장
4. replica shard를 생성하여 백업 유지

---

### 데이터 검색 과정

1. 검색 요청(GET) -> coordinating node에서 요청 분배
2. 각 shard에서 병렬 검색 수행
3. TF-IDF 또는 BM25 알고리즘을 통해 결과 랭킹
4. 최종 결과를 사용자에게 반환

---

## 5. Elastic Stack 구성 요소

### Kibana

- elasticsearch를 위한 시각화 및 관리 플랫폼
- 대시보드, 차트, 지도 등 다양한 시각화 제공
- 데이터 탐색 및 분석 도구

---

### Logstash

- 데이터 수집 및 변환 pipeline
- 다양한 소스에서 데이터를 수집하고 가공한 후 elasticsearch로 전송

---

### Beats

- 경량 데이터 수집기
- Filebeat(로그), Metricbeat(시스템 메트릭), Packetbeat(네트워크) 등

---

## 6. 분산 시스템 이론

### 분산 아키텍처

- 데이터는 여러 node에 분산 저장
- cluster 내 모든 node는 서로 통신
- master node가 custer 상태를 관리

---

### CAP 이론

- consistency(일관성), availability(가용성), partition tolerance(분할 내구성)
- elasticsearch는 기본적으로 AP 시스템이지만, 설정을 통해 조정 가능

---

## 7. 성능 최적화

### Index 설계

- 적절한 shard 개수 설정
- 필드 mapping 최적화
- index life cycle 관리

---

### Query 최적화

- 필터 컨텍스트 활용
- 집계 성능 향상 기법
- 캐싱 활용

---