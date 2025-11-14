---
created: 2025-11-14 22:51
updated: 2025-11-14 22:51
status:
  - draft
tags:
  - Configuration
  - Annotation
source:
---
# 📝 TIL 2025-11-14 - Annotation - Configuration

> **관련 주제:** [[MOC_TBD]] 

## 🚀 개요 및 핵심 (Summary)

`@Component`는 다음과 같은 핵심 역할을 수행하는 **베이스(Base) 어노테이션**이며 **단독으로 쓰이기보다는** 그 의미를 확장하고 특화한 자식 어노테이션들의 기반임
`@Bean` 등록은 반드시 @Component가 아닌 `@Configuration`을 사용해야 함

---

## 📚 상세 내용 (Deep Dive)

🏷️ 역할별 특화 어노테이션 사용

실제 엔터프라이즈 애플리케이션 개발에서는 `@Component`를 직접 사용하기보다, 다음과 같이 역할에 따라 구분된 어노테이션을 사용하는 것이 **표준**입니다.

| **어노테이션**             | **사용 계층**         | **기능적 역할 명시**                     |
| --------------------- | ----------------- | --------------------------------- |
| **`@Controller`**     | 프레젠테이션 계층 (Web)   | HTTP 요청 처리 및 응답(View) 결정          |
| **`@RestController`** | 프레젠테이션 계층 (API)   | JSON/XML 형태의 데이터 응답 (RESTful API) |
| **`@Service`**        | 비즈니스 계층 (Service) | 핵심 비즈니스 로직 처리                     |
| **`@Repository`**     | 영속성 계층 (Data)     | 데이터베이스 접근 및 조작 (DAO)              |
| **`@Configuration`**  | 설정 계층 (Config)    | `@Bean` 메서드를 통한 빈 정의              |

