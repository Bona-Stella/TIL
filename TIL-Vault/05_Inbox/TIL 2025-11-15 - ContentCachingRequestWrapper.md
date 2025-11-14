---
created: 2025-11-15 00:03
updated: 2025-11-15 00:03
status:
  - draft
tags:
source:
---
# 📝 TIL 2025-11-15 - ContentCachingRequestWrapper

> **관련 주제:** [[MOC_TBD]] 

## 🚀 개요 및 핵심 (Summary)

**`ContentCachingRequestWrapper`**를 사용하면 **언제 어디서든** (`Filter`, `Interceptor`, `Controller` 등) **Request Body의 데이터를 안전하게 재확인(재사용)**할 수 있습니다.

---

## 📚 상세 내용 (Deep Dive)

## 🛠️ ContentCachingRequestWrapper의 재사용 원리

`ContentCachingRequestWrapper`를 통해 Body 데이터를 재확인할 수 있는 원리는 다음과 같습니다.

1. **데이터 캐싱:** 이 래퍼 객체가 생성될 때, 원본 `HttpServletRequest`의 Body 스트림을 **모두 읽어서** 내부 메모리(바이트 배열)에 **복사(캐싱)**하여 저장합니다. 이 과정에서 원본 스트림은 소비되지만, 이제 데이터는 래퍼 객체 안에 안전하게 보존됩니다.
    
2. **스트림 재생성:** `Controller`의 `@RequestBody` 처리 등에서 `request.getInputStream()`이 호출되면, 래퍼는 **원본 스트림이 아닌** 자신이 **캐시해 둔 메모리 배열**을 기반으로 **새로운 `InputStream` 객체를 생성**하여 반환합니다.
    

### 📌 사용 시 유의 사항

`ContentCachingRequestWrapper`는 매우 유용하지만, 다음 사항을 유의해야 합니다.

- **Filter에서 래핑:** 이 래퍼 객체는 **가장 앞단인 `Filter`**에서 생성하여 `chain.doFilter(wrapper, response)`로 넘겨줘야, 뒤따라오는 `DispatcherServlet`, `Interceptor`, `Controller` 모두 이 **재사용 가능한 래퍼 객체**를 참조할 수 있습니다.
    
- **메모리 사용:** Body 전체를 메모리에 복사해 두기 때문에, **대용량 파일 업로드**와 같은 매우 큰 요청에는 서버 메모리 부담이 커질 수 있습니다. 일반적인 JSON 데이터에는 문제가 없습니다.
    
- **데이터 접근:** 래퍼에 저장된 캐시 데이터는 `wrapper.getContentAsByteArray()` 메서드를 통해 바이트 배열 형태로 접근할 수 있습니다.
    

요약하자면, `ContentCachingRequestWrapper`를 사용하면 스트림의 **'한 번만 읽히는'** 성질을 우회하고 데이터를 **메모리에 저장**하여 여러 번 재사용할 수 있게 됩니다.

