---
created: 2025-11-14 23:57
updated: 2025-11-14 23:57
status:
  - draft
tags:
source:
---
# 📝 TIL 2025-11-14 - Filter와 Interceptor의 예외 처리

> **관련 주제:** [[MOC_TBD]] 

## 🚀 개요 및 핵심 (Summary)

Spring 애플리케이션에서 Filter와 Interceptor를 통해 예외를 안전하게 처리하는 전략은 **예외가 발생하는 위치와 시점**에 따라 달라집니다.

---

## 📚 상세 내용 (Deep Dive)

### . 🌐 Interceptor와 Controller에서의 예외 처리

Interceptor 또는 Controller에서 발생하는 예외는 **Spring 프레임워크의 통제 범위 내**에 있으므로, 가장 깔끔하고 일관성 있게 처리할 수 있습니다.

- **처리 주체:** **`@ControllerAdvice`** (글로벌 예외 핸들러)
    
- **동작 방식:** Interceptor의 `preHandle()` 이후 단계(Controller, Service, Repository 등)에서 예외가 발생하면, 이 예외는 Dispatcher Servlet을 거쳐 `@ControllerAdvice`가 선언된 클래스로 전달됩니다.
    
- **`@ExceptionHandler`:** `@ControllerAdvice` 내부에 정의된 `@ExceptionHandler` 메서드가 예외 타입에 맞는 처리를 수행하고, 클라이언트에게 적절한 HTTP 상태 코드와 에러 메시지를 포함한 응답을 반환합니다.
    

### 2. 🛡️ Filter에서의 예외 처리

Filter는 Spring 컨테이너의 외부에 존재하며, Dispatcher Servlet보다 먼저 실행됩니다. 따라서 Filter에서 예외가 발생하거나 감지될 경우, **Spring의 `@ControllerAdvice`가 동작하지 않습니다.**

- **문제점:** Filter에서 예외가 발생하면, 예외는 **서블릿 컨테이너(Tomcat 등)**로 바로 던져지고, 컨테이너는 기본 에러 페이지(혹은 Whitelabel Error Page)를 보여주거나 500 에러를 반환합니다.
    
- **처리 방법 (가장 일반적):**
    
    1. **`try-catch`로 직접 처리:** Filter의 `doFilter()` 메서드 내에서 예외를 `try-catch` 블록으로 잡습니다.
        
    2. **응답 생성:** 잡은 예외를 분석하여, `HttpServletResponse` 객체를 사용하여 **직접 JSON 형태의 응답을 구성**하고 **HTTP 상태 코드를 설정**합니다 (예: `response.setStatus(401)`). 이후 `chain.doFilter()`를 호출하지 않고 응답을 종료합니다.
        
- **처리 방법 (Spring 내부 위임 - 권장):**
    
    1. Filter에서 예외를 `catch`합니다.
        
    2. `HttpServletRequest.getRequestDispatcher("/error")` 등을 사용하여 요청을 **Spring Controller의 에러 핸들링 엔드포인트**로 **강제 포워딩(Forward)**합니다. 이렇게 하면 Spring의 `@ControllerAdvice`가 해당 포워딩 요청을 받아 예외 처리를 수행할 수 있게 됩니다.
        

### 3. 📝 예외 발생 시점별 최종 처리

|**발생 위치**|**Spring IoC 통제 여부**|**처리 전략**|
|---|---|---|
|**Servlet Filter**|**외부 (Out of Scope)**|`try-catch` 후 `HttpServletResponse`로 직접 응답 구성 **OR** Spring 에러 컨트롤러로 **포워딩**|
|**Interceptor**|**내부 (In Scope)**|`@ControllerAdvice`에 의해 자동으로 처리|
|**Controller/Service**|**내부 (In Scope)**|`@ControllerAdvice`에 의해 자동으로 처리|

