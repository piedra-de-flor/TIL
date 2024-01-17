# Spring에서는 어떻게 예외 처리를 지원할까?
### 1. BasicErrorController
   - Spring은 초기 Spring 1.0에서 부터 에러 처리를 위한 BasicErrorController를 구현해 놓았음
   - Spring Boot는 기본적으로 예외가 발생할 시에 /error로 에러 요청을 다시 전달하도록 되어있음
   - 하지만 이 방식은 필터 혹은 인터셉터와 컨트롤러가 불필요하게 2번씩 호출됨
   - 필터와 인터셉터가 2번 호출되지 않기 위해서는 별도의 작업을 진행해야 함
   - 에러 메세지 또한 클라이언트에게 유용하지 않음

### 2. HandlerExceptionResolver interface
   - 전략패턴 사용
   - AOP
   - Exception을 catch하고, HTTP 상태나 응답 메세지 등을 설정함
   - 위의 이유로 WAS에서 이를 error가 아닌 정상적인 응답으로 인식하기에, BasicErrorController의 문제점 해결
   - 우선 순위대로 4가지 구현체들이 빈으로 등록되어 있음
     - DefaultErrorAttributes: 에러 속성을 저장하며 직접 예외를 처리하지는 않음
     - ExceptionHandlerExceptionResolver: 에러 응답을 위한 Controller나 ControllerAdvice에 있는 ExceptionHandler를 처리함
     - ResponseStatusExceptionResolver: Http 상태 코드를 지정하는 @ResponseStatus 또는 ResponseStatusException를 처리함
     - DefaultHandlerExceptionResolver: 스프링 내부의 기본 예외들을 처리함

### 3. 그럼 HandlerExceptionResolver를 어떻게 사용할 수 있을까?
   - @ResponseStatus
      - 에러 HTTP 상태를 변경하도록 도와주는 어노테이션
      - 하지만 이는 BasicErrorController를 사용. 즉, WAS까지 오류가 전파됨
      - 외부 라이브러리에서 정의한 Exception 클래스에는 사용 불가
   - ResponseStatusException
      - @ResponseStatus의 프로그래밍적 대안
      - 하지만 이 또한 WAS까지 오류가 전파됨
   - @ExceptionHandler
      - controller의 메서드나 @ControllerAdvice나 @RestControllerAdvice가 있는 클래스의 메소드에 적용
      - Exception 클래스들을 속성으로 받아 처리할 예외를 지정
      - 에러 응답(payload)을 자유롭게 다룰 수 있음 
   - @ControllerAdvice와 @RestControllerAdvice
      - 여러 컨트롤러에 대해 전역적으로 ExceptionHandler를 적용
      - 직접 정의한 에러 응답을 일관성있게 클라이언트에게 내려줄 수 있음
      - 별도의 try-catch문이 없어 코드의 가독성이 높아짐

### 4. 그래서 뭘 사용하는게 가장 좋은 방법이지?
   - ControllerAdvice를 이용하는 것이 가장 좋은 방식
   
