
# Spring + Elasticsearch 성능 개선 트러블슈팅 문서

> **목표**: 동시 사용자 1000명 이상 처리 및 CPU 점유율 30% 이하 유지  
> **환경**: Spring Boot, Elasticsearch, Docker, Prometheus, nGrinder

---

## 📌 개요

- **부하 조건**: nGrinder로 2분간 100,000건 요청 발생
- **vuser 증가 테스트**: 1 → 1000까지 점진 증가
- **문제 발생 지점**: vuser 약 600 이상부터 에러 폭증
- **문제 증상**
    - Spring thread 수가 250 이상 증가하지 않음
    - Elasticsearch `NullPointerException`
    - nGrinder 커넥션 끊김 (`Connection reset` 등)
    - CPU, Memory는 여유 있음 (`docker stats` 기준)

---

## 🔧 초기 설정

### application.properties
```properties
server.tomcat.max-threads=500
server.tomcat.accept-count=500
server.tomcat.connection-timeout=5000
server.tomcat.threads.min-spare=100
```

### 문제점
- 일부 프로퍼티 인식 불가 → actuator로 확인 필요
- `server.tomcat.max-threads`가 적용되지 않음 → 의존성 문제 또는 오타 의심

---

## 🐳 Docker 환경 최적화

### JVM 컨테이너 리소스 인식
기본 JVM은 Docker의 리소스를 제대로 인식하지 못함

✅ 해결:
```dockerfile
ENTRYPOINT ["java",
  "-XX:+UseContainerSupport",
  "-XX:MaxRAMPercentage=75.0",
  "-XX:InitialRAMPercentage=50.0",
  "-jar", "/app.jar"]
```

### 확인 명령
```bash
docker stats
docker inspect <container_id> | grep -i cpus
```

---

## 🧵 스레드 제한 문제

### 스레드 수 확인
```bash
ps -T | wc -l      # BusyBox 기준 thread 수 확인
```

### actuator 활용
```http
/actuator/metrics/tomcat.threads.max
/actuator/metrics/tomcat.threads.busy
```

### CPU 코어 인식
```java
Runtime.getRuntime().availableProcessors();
```

---

## 🛠 Elasticsearch 연결 풀 문제

### RestClient 기본값
기본 커넥션 수가 낮아 대량 요청 시 병목 발생

✅ 해결:
```java
RestClientBuilder builder = RestClient.builder(...);
builder.setHttpClientConfigCallback(httpClientBuilder ->
    httpClientBuilder
        .setMaxConnTotal(1000)
        .setMaxConnPerRoute(500)
);
```

---

## 💥 DTO NullPointerException

### 문제
Elasticsearch에서 score, lentPrice 등의 필드가 `null` → DTO 생성 시 NPE

### 로그 예시
```
java.lang.NullPointerException: Cannot invoke "java.lang.Long.longValue()" because the return value of ...
```

### 해결 방법
- DTO 생성자에서 null 체크
- 또는 `Optional.ofNullable().orElse(default)` 처리

---

## 📉 nGrinder 부하 테스트 에러

### vuser 600 이상
- `Connection reset by peer`
- `SocketTimeoutException`
- `Connection refused`

→ 서버 병목 or 커넥션 풀 초과 확실

---

## 📊 Prometheus / Grafana 메트릭 활용

### 주요 지표
- `tomcat_threads_busy`
- `jvm_threads_live`
- `es_search_rejected_total`
- `process_cpu_usage`
- `jvm_memory_used_bytes`

→ Grafana 대시보드 연동하여 실시간 관측

---

## ✅ 정리

| 항목 | 조치 내용 |
|------|-----------|
| **Tomcat** | max-threads 재설정, actuator로 모니터링 |
| **JVM** | UseContainerSupport 설정, RAM 비율 조정 |
| **ES 커넥션 풀** | RestClient 설정으로 확장 |
| **DTO 오류** | null-safe 생성자 또는 유효성 체크 |
| **컨테이너 모니터링** | docker stats / inspect / Prometheus |
| **부하 테스트 대응** | 에러 유형 분석 + 서버 튜닝 연계 |

---

## 📎 참고 명령어 모음

```bash
# 현재 컨테이너 CPU/메모리 확인
docker stats

# 컨테이너의 CPU 개수 확인
docker inspect <container_id> | grep -i cpus

# JVM 내 CPU 코어 확인
java.lang.Runtime.getRuntime().availableProcessors();

# 컨테이너 내 스레드 수 확인
ps -T | wc -l

# actuator로 tomcat 스레드 확인
curl localhost:8080/actuator/metrics/tomcat.threads.max
```
