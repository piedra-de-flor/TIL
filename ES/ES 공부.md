# Elasticsearch(ES)

## 1. Elasticsearch의 특징

- **분산 검색 엔진**: 데이터를 샤드(Shard)로 나누어 분산 저장 및 검색 가능
- **RESTful API 지원**: JSON 기반의 REST API를 제공하여 다양한 프로그래밍 언어에서 쉽게 접근 가능
- **실시간 검색 및 분석**: 대량의 데이터를 빠르게 검색하고 분석하는 기능 제공
- **Full-Text Search 지원**: 강력한 자연어 검색 기능 제공
- **확장성(Scalability)**: 노드를 추가하여 수평 확장이 가능
- **High Availability**: 다중 노드를 활용한 장애 대응 가능

## 2. Elasticsearch의 장단점

### 장점
- 빠른 검색 성능
- 확장성이 뛰어남
- 다양한 필터링 및 분석 기능 지원
- 다양한 플러그인 및 생태계 지원(Kibana, Logstash 등)

### 단점
- 데이터 일관성 유지가 어려울 수 있음
- 운영 및 관리가 복잡함
- 메모리 사용량이 많음
- 복잡한 쿼리는 성능 저하 가능성

## 3. Elasticsearch의 내부 구조

### 주요 컴포넌트
- **Cluster**: 하나 이상의 노드로 구성된 ES 시스템
- **Node**: 클러스터 내에서 데이터를 저장하고 검색을 수행하는 단위
- **Index**: 여러 개의 Document를 포함하는 논리적인 데이터 저장소
- **Document**: JSON 형태로 저장되는 개별 데이터 단위
- **Shard**: 인덱스를 작은 단위로 나눈 파티션 (Primary, Replica)

## 4. Elasticsearch의 동작 과정

### 1. 데이터 색인(Indexing)
- JSON 형식의 데이터를 `PUT` 요청으로 색인
- 데이터는 내부적으로 Inverted Index 구조로 저장됨

### 2. 검색(Querying)
- `GET` 또는 `POST` 요청을 통해 검색 수행
- `Match`, `Term`, `Bool` 등 다양한 쿼리 지원

### 3. 샤드(Shard) 분배 및 조회
- Primary Shard와 Replica Shard로 분산 저장
- 검색 시 여러 샤드에서 병렬 조회 후 결과 집계

## 5. Spring에서 Elasticsearch 설정 방법

### 1. 의존성 추가
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

### 2. 설정 파일 (application.yml)
```yaml
spring:
  elasticsearch:
    uris: http://localhost:9200
```

### 3. Elasticsearch Repository 생성
```java
@Repository
public interface UserRepository extends ElasticsearchRepository<User, String> {
    List<User> findByName(String name);
}
```

### 4. Elasticsearch Configuration 설정
```java
@Configuration
public class ElasticsearchConfig {
    @Bean
    public RestHighLevelClient client() {
        return new RestHighLevelClient(
            RestClient.builder(new HttpHost("localhost", 9200, "http"))
        );
    }
}
```

## 6. ES 도입 후 전체적인 아키텍처

1. **데이터 저장소**
    - MySQL 기존 RDBMS + Elasticsearch
2. **데이터 동기화**
    - Kafka, Logstash, Beats 등을 이용한 실시간 동기화
3. **데이터 검색 및 분석**
    - Elasticsearch에서 데이터를 검색 및 분석
4. **애플리케이션 연동**
    - Spring Boot + Elasticsearch Client 사용
5. **모니터링 및 시각화**
    - Kibana를 이용한 대시보드 구성

```
[Client] -> [Spring Boot] -> [Elasticsearch] -> [Shard] -> [Index]
                                  ↘
                                   [Kibana]
```
