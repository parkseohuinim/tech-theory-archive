# Elastic 기본 개념과 이론

<h2>1. 기본</h2>
<h3>분산형 검색 및 분석 엔진</h3>

- Apache Lucene을 기반으로 구축된 분산형 RESTful 검색 및 분석 엔진
- JSON 기반 Document를 저장하고 실시간 검색 및 분석 가능
- 대량의 데이터를 처리하기 위해 수평적 확장 가능

<h3>핵심 용어</h3>

- **Cluster**: 여러 Node의 집합
  - 같은 Cluster에 속한 Node는 데이터를 공유하고 부하를 분산
  - Cluster의 이름을 기반으로 같은 Cluster인지 판별

- **Node**: Elasticsearch의 단일 서버 인스턴스
  - Master Node, Data Node, Coordinating Node 등의 역할이 있음
  - Node들은 서로 데이터를 주고 받으며 분산 저장 구조를 형성

- **Index**: Document들의 논리적 컨테이너
  - 하나의 Index는 여러개의 Document를 포함

- **Shard**: Index 데이터를 나누어 저장하는 단위(수평적 확장 가능)
  - 데이터 양이 많을 경우 Sharding을 통해 분산 처리ssss
  - Primary Shard, Replica Shard가 있음


<h2>2. Document 구조와 데이터 모델</h2>

<h3>Document</h3>

- Elasticsearch는 JSON 형식의 문서를 저장
- 각 문서는 Unique한 ID를 가지며 필드와 값으로 구성
- Schemaless하지만, Mapping을 통해 Schema 정의 가능

<h3>Mapping</h3>

- Document의 필드와 데이터 타입을 정의
- 동적, 명시적 Mapping이 있음
- 필드 타입: text, keyword, date, numeric, boolean, geo 등

<h2>3. 검색과 쿼리의 개념</h2>

<h3>Inverted Index(역색인) 구조</h3>

- 데이터를 저장할 때 미리 색인을 생성하여 빠르게 검색 가능
- 검색어를 키워드 단위로 분리하여 인덱싱(토큰화)

<h3>Full-text Search(전문 검색)</h3>

- 검색어를 자연어로 입력해도 유사도를 고려하여 검색 가능
- Elastic은 TF-IDF 및 BM25 알고리즘을 활용하여 검색 결과를 랭킹

<h3>Query DSL</h3>

- JSON 기반의 강력한 Query Language를 제공
- 다양한 Query 유형:
  - Match Query: 전문 검색
  - Term Query: 정확한 값 검색
  - Range Query: 범위 검색
  - Bool Query: 여러 Query 조합

<h3>분석과 토큰화</h3>

- 텍스트를 검색 가능한 토큰으로 변환하는 과정
- Analyzer는 Tokenizer와 Filter로 구성
- 언어별 분석기를 제공(영어, 한국어 등)

<h2>4. Elasticsearch 데이터 저장 및 검색 프로세스</h2>

<h3>데이터 저장 과정</h3>

1. JSON 데이터 입력(POST 요청)
2. Inverted Index(역색인) 생성(검색 속도 향상)
3. Shard에 분산 저장
4. Replica Shard를 생성하여 백업 유지

<h3>데이터 검색 과정</h3>

1. 검색 요청(GET) -> Coordinating Node에서 요청 분배
2. 각 Shard에서 병렬 검색 수행
3. TF-IDF 또는 BM25 알고리즘을 통해 결과 랭킹
4. 최종 결과를 사용자에게 반환

<h2>5. Elastic Stack 구성 요소</h2>

<h3>Kibana</h3>

- Elasticsearch를 위한 시각화 및 관리 플랫폼
- 대시보드, 차트, 지도 등 다양한 시각화 제공
- 데이터 탐색 및 분석 도구

<h3>Logstash</h3>

- 데이터 수집 및 변환 Pipeline
- 다양한 소스에서 데이터를 수집하고 가공한 후 Elasticsearch로 전송

<h3>Beats</h3>

- 경량 데이터 수집기
- Filebeat(로그), Metricbeat(시스템 메트릭), Packetbeat(네트워크) 등

<h2>6. 분산 시스템 이론</h2>

<h3>분산 아키텍처</h3>

- 데이터는 여러 Node에 분산 저장
- Cluster 내 모든 Node는 서로 통신
- Master Node가 Custer 상태를 관리

<h3>CAP 이론</h3>

- Consistency(일관성), Availability(가용성), Partition tolerance(분할 내구성)
- Elasticsearch는 기본적으로 AP 시스템이지만, 설정을 통해 조정 가능

<h2>7. 성능 최적화</h2>

<h3>Index 설계</h3>

- 적절한 Shard 개수 설정
- 필드 Mapping 최적화
- Index life cycle 관리

<h3>Query 최적화</h3>

- 필터 컨텍스트 활용
- 집계 성능 향상 기법
- 캐싱 활용