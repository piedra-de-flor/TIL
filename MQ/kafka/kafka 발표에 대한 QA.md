# Kafka 성능 및 설정 관련 정리

## 1. Kafka의 레플리카 수 조정 기준
Kafka에서 레플리카(replication factor)를 조정할 때 고려해야 할 일반적인 기준은 다음과 같다.

- **가용성(Availability)**
    - 브로커 장애 발생 시 데이터를 복구할 수 있도록 최소 3개의 복제본을 유지하는 것이 일반적이다.

- **내구성(Durability)**
    - 데이터 유실을 방지하려면 `replication factor >= 3` 권장.
    - 최소 `min.insync.replicas = 2` 설정하여 과반수의 복제본이 유지되는지 확인.

- **성능(Performance)**
    - 높은 replication factor는 쓰기 성능을 저하시킬 수 있음.
    - 가용성과 성능 간 트레이드오프를 고려하여 조정해야 함.

- **비용(Cost)**
    - 높은 replication factor는 저장 비용 증가를 초래.
    - 필요 이상의 복제본을 유지하는 것은 비효율적일 수 있음.

### 일반적인 권장 사항
- 단일 노드 장애 대비: `replication factor = 3`
- 고가용성 요구되는 시스템: `replication factor = 5`
- 로그 데이터 등 중요도 낮은 데이터: `replication factor = 2`

---

## 2. Kafka 성능 테스트 방법
Kafka의 성능을 테스트하는 방법에는 여러 가지가 있으며, 대표적으로 다음과 같은 방법을 사용한다.

### 1) `kafka-producer-perf-test.sh` 사용
Kafka는 기본적으로 제공하는 `kafka-producer-perf-test.sh` 스크립트를 활용하여 성능 테스트 가능.

```bash
kafka-producer-perf-test.sh \
  --topic test-topic \
  --num-records 1000000 \
  --record-size 100 \
  --throughput -1 \
  --producer-props acks=all bootstrap.servers=localhost:9092
```

### 2) kafka-consumer-perf-test.sh 사용
```bash
kafka-consumer-perf-test.sh \
  --bootstrap-server localhost:9092 \
  --topic test-topic \
  --messages 1000000 \
  --group test-group
```

### 3) Apache JMeter 활용
3. Kafka 설정 변경 시 데이터 유실 고려 

---
<br>
Kafka 설정을 변경할 때 데이터 유실 가능성을 최소화하는 것이 중요하다. 특히 acks 설정이 데이터 유실과 성능에 큰 영향을 미친다.

acks=1 설정 시 데이터 유실 가능성
acks=1으로 설정하면 리더 브로커가 데이터를 수신하고 즉시 응답을 보낸다.

팔로워 복제본이 동기화되지 않더라도 ACK이 반환되므로 장애 발생 시 데이터 유실 가능성이 존재.

성능은 향상되지만 내구성이 약해짐.

대책:

장애 시 데이터 손실을 수용할 수 있는지 검토.

min.insync.replicas 설정으로 최소한의 동기화된 복제본 유지.

acks=all 설정 시 성능 저하 vs 데이터 안정성
acks=all 설정 시 모든 ISR(In-Sync Replicas)이 데이터를 저장해야 ACK을 반환.

데이터 안정성은 확보되지만 성능이 저하됨.

설정	데이터 안정성	성능
acks=0	매우 낮음 (데이터 유실 위험)	매우 빠름
acks=1	중간 (리더 장애 시 유실 가능)	빠름
acks=all	높음 (ISR 모두 저장 시 ACK 반환)	상대적으로 느림
트레이드오프 분석:

중요한 데이터는 acks=all + min.insync.replicas=2 설정 권장.

대량 트래픽 처리를 위해 acks=1을 고려할 수 있으나, 장애 대비 필요.

4. Kafka 성능 병목 원인
Kafka에서 성능 병목이 발생할 수 있는 주요 원인은 다음과 같다.

1) 네트워크 병목
브로커 간 데이터 복제 및 전송량이 많을 경우 네트워크 대역폭 제한 발생.

compression.type을 snappy 또는 lz4로 설정하여 트래픽 감소 가능.

2) 디스크 I/O 병목
Kafka는 기본적으로 디스크 기반 로그를 사용하므로 SSD 사용이 유리.

log.segment.bytes, log.retention.hours 조정을 통해 불필요한 디스크 사용량 줄이기.

3) 컨슈머 병목
컨슈머가 처리 속도가 느릴 경우 메시지 처리 지연 발생.

fetch.min.bytes 및 fetch.max.wait.ms 조정으로 효율적인 메시지 소비 가능.

4) GC(가비지 컬렉션) 문제
JVM 기반 Kafka에서는 GC가 성능 저하를 유발할 수 있음.

G1GC 또는 ZGC를 사용하여 GC 튜닝 가능.

5) Partition 개수 조정 문제
너무 적은 파티션: 병렬 처리 제한

너무 많은 파티션: 컨트롤러 오버헤드 증가

일반적으로 브로커당 100~200개 이하의 파티션 권장.

