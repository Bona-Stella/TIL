---
created: 2025-11-13 18:35
updated: 2025-11-13 18:35
status:
  - draft
tags:
  - Transaction
  - Rollback
  - CheckedException
source:
---
# 📝 TIL 2025-11-13 - Spring 트랜잭션과 Rollback 대상

> **관련 주제:** [[MOC_TBD]] 

## 🚀 개요 및 핵심 (Summary)

Spring 트랜잭션의 기본 정책은 다음과 같습니다.

1. **Unchecked Exception (RuntimeException 및 Error):** 발생 시 트랜잭션을 **자동으로 롤백**합니다.
    
2. **Checked Exception:** 발생 시 트랜잭션을 **자동으로 커밋(Commit)**합니다.

---

## 📚 상세 내용 (Deep Dive)

### ❓ 왜 Checked Exception은 커밋될까?

Spring은 Checked Exception을 **비즈니스 로직에서 의도적으로 발생시켜 복구를 유도**할 수 있는 예외로 간주합니다.

- 예를 들어, "잔액 부족"을 나타내는 `InsufficientBalanceException` (Checked Exception으로 정의)이 발생했을 때, Spring은 기본적으로 이것을 **정상적인 비즈니스 흐름**으로 보고 롤백하지 않고 커밋하려고 합니다.
    

### 📝 롤백 대상 재정의

따라서 Spring 트랜잭션을 사용할 때 개발자는 이 기본 동작을 재정의해야 할 경우가 많습니다.

- **특정 Checked Exception에 대한 롤백:** `@Transactional` 애노테이션의 `rollbackFor` 속성을 사용하여, 특정 Checked Exception이 발생했을 때도 롤백하도록 명시할 수 있습니다.

**요약:** Spring에서 기본적으로 롤백 대상은 **Unchecked Exception**이며, Checked Exception은 기본적으로 커밋 대상이지만, 개발자가 명시적으로 롤백하도록 설정할 수 있습니다.

