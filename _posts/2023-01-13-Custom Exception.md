---
categories: Study
tags: [spring, Exception, Java]
---

# Custom Exception

Custom Exception을 만들기 전에는 기존에 작성했던 것처럼 표준 예외에 메시지만 담아서 사용하면 충분하지 않을까라는 생각이 들었다.

실제로 자바에서는 이미 다양한 예외를 제공하고 있고 메시지를 잘 작성하면 상황 설명도 가능해 보였기 때문이다.

원 글은 해당 [블로그](https://tecoble.techcourse.co.kr/post/2020-08-17-custom-exception/)에서 보면 되고 나는 각각의 장단점에 대해서 요약해봤다. 

<br><br><br>

## 표준 예외로 처리할 때의 장점

### 1. 예외 메시지로도 충분히 의미 전달이 가능하다.

단순히 메시지 하나를 표현하기 위해 매번 새로운 Custom Exception을 만드는 것은 구현 측면에서 다소 과할 수 있다.

이런 경우라면 이미 존재하는 표준 예외에 메시지를 담아 사용하는 것도 충분히 합리적인 선택이 될 수 있다.

<br><br>

### 2. 표준 예외를 사용하면 코드 가독성이 높아진다.
표준 예외들은 이미 이름과 쓰임새가 명확하게 정해져 있고 많은 개발자들이 그 의미를 알고 있다.

반면 처음 보는 Custom Exception은 이름만으로 그 용도를 바로 파악하기 어려울 수 있다.

이 때문에 낯선 예외보다는 익숙한 표준 예외를 사용하는 쪽이 가독성 측면에서 더 유리할 수 있다.

<br><br>

### 3. Custom Exception이 지나치게 많아질 수 있다.

모든 상황마다 Custom Exception을 만들기 시작하면 예외 클래스의 수가 빠르게 증가한다.

예외 클래스가 많아질수록 클래스 로딩 비용이 증가할 수 있고 관리 또한 점점 어려워질 수 있다.

이미 자바에서 제공하는 표준 예외를 적절히 재사용한다면 이런 문제를 어느 정도 피할 수 있다.

<br><br><br><br> 

## 사용자 정의 예외(Custom Exception)를 사용할 때의 장점

### 1. 더 많은 예외 정보를 함께 전달할 수 있다.

컬렉션에서 index 접근 오류가 발생했을 때를 예로 들 수 있다.

Custom Exception을 사용하면 요청한 index 값과 컬렉션의 최대 크기 같은 정보를 함께 담아 전달할 수 있다.

이런 정보는 디버깅 과정에서 큰 도움이 된다.

<br><br>

### 2. 예외에 대한 응집도가 높아진다.

클래스를 만든다는 것은 관련된 정보를 하나의 책임 단위로 묶겠다는 의미이다.

같은 예외가 여러 곳에서 반복해서 사용될수록 Custom Exception을 통해 해당 예외와 관련된 정보를 한 곳에서 관리할 수 있다.

<br><br>

### 3. 예외 발생 이후의 처리 흐름을 제어하기 쉬워진다.

예외는 상속 구조를 가지기 때문에 `Exception`이나 `RuntimeException`을 통해 여러 예외를 한 번에 처리할 수 있다.

하지만 표준 예외만 사용하다 보면 의도하지 않은 예외까지 함께 잡히는 문제가 발생할 수 있다.

Custom Exception을 사용하면 특정 예외만 골라서 처리할 수 있어 예외 처리 흐름이 더 명확해진다.

<br>

아래는 이러한 기준을 바탕으로 Custom Exception을 구성한 예시이다.

<br><br><br><br>  

## ErrorCode

먼저 애플리케이션 전반에서 공통으로 사용할 ErrorCode를 정의한다.

```java
@Getter
public enum ErrorCode {

    REGISTRY_NOT_FOUND(HttpStatus.NOT_FOUND, "게시글이 존재하지 않습니다."),
    COMMENT_NOT_FOUND(HttpStatus.NOT_FOUND, "해당 댓글이 존재하지 않습니다."),
    USER_NOT_FOUND(HttpStatus.NOT_FOUND, "해당 유저를 찾을 수 없습니다."),
    PERMISSION_DENIED(HttpStatus.UNAUTHORIZED, "사용자 권한이 없습니다.");

    private final HttpStatus httpStatus;
    private final String message;

    ErrorCode(HttpStatus httpStatus, String message) {
        this.httpStatus = httpStatus;
        this.message = message;
    }
}
```

ErrorCode에는 HTTP 상태 코드와 클라이언트에게 전달할 메시지를 함께 담는다.

이렇게 하면 예외가 발생했을 때 어떤 상태 코드와 메시지를 내려줄지 한 곳에서 관리할 수 있다.

HTTP 상태 코드를 사용하지 않고 `USER_NOT_FOUND(40410, "유저 없음")`처럼 직접 정의할 수도 있다.

다만 이 경우 HttpStatus 타입이 아닌 정수형 코드로 관리해야 한다는 점을 고려해야 한다.

<br><br><br><br>  

## CustomException

ErrorCode를 포함하는 CustomException 클래스를 정의한다.

```java
public class CustomException extends RuntimeException {

    private static final long serialVersionUID = 268398153652135694L;

    private final ErrorCode errorCode;

    public CustomException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }

    public ErrorCode getErrorCode() {
        return errorCode;
    }
}
```

CustomException은 RuntimeException을 상속받아 Unchecked Exception으로 사용한다.

ErrorCode 하나만 가지고 있어도 HTTP 상태 코드와 메시지를 모두 관리할 수 있다.

<br><br><br><br>  

## CustomExceptionHandler

컨트롤러 전역에서 발생하는 CustomException을 공통으로 처리하기 위한 Handler를 만든다.

```java
@RestControllerAdvice
public class CustomExceptionHandler {

    @ExceptionHandler(CustomException.class)
    protected ResponseEntity<ErrorResponseEntity> handleCustomException(CustomException e) {
        return ErrorResponseEntity.toResponseEntity(e.getErrorCode());
    }
}
```

`@ExceptionHandler(CustomException.class)`는 CustomException이 발생했을 때 해당 메소드가 실행되도록 한다.

`@RestControllerAdvice`는 모든 컨트롤러에서 발생한 예외를 공통으로 처리하겠다는 의미이다.

<br><br><br><br>  

## ErrorResponseEntity

에러 응답으로 내려줄 Response 객체를 정의한다.

```java
@Data
@Builder
public class ErrorResponseEntity {

    private int status;
    private String code;
    private String message;

    public static ResponseEntity<ErrorResponseEntity> toResponseEntity(ErrorCode e) {
        return ResponseEntity
                .status(e.getHttpStatus())
                .body(ErrorResponseEntity.builder()
                        .status(e.getHttpStatus().value())
                        .code(e.name())
                        .message(e.getMessage())
                        .build());
    }
}
```

CustomException에 담긴 ErrorCode를 이용해 일관된 형태의 에러 응답을 생성한다.

<br><br><br><br>  

## 사용 예시

```java
// 게시글이 없을 때
public class RegistryNotFoundException extends CustomException {

    public RegistryNotFoundException() {
        super(ErrorCode.REGISTRY_NOT_FOUND);
    }
}
```

```java
// 게시글 상세 보기
public ResRegistryDto getIdxRegistry(Long idx) {
    Registry registry = registryRepository.findById(idx)
            .orElseThrow(RegistryNotFoundException::new);
    return ResRegistryDto.of(registry);
}
```

<br>

![image](/assets/img/posts/212059902-96820ae8-ba63-482f-972a-9061209634c8.png)

<br><br><br><br>  

---

***reference***       
[custom exception을 언제 써야 할까?](https://tecoble.techcourse.co.kr/post/2020-08-17-custom-exception)      
[woowacourse-teams/2022-f12](https://github.com/woowacourse-teams/2022-f12)           

