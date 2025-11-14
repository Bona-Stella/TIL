---
created: 2025-11-14 23:13
updated: 2025-11-14 23:13
status:
  - draft
tags:
  - Scope
source:
---
# 📝 TIL 2025-11-14 - Request Scope

> **관련 주제:** [[MOC_TBD]] 

## 🚀 개요 및 핵심 (Summary)

**HTTP 요청**과 이에 종속된 **`request` 스코프 Bean**은 서버가 클라이언트에게 **응답(Response)까지 완벽히 보낸 상태여야만** 소멸

---

## 📚 상세 내용 (Deep Dive)

## 🌉 `Request` 스코프 Bean의 동작 메커니즘 요약

정확히 말씀하신 대로, 연결은 다음과 같이 이루어지며, 각 구성 요소의 생명주기와 역할이 분리됩니다.

$$\text{Singleton Service} \rightarrow \text{Proxy Class} \rightarrow \text{Request Scope Bean}$$

### 1. 🥇 Singleton Service와 Proxy의 관계 (장수)

- **`Singleton Service`**는 애플리케이션 수명 동안 **하나만** 존재합니다.
    
- 이 Service가 주입받는 **`Proxy Class`** 객체 역시 **Service와 함께** 생성되어 애플리케이션 종료 시까지 메모리에 **떠 있는(남아있는)** 상태를 유지합니다.
    

### 2. 🔄 Request Scope Bean의 생명주기 (단명)

- 클라이언트로부터 **새로운 HTTP 요청이 들어올 때마다** Spring은 **새로운 `Request Scope Bean` 인스턴스**를 생성합니다.
    
- 요청 처리가 **완료되고 응답이 나가면** 해당 인스턴스는 **소멸**됩니다.
    

### 3. 🎯 Proxy의 역할: 단 하나의 스위치

- 모든 새로운 요청은 **단 하나의 `Proxy Class` 객체**를 바라봅니다.
    
- Service가 `Proxy`를 통해 메서드를 호출할 때마다, `Proxy`는 그 순간 실행 중인 **현재 스레드**에 할당된 **가장 최근에 생성된 유효한 `Request Scope Bean` 인스턴스를 찾아 연결**하고 메서드 호출을 위임합니다.
    

결론적으로, `Proxy`는 **모든 요청이 공유하는** 하나의 영구적인 **스위치** 역할을 하며, 이 스위치가 요청이 들어올 때마다 **새로 생성되는 인스턴스**로 연결을 돌려주는 것입니다.

## 🆚 `request` 스코프 Bean vs. `ThreadLocal` / AOP

|**구분**|**Request Scope Bean**|**ThreadLocal (AOP로 구현 시)**|
|---|---|---|
|**관리 주체**|**Spring IoC 컨테이너**|**개발자 (수동 관리) 또는 AOP/필터**|
|**접근 방식**|**DI (의존성 주입)**를 통해 주입받아 사용|**`ThreadLocal.get()`** 메서드를 직접 호출하여 사용|
|**생명 주기**|**HTTP 요청의 시작과 끝**에 정확히 맞춤|개발자가 직접 `remove()`를 호출하지 않으면 **메모리 누수 위험** 있음|
|**재사용성**|해당 객체에 **DI/AOP/프록시** 기능 적용 가능|순수한 스레드 로컬 저장소 역할만 수행 (Spring 기능 적용 불가)|
|**주요 목적**|**Bean의 형태로** Spring의 관리 기능(DI/AOP)을 활용하며 **요청별 상태 분리**|**단순히 스레드별 데이터** 저장 및 접근 (프레임워크 비종속적)|