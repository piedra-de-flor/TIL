# 📌 Elasticsearch(ES)와 트랜잭션(Tx)의 관계 및 재시도/실패 처리 전략

Elasticsearch(ES)는 **분산 검색 및 분석 엔진**으로, 일반적인 RDBMS와는 **트랜잭션(ACID) 특성이 다름**. 이에 따라 ES를 사용할 때 트랜잭션 관리 및 재시도 로직을 적절히 설계해야 함.

---

## 1️⃣ ES와 트랜잭션(Tx)의 관계

### 🔹 ES는 완전한 ACID 트랜잭션을 지원하지 않음
- ES는 기본적으로 **"최종 일관성(Eventual Consistency)"** 모델을 따름
- `refresh_interval`에 따라 **실시간으로 데이터가 반영되지 않을 수도 있음**
- ES는 **분산 시스템**이므로, 네트워크 장애나 노드 장애 발생 시 **데이터 정합성 문제 발생 가능**

### 🔹 ES의 트랜잭션적 특징
| 특징 | 설명 |
|------|------|
| **단일 문서 수준 트랜잭션** | ES는 개별 문서 수준에서만 **원자적(Atomic) 저장**을 보장 |
| **다중 문서 업데이트 시 일관성 없음** | `_bulk API`를 사용해도 중간에 일부 문서만 저장될 수 있음 |
| **동시성 제어** | `_seq_no`, `_primary_term`을 활용해 동시성 충돌 방지 가능 |
| **트랜잭션 롤백 미지원** | ES는 RDBMS처럼 트랜잭션 롤백을 지원하지 않음 |

### 🔹 ES에서 트랜잭션이 필요한 경우 해결 방법
1. **RDBMS에 저장 후 ES에 반영 (CQRS 패턴 활용)**
2. **Kafka, Redis 등의 이벤트 기반 시스템과 연계하여 보장**
3. **ES 내 `_version`, `_seq_no`, `_primary_term`을 활용하여 정합성 유지**

---

## 2️⃣ ES에서 재시도 로직 관리

### 🔹 ES 요청 실패 유형
| 실패 유형 | 원인 | 해결 방법 |
|---------|------|---------|
| **일시적 장애 (Timeout, Too many requests)** | ES 부하 증가, 네트워크 문제 | ✅ **재시도 로직 적용** (Exponential Backoff) |
| **데이터 충돌 (Version Conflict)** | 동시 업데이트로 인한 충돌 | ✅ `_seq_no` & `_primary_term` 기반 동시성 제어 |
| **노드 장애** | ES 노드 다운 또는 분할 브레인(Split Brain) 현상 | ✅ `cluster.routing.allocation` 설정 및 장애 조치 |
| **인덱스 매핑 오류** | 매핑 타입 불일치 | ✅ 인덱스 매핑 사전 점검 및 동적 매핑 활성화 조정 |

### 🔹 **재시도 로직 설계**
- **재시도 횟수 설정**: 최대 3~5회 (`MAX_RETRIES`)
- **지수 백오프 (Exponential Backoff)** 적용: 첫 재시도 1초, 다음 2초, 4초... 증가
- **재시도 대상 예외만 처리** (예: `IOException`, `ElasticsearchTimeoutException`)

```java
private static final int MAX_RETRIES = 5;

public void indexWithRetry(String index, String documentId, String jsonData) {
    int attempt = 0;
    while (attempt < MAX_RETRIES) {
        try {
            elasticsearchClient.index(new IndexRequest(index).id(documentId).source(jsonData, XContentType.JSON));
            return; // 성공하면 종료
        } catch (ElasticsearchException e) {
            attempt++;
            log.warn("ES 인덱싱 실패 ({}회), 재시도 중...", attempt);
            sleepBeforeRetry(attempt);
        }
    }
    log.error("ES 인덱싱 최종 실패: {}", documentId);
    sendToFailureQueue(jsonData);
}

private void sleepBeforeRetry(int attempt) {
    try {
        Thread.sleep((long) Math.pow(2, attempt) * 100L); // 100ms, 200ms, 400ms...
    } catch (InterruptedException ignored) {}
}
```

---


## 3️⃣ ES에서 재시도 실패 시 관리

### 🔹 완전 실패(Failure) 처리 전략

| 전략 | 설명 |
|------|------|
| **Fallback (대체 저장소 사용)** | 실패 시 **DB 또는 Redis** 등에 임시 저장 후, 이후 동기화 |
| **Kafka, RabbitMQ 기반 비동기 재처리** | 실패 데이터를 메시지 큐에 저장하고, 별도 Worker가 재처리 |
| **운영자 개입을 위한 대시보드 구축** | Kibana/Prometheus/ELK로 모니터링 & 실패 로그 제공 |
| **Batch Job을 활용한 정합성 보정** | DB와 ES 간 정합성을 맞추는 주기적 싱크 배치 실행 |

---

### 🔹 Kafka를 활용한 재시도 실패 관리 예제

#### ✅ **ES 저장 실패 시 Kafka 토픽(`es-failure`)에 메시지 저장**
```java
try {
        elasticsearchClient.index(...);
        } catch (Exception e) {
        kafkaTemplate.send("es-failure", jsonData);
        }


```java
try {
    elasticsearchClient.index(...);
} catch (Exception e) {
    kafkaTemplate.send("es-failure", jsonData);
}
Kafka Consumer가 실패 데이터 재시도

java
Copy
Edit
@KafkaListener(topics = "es-failure", groupId = "es-retry-group")
public void retryFailedIndexing(String message) {
    try {
        elasticsearchClient.index(...);
    } catch (Exception e) {
        log.error("ES 재시도 실패, 다시 큐에 저장", e);
        kafkaTemplate.send("es-failure", message);
    }
}

```

## 4️⃣ ES 정합성 보정 (DB ↔ ES 싱크 맞추기)

### 🔹 **DB와 ES 정합성을 유지하는 방법**
- ✅ **DB → ES 데이터 싱크를 정기적으로 점검**
- ✅ **Batch Job을 통해 ES에서 누락된 데이터 보정**
- ✅ **ES의 `_seq_no` & `_primary_term`을 활용하여 최신 데이터 유지**

---

### 🔹 **Batch Job을 활용한 정합성 보정**
```java
@Scheduled(fixedRate = 60000) // 1분마다 실행
public void syncMissingDataToEs() {
    List<DataEntity> missingData = dataRepository.findMissingDataInEs();
    for (DataEntity data : missingData) {
        try {
            elasticsearchClient.index(...);
        } catch (Exception e) {
            log.error("ES 보정 실패: {}", data.getId(), e);
        }
    }
}
```
## 5️⃣ ES 장애 감지 및 모니터링

### 🔹 **ES 모니터링 주요 지표**
| 지표 | 설명 |
|------|------|
| **Cluster Health** | 클러스터 상태 확인 (`GET _cluster/health`) |
| **Pending Tasks** | 대기 중인 작업 점검 (`GET _cluster/pending_tasks`) |
| **Index 상태 점검** | 인덱스별 상태 확인 (`GET _cat/indices?v`) |
| **검색 성능 모니터링** | 노드별 검색 성능 분석 (`GET _nodes/stats`) |
| **Elasticsearch Slow Logs 설정** | 느린 쿼리 감지 (`index.search.slowlog.threshold.query.warn`) |

---

### 🔹 **Kibana, Prometheus, Grafana를 활용한 모니터링**
- ✅ **Kibana Dev Tools**로 ES 상태 점검
- ✅ **Prometheus + Grafana 연동**하여 ES 메트릭 시각화
- ✅ **Elasticsearch Alerting**을 통해 장애 감지 자동화

---

## 🎯 **결론**

- ✅ **ES는 ACID 트랜잭션을 지원하지 않으므로, RDBMS 또는 Kafka 기반으로 보완 필요**
- ✅ **ES 요청 실패 시 지수 백오프(Exponential Backoff) 적용하여 재시도 관리**
- ✅ **재시도 실패 시 Kafka/RabbitMQ 기반으로 비동기 처리**
- ✅ **Batch Job 또는 대시보드를 활용해 ES 정합성 유지**
- ✅ **Kibana, Prometheus, ELK Stack을 활용한 실시간 모니터링 필수**

> **🚀 ES는 RDBMS처럼 동작하지 않으므로, 데이터 정합성 및 장애 대응을 위한 체계적인 설계가 필수!**  
