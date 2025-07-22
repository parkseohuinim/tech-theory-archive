# CDC를 활용한 RAG 시스템의 실시간 데이터 통합

## 목차

- [1. CDC(Change Data Capture)의 기본 개념](#1-CDC(Change Data Capture)의-기본-개념)
- [2. RAG(Retrieval-Augmented Generation)와 통합](#2-RAG(Retrieval-Augmented Generation)와-통합)
- [3. 시스템 복원력 향상 메커니즘](#3-시스템-복원력-향상-메커니즘)
  - [1. 시스템 신뢰성](#1-시스템-신뢰성)
  - [2. 일관성과 확장성](#2-일관성과-확장성)
  - [3. 이상 징후 탐지 강화](#3-이상-징후탐지-강화)
  - [4. 적응형 응답 메커니즘](#4-적응형-응답-메커니즘)
- [4. 실시간 RAG 업데이트의 중요성](#4-실시간-RAG-업데이트의-중요성)
- [5. CDC 구현 방식의 분류](#5-CDC-구현-방식의-분류)
  - [1. 로그 기반 CDC(권장)](#1-로그-기반-CDC(권장))
  - [2. 트리거 기반 CDC](#2-트리거-기반-CDC)
  - [3. 쿼리 기반 CDC](#3-쿼리-기반-CDC)
- [6. 구현 시 주요 복잡성](#6-구현-시-주요-복잡성)
  - [1. 스키마 진화 관리](#1-스키마-진화-관리)
  - [2. 데이터 변환 및 직렬화](#2-데이터-변환-및-직렬화)
  - [3. 멱등성 보장](#3-멱등성-보장)
- [7. 정리](#7-정리)
- [8. CDC 핵심 구현 분석](#8-CDC-핵심-구현-분석)
  - [1. PostgreSQL CDC 설정 및 WAL 구성](#1-PostgreSQL-CDC-설정-및-WAL-구성)
  - [2. Docker Compose PostgreSQL 설정](#2-Docker-Compose-PostgreSQL-설정)
  - [3. Debezium CDC 커넥터 설정 및 구성](#3-Debezium-CDC-커넥터-설정-및-구성)
  - [4. Kafka Connect 설정](#4-Kafka-Connect-설정)
  - [5. Docker Compose Kafka 설정](#5-Docker-Compose-Kafka-설정)
  - [6. CDC 컨슈머 구현 및 이벤트 처리](#6-CDC-컨슈머-구현-및-이벤트-처리)
  - [7. CDC 이벤트 메시지 구조 분석](#7-CDC-이벤트-메시지-구조-분석)
  - [8. CDC 이벤트 처리 API 엔드포인트](#8-CDC-이벤트-처리-API-엔드포인트)
  - [9. 통합 벡터 프로세서](#9-통합-벡터-프로세서)
  - [10. 통합 RAG 서비스](#10-통합-RAG-서비스)
  - [11. FastAPI 엔드포인트 설계](#11-FastAPI-엔드포인트-설계)
- [9. 결론](#9-결론)

---

## 1. CDC(Change Data Capture)의 기본 개념

- CDC는 시스템의 데이터베이스 **변경사항을 실시간으로 모니터링하고 기록**하는 기술로, 시스템 운영을 중단하지 않고, 지속적인 데이터 업데이트 가능
- **변경 감지와 처리 간의 시간 지연을 최소화**하여 **데이터가 항상 최신 상태를 유지**

---

## 2. RAG(Retrieval-Augmented Generation)와 통합

- RAG는 검색 기반과 생성 기반 접근법을 결합한 고급 AI 프레임워크로, **외부 데이터베이스나 문서에서 가장 관련성 높은 정보를 검색**하고, 이를 바탕으로 **맥락적으로 적절한 응답 생성**
- CDC를 통해 통합된 **실시간 데이터로 RAG 시스템이 더욱 일관성있고, 정확하며 맥락적으로 관련성 있는 응답 생성** 가능

---

## 3. 시스템 복원력 향상 메커니즘

### 1. 시스템 신뢰성

- 지속적인 데이터 동기화를 통해 **모든 변경사항이 즉시 시스템 전반에 복제되어 데이터 손실 방지**
- **시스템 장애나 유지보수 기간에도 RAG 시스템 운영 지속** 가능

### 2. 일관성과 확장성

- CDC는 모든 AI 시스템 구성 요소가 **일관된 데이터에 접근**할 수 있도록 보장하여 **일관되고 정확한 응답 생성**에 필수적
- RAG 시스템이 확장되고 더 많은 데이터 소스를 통합할 때 효율적인 데이터 변경 관리 지원

### 3. 이상 징후 탐지 강화

- 실시간 데이터 통합을 통해 AI 시스템이 표준에서 벗어난 변화를 더 효과적으로 식별하고 문제가 확대되기 전에 이해 관계자에게 경고

### 4. 적응형 응답 메커니즘

- CDC가 제공하는 지속적인 최신 데이터 스트림을 통해 RAG 모델이 실시간으로 출력을 조정할 수 있는 동적 적응성 제공

---

## 4. 실시간 RAG 업데이트의 중요성

- 대규모로 운영되는 RAG 시스템에서 기반 지식 베이스의 **신선도는 "있으면 좋은" 기능이 아닌 답변 품질과 사용자 신뢰의 근본적 결정**을 요소
- 배치 업데이트는 구현이 간단하지만 지연을 초래하여 불완전한 정보 응답을 야기할 수 있음

---

## 5. CDC 구현 방식의 분류

### 1. 로그 기반 CDC(권장)

- 대부분의 프로덕션 데이터베이스는 복제, 복구, 특정 시점 복원을 위해 트랜잭션 로그를 유지하며, 로그 기반 CDC 도구는 이러한 네이티브 로그를 활용하여 **비침습적으로 커밋된 변경사항의 스트림을 읽음**
- 높은 충실도, 소스 데이터베이스에 대한 **낮은 오버헤드**, 삭제 및 스키마 수정을 포함한 모든 변경 사항 캡처
- Debezium과 같은 도구가 Apache Kafka 등의 플랫폼으로 변경 이벤트를 스트리밍

### 2. 트리거 기반 CDC

- 모니터링하려는 테이블에 데이터베이스 트리거(AFTER INSERT, AFTER UPDATEm AFTER DELETE)를 생성하는 기법

### 3. 쿼리 기반 CDC

- 행이 마지막으로 수정된 시점을 나타내는 소스 테이블의 컬럼에 의존하는 방법으로, 가장 침습적이지 않지만 삭제를 안정적으로 캡처할 수 없고 중간 업데이트를 놓칠 수 있는 단점

---

## 6. 구현 시 주요 복잡성

### 1. 스키마 진화 관리

- 소스 데이터베이스 스키마가 변경 될 때 **CDC 파이프라인과 RAG 업데이트 핸들러가 이러한 변경사항을 처리할 수 있어야 함**

### 2. 데이터 변환 및 직렬화

- 변경 이벤트는 종종 JSON이나 Avro 형식으로 생성되며, 스트림 프로세서와 RAG 업데이트 핸들러가 이러한 이벤트를 올바르게 역직렬화하고 해석할 수 있어야 함

### 3. 멱등성 보장

- 분산 시스템에서 메시지 전달 의미론(특히 재시도 포함)으로 인해 중복 이벤트 처리가 발생할 수 있으므로,  RAG 업데이트 핸들러는 **멱등성을 가지도록 설계**되어야 함

---

## 7. 정리

![cdc-rag-system-1](/Users/seohuipark/Desktop/Document/tech-theory-archive/ai/cdc-rag-system-1.png)

![cdc-rag-system-2](/Users/seohuipark/Desktop/Document/tech-theory-archive/ai/cdc-rag-system-2.png)

---

## 8. CDC 핵심 구현 분석

### 1. PostgreSQL CDC 설정 및 WAL 구성

```sql
-- CDC를 위한 PostgreSQL 설정
ALTER SYSTEM SET wal_level = logical;  -- 논리적 복제 활성화
ALTER SYSTEM SET max_replication_slots = 10;  -- 복제 슬롯 증가
ALTER SYSTEM SET max_wal_senders = 10;  -- WAL 전송자 증가
ALTER SYSTEM SET wal_keep_size = '1GB';  -- WAL 보관 크기

-- CDC 대상 테이블 생성
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    content TEXT NOT NULL,
    author_id INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_deleted BOOLEAN DEFAULT FALSE
);

CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    post_id INTEGER REFERENCES posts(id),
    author_id INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_deleted BOOLEAN DEFAULT FALSE
);

-- CDC를 위한 복제 슬롯 생성
SELECT pg_create_logical_replication_slot('debezium_slot', 'pgoutput');

-- CDC를 위한 Publication 생성
CREATE PUBLICATION debezium_pub FOR TABLE posts, comments, users;
```

### 2. Docker Compose PostgreSQL 설정

```yaml
# docker-compose.yml - PostgreSQL 서비스
postgres:
  image: postgres:15
  container_name: postgres
  environment:
    POSTGRES_DB: boarddb
    POSTGRES_USER: postgres
    POSTGRES_PASSWORD: password
    POSTGRES_INITDB_ARGS: "--encoding=UTF-8 --lc-collate=C --lc-ctype=C"
  volumes:
    - ./data/postgres:/var/lib/postgresql/data
    - ./infrastructure/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
  ports:
    - "5432:5432"
  command: >
    postgres
    -c wal_level=logical # CDC를 위한 WAL 레벨 설정
    -c max_replication_slots=10 # Debezium 같은 CDC 도구 연결 준비
    -c max_wal_senders=10 # 복제 연결 처리 용량
    -c wal_keep_size=1GB
    -c shared_preload_libraries=pg_stat_statements # 쿼리 성능 분석 확장 로드
```

### 3. Debezium CDC 커넥터 설정 및 구성

```json
// config/debezium/connector-config.json
{
  "name": "boarddb-connector", // 게시판 DB 전용 CDC 커넥터
  "config": {
    // PostgreSQL 전용 Debezium 커넥터 사용
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    
    // 데이터베이스 연결 설정
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "postgres",
    "database.password": "password",
    "database.dbname": "boarddb",
    "database.server.name": "cdc",
    
    // CDC 대상 테이블 설정
    "table.include.list": "public.posts,public.comments,public.users",
    "table.exclude.list": "",
    
    // 논리적 복제 설정
    "plugin.name": "pgoutput", // 플러그인으로 논리적 복제 활용
    "publication.autocreate.mode": "filtered",
    "slot.name": "debezium_slot", // 복제 슬롯 사용
    
    // 스냅샷 설정
    "snapshot.mode": "initial", // 최초 실행 시 기존 데이터 전체 캐치
    "snapshot.locking.mode": "minimal", // 테이블 락 최소화
    "snapshot.delay.ms": 0,
    
    // 이벤트 설정
    "include.schema.changes": true, // 스키마 변경사항도 포함
    "include.query": false,
    "tombstones.on.delete": true,
    
    // 오프셋 관리
    "offset.storage": "org.apache.kafka.connect.storage.FileOffsetBackingStore",
    "offset.storage.file.filename": "/tmp/connect.offsets",
    "offset.flush.interval.ms": 60000,
    
    // 에러 처리
    "errors.retry.timeout": 300000,
    "errors.retry.delay.max.ms": 10000,
    "errors.log.enable": true,
    "errors.log.include.messages": true
  }
}
```

### 4. Kafka Connect 설정

```properties
# infrastructure/kafka/connect.properties
bootstrap.servers=kafka:9092
group.id=connect-cluster
key.converter=org.apache.kafka.connect.json.JsonConverter # 메시지를 JSON 형태로 직렬화/역직렬화
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=false
value.converter.schemas.enable=false
offset.storage.topic=connect-offsets
config.storage.topic=connect-configs
status.storage.topic=connect-status
config.storage.replication.factor=1 # 개발 환경으로 인한 단일 브로커
offset.storage.replication.factor=1
status.storage.replication.factor=1
```

### 5. Docker Compose Kafka 설정

```yaml
# docker-compose.yml - Kafka 서비스
kafka: # 메시지 브로커, 데이터 스트림 저장소
  image: confluentinc/cp-kafka:7.4.0
  container_name: kafka
  depends_on:
    - zookeeper
  environment:
    KAFKA_BROKER_ID: 1
    KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
    KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
    KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1
    KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
    KAFKA_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
    KAFKA_JMX_PORT: 9101
    KAFKA_JMX_HOSTNAME: localhost
  ports:
    - "9092:9092"
    - "9101:9101"
  volumes:
    - ./data/kafka:/var/lib/kafka/data

kafka-connect: # Debezium 커넥터 실행 플랫폼
  image: confluentinc/cp-kafka-connect:7.4.0
  container_name: kafka-connect
  depends_on:
    - kafka
  environment:
    CONNECT_BOOTSTRAP_SERVERS: kafka:29092
    CONNECT_REST_PORT: 8083
    CONNECT_GROUP_ID: connect-cluster
    CONNECT_CONFIG_STORAGE_TOPIC: connect-configs
    CONNECT_OFFSET_STORAGE_TOPIC: connect-offsets
    CONNECT_STATUS_STORAGE_TOPIC: connect-status
    CONNECT_KEY_CONVERTER: org.apache.kafka.connect.json.JsonConverter
    CONNECT_VALUE_CONVERTER: org.apache.kafka.connect.json.JsonConverter
    CONNECT_KEY_CONVERTER_SCHEMAS_ENABLE: false
    CONNECT_VALUE_CONVERTER_SCHEMAS_ENABLE: false
    CONNECT_CONFIG_STORAGE_REPLICATION_FACTOR: 1
    CONNECT_OFFSET_STORAGE_REPLICATION_FACTOR: 1
    CONNECT_STATUS_STORAGE_REPLICATION_FACTOR: 1
    CONNECT_PLUGIN_PATH: "/usr/share/java,/usr/share/confluent-hub-components"
  ports:
    - "8083:8083"
  volumes:
    - ./infrastructure/kafka/connect.properties:/etc/kafka/connect.properties
```

```json
// config/debezium/connector-config.json
{
  "name": "boarddb-connector",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "postgres",
    "database.password": "password",
    "database.dbname": "boarddb",
    "database.server.name": "cdc",
    "table.include.list": "public.posts,public.comments,public.users",
    "plugin.name": "pgoutput",
    "publication.autocreate.mode": "filtered",
    "slot.name": "debezium_slot"
  }
}
```

### 6. CDC 컨슈머 구현 및 이벤트 처리

```python
import json
import logging
import threading
import time
from datetime import datetime
from kafka import KafkaConsumer, KafkaProducer
from kafka.errors import KafkaError
import requests

logger = logging.getLogger(__name__)

class CDCConsumer:
    """CDC 이벤트를 소비하고 처리하는 컨슈머"""
    
    def __init__(self):
        # Kafka 컨슈머 설정
        self.consumer = KafkaConsumer(
            'cdc.public.posts',  # CDC 토픽
            'cdc.public.comments',  # 댓글 CDC 토픽
            'cdc.public.users',  # 사용자 CDC 토픽
            bootstrap_servers=['localhost:9092'],
            auto_offset_reset='earliest',  # 처음부터 읽기
            enable_auto_commit=True,
            group_id='cdc-consumer-group',
            value_deserializer=lambda m: json.loads(m.decode('utf-8')),
            consumer_timeout_ms=1000
        )
        
        # API 서버 통신 설정
        self.api_base_url = "http://localhost:8000"
        self.session = requests.Session()
        self.session.timeout = 5
        
        # 하트비트 설정
        self.heartbeat_interval = 30  # 30초마다 하트비트
        self.last_heartbeat = time.time()
        
        # 스레드 관리
        self.running = False
        self.processing_thread = None
        self.heartbeat_thread = None
        
        logger.info("CDC Consumer initialized (Redis-free version)")
    
    def start(self):
        """CDC 컨슈머 시작"""
        self.running = True
        
        # 메시지 처리 스레드 시작
        self.processing_thread = threading.Thread(target=self._process_messages)
        self.processing_thread.daemon = True
        self.processing_thread.start()
        
        # 하트비트 스레드 시작
        self.heartbeat_thread = threading.Thread(target=self._heartbeat_loop)
        self.heartbeat_thread.daemon = True
        self.heartbeat_thread.start()
        
        logger.info("CDC Consumer started successfully")
    
    def stop(self):
        """CDC 컨슈머 중지"""
        self.running = False
        if self.consumer:
            self.consumer.close()
        logger.info("CDC Consumer stopped")
    
    def _process_messages(self):
        """메시지 처리 메인 루프"""
        logger.info("Vector processor started")
        logger.info("Vector processing thread started")
        
        try:
            for message in self.consumer:
                if not self.running:
                    break
                
                try:
                    self._handle_cdc_message(message)
                except Exception as e:
                    logger.error(f"Error processing message: {e}")
                    
        except Exception as e:
            logger.error(f"Error in message processing loop: {e}")
    
    def _handle_cdc_message(self, message):
        """CDC 메시지 처리 핵심 로직"""
        try:
            # 메시지 파싱
            data = message.value
            topic = message.topic
            partition = message.partition
            offset = message.offset
            
            logger.info(f"Processing message from topic: {topic}")
            
            # 스냅샷 단계 확인
            if self._is_snapshot_message(data):
                logger.info("Initial snapshot phase completed (0 records), starting real-time processing")
                return
            
            # 이벤트 타입별 처리
            operation = data.get('payload', {}).get('op')
            
            if operation == 'c':  # CREATE
                self._handle_create_event(data, topic)
            elif operation == 'u':  # UPDATE
                self._handle_update_event(data, topic)
            elif operation == 'd':  # DELETE
                self._handle_delete_event(data, topic)
            else:
                logger.warning(f"Unknown operation: {operation}")
                
        except Exception as e:
            logger.error(f"Error handling CDC message: {e}")
    
    def _is_snapshot_message(self, data):
        """스냅샷 메시지인지 확인"""
        return data.get('payload', {}).get('op') == 'r'  # READ (스냅샷)
    
    def _handle_create_event(self, data, topic):
        """CREATE 이벤트 처리"""
        try:
            payload = data['payload']
            after_data = payload['after']
            
            if topic == 'cdc.public.posts':
                post_id = after_data['id']
                title = after_data['title']
                logger.info(f"Creating post: {title}")
                
                # 데이터 플로우 이벤트 기록
                self.record_data_flow_event("debezium", "kafka", "event_published", f"게시글 {post_id} 이벤트 발행")
                
                # API 서버에 벡터 처리 요청
                self.notify_api_server_post_event(post_id)
                
                # 데이터 플로우 이벤트 기록
                self.record_data_flow_event("kafka", "vector_processor", "event_consumed", f"게시글 {post_id} 벡터 처리 시작")
                
            elif topic == 'cdc.public.comments':
                comment_id = after_data['id']
                logger.info(f"Creating comment: {comment_id}")
                # 댓글 처리 로직
                
        except Exception as e:
            logger.error(f"Error handling CREATE event: {e}")
    
    def _handle_update_event(self, data, topic):
        """UPDATE 이벤트 처리"""
        try:
            payload = data['payload']
            after_data = payload['after']
            
            if topic == 'cdc.public.posts':
                post_id = after_data['id']
                title = after_data['title']
                logger.info(f"Updating post: {title}")
                
                # API 서버에 벡터 처리 요청
                self.notify_api_server_post_event(post_id)
                
        except Exception as e:
            logger.error(f"Error handling UPDATE event: {e}")
    
    def _handle_delete_event(self, data, topic):
        """DELETE 이벤트 처리"""
        try:
            payload = data['payload']
            before_data = payload['before']
            
            if topic == 'cdc.public.posts':
                post_id = before_data['id']
                title = before_data['title']
                logger.info(f"Deleting post: {title}")
                
                # API 서버에 벡터 삭제 요청
                self.notify_api_server_post_delete_event(post_id)
                
        except Exception as e:
            logger.error(f"Error handling DELETE event: {e}")
    
    def notify_api_server_post_event(self, post_id: int):
        """API 서버에 게시글 이벤트 알림"""
        try:
            # 데이터 플로우 이벤트 기록 (Debezium → Kafka)
            self.record_data_flow_event("debezium", "kafka", "event_published", f"게시글 {post_id} 이벤트 발행")
            
            url = f"{self.api_base_url}/internal/cdc/post"
            params = {"post_id": post_id}
            
            response = self.session.post(url, params=params)
            
            if response.status_code == 200:
                logger.info(f"CDC: Successfully notified API server about post {post_id}")
                # 이벤트 처리 상태 업데이트
                self.notify_event_processed(post_id)
                
                # 데이터 플로우 이벤트 기록 (Kafka → Vector Processor)
                self.record_data_flow_event("kafka", "vector_processor", "event_consumed", f"게시글 {post_id} 벡터 처리 시작")
            else:
                logger.error(f"CDC: Failed to notify API server about post {post_id}: {response.status_code}")
                
        except Exception as e:
            logger.error(f"CDC: Error notifying API server about post {post_id}: {e}")
    
    def notify_api_server_post_delete_event(self, post_id: int):
        """API 서버에 게시글 삭제 이벤트 알림"""
        try:
            url = f"{self.api_base_url}/internal/cdc/post/delete"
            params = {"post_id": post_id}
            
            response = self.session.post(url, params=params)
            
            if response.status_code == 200:
                logger.info(f"CDC: Successfully notified API server about post deletion {post_id}")
            else:
                logger.error(f"CDC: Failed to notify API server about post deletion {post_id}: {response.status_code}")
                
        except Exception as e:
            logger.error(f"CDC: Error notifying API server about post deletion {post_id}: {e}")
    
    def record_data_flow_event(self, from_component: str, to_component: str, event_type: str, details: str = ""):
        """데이터 플로우 이벤트를 API 서버에 기록"""
        try:
            url = f"{self.api_base_url}/internal/data-flow/event"
            data = {
                "from": from_component,
                "to": to_component,
                "type": event_type,
                "details": details
            }
            response = self.session.post(url, json=data)
            if response.status_code == 200:
                logger.debug(f"Data flow event recorded: {from_component} -> {to_component}")
        except Exception as e:
            logger.debug(f"Failed to record data flow event: {e}")
    
    def notify_event_processed(self, post_id: int):
        """이벤트 처리 완료 알림"""
        try:
            url = f"{self.api_base_url}/internal/cdc/event-processed"
            params = {"post_id": post_id}
            self.session.post(url, params=params)
        except Exception as e:
            logger.debug(f"Failed to notify event processed: {e}")
    
    def _heartbeat_loop(self):
        """하트비트 전송 루프"""
        logger.info("Heartbeat thread started")
        
        while self.running:
            try:
                time.sleep(self.heartbeat_interval)
                self._send_heartbeat()
            except Exception as e:
                logger.error(f"Error in heartbeat loop: {e}")
    
    def _send_heartbeat(self):
        """하트비트 전송"""
        try:
            url = f"{self.api_base_url}/internal/cdc/heartbeat"
            response = self.session.post(url)
            
            if response.status_code == 200:
                self.last_heartbeat = time.time()
                logger.debug("Heartbeat sent successfully")
            else:
                logger.warning(f"Heartbeat failed: {response.status_code}")
                
        except Exception as e:
            logger.error(f"Error sending heartbeat: {e}")

# 메인 실행 함수
def main():
    """CDC 컨슈머 메인 함수"""
    logging.basicConfig(
        level=logging.INFO,
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    
    consumer = CDCConsumer()
    
    try:
        consumer.start()
        
        # 메인 스레드 대기
        while True:
            time.sleep(1)
            
    except KeyboardInterrupt:
        logger.info("Shutting down CDC Consumer...")
        consumer.stop()

if __name__ == "__main__":
    main()
```

### 7. CDC 이벤트 메시지 구조 분석

```json
// Debezium에서 생성되는 CDC 이벤트 메시지 예시
{
  "schema": {
    "type": "struct",
    "fields": [
      {"field": "id", "type": "int32"},
      {"field": "title", "type": "string"},
      {"field": "content", "type": "string"},
      {"field": "author_id", "type": "int32"},
      {"field": "created_at", "type": "int64"},
      {"field": "updated_at", "type": "int64"},
      {"field": "is_deleted", "type": "boolean"}
    ]
  },
  "payload": {
    "before": null,  // CREATE 이벤트에서는 null
    "after": {
      "id": 13,
      "title": "새로운 게시글",
      "content": "게시글 내용입니다.",
      "author_id": 1,
      "created_at": 1720944000000,
      "updated_at": 1720944000000,
      "is_deleted": false
    },
    "source": {
      "version": "2.4.0.Final",
      "connector": "postgresql",
      "name": "cdc",
      "ts_ms": 1720944000000,
      "snapshot": "false",
      "db": "boarddb",
      "sequence": "[null,\"1234567890\"]",
      "schema": "public",
      "table": "posts",
      "txId": 12345,
      "lsn": 1234567890,
      "xmin": null
    },
    "op": "c",  // c=CREATE, u=UPDATE, d=DELETE, r=READ(snapshot)
    "ts_ms": 1720944000000
  }
}
```

### 8. CDC 이벤트 처리 API 엔드포인트

```python
# backend/api/main.py - CDC 이벤트 처리 엔드포인트
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session
import logging

logger = logging.getLogger(__name__)

# Internal CDC endpoints (내부 통신용)
@app.post("/internal/cdc/post")
async def handle_cdc_post_event(post_id: int, db: Session = Depends(get_db)):
    """CDC 게시글 이벤트 처리 - 내부 통신용"""
    try:
        # 통합 벡터 프로세서 사용
        vector_processor, _ = get_integrated_services()
        
        # 벡터 인덱싱 처리
        vector_processor.process_vector_indexing(post_id)
        
        logger.info(f"CDC: Successfully processed vector indexing for post {post_id}")
        return {"message": f"Successfully processed post {post_id}"}
    
    except Exception as e:
        logger.error(f"Error processing CDC post event for {post_id}: {e}")
        return {"error": str(e)}

@app.post("/internal/cdc/post/delete")
async def handle_cdc_post_delete_event(post_id: int, db: Session = Depends(get_db)):
    """CDC 게시글 삭제 이벤트 처리 - 내부 통신용"""
    try:
        # 통합 벡터 프로세서 사용
        vector_processor, _ = get_integrated_services()
        
        # 벡터 삭제 처리
        vector_processor.process_vector_deletion(post_id)
        
        logger.info(f"CDC: Successfully processed vector deletion for post {post_id}")
        return {"message": f"Successfully deleted post {post_id}"}
    
    except Exception as e:
        logger.error(f"Error processing CDC post delete event for {post_id}: {e}")
        return {"error": str(e)}

@app.post("/internal/cdc/heartbeat")
async def cdc_heartbeat():
    """CDC 컨슈머 하트비트 수신"""
    global cdc_consumer_last_heartbeat
    cdc_consumer_last_heartbeat = datetime.now()
    return {"status": "heartbeat_received"}

@app.post("/internal/cdc/event-processed")
async def cdc_event_processed(post_id: int):
    """CDC 이벤트 처리 완료 알림"""
    global cdc_event_history
    
    # CDC 이벤트 히스토리에 처리 완료 기록
    event_data = {
        "type": "event_processed",
        "post_id": post_id,
        "details": f"게시글 {post_id} CDC 이벤트 처리 완료",
        "timestamp": datetime.now().isoformat()
    }
    
    cdc_event_history.append(event_data)
    
    # 최대 개수 제한
    if len(cdc_event_history) > MAX_EVENT_HISTORY:
        cdc_event_history.pop(0)
    
    return {"status": "event_processed"}

@app.post("/internal/data-flow/event")
async def record_data_flow_event(request: dict):
    """데이터 플로우 이벤트 기록"""
    global data_flow_events
    
    try:
        event_data = {
            "from": request.get("from", ""),
            "to": request.get("to", ""),
            "type": request.get("type", ""),
            "details": request.get("details", ""),
            "timestamp": datetime.now().isoformat()
        }
        
        data_flow_events.append(event_data)
        
        # 최대 개수 제한
        if len(data_flow_events) > MAX_FLOW_EVENTS:
            data_flow_events.pop(0)
        
        return {"status": "flow_event_recorded"}
    except Exception as e:
        logger.error(f"Error recording data flow event: {e}")
        return {"error": str(e)}
```

### 9. 통합 벡터 프로세서

```python
# backend/api/main.py - IntegratedVectorProcessor 클래스
class IntegratedVectorProcessor:
    def __init__(self):
        self.chroma_client = chromadb.PersistentClient(path="./data/chroma")
        self.collection = self.chroma_client.get_or_create_collection("posts")
        self.openai_client = OpenAI(api_key=config.openai_api_key)
    
    def process_vector_indexing(self, post_id: int):
        """게시글 벡터 인덱싱 처리"""
        try:
            # 1. 데이터베이스에서 게시글 조회
            post_data = self._get_post_data(post_id)
            
            # 2. 텍스트 결합 (제목 + 내용 + 댓글)
            combined_text = self._combine_post_content(post_data)
            
            # 3. OpenAI 임베딩 생성
            embedding = self._generate_embedding(combined_text)
            
            # 4. ChromaDB에 벡터 저장
            self._store_in_vector_db(post_id, embedding, post_data, combined_text)
            
            logger.info(f"Successfully indexed post {post_id}")
            
        except Exception as e:
            logger.error(f"Error processing vector indexing for post {post_id}: {e}")
            raise
    
    def _generate_embedding(self, text: str) -> List[float]:
        """OpenAI API를 사용한 텍스트 임베딩 생성"""
        try:
            response = self.openai_client.embeddings.create(
                model="text-embedding-3-small",
                input=text,
                encoding_format="float"
            )
            embedding = response.data[0].embedding
            logger.info(f"Generated embedding with {len(embedding)} dimensions")
            return embedding
            
        except Exception as e:
            logger.error(f"Error generating embedding: {e}")
            raise
    
    def search_similar_posts(self, query: str, limit: int = 5) -> List[Dict[str, Any]]:
        """유사한 게시글 검색"""
        try:
            # 1. 쿼리 임베딩 생성
            query_embedding = self._generate_embedding(query)
            
            # 2. ChromaDB에서 유사도 검색
            results = self.collection.query(
                query_embeddings=[query_embedding],
                n_results=limit,
                include=["metadatas", "documents"]
            )
            
            # 3. 결과 포맷팅
            formatted_results = []
            for i, (metadata, document, distance) in enumerate(zip(
                results['metadatas'][0], 
                results['documents'][0], 
                results['distances'][0]
            )):
                similarity_score = 1 - distance  # 거리를 유사도로 변환
                if similarity_score >= 0.1:  # 최소 유사도 임계값
                    formatted_results.append({
                        'post_id': metadata['post_id'],
                        'title': metadata['title'],
                        'content': document,
                        'score': similarity_score,
                        'created_at': metadata['created_at']
                    })
            
            return formatted_results
            
        except Exception as e:
            logger.error(f"Error searching similar posts: {e}")
            raise
```

### 10. 통합 RAG 서비스

```python
# backend/api/main.py - IntegratedRAGService 클래스
class IntegratedRAGService:
    def __init__(self, vector_processor: IntegratedVectorProcessor):
        self.vector_processor = vector_processor
        self.openai_client = OpenAI(api_key=config.openai_api_key)
    
    async def generate_answer(self, question: str, context_limit: int = 5) -> Dict[str, Any]:
        """RAG 답변 생성"""
        start_time = time.time()
        
        try:
            # 1. 관련 컨텍스트 검색
            retrieval_start = time.time()
            context_posts = await self._retrieve_relevant_context(question, context_limit)
            retrieval_time = time.time() - retrieval_start
            
            # 2. 컨텍스트 기반 답변 생성
            generation_start = time.time()
            answer, token_usage, prompt_content = await self._generate_contextual_answer(question, context_posts)
            generation_time = time.time() - generation_start
            
            # 3. 디버그 정보 수집
            debug_info = {
                'retrieval_time': retrieval_time,
                'generation_time': generation_time,
                'total_tokens_used': token_usage.get('total_tokens', 0),
                'prompt_tokens': token_usage.get('prompt_tokens', 0),
                'completion_tokens': token_usage.get('completion_tokens', 0),
                'context_length': len(str(context_posts)),
                'similarity_scores': [post.get('score', 0) for post in context_posts],
                'retrieved_context': self._format_context(context_posts),
                'prompt_content': prompt_content,
                'response_content': answer
            }
            
            return {
                'answer': answer,
                'sources': context_posts,
                'debug_info': debug_info
            }
            
        except Exception as e:
            logger.error(f"Error generating RAG answer: {e}")
            raise
    
    async def _retrieve_relevant_context(self, question: str, limit: int) -> List[Dict[str, Any]]:
        """질문과 관련된 컨텍스트 검색"""
        try:
            # 벡터 프로세서를 사용한 유사도 검색
            similar_posts = self.vector_processor.search_similar_posts(question, limit)
            return similar_posts
            
        except Exception as e:
            logger.error(f"Error retrieving context: {e}")
            raise
    
    async def _generate_contextual_answer(self, question: str, context_posts: List[Dict[str, Any]]) -> tuple[str, Dict[str, int], str]:
        """컨텍스트를 활용한 답변 생성"""
        try:
            # 1. 컨텍스트 포맷팅
            context = self._format_context(context_posts)
            
            # 2. RAG 프롬프트 생성
            prompt = self._create_rag_prompt(question, context)
            
            # 3. OpenAI API 호출
            response = self.openai_client.chat.completions.create(
                model="gpt-4o-mini",
                messages=[
                    {"role": "system", "content": self._get_system_prompt()},
                    {"role": "user", "content": prompt}
                ],
                temperature=0.7,
                max_tokens=1000
            )
            
            answer = response.choices[0].message.content
            token_usage = {
                'prompt_tokens': response.usage.prompt_tokens,
                'completion_tokens': response.usage.completion_tokens,
                'total_tokens': response.usage.total_tokens
            }
            
            return answer, token_usage, prompt
            
        except Exception as e:
            logger.error(f"Error generating contextual answer: {e}")
            raise
    
    def _create_rag_prompt(self, question: str, context: str) -> str:
        """RAG 프롬프트 생성"""
        return f"""
다음 컨텍스트를 참고하여 질문에 답변해주세요.

컨텍스트:
{context}

질문: {question}

답변:
"""
```

### 11. FastAPI 엔드포인트 설계

```python
# backend/api/main.py
from fastapi import FastAPI, Depends, HTTPException
from sqlalchemy.orm import Session
from typing import List

app = FastAPI(title="CDC-RAG System API")

# 게시글 CRUD 엔드포인트
@app.post("/posts", response_model=PostResponse)
async def create_post(post: PostCreate, db: Session = Depends(get_db)):
    """게시글 생성 - CDC 이벤트 트리거"""
    # 1. 데이터베이스에 게시글 저장
    db_post = Post(**post.dict(), author_id=1)
    db.add(db_post)
    db.commit()
    db.refresh(db_post)
    
    # 2. CDC 이벤트 기록
    add_cdc_event("post_created", db_post.id, f"게시글 생성: {db_post.title}")
    
    # 3. 데이터 플로우 이벤트 기록
    add_data_flow_event("postgresql", "debezium", "data_inserted", f"게시글 {db_post.id} 생성됨")
    
    return db_post

# 검색 엔드포인트
@app.post("/search", response_model=SearchResponse)
async def search_posts(search_request: SearchRequest):
    """벡터 기반 유사도 검색"""
    try:
        vector_processor, _ = get_integrated_services()
        search_results = vector_processor.search_similar_posts(
            query=search_request.query,
            limit=search_request.limit
        )
        
        return SearchResponse(
            results=search_results,
            query=search_request.query,
            total=len(search_results)
        )
    except Exception as e:
        logger.error(f"Error in search: {e}")
        raise HTTPException(status_code=500, detail="Search failed")

# RAG 엔드포인트
@app.post("/rag", response_model=RAGResponse)
async def ask_rag(rag_request: RAGRequest):
    """RAG 기반 질의응답"""
    try:
        _, rag_service = get_integrated_services()
        response = await rag_service.generate_answer(
            question=rag_request.question,
            context_limit=rag_request.context_limit
        )
        
        return RAGResponse(
            answer=response.get('answer', ''),
            sources=response.get('sources', []),
            question=rag_request.question,
            debug_info=response.get('debug_info')
        )
    except Exception as e:
        logger.error(f"Error in RAG: {e}")
        raise HTTPException(status_code=500, detail="RAG processing failed")

# 시스템 모니터링 엔드포인트
@app.get("/system/status")
async def get_system_status():
    """시스템 상태 모니터링"""
    try:
        # 데이터베이스 연결 확인
        db_status = check_database_connection()
        
        # Kafka 연결 확인
        kafka_status = check_kafka_connection()
        
        # ChromaDB 연결 확인
        chromadb_status = check_chromadb_connection()
        
        # CDC 컨슈머 상태 확인
        cdc_status = check_cdc_consumer_status()
        
        return {
            "database": db_status,
            "kafka": kafka_status,
            "chromadb": chromadb_status,
            "cdc_consumer": cdc_status,
            "timestamp": datetime.now().isoformat()
        }
    except Exception as e:
        logger.error(f"Error getting system status: {e}")
        raise HTTPException(status_code=500, detail="System status check failed")
```

---

## 9. 결론

1. **실시간성의 중요성**: 실시간 데이터 신선도 확보
2. **시스템 복원력**: 장애 상황에서도 지속적인 서비스 제공과 빠른 복구 능력
3. **확장성과 일관성**: 대규모 분산 환경에서도 데이터 일관성 유지
4. **기술적 복잡성**: 스키마 진화, 멱등성, 데이터 변환 등 구현 시 고려해야 할 다양한 기술적 과제

CDC는 단순한 데이터 동기화 도구를 넘어 AI 시스템의 신뢰성과 성능을 보장하는 핵심 인프라 구성 요소로 자리잡고 있으며, 특히 실시간 의사결정이 중요한 현대 비즈니스 환경에서 그 중요성이 더욱 부각되고 있다.
