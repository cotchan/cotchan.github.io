---
layout: post
title: throwable.getCause()
subtitle: 멀티 쓰레드 사용 시 원인이 되는 비즈니스 예외 식별하기
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### throwable.getCause()란?

- 예외가 발생한 근본적인 `원인 예외를 반환`합니다. 원인 객체가 없는 경우 `null`을 반환합니다.

```java
Throwable getCause();
```

### 문제 상황

#### CompletionException

- 멀티쓰레드를 사용하는 곳에 `CompletableFuture.handle()`을 적용해서 로직을 처리했었습니다.
- 이 때 해당 쓰레드에서 exception이 발생하면 `CompletionException` 예외가 반환이 됩니다.

```bash
# 해당 예외 설명
Exception thrown when an error or other exception is encountered in the course of completing a result or task.
```

- 이 예외는 `정의한 비즈니스 예외`와는 무관한 예외입니다.
- 그래서 이 예외가 왜 발생했는지를 알아야 했는데 `.getMessage()`만으로 식별하기에는 방법이 좋지 않았습니다.

### 해결방법

#### throwable.getCause()

- 그래서 `원인이 되는 비즈니스 예외를 식별`할 방법이 없는지 이것저것 호출하며 테스트를 해봤고 `throwable.getCause()`을 통해 해결할 수 있었습니다.
- 방법은 `throwable.getCause()`와 `instanceof`의 조합으로 원인이 되는 비즈니스 예외를 식별하여 해당 예외에 맞는 예외 처리 로직을 태우면 됩니다.

```java
public class ErrorResponseEntity {

    public static ResponseEntity<ApiResult<?>> from(Throwable throwable) {

        if (throwable.getCause() instanceof ApiException) {
            ApiException e = (ApiException) throwable.getCause();
            return new ResponseEntity<>(ERROR(e, e.getType().getStatus()), e.getType().getStatus());
        } else if (throwable.getCause() instanceof IllegalArgumentException) {
            return new ResponseEntity<>(ERROR(throwable.getCause(), BAD_REQUEST), BAD_REQUEST);
        } else if (throwable.getCause() instanceof IllegalStateException) {
            return new ResponseEntity<>(ERROR(throwable.getCause(), BAD_REQUEST), BAD_REQUEST);
        }

        return new ResponseEntity<>(ERROR(throwable, INTERNAL_SERVER_ERROR), INTERNAL_SERVER_ERROR);
    }
}
```

---

- 출처
  - [[JAVA] 예외의 종류](https://dololak.tistory.com/52)
