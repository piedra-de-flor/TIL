# Spring 테스트 코드에서의 리플렉션(Reflection) 활용 정리

Spring 기반의 테스트 코드를 작성할 때, 리플렉션(Reflection)은 **private 필드나 메서드에 접근**, **mock 객체 주입**, **테스트 커버리지 확대** 등의 용도로 유용하게 사용됩니다. 이 문서에서는 리플렉션의 기본 개념부터 Spring 테스트에서의 활용 예시까지 정리합니다.

---

## 📌 1. 리플렉션(Reflection) 이란?

리플렉션은 **자바의 런타임 동적 클래스 분석 및 조작 기능**입니다.  
즉, 실행 중인 클래스의 필드, 메서드, 생성자 등에 접근하거나 수정할 수 있는 기능을 제공합니다.

### ✨ 주요 클래스
- `Class`
- `Field`
- `Method`
- `Constructor`
- `java.lang.reflect` 패키지 활용

---

## 🔧 2. 리플렉션을 활용하는 이유 (Spring 테스트에서)

| 상황 | 리플렉션 활용 이유 |
|------|-------------------|
| private 필드 접근 | 테스트에서 내부 상태를 검증하거나 강제로 값 설정 |
| private 메서드 호출 | 테스트 범위를 넓히고 내부 로직 검증 |
| 의존성 주입 강제 설정 | 테스트 대상 객체에 mock 객체를 강제로 주입 |
| final 필드 설정 | 상수 변경이나 특정 조건 설정 필요 시 |

---

## ✅ 3. 사용 예제

### 📍 3.1 private 필드에 값 설정하기

```java
Field field = MyService.class.getDeclaredField("privateField");
field.setAccessible(true);
field.set(myServiceInstance, "testValue");
```

### 📍3.2 private 메서드 호출하기
```
Method method = MyService.class.getDeclaredMethod("privateMethod", String.class);
method.setAccessible(true);
Object result = method.invoke(myServiceInstance, "param");
```

### 📍 3.3 테스트에서 mock 객체 강제 주입
```
@InjectMocks
private MyService myService;

@Mock
private Dependency dependency;

@BeforeEach
void setUp() throws Exception {
    Field field = MyService.class.getDeclaredField("dependency");
    field.setAccessible(true);
    field.set(myService, dependency);
}
```