---
categories: Study
tags: [spring, Exception, Java]
---

# Custom Exception

CustomException을 만들기 전에는 기존에 작성한 것처럼 표준 예외를 사용해서 message를 주면 되지 않을까 라는 생각이 들었었다.

원 글은 해당 [블로그](https://tecoble.techcourse.co.kr/post/2020-08-17-custom-exception/)에서 보면 되고 나는 각각의 장단점에 대해서 요약해봤다. 

<br>

### 표준 예외로 처리할 때 장점

**1. 예외 메시지로도 충분히 의미를 전달할 수 있다.**

`UserNameEmptyException` 처럼 이름만 봐도 사용자 이름의 입력값이 비어있는 경우 발생하는 예외임을 알 수 있다.

하지만 그 이유를 위해서 커스텀 예외를 만드는 것은 지나친 구현이다.

<br>

**2. 표준 예외를 사용하면 가독성이 높아진다.**

- `IllegalArgumentException` : 인수로 부적절한 값이 들어올 때 던지는 예외

- `IllegalStateException`: 일을 수행하기에 적합하지 않은 상태의 객체인 경우 던지는 예외

- `UnsupportedOperationException` : 요청받은 작업을 지원하지 않는 경우에 던지는 예외

이미 익숙하고, 쓰임에 대해 잘 알고있는 예외들이 많은 반면에 처음 보는 예외들은 당연히 구체적인 쓰임을 잘 모른다. 

이런 이유로 낯선 예외보다는 익숙한 예외를 마주치는 것이 당연히 가독성이 높을 수 밖에 없다.

<br>

**3. 일일히 예외 클래스를 만들다보면 지나치게 커스텀 예외가 많아질 수 있다.**

예외가 지나치게 많아진다면 메모리 문제도 발생할 수 있고, 클래스 로딩에도 시간이 더 소요될 가능성이 있다.

이미 자바에서는 충분히 많은 표준 예외를 제공하고 있으므로 표준 예외를 재사용한다면 이를 막을 수 있다.

<br><br>

### 사용자 정의로 예외를 처리할 때 장점

**1. 이름으로도 정보 전달이 가능하다.**

`NoSuchElementException`만으로는 어떤 요소가 없는지 알 수 없다. 

하지만 `PostNotFoundException`이 발생했다면, Post를 찾는 요청을 보냈지만 해당 요소가 없다는 상황을 유추할 수 있을 것이다.

<br>

**2. 상세한 예외 정보를 제공할 수 있다.**

컬렉션의 범위를 벗어난 index 접근 요청이 생겼을 때 custom exception을 통해서 

요청 받은 컬렉션의 최대 범위가 어디까지인지, 요청한 index는 몇인지 바로 알 수 있다.

<br>

**3. 예외에 대한 응집도가 향상된다.**

클래스를 만드는 행위는 관련 정보를 해당 클래스에서 최대한 관리하겠다는 이야기다.

같은 예외가 여러 곳에서 사용될수록 이 장점은 더 극대화될 것이다.

<br>

**4. 예외 발생 후처리가 용이하다.**

예외는 상속 관계에 있기 때문에,             
`Exception`이나 `RuntimeException`을 잡아두면 프로그램 내에서 발생하는 거의 모든 예외에 대해 처리가 가능하다. 

하지만 이는 프로그래머가 의도하지 않은 예외까지 모두 잡아내 혼란을 야기할 수 있다.

재사용성이 높은 것은 표준 예외들의 장점이다. 하지만 그 장점 때문에 발생 위치를 정확하게 파악하기 힘들다는 단점도 생긴다.

<br><br>

마구잡이로 쓰기 보다는 적절한 상황에 맞춰서 Custom Exception을 작성해야할 것 같다.

아래는 Custom Exception을 작성한 코드이다. 

<br><br><br>

## ErrorCode
다양한 상황에서 쓰일 Error Code를 만든다.

```java
@Getter
public enum ErrorCode {
    // Registry
    REGISTRY_NOT_FOUND(HttpStatus.NOT_FOUND, "게시글이 존재하지 않습니다."),

    // Comment
    COMMENT_NOT_FOUND(HttpStatus.NOT_FOUND, "해당 댓글이 존재하지 않습니다."),

    // User
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
☑️ 상태를 담을 **HttpStatus**와 **message**를 담을 String 속성을 추가했다.

<br><br>

🐣 HTTP Status Code(HTTP 상태 코드) :              

HTTP Status Code(HTTP 상태 코드)는 클라이언트가 보낸 HTTP 요청에 대한 서버의 응답을 코드로 표현한 것으로                 
해당 코드로 요청의 성공 / 실패 / 실패요인 등을 알 수 있다.                    

<br>

🐣 HttpStatus를 따르지 않고 `USER_NOT_FOUND(40410, "유저 없음")` 과 같이 custom해서 만들어도 된다. 

다만 이럴 경우에는 HttpStatus타입이 아닌 Integer타입으로 바꿔야 한다고 한다.                       

<br><br><br>

## CustomException
ErrorCode를 담을 class를 만든다.
 
 ```java
 public class CustomException extends RuntimeException {
    private static final long serialVersionUID = 4663380430591151694L;

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
☑️ RuntimeException을 상속

☑️ ErrorCode만 속성으로 추가했다. ➡️ 상태(**HttpStatus**)와 메세지(**message**) 정보 모두 담겨있다.

<br><br>

- **Exception** 클래스를 상속 받아 정의한 **checked** Exception
    - **반드시 오류를 처리 해야만 하는 Exception**
    - 예외 처리하지 않으면 컴파일 오류를 발생 시킨다.

- **RuntimeException** 클래스를 상속 받아 정의한 **unChecked** Exception
    - 예외 처리하지 않아도 컴파일 시에는 오류를 발생시키지 않는다.

<br><br>

🐣 **serialVersionUID 작성 이유**

serialVersionUID 값을 파일 등으로 저장을 할 때 해당하는 클래스의 버전이 맞는지를 확인하는 중요한 장치이다.

serialVersionUID 는 직렬화에 사용되는 고유 아이디인데, 선언하지 않으면 JVM에서 디폴트로 자동 생성된다.

Java VM에서 내부 알고리즘에 따라서 자동으로 작성을 하게 되는데, 이것은 어떤 Java VM을 사용하는지에 따라서 달라지게 된다.

클라이언트가 Windows가 서버가 Linux일 경우 Java VM이 다르므로 이 값이 다르게 설정이 되고,

역직렬화를 할때 exception이 발생할 수 있다. 따라서 무조건 serialVersionUID 값을 설정하기를 권장한다.

serialVersionUID는 `private static final` 로 선언하면 된다.

<br><br><br>

## CustomExceptionHandler
컨트롤러 전역에서 발생하는 Custom에러를 잡아줄 Handler를 만든다.

ExceptionHandler가 붙은 함수는 꼭 protected / private 처리를 해준다. (외부에서 함수를 부르게 되면 그대로 에러 객체를 리턴하기 떄문이다.)

```java
@RestControllerAdvice
public class CustomExceptionHandler {
    private final Logger LOGGER = LoggerFactory.getLogger(CustomExceptionHandler.class); // Logger를 등록

    @ExceptionHandler(CustomException.class)
    protected ResponseEntity<ErrorResponseEntity> handleCustomException(CustomException e) {
        return ErrorResponseEntity.toResponseEntity(e.getErrorCode());
    }
}
```
#### @ExceptionHandler(CustomException.class)
🐣  발생한 CustomException 예외를 잡아서 하나의 메소드에서 공통 처리할 수 있게 해준다.

- 해당 메소드가 () 들어가는 예외타입을 처리한다.
- Controller에만 등록이 가능하다. (Service영역은 ❌!)
- 등록한 Controller 영역 안에서만 작동한다.
- `@ExceptionHandler({ Exception1.class, Exception2.class})` 와 같이 두 개 이상 등록도 가능하다.

<br><br>

#### @ControllerAdvice + @ExceptionHandler
🐣  모든 컨트롤러에서 발생하는 CustomException을 캐치한다.

해당 객체가 스프링의 controller에서 발생하는 예외를 처리하는 존재임을 명시한다.

<br><br>

**`@ControllerAdvice` 와 `@RestControllerAdvice` 차이**

Spring은 전역적으로 예외를 처리할 수 있는 `@ControllerAdvice`와 `@RestControllerAdvice` 어노테이션을 

각각 Spring3.2, Spring4.3부터 제공하고 있다. 

<br>

두 개의 차이는 `@Controller`와 `@RestController`와 같은데, `@RestControllerAdvice`는 `@ControllerAdvice`와 달리 

`@ResponseBody`가 붙어 있어 응답을 Json으로 내려준다는 점에서 다르다.

<br><br><br>

## ErrorResponseEntity
Error 내용을 담을 Response Entity를 작성한다.

```java
@Data
@Builder
public class ErrorResponseEntity {
    private int status;
    private String code;
    private String message;

    public static ResponseEntity<ErrorResponseEntity> toResponseEntity(ErrorCode e){
        return ResponseEntity
                .status(e.getHttpStatus())
                .body(ErrorResponseEntity.builder()
                        .status(e.getHttpStatus().value())
                        .code(e.name())
                        .message(e.getMessage())
                        .build()
                );
    }
}
```
CustomException을 toResponseEntity로 보내면, ErrorCode e안의 내용을 가지고 Response를 만든다.

<br><br>

🐣 ResponseEntity란?

Spring Framework에서 제공하는 클래스 중 HttpEntity 라는 클래스가 존재한다. 

이것은 HTTP 요청(Request) 또는 응답(Response)에 해당하는 HttpHeader와 HttpBody를 포함하는 클래스이다. 

HttpEntity 클래스를 상속받아 구현한 클래스가 RequestEntity, ResponseEntity 클래스이다.

<br><br><br>

## 출력 예시
### RegistryNotFoundException
```java
// 게시글이 없을 때
public class RegistryNotFoundException extends CustomException {
    public RegistryNotFoundException(){
        super(ErrorCode.REGISTRY_NOT_FOUND);
    }
}
```
<br>

### RegistryServiceImpl
```java
// 게시글 상세 보기
public ResRegistryDto getIdxRegistry(Long idx) throws CustomException {
    Registry getIdxRegistry = registryRepository.findById(idx).orElseThrow(RegistryNotFoundException::new);
    return ResRegistryDto.of(getIdxRegistry);
}
```
<br>

![image](https://user-images.githubusercontent.com/74857364/212059902-96820ae8-ba63-482f-972a-9061209634c8.png)

<br><br><br>

***reference***       
[이론 참고 사이트]               
[[SpringBoot] HTTP Status Code 제어 중요성 및 방법](https://atoz-developer.tistory.com/121)           
[[Spring] @RestControllerAdvice를 이용한 Spring 예외 처리 방법 - (2/2)](https://mangkyu.tistory.com/205)                        
[[Spring] Controller에서의 Exception 처리](https://hanna97.tistory.com/entry/Spring-Controller%EC%97%90%EC%84%9C%EC%9D%98-Exception%EC%B2%98%EB%A6%AC)                                    
[[완벽해설] serialVersionUID에 대한 정확한 설명](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=kkson50&logNo=220564273220)                  
[[Java] "serialVersionUID"이란? 어떤 역할을 가지고 있기에 선언이 되어 있는가?](https://whitekeyboard.tistory.com/496)                                    
[[Spring Boot] ResponseEntity란 무엇인가?](https://devlog-wjdrbs96.tistory.com/182)                   
[[스프링부트] @ExceptionHandler를 통한 예외처리](https://velog.io/@kiiiyeon/%EC%8A%A4%ED%94%84%EB%A7%81-ExceptionHandler%EB%A5%BC-%ED%86%B5%ED%95%9C-%EC%98%88%EC%99%B8%EC%B2%98%EB%A6%AC)  

[코드 참고 사이트 및 영상]                               
[[SpringBoot] Custom Exception Response 만들기](https://velog.io/@dot2__/SpringBoot-Custom-Exception-Response-%EB%A7%8C%EB%93%A4%EA%B8%B0)         
[woowacourse-teams/2022-f12](https://github.com/woowacourse-teams/2022-f12)           
[서비스 특성에 맞춘 예외 처리 방법 Custom Exception [ 스프링 부트 (Spring Boot) ]](https://www.youtube.com/watch?v=5XHhAhN-9po&t=104s)           
              
