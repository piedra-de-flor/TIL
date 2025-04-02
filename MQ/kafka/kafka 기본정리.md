# Apache Kafka 개요

## 📌 Kafka란?
Apache Kafka는 대용량 데이터를 실시간으로 처리하고 스트리밍하는 분산 메시징 시스템입니다. 높은 처리량과 확장성을 제공하며, 데이터 파이프라인, 스트리밍 애플리케이션, 로그 집계 등에 사용됩니다.

---

## 🔹 Kafka의 주요 특징

### 1. **분산 시스템**
- 여러 개의 브로커(서버)로 구성되어 있어 확장성과 내결함성이 뛰어남

### 2. **고성능 및 고가용성**
- 데이터가 여러 개의 복제본으로 저장되므로 장애 발생 시에도 데이터 유실이 최소화됨

### 3. **높은 처리량**
- 배치 처리 및 스트리밍 처리를 지원하며, 초당 수백만 개의 메시지를 처리할 수 있음

### 4. **Publisher-Subscriber 모델 지원**
- 데이터를 Producer가 생성하고 Consumer가 구독하는 구조

### 5. **로그 저장소 역할**
- 데이터를 일정 기간 동안 유지하며, 장애 발생 시 재처리가 가능함

---

## 🔹 Kafka의 아키텍처

### 1. **Producer**
- 데이터를 생성하여 Kafka의 특정 토픽(Topic)으로 전송하는 역할

### 2. **Broker**
- 메시지를 저장하고 관리하는 Kafka 서버
- 여러 개의 Broker가 클러스터를 형성하며 데이터를 분산 처리

### 3. **Topic & Partition**
- Topic: 메시지를 저장하는 단위 (ex. 로그, 이벤트 데이터)
- Partition: 하나의 Topic을 여러 개의 파티션으로 나누어 병렬 처리 가능

### 4. **Consumer & Consumer Group**
- Consumer: 특정 Topic에서 메시지를 읽어 가는 역할
- Consumer Group: 여러 Consumer가 협력하여 데이터 처리를 분산 수행

### 5. **Zookeeper**
- 클러스터 상태 관리 및 노드 간 조정 역할 수행

---

## 🔹 Kafka의 장단점

### ✅ 장점
- **고성능 & 대용량 처리**: 초당 수백만 개의 메시지를 처리할 수 있음
- **확장성**: 브로커 및 파티션을 추가하여 손쉽게 확장 가능
- **내결함성**: 데이터 복제 및 분산 저장으로 안정적인 데이터 보장
- **유연한 데이터 처리**: 실시간 스트리밍과 배치 처리를 모두 지원
- **오픈소스 및 다양한 생태계**: 다양한 오픈소스 프레임워크(Flume, Spark, Storm 등)와 연동 가능

### ❌ 단점
- **운영 복잡성**: 클러스터를 관리하려면 Zookeeper와 브로커 설정이 필요함
- **데이터 순서 보장 어려움**: 여러 파티션에서 병렬 처리될 경우 메시지 순서가 보장되지 않을 수 있음
- **학습 곡선이 가파름**: 초보자가 설정 및 운영하기에 다소 어려움

---

## 🔹 Kafka 사용 방법

### 1. Kafka 설치
```sh
# Kafka 다운로드 및 압축 해제
wget https://downloads.apache.org/kafka/3.0.0/kafka_2.13-3.0.0.tgz
tar -xvzf kafka_2.13-3.0.0.tgz
cd kafka_2.13-3.0.0
```

### 2. Zookeeper 및 Kafka 실행
```sh
# Zookeeper 실행
bin/zookeeper-server-start.sh config/zookeeper.properties &

# Kafka 실행
bin/kafka-server-start.sh config/server.properties &
```

### 3. Topic 생성
```sh
bin/kafka-topics.sh --create --topic test-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1
```

### 4. Producer에서 메시지 전송
```sh
bin/kafka-console-producer.sh --topic test-topic --bootstrap-server localhost:9092
>
```

### 5. Consumer에서 메시지 수신
```sh
bin/kafka-console-consumer.sh --topic test-topic --from-beginning --bootstrap-server localhost:9092
```

---

## 🔹 Kafka 활용 사례

- **로그 수집 및 분석**: 실시간 로그 데이터를 수집하여 분석 시스템과 연동
- **실시간 데이터 스트리밍**: 주문 처리, 금융 거래 데이터, IoT 센서 데이터 분석
- **메시지 브로커**: 마이크로서비스 간 메시지 전달 및 이벤트 기반 아키텍처 구현
- **데이터 파이프라인**: 데이터 웨어하우스로 전송 및 실시간 ETL (Extract, Transform, Load)

---

## 🔹 결론
Kafka는 확장성과 성능이 뛰어나고, 대규모 데이터 스트리밍 및 실시간 분석에 최적화된 메시징 시스템입니다. 하지만 운영 및 설정이 복잡하므로,
적절한 아키텍처 설계와 운영 경험이 필요합니다.
이를 활용하여 안정적인 데이터 파이프라인과 스트리밍 시스템을 구축할 수 있습니다.

