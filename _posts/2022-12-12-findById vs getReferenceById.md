---
categories: Project
tags: [JPA, study, project, spring, Exception]
---

# findById() vs getReferenceById()

`findById()`는 **EAGER**방식의 조회기법이라면 `getReferenceById(ID)`는 **LAZY**방식으로 조회된다.

`getReferenceById(ID)`는 실제 테이블을 조회하는 대신 프록시 객체만 가져온다.

프록시 객체만 있는 경우 ID 값을 제외한 나머지 값을 사용하기 전까지는 실제 DB에 액세스하지 않기 때문에 SELECT 쿼리가 날아가지 않는다.

`findById()`를 사용하면 DB에 바로 액세스해서 데이터를 가져온다.

실제 DB에 접근하느냐 하지 않느냐는 성능에 영향을 줄 수 있다.

단순히 특정 엔티티의 ID 값만 필요한 경우에는 모든 데이터를 가져올 필요가 없다.

연관 관계를 갖는 엔티티를 저장할 때 연관된 엔티티 조회 시 `getReferenceById(ID)`를 사용하는 것이 성능 개선에 도움이 된다.

상황에 따라 적절한 메소드를 사용하면 된다.

<br><br><br>

## test

Comment와 Registry로 테스트를 진행했다. (N:1)

<br>

Comment → Registry가 **LAZY** 관계인 경우

Comment 테이블을 조회하는 시점에 registryId는 이미 FK로 Comment 테이블의 컬럼 값에 포함되어 있다.

JPA는 이 값으로 Registry의 프록시 객체를 생성하기 때문에 프록시 객체 내부에는 이미 registryId를 가지고 있다.

따라서 이 경우 registryId를 조회할 때는 프록시 초기화가 발생하지 않는다.

<br><br><br> 

### findById()

```java
System.out.println(" = = = = = = = = = = == = = = = = = = = = = ");
System.out.println("findById");
System.out.println(registryRepository.findById(commentDto.getRegistryIdx()));
System.out.println();
```
![image](/assets/img/posts/206918932-aca53467-ebc6-4af0-9b93-e1ba8c856b84.png){: width="50%"}

`findById()`를 사용하니 select 쿼리가 떴다.  

<br><br><br> 

### getReferenceById()

```java
System.out.println(" = = = = = = = = = = == = = = = = = = = = = ");
System.out.println();
System.out.println("getReferenceById");
System.out.println(registryRepository.getReferenceById(commentDto.getRegistryIdx()));
System.out.println();
System.out.println("  == = = = = = = = == = = = = =  끝 = = = = = = = == = = = =  ");
```

![image](/assets/img/posts/206918980-e06600ff-c466-40d9-87a5-15d5ae6860c0.png){: width="60%"}


`getReferenceById()`를 사용하자 SELECT 쿼리가 실행되지 않았다.

<br><br><br>  

#### 전체 코드

```java
public Comment setComment(CommentDto commentDto) {
    System.out.println("쀼뷰뷰뷰ㅠ븁");
    Registry registry = registryRepository.getReferenceById(commentDto.getRegistryIdx());
    System.out.println();

    System.out.println(" = = = = = = = = = = == = = = = = = = = = = ");
    System.out.println("findById");
    System.out.println(registryRepository.findById(commentDto.getRegistryIdx()));
    System.out.println();

    System.out.println(" = = = = = = = = = = == = = = = = = = = = = ");
    System.out.println();
    System.out.println("getReferenceById");
    System.out.println(registryRepository.getReferenceById(commentDto.getRegistryIdx()));
    System.out.println();
    System.out.println("  == = = = = = = = == = = = = =  끝 = = = = = = = == = = = =  ");

    Comment comment = commentDto.toEntity(registry);
    Comment save = commentRepository.save(comment);
    return save;
}
```
![image](/assets/img/posts/206919045-d011dc0a-fa9f-42d6-8e16-cb59aeec39a2.png){: width="50%"}

<br><br><br><br>

## 예외 처리

Negative Test를 작성하면서 `getReferenceById()`의 예외 처리에 대해 고민하게 되었다.

<br><br>    

### CommentService.java

```java
@Override
public Comment postComment(CommentRequestDto commentDto) {
    Registry registry = registryRepository.getReferenceById(commentDto.getRegistryIdx());
    User user = userRepository.findByNickname(commentDto.getNickname())
        .orElseThrow(UserNotFoundException::new);
    Comment comment = commentDto.toEntity(registry, user);
    Comment save = commentRepository.save(comment);
    return save;
}
```

본 코드에서는 `findByNickname()`뿐만 아니라 그 이전에 `getReferenceById()`로도 값을 조회한다.

`userRepository.findByNickname()`에는 예외 처리가 되어 있지만  

`registryRepository.getReferenceById()`에는 예외 처리가 되어 있지 않다.

그래서 registry의 idx 값이 존재하지 않는 경우에 대한 의문이 생겼다.

<br><br><br> 

### getReferenceById??

```java
@Override
public Comment postComment(CommentRequestDto commentDto) {
    System.out.println(" = = = = = = = = = = =  postComment 시작  = = = = = = = = = = =  ");

    Registry registry = registryRepository.getReferenceById(commentDto.getRegistryIdx());
    System.out.println("registry.getIdx() : " + registry.getIdx());

    System.out.println(" = = = = = = registry = = = = = = = = ");
    System.out.println("registry :  " + registry);

    System.out.println(" = = = = = = registryRepository.existsById = = = = = = = = = ");
    registryRepository.existsById(commentDto.getRegistryIdx());

    System.out.println(" = = = = = = = = = = = = = = = = = = = = = = ");
}
```

![image](/assets/img/posts/212552851-c9fd6d80-6b41-461f-95e5-cf6adf0cd55d.png){: width="65%"}

`getReferenceById()`는 실제 테이블을 조회하지 않고 프록시 객체만 가져온다.

ID 값을 제외한 다른 값을 사용하기 전까지는 DB에 접근하지 않기 때문에 SELECT 쿼리가 실행되지 않는다.

따라서 `registry.getIdx()`에서는 쿼리가 실행되지 않지만  

registry 객체를 출력하면 프록시 초기화가 발생하며 SELECT 쿼리가 실행된다.

*existsById는 비교를 위해 출력

<br><br>

그러므로 ID 값만 사용할 경우에는  

`findById()`보다 `getReferenceById(ID)`를 사용하는 것이 쿼리를 줄이는 데 유리하다.

![image](/assets/img/posts/212553053-014af67b-1c18-4b4c-b0a4-1814ae9052c9.png)

<br><br><br>

그러나 문제가 존재한다.

ID 값이 실제로 존재하는지 여부를 확인하지 않고 단순히 해당 ID를 가진 프록시 객체를 생성한다.

```java
public Comment postComment(CommentRequestDto commentDto) {
    System.out.println(" = = = = = = = = = = =  postComment 시작  = = = = = = = = = = =  ");

    Registry registry = registryRepository.getReferenceById(commentDto.getRegistryIdx());
    System.out.println("registry.getIdx() :  " + registry.getIdx());

    if (registry.getIdx() == null) {
        System.out.println("hello");
    }

    User user = userRepository.findByNickname(commentDto.getNickname())
        .orElseThrow(UserNotFoundException::new);
    Comment comment = commentDto.toEntity(registry, user);
    Comment save = commentRepository.save(comment);
    return save;
}
```

존재하지 않는 registry idx를 테스트했음에도  

`registry.getIdx() : 40`과 같이 출력되었고 에러가 발생했다.

![image](/assets/img/posts/212553279-085ea4a2-ba2e-4df5-b948-33e76beebcf0.png)


<br><br><br>

### return

### findById()

![image](/assets/img/posts/212559667-338810b3-4d0e-4e48-89b1-e08301273011.png)

매개변수로 전달된 ID에 해당하는 entity를 반환하거나 존재하지 않을 경우 `Optional.empty()`를 반환한다.

즉 조회 결과가 없어도 내부적으로 예외를 발생시키지 않는다.

ID가 null일 경우 IllegalArgumentException이 발생할 수 있다.

<br><br>

### getReferenceById()
![image](/assets/img/posts/212559690-c85344fd-c981-4e65-9020-0cd31727eba6.png)

반환 타입이 Optional이 아닌 엔티티 자체이다.

해당 ID의 엔티티가 존재하지 않을 경우 예외가 발생한다.

존재하지 않는 대상에 대한 예외는 XXNotFoundException과 같은 커스텀 예외로 처리할 수 있다.

<br><br><br>  

### 예외 처리 코드 작성
*[Custom Exception]() 정리 글             

```java
@RestControllerAdvice
public class CustomExceptionHandler {

    @ExceptionHandler(SQLException.class)
    public ResponseEntity<ErrorResponseEntity> handleSQLException() {
        return ErrorResponseEntity.toResponseEntity(ErrorCode.DATABASE_ERROR);
    }

}
```

`@ExceptionHandler`를 이용해 Controller에서 예외 처리 메소드를 추가했다.

`@ExceptionHandler`에 정한 예외가 발생하면 해당 handler가 실행된다. (SQLException 예외를 처리)  

*@Service나 @Repository가 적용된 Bean에서는 사용할 수 없다.

<br>

`@ControllerAdvice`는 모든 Controller에서 발생하는 예외를 처리할 수 있도록 해준다.

*`@RestControllerAdvice`: `@ControllerAdvice`+ `@ResponseBody`

<br><br>

위와 같이 작성하면 존재하지 않는 ID로 조회 시 서버 에러가 아닌 정의한 예외 응답 형태로 반환된다.

![image](/assets/img/posts/212668238-13a29e52-1375-421c-a1e3-16dca3378501.png)

<img src="/assets/img/posts/37bf46f5-cea3-4e10-9d18-8f58c8ee707f.png" />


<br><br><br><br> 

## LAZY 오류

LAZY를 사용할 경우 아래와 같은 오류를 만날 수 있다.

*No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer*

LAZY는 필요한 시점에만 조회하는 전략이다.

해당 오류는 아직 초기화되지 않은 프록시 객체를 직렬화하려고 시도하면서 발생한다.

<br>

대표적인 해결 방법은 3가지가 있다. (3번 권장)  

**1.** application 설정에 `spring.jackson.serialization.fail-on-empty-beans=false` 추가

**2.** LAZY 설정을 EAGER로 변경

**3.** 문제가 발생하는 필드에 `@JsonIgnore` 적용

<br><br><br>

### getReferenceById version

버전에 따라 `getReferenceById()`가 존재하지 않을 수 있다.

Spring Boot 2.5 미만 : `getOne()`

Spring Boot 2.7 미만 : `getById(ID)`

Spring Boot 2.7 이상 : `getReferenceById(ID)`

<br>

Spring Boot 2.7 미만

![image](/assets/img/posts/204102709-4319feaa-e7fd-4491-91a2-36850457b33d.png){: width="65%"}


<br><br>

Spring Boot 2.7 이상

![image](/assets/img/posts/205506648-30bfa185-71e0-47bc-9b7f-4cd733f7abbb.png){: width="50%"}

<br><br><br><br>  

---

*reference*            
[find vs get (네이밍 컨벤션과 JPA에서의 내부 동작 차이)](https://creampuffy.tistory.com/162)         
[ExceptionHandler 와 ControllerAdvice](https://tecoble.techcourse.co.kr/post/2021-05-10-controller_advice_exception_handler/)                                              
