# Change Data Capture (CDC) — 완전 정리

## 개념

**CDC (Change Data Capture)** 는 **데이터베이스에서 발생하는 변경 사항 (INSERT, UPDATE, DELETE)** 을 **실시간 또는 근실시간으로 감지하여 다른 시스템으로 전달하는 기술**입니다.

CDC는 전통적인 Full Dump 또는 Batch ETL과 달리 **변경된 데이터만 추출**하므로 **실시간성, 효율성, 확장성** 면에서 우수합니다.

---

## CDC 도입의 필요성

| 기존 방식 | 한계 |
|-----------|------|
| Full Dump (전체 백업 후 전송) | 대용량 데이터에 비효율, 지연 큼, 중복 부담 |
| Batch ETL (주기적 쿼리) | 삭제 감지 어려움, 지연 발생, 부하 증가 |
| Application Level 이벤트 | 개발 비용 높음, 누락 가능성 |

 **CDC는 데이터 변경을 DB 수준에서 감지 → 처리 시스템으로 전파**함으로써 이 한계를 극복

---

## CDC 동작 방식 (종류별 비교)

| 방식 | 원리 | 장점 | 단점 |
|------|------|------|------|
| **① 트리거 기반** | DB 트리거로 로그 테이블에 기록 | - 정확함<br>- 모든 변경 추적 | - DB 부하 큼<br>- 관리 복잡 |
| **② 타임스탬프 기반** | `updated_at` 컬럼 비교 | - 구현 쉬움 | - 삭제 감지 불가<br>- 누락 위험 |
| **③ 로그 기반 (binlog, WAL, oplog)** | DB 내부 변경 로그 분석 | - 성능 우수<br>- 삭제 추적 가능 | - 로그 접근 필요<br>- 초기 설정 복잡 |

→ **실무에서는 로그 기반 CDC가 가장 많이 사용됨**

---

## 주요 CDC 도구 비교

| 도구 | 특징 | 장점 | 단점 |
|------|------|------|------|
| **Debezium** | Kafka 기반 오픈소스 | 다양한 DB 지원, Kafka와 연동 용이 | Kafka 전제, 설정 복잡 |
| **AWS DMS** | AWS CDC 솔루션 | 관리형, 간편 설정 | AWS 종속성 |
| **Oracle GoldenGate** | 상용 Oracle 전용 | 신뢰성, 고성능 | 비용 높음 |
| **Maxwell's Daemon** | MySQL binlog 기반 | 경량화, JSON output | 기능 한계 있음 |
| **Fivetran / Airbyte** | GUI 기반 통합 파이프라인 | 코드 없이 구성 가능 | 커스터마이징 제약 |

---

## CDC → 메시징 → 싱크 구조 예시

```plaintext
[MySQL binlog]
    ↓
[Debezium Connector]
    ↓
Kafka → Kafka Topic → Kafka Connect Sink
    ↓
Elasticsearch / MongoDB / PostgreSQL
```

---

## CDC 이벤트 구조 (Debezium 기준)

```json
{
  "before": { "id": 1, "price": 100 },
  "after":  { "id": 1, "price": 120 },
  "op": "u",      // "c"=create, "u"=update, "d"=delete
  "ts_ms": 1628272894738
}
```

---

## CDC 실무 활용 예시

| 분야 | 활용 방식 |
|------|-----------|
| 실시간 검색 | RDB → Kafka → Elasticsearch |
| 마이크로서비스 | DB 변경 → 이벤트 발행 (Event Sourcing 유사) |
| 데이터 웨어하우징 | OLTP → Kafka → Redshift/BigQuery |
| 보안/감사 | CDC 로그 → 감사 시스템 저장 |

---

## 실무 설계 시 고려 사항

### 1. Idempotency (중복 처리 방지)
- Kafka key 기반 UPSERT
- 이벤트 ID, 버전 정보 사용

### 2. 정합성 보장
- Kafka offset 순서 보장
- 파티션 키 유지로 순서 유지

### 3. 장애 복구 / DLQ
- Sink 오류 시 DLQ 구성
- Retention 시간 고려해 재처리

### 4. 보안 이슈
- binlog 접근 권한 관리
- 개인정보 마스킹

### 5. 스키마 진화 대응
- Schema Registry + Avro 사용
- Dynamic JSON 처리 대응

---

## 실시간성 확보 전략

| 전략 | 설명 |
|------|------|
| Snapshot 생략 | `snapshot.mode=schema_only` 또는 `never` 설정 |
| Event 필터링 | 필요한 테이블/컬럼만 감지 |
| Kafka 튜닝 | acks, linger.ms, compression.type 등 |
| Sink 병렬화 | tasks.max, async 처리 |
| Debezium 설정 | poll.interval.ms, buffer.size 조정 등 |

---

## CDC vs Event Sourcing vs CQRS

| 항목 | CDC | Event Sourcing | CQRS |
|------|-----|----------------|------|
| 목적 | DB 복제/동기화 | 상태 변경의 이벤트 기록 | 읽기/쓰기 분리 |
| 저장 방식 | 변경된 데이터 | 도메인 이벤트 로그 | 명령/조회 모델 분리 |
| Kafka 활용 | 중간 브로커 | 이벤트 저장소 | 읽기 side 구성 |
| 적합한 상황 | 이기종 시스템 연동 | 상태 이력 보존 | 복잡한 조회 성능 분리 |

---

## 테스트 및 운영 가이드

| 항목 | 권장 방안 |
|------|----------|
| 테스트 | DB 변경 → Kafka 메시지 → Sink 결과 검증 |
| 모니터링 | Kafka Connect metrics, 토픽 lag, 로그 확인 |
| 데이터 검증 | 원본 DB ↔ 대상 DB 비교 스크립트 |
| 스케일 아웃 | Connector, Sink, Kafka Broker 병렬화 구성 |

---

## 미래 확장

- LLM 기반 CDC 이벤트 요약/의미 추출
- CDC → 자동 감사 로그 변환 ("가격 100 → 120 수정됨")
- Kafka + Debezium + OpenSearch 실시간 분석 대시보드
- CQRS + CDC 융합으로 트리거리스 이벤트 시스템 구성


---

## CDC를 통해 확장할 수 있는 CS적 고민과 설계 관점

### 1. **데이터 일관성 (Data Consistency)**

| 질문 | 설명 |
|------|------|
| **Kafka topic의 메시지 순서를 어떻게 보장할까?** | Kafka 파티션 단위로만 순서 보장이 되며, 이를 유지하려면 파티셔닝 키를 일관되게 유지해야 함 |
| **동시성 변경이 발생한 경우, 어떤 데이터가 옳은가?** | 최종 일관성(Eventual Consistency)을 허용할지, 강한 일관성(Strong Consistency)이 필요한지 도메인별 판단 필요 |
| **CDC로 생성된 메시지가 실제로 Sink에 들어갔는지 어떻게 검증할 수 있을까?** | Exactly-once 처리를 위한 Kafka Connect Sink 설정 + DB ↔ 대상 시스템의 정합성 검증 로직 필요 |

---

### 2. **분산 시스템 이론 적용**

| 질문 | 설명 |
|------|------|
| **CAP Theorem에서 CDC는 어느 지점을 선택하는가?** | CDC 기반 시스템은 **가용성(A)**과 **파티션 허용성(P)**을 우선시함. 일관성(C)은 보완 기술로 관리 |
| **CDC 처리 중 일부 노드 실패 시 시스템은 어떻게 복구해야 하는가?** | Kafka 오프셋 기반으로 정확히 한 번 처리(Exactly-Once Semantics) 보장 필요. DLQ + 재처리 체계로 복원 가능 |
| **CDC 시스템에서 지연(Latency)은 어디서 발생하는가?** | Binlog → Kafka Connector polling 지연, Kafka 처리 지연, Sink 처리 시간 등 모든 단계에서 지연이 누적될 수 있음 |

---

### 3. **이벤트 처리 패턴과 Event-Driven Architecture**

| 질문 | 설명 |
|------|------|
| **이벤트 소싱(Event Sourcing)과 CDC의 근본적 차이는?** | CDC는 외부 DB의 *변경 결과*를 추적하는 반면, Event Sourcing은 시스템 자체가 *이벤트를 직접 생성*하고 그것이 진실의 원천(Source of Truth)이 됨 |
| **CDC와 CQRS를 결합하면 어떤 문제가 생길 수 있나?** | 쓰기 모델과 읽기 모델 간 비동기 지연 발생 가능. 동기화된 상태를 사용자에게 보여주려면 적절한 지연 허용 전략 필요 |
| **하나의 트랜잭션을 여러 마이크로서비스로 퍼뜨릴 때 이벤트 설계는 어떻게 할까?** | 이벤트 카디널리티, 순서 보장, 중복 이벤트 처리(idempotency), fail-over 전파 전략 등을 설계해야 함 |

---

### 4. **테스트 전략과 관찰 가능성(Observability)**

| 고민 포인트 | 설명 |
|-------------|------|
| **CDC 이벤트의 단위 테스트/통합 테스트는 어떻게 구성해야 할까?** | Kafka Embedded 서버 사용 or Mocking → Source/Connector/Sink 통합 흐름 테스트 |
| **장애가 발생했을 때 어떤 지표로 원인을 추적할 수 있을까?** | Lag 지표, DLQ에 쌓인 메시지 수, Connector 로그 등으로 문제 단계 구분 가능 |
| **Kafka Connect의 Task 병렬성이 성능에 어떤 영향을 주는가?** | `tasks.max` 값을 조정해 병렬성을 조절하되, DB의 binlog 소비 속도와 Sink의 처리 속도 사이 균형 필요 |

---

### 5. **보안 및 데이터 보호**

| 질문 | 설명 |
|------|------|
| **binlog에 민감 정보가 포함되어 있는 경우 어떻게 할 것인가?** | Debezium의 필드 마스킹 기능을 활용하거나 Kafka 중간에서 필터링 필요 |
| **Kafka 토픽의 보안은 어떻게 보장할 수 있을까?** | Kafka ACL, 전송 암호화 (TLS), 인증/인가 설정을 통해 보호 |
| **Sink로 보낸 후 로그를 얼마나 오래 보관할 것인가?** | GDPR이나 내부 감사 정책에 따라 로그 보존 기간, Masking 여부 결정 필요 |

---

### 6. **스키마 진화(Schema Evolution)**

| 질문 | 설명 |
|------|------|
| **컬럼이 추가되거나 이름이 변경되었을 때 CDC는 어떻게 대응할 수 있나?** | Debezium + Kafka Schema Registry + Avro를 사용하면 스키마 버전 관리 및 호환성 확인 가능 |
| **Sink의 스키마와 Source가 불일치할 경우 어떻게 처리할 것인가?** | 강제 변환, Null 허용, JSON dynamic schema 적용 등으로 대응 가능 |

---

