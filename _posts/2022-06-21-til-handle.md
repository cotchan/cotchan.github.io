---
layout: post
title: 프로젝트에 CompletableFuture handle() 적용하기
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

---

### 적용배경

- 모든 요청을 단일 쓰레드에서 처리할 수는 없으므로 멀티 쓰레드 구조로의 전환이 필요했습니다.
- 최소 `리소스 생성` 같은 역할은 별도의 쓰레드로 분리하여 리소스를 생성하는 과정에서 생기는 오류는 분리하고자 했습니다.

### handle

- `handle()`은 `작업이 완료된 다음에 얻게될 결과`와 `예외`를 모두 parameter로 사용할 수 있습니다.
- `handle()`은 현재얻은 결과 또는 예외를 `다른 값으로 변환할 수 있습니다.`

```java
public <U> CompletableFuture<U> handle(
    BiFunction<? super T, Throwable, ? extends U> fn) {
  ...
}
```

#### 사용하면 좋은 경우

- `handle()`을 사용하면 좋은 경우는, `정상 결과와 예외가 모두 필요`하며 `결과 유형을 변환`해야 할 때 적합합니다.

```java
// CompletableFuture<User> to CompletableFuture<Response>
cf.handle((user, ex) -> {
  if (ex != null) {
    return Response.failure("Unknown user");
  } else {
    return Response.success(user);
  }
}
```

#### 예시 코드

- `handle()`은 정상 결과를 첫 번째 인자로, 예외를 두 번째 인자로 받는 BiFunction 인터페이스를 콜백 함수로 받아서 작업을 진행합니다.
- 만약 앞의 비동기 작업이 `정상적으로 진행되었다면` 두 번째 인자의 `throwable 예외는 null` 값이고,
- `예외가 발생했다면` 결과값으로 받는 첫 번째 인자의 `userId이 null`값이 됩니다.

```java
@PostMapping("/join")
public CompletableFuture<ResponseEntity<ApiResult<?>>> join(@RequestBody JoinRequest request) {

    String accessToken = request.getAccessToken();
    OAuthResponse.LoginResponse userInfo = oAuthService.getUserInfo(OAuthRequest.LoginRequest.from(accessToken));

    return userService.create(userInfo.getId(), userInfo.getProfileImage()).handle((userId, throwable) -> {

        if (userId != null) {
            User user = userService.getUser(userId);
            return new ResponseEntity<>(OK(JoinResponse.from(user)), OK);
        }

        return ErrorResponseEntity.from(throwable, true);
    });
}
```

#### ErrorResponseEntity

- 멀티 쓰레드를 적용하면서 가장 안 좋아진 점은 `@RestControllerAdvice`의 도움을 받을 수 없다는 점입니다.
- 기존 방식에서는 성공 케이스에 대해서는 별도로 `ResponseEntity`를 래핑할 필요가 없었지만 별도 쓰레드에서 처리하고자 하는 로직은 다르게 처리해줘야 합니다.
- 그래서 `handle` 메소드의 두 번째 파라미터로 받을 수 있는 `Throwable`을 전담해서 처리하는 클래스를 생성해서 정적 메소드로 예외 응답값을 만들어주었습니다.

```java
public class ErrorResponseEntity {
    public static ResponseEntity<ApiResult<?>> from(Throwable throwable, boolean logFlag) {...}
}
```

### Service 메소드

- 간단하게 별도의 쓰레드에서 처리하도록 `@Async` 애노테이션을 적용했기 때문에 리소스를 생성하는 서비스 메소드는 다음과 같은 형태를 가집니다.

```java
public class UserServiceImpl implements UserService {

    //...

    @Async
    @Override
    @Transactional
    public CompletableFuture<Long> create(Long oauthId, String profileImageFullPath) {...}

}
```

---

- 참고
  - [3 Ways to Handle Exception In Completable Future](https://mincong.io/2020/05/30/exception-handling-in-completable-future/)
  - [CompletableFuture 톺아보기](https://wbluke.tistory.com/50)
