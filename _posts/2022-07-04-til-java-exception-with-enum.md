---
layout: post
title: Enum을 사용한 자바 예외처리
subtitle: Enum + MessageSourceAccessor를 사용하여
categories: todayilearned
tags: [todayilearned]
---

- 개인 공부 목적으로 작성한 포스팅입니다.
- 아래 출처를 참고하여 작성하였습니다. :)

### INTRO

- 기존에는 `MessageSourceAccessor`를 사용하여 예외 메시지 처리를 했었습니다.
- 근데 사용자 정의 예외가 늘어날 때마다 해당 예외 클래스를 추가로 정의해야하는 게 옳은 방식인가 의문이 들었습니다.
- 그래서 Enum을 사용한 예외 처리 방식을 공부했고, 거기에 `MessageSourceAccessor`를 적용해본 내용을 정리합니다.

### MessageSourceAccessor

- 선택사항이지만 예외를 생성하는 곳과 예외 메시지를 만드는 곳을 분리하기 위해 적용했습니다.
- Enum을 사용해서는 `RuntimeException`을 상속받은 사용자 정의 예외를 만들고, 그에 해당하는 실제 메시지는 `MessageSourceAccessor`가 처리합니다.

#### MessageUtils

- MessageSourceAccessor를 편리하게 사용하기 위해 `MessageUtils` 클래스를 사용했습니다.

```java
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public class MessageUtils {

    private static MessageSourceAccessor messageSourceAccessor;

    public static String getMessage(String key) {
        return messageSourceAccessor.getMessage(key);
    }

    public static String getMessage(String key, Object... params) {
        return messageSourceAccessor.getMessage(key, params);
    }

    //...
}
```

#### message.properties

```properties
error.authentication=
error.authentication.details=
error.authority=
error.authority.details=
error.notfound=
error.notfound.details=
error.duplicate=
error.duplicate.details=
```

### ServiceRuntimeException

- 사용자 정의 최상단 예외 클래스입니다.

```java
@Getter
@RequiredArgsConstructor
public abstract class ServiceRuntimeException extends RuntimeException {

    // 필요 시 에러 메시지에 추가할 parameter
    private final Object[] params;
}
```

### EnumApiException

- `api`라는 모듈에서 사용할 모든 예외 케이스를 Enum으로 정의합니다.
- `messageKey`, `messageDetailKey` 필드는 `message.properties`에 정의한 key값입니다.
- 예외 메시지 생성 시 `MessageSourceAccessor`에 key 값을 넘겨줘서 예외 메시지를 만들기 위해 사용합니다.

```java
@Getter
@AllArgsConstructor
public enum EnumApiException {

    NOT_FOUND("error.notfound", "error.notfound.details"),
    UNAUTHORIZED("error.authority","error.authority.details"),
    DUPLICATED_VALUE("error.duplicate","error.duplicate.details");

    private String messageKey;
    private String messageDetailKey;
}
```

### ApiException

- `api`라는 모듈에서 사용할 사용자 정의 예외입니다.
- 호출하고자 했던 형태는 아래와 같은 두 가지입니다.
  - `new ApiException(EnumApiException, Clazz.class, param);`
    - `Clazz` 타입 Entity 조회 시 예외가 발생했고 그 때의 `param`도 같이 전달
  - `new ApiException(EnumApiException, message);`
    - 예외 메시지만 추가로 입력하고 싶을 때

```java
@Getter
public class ApiException extends ServiceRuntimeException {

    private final EnumApiException exceptionType;

    /**
     * ApiException TYPE1
     *
     * Usage: 단순히 어떤 타입의 예외이고, 추가적으로 message만 기입하고 싶을 때
     */
    public ApiException(EnumApiException exceptionType, String message) {
        super(new String[]{message});
        this.exceptionType = exceptionType;
    }

    /**
     * ApiException TYPE2
     *
     * Usage: Entity 데이터 조회 전용. 어떤 Entity Class를 못 찾았고, 그 때의 해당 parameter return
     */
    public ApiException(EnumApiException exceptionType, Class<?> cls, Object... values) {
        this(exceptionType, cls.getSimpleName(), values);
    }

    private ApiException(EnumApiException exceptionType, String targetName, Object... values) {
        super(new String[]{targetName,
                (isNotEmpty(values)) ? StringUtils.join(values, ",") : ""});

        this.exceptionType = exceptionType;
    }

    /**
     * 해당 Exception 출력 시 message.properties의 detailKey를 읽고, 거기에 해당 parameter를 넣어줌
     */
    @Override
    public String getMessage() {
        return MessageUtils.getMessage(exceptionType.getMessageDetailKey(), getParams());
    }

    @Override
    public String toString() {
        return MessageUtils.getMessage(exceptionType.getMessageKey());
    }
}
```

### RestControllerAdvice

- 결국 이걸 하는 것도 한 곳에서 예외처리를 전담하기 위함입니다.
- 아래와 같이 `ApiException` 발생 시 `EnumApiException` 코드에 해당하는 httpStatusCode를 만들어서 응답을 주면 됩니다.

```java
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {
    /**
     * api 모듈에서 정의한 비즈니스 예외가 발생하는 경우
     */
    @ExceptionHandler(ApiException.class)
    public ResponseEntity<?> handleApiException(ApiException e) {
        switch (e.getExceptionType()) {
            case NOT_FOUND:
                return createResponse(e, HttpStatus.NOT_FOUND);
            case UNAUTHORIZED:
                return createResponse(e, HttpStatus.FORBIDDEN);
            case DUPLICATED_VALUE:
                return createResponse(e, HttpStatus.BAD_REQUEST);
            default:
                log.warn("Unexpected apiException occurred: {}", e.getMessage(), e);
                return createResponse(e, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    }

    //...
}
```

### 추신

- Enum을 사용해서 사용자 예외를 처리할 때 `"어떤 단위로 예외를 나눠야 하나"` 고민하는 과정에서 아래 참고한 포스팅이 많은 도움이 되었습니다.
  - [[Spring] Enum을 사용하여 예외처리 해보기](https://fenderist.tistory.com/116)
- 해당 포스팅에서 `"Enum을 사용했을 때 장점은 모듈 내에서 발생하는 다양한 예외를 하나의 Exception class로 처리할 수 있다"`라는 부분이 인상 깊었는데, 그 이유는 같은 의미인 예외더라도 모듈마다 쓰임새가 다를 수 있기 때문입니다.
- 예를 들어, 모든 모듈에 사용하는 공통 예외가 있는데 현재 모듈에서 그 의미가 약간 안 맞는다면 이때부터 고민이 시작됩니다.
  - 비슷한 게 있긴 하니 그냥 재정의하기도 애매하고, 그렇다고 갖다쓰자니 의미가 명확히 일치하지 않아서 찜찜한 경우가 있기 때문입니다.
- 이럴 때 모듈별로 `하나의 예외 클래스`를 만들고 모듈별로 예외를 관리할 수 있다면 이런 고민이 좀 줄어들지 않을까 싶습니다.
- `Enum`을 적용해서 개별 예외 클래스를 계속 생성하는 불편함도 없앴고, `MessageSourceAccessor`도 계속 사용할 수 있어서 뿌듯했던 작업입니다.

---

- 참고
  - [[Spring] Enum을 사용하여 예외처리 해보기](https://fenderist.tistory.com/116)
