# 🧠 Mapped Diagnostic Context (MDC)란?

## 1. 개요

MDC(Mapped Diagnostic Context)는 **로그에 컨텍스트 정보를 자동으로 포함시키기 위한 Thread-local 기반의 기능**이다.  
Spring Boot나 Java 백엔드 애플리케이션에서 **사용자 ID, traceId, 세션 ID** 등을 로깅할 때 유용하게 사용된다.

---

## 2. 왜 필요한가?

### 문제 상황
- 로그가 남아도 **누가, 어떤 요청에서 발생한 것인지 추적이 어려움**
- 특히 다중 사용자 환경이나 분산 시스템에서 문제 분석이 어려움

### 해결
MDC를 활용하면 다음과 같은 정보를 로그에 **자동 포함**시킬 수 있음:

- `userId`: 요청을 보낸 사용자
- `traceId`: 하나의 요청을 추적할 수 있는 식별자
- `transactionId`: 트랜잭션 단위 식별자

---

## 3. 사용 방법 (Spring Boot + Logback)

### 3.1. 필터에서 컨텍스트 주입

```java
@Component
public class LoggingFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        try {
            MDC.put("traceId", UUID.randomUUID().toString());
            MDC.put("userId", extractUserId(request));
            filterChain.doFilter(request, response);
        } finally {
            MDC.clear(); // 메모리 누수 방지
        }
    }
}
```

## 3.2. logback-spring.xml 설정
```java
<pattern>
    {
      "timestamp": "%d{yyyy-MM-dd'T'HH:mm:ss.SSSZ}",
      "level": "%level",
      "logger": "%logger",
      "message": "%message",
      "traceId": "%X{traceId}",
      "userId": "%X{userId}"
    }
</pattern>
```

## 4. 특징
   항목	설명
   ThreadLocal 기반	하나의 스레드 내에서만 MDC 값 유지
   자동 로그 삽입	logback, log4j 등에서 %X{key}로 로그 자동 포함
   비동기 환경 주의	@Async, CompletableFuture 등에서는 별도 전달 필요

## 5. 비동기 환경에서의 MDC 전파
   비동기 작업에서는 MDC 값이 전달되지 않으므로, 별도의 전달 로직이 필요하다.
```java
java
Copy
Edit
public <T> Callable<T> wrapWithMdc(Callable<T> task) {
Map<String, String> contextMap = MDC.getCopyOfContextMap();
return () -> {
MDC.setContextMap(contextMap);
return task.call();
};
}
```

## 6. 결론
   MDC는 로그 추적성(traceability) 을 확보하는 데 매우 유용한 도구

Spring Boot 기반의 실무 환경에서 거의 필수적인 로깅 패턴

단, 비동기 환경에서는 MDC 전파를 신경 써야 한다