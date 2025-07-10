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


