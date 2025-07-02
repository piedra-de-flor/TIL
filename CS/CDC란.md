📌 Change Data Capture (CDC)
✅ 개념
**CDC(Change Data Capture)**는 **데이터베이스에서 발생하는 변경 사항(삽입, 수정, 삭제)**을 실시간 또는 준실시간으로 캡처하여 다른 시스템에 전달하는 기술입니다.
주로 실시간 데이터 파이프라인, 이벤트 기반 아키텍처, 데이터 웨어하우징 등에 사용됩니다.

🧩 CDC의 필요성
기존 방식	문제점
정기적인 전체 덤프 (Full Dump)	데이터 양 증가 시 비효율적, 리소스 낭비
Polling 방식 (주기적 질의)	지연 발생, 중복 가능성, DB 부하 증가

➡️ CDC는 변경된 데이터만 추적하므로 리소스 절약과 실시간성 확보에 유리합니다.

🔍 CDC 동작 방식
CDC는 변경 이력을 추적하기 위해 다양한 방식을 사용합니다:

1. 트리거 기반
   DB 트리거를 활용하여 변경 시 로그 테이블에 기록

✅ 장점: 모든 변경 사항 포착 가능

❌ 단점: 성능 부하 발생 가능

2. 타임스탬프 기반
   updated_at 등 타임스탬프 필드를 기준으로 변경 감지

✅ 장점: 구현 간단

❌ 단점: 삭제는 감지 불가, 정확도 낮음

3. 로그 기반 (Binary Log, WAL 등)
   DB 내부의 변경 로그(binlog, WAL)를 분석

✅ 장점: 실시간 처리, 삭제까지 추적 가능

❌ 단점: 로그 접근 권한 필요, 복잡한 파싱 필요

🔧 주요 CDC 도구
도구	특징
Debezium	Kafka 기반, MySQL/Postgres 등 다양한 DB 지원
AWS DMS	AWS 환경에서 CDC를 쉽게 구성 가능
Oracle GoldenGate	Oracle 전용 고급 CDC 솔루션
StreamSets / Fivetran	상용 ETL/CDC 플랫폼들

🎯 CDC 활용 사례
실시간 데이터 동기화 (DB → Kafka → Elasticsearch)

마이크로서비스 간 이벤트 발행

데이터 웨어하우징 (예: Redshift, BigQuery)

감사 로그 및 변경 이력 추적

⚠️ 주의사항
이중 처리 방지: 동일 이벤트 중복 처리 방지 필요 (idempotent 처리)

데이터 일관성 확보: 변경 순서 보장

보안 및 접근 제어: binlog 등 민감한 정보 노출 주의