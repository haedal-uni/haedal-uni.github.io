---
categories: Project
tags: [JPA, study, project, spring, Exception]
---

# findById() vs getReferenceById()

`findById()`는 **EAGER**방식의 조회기법이라면 `getReferenceById(ID)`는 **LAZY**방식으로 조회된다.

`getReferenceById(ID)` 는 실제 테이블을 조회하는 대신 프록시 객체만 가져온다.

프록시 객체만 있는 경우 ID 값을 제외한 나머지 값을 사용하기 전까지는 실제 DB 에 액세스 하지 않기 때문에               
 **SELECT** 쿼리가 날아가지 않는다.

`findById()`를 사용하면 DB 에 바로 액세스해서 데이터를 가져온다.

실제 DB 에 접근하냐 하지 않냐는 성능에 영향이 갈 수 있다.

단순히 특정 엔티티의 ID 값만 필요한 경우에는 모든 데이터를 가져올 필요가 없다.

연관 관계를 갖는 엔티티를 저장할 때, 연관된 엔티티 조회시 `getReferenceById(ID)`를 사용하는 것이 성능 개선에 도움이 된다.

*상황에 따라 적절한 메소드를 사용하면 된다.

<br><br><br>

## test
Comment와 Registry로 test를 해봤다. (N:1)

<br>

Comment → Registry 가 **LAZY** 관계이면

Comment 테이블을 조회하는 시점에 registryId가 이미 FK로 Comment 테이블의 값에 포함되어 있다.

JPA는 이 값으로 Registry의 프록시 객체를 만들기 때문에 프록시 객체는 내부에 이미 registryId를 가지고 있다.

따라서 이 경우 registryId를 조회할 때는 프록시를 초기화 하지 않는다.

<br><br>

### findById()
```java
System.out.println(" = = = = = = = = = = == = = = = = = = = = = ");
System.out.println("findById");
System.out.println(registryRepository.findById(commentDto.getRegistryIdx()));
System.out.println();
```
![image](https://user-images.githubusercontent.com/74857364/206918932-aca53467-ebc6-4af0-9b93-e1ba8c856b84.png){: width="50%"}

`findById()`를 사용하니 select 쿼리가 떴다.

<br><br>

### getReferenceById()  
```java
System.out.println(" = = = = = = = = = = == = = = = = = = = = = ");
System.out.println();
System.out.println("getReferenceById");
System.out.println(registryRepository.getReferenceById(commentDto.getRegistryIdx()));
System.out.println();
System.out.println("  == = = = = = = = == = = = = =  끝 = = = = = = = == = = = =  ");
```
![image](https://user-images.githubusercontent.com/74857364/206918980-e06600ff-c466-40d9-87a5-15d5ae6860c0.png){: width="60%"}


`getReferenceById()`를 사용하니 실제로 SELECT 쿼리가 날아가지 않는다.

<br><br>

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

![image](https://user-images.githubusercontent.com/74857364/206919045-d011dc0a-fa9f-42d6-8e16-cb59aeec39a2.png){: width="50%"}


<br><br><br>

## 예외 처리
[Negative Test]()를 작성하면서 `getReferenceById()`에 관한 예외 처리를 생각하게 되었다.

<br>

### CommentService.java
```java
@Override
public Comment postComment(CommentRequestDto commentDto) {
    Registry registry = registryRepository.getReferenceById(commentDto.getRegistryIdx());
    User user = userRepository.findByNickname(commentDto.getNickname()).orElseThrow(UserNotFoundException::new);
    Comment comment = commentDto.toEntity(registry, user);
    Comment save = commentRepository.save(comment);
    return save;
}
```
본 코드에서 *findByNickname*만 사용하는게 아니라 그 위 코드 *getReferenceById*로도 값을 찾는다.

`userRepository.findByNickname()`에 exception 처리가 되어있지만 

`registryRepository.getReferenceById()`에는 exception 처리가 되어있지 않다.

그래서 문득 예외상황에 대한 test 코드를 작성하면서 registry의 idx값도 존재하지 않을 수도 있을텐데? 싶었다.

<br><br>

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
`getReferenceById()`를 사용하면 실제 테이블을 조회하는 대신 프록시 객체만 가져온다.

프록시 객체만 있는 경우 ID 값을 제외한 나머지 값을 사용하기 전까지는 실제 DB 에 액세스 하지 않기 때문에 SELECT 쿼리가 날아가지 않는다.

그렇기 때문에 `registry.getIdx()`에서는 쿼리문이 로그에 찍히지 않지만

registry 자체로 출력할 때는 로그에 select문이 찍히는 것을 볼 수 있다. (*existsById는 참고로 출력해봤다.)

<br>

![image](https://user-images.githubusercontent.com/74857364/212552851-c9fd6d80-6b41-461f-95e5-cf6adf0cd55d.png){: width="65%"}

<br><br>

그러므로 id 값만 사용할 것이라면 `findById()`보다는 `getReferenceById(ID)`를 쓰는게 쿼리가 덜 날라가기 때문에 좋을 것이다.

![image](https://user-images.githubusercontent.com/74857364/212553053-014af67b-1c18-4b4c-b0a4-1814ae9052c9.png)

<br><br><br>

그런데 문제가 있다.

id값이 있는지 없는지를 따지는게 아니라 그냥 그 값을 가져온다. 

```java
public Comment postComment(CommentRequestDto commentDto) {
    System.out.println(" = = = = = = = = = = =  postComment 시작  = = = = = = = = = = =  ");
    
    Registry registry = registryRepository.getReferenceById(commentDto.getRegistryIdx());
    System.out.println("registry.getIdx() :  " + registry.getIdx());
    
    if(registry.getIdx() == null) {
        System.out.println("hello");
    }
    
    User user = userRepository.findByNickname(commentDto.getNickname()).orElseThrow(UserNotFoundException::new);
    Comment comment = commentDto.toEntity(registry, user);
    Comment save = commentRepository.save(comment);

    return save;
}
```
registry의 없는 idx값을 test하였으나 `registry.getIdx() :  40`로 출력이 되고 아래와 같이 에러가 떴다.

![image](https://user-images.githubusercontent.com/74857364/212553279-085ea4a2-ba2e-4df5-b948-33e76beebcf0.png)

<br><br><br>

### return

**findById()**

![image](https://user-images.githubusercontent.com/74857364/212559667-338810b3-4d0e-4e48-89b1-e08301273011.png)

매개변수로 전달된 ID에 해당하는 entity를 반환하거나, 해당하는 entity가 없을 경우 Optional.empty()를 반환한다.

즉, 탐색 결과가 없더라도 내부에서 예외를 발생시키지 않는다.


Throws를 살펴보면, ID가 null일 경우엔 IllegalArgumentException이 발생할 수 있다.

<br><br>

**getReferenceById()**

![image](https://user-images.githubusercontent.com/74857364/212559690-c85344fd-c981-4e65-9020-0cd31727eba6.png)

`Optional<T>` 가 아닌 T가 반환값이다.
  
즉, 매개변수로 전달된 ID에 해당하는 entity를 반환하되, 없을 경우 내부에서 예외를 발생시킨다.

특정 대상이 존재하지 않을 때의 커스텀 예외를 XXNotFoundException이라고 할 수 있다.
  
<br><br>  

### 예외 처리 코드 작성
*[Custom Exception](https://haedal-uni.github.io/posts/Custom-Exception/) 정리 글

```java
@RestControllerAdvice
public class CustomExceptionHandler {
  
    @ExceptionHandler(SQLException.class)
    public ResponseEntity<ErrorResponseEntity> handleSQLException(){
        return ErrorResponseEntity.toResponseEntity(ErrorCode.DATABASE_ERROR);
    }

}
```
`@ExceptionHandler`를 이용해 Controller에 예외 처리 메소드를 추가했다.

`@ExceptionHandler` 에 설정한 예외가 발생하면 handler가 실행된다. → SQLException 예외를 처리하기 위해 작성

🐣 @Controller, @RestController가 아닌 @Service 나 @Repository 가 적용된 Bean에서는 사용할 수 없다.

<br>

`@ControllerAdvice`는 `@Controller` 어노테이션이 있는 모든 곳에서의 예외를 잡을 수 있도록 해준다.

`@ControllerAdvice` 안에 있는 `@ExceptionHandler`는 모든 컨트롤러에서 발생하는 예외상황을 잡을 수 있다.   

🐣 `@ControllerAdvice` + `@ResponseBody` → `@RestControllerAdvice` : `@ControllerAdvice` + 객체를 반환할 수 있다.

<br><br>

위와 같이 작성하면 없는 id값을 가지고 조회할 때 위처럼 에러가 뜨는 것이 아니라 예외처리가 되어 아래와 같이 출력된다.

![image](https://user-images.githubusercontent.com/74857364/212668238-13a29e52-1375-421c-a1e3-16dca3378501.png)


<br><br><br>

## LAZY 오류
LAZY를 사용하다보면 아래와 같은 오류를 볼 수 있다.

*No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer*

<br>

**LAZY** 옵션은 필요할때 조회를 해오는 옵션인데           

위 에러는 필요가 없으면 조회를 안해서 비어있는 객체를 serializer 하려고 해서 발생되는 문제이다.                 

<br>

대표적으로 3가지 해결방법이 있지만 3번을 추천한다. 나머지는 근본적인 해결방법이 아니다.                       
(*자세한 내용은 [Proxy](https://haedal-uni.github.io/posts/Proxy/)에서 다룰 예정이다.)


1. application 파일에 *spring.jackson.serialization.fail-on-empty-beans=false* 설정해주기

2. 오류가 나는 엔티티의 LAZY 설정을 EAGER로 바꿔주기

3. 오류가 나는 컬럼에 @JsonIgnore를 설정해주기

<br><br><br>

### getReferenceById version
버전에 따라서 `getReferenceById()`로 나타나지 않을 수 있다. 

spring boot version 2.5 미만의 경우 `getOne()`을 사용하고

spring boot version 2.7 미만의 경우 `getOne(ID)` is deprecated 되고 `getById(ID)`로 대체 되었다. 

그리고 spring boot version 2.7 이상 부터는 `getById()` 대신 `getReferenceById(ID)`로 대체 되었다.

<br>

**spring boot version 2.7 미만**       

![image](https://user-images.githubusercontent.com/74857364/204102709-4319feaa-e7fd-4491-91a2-36850457b33d.png){: width="65%"}

<br><br>

**spring boot version 2.7 이상**       

![image](https://user-images.githubusercontent.com/74857364/205506648-30bfa185-71e0-47bc-9b7f-4cd733f7abbb.png){: width="50%"}


<br><br><br>

*reference*            
[find vs get (네이밍 컨벤션과 JPA에서의 내부 동작 차이)](https://creampuffy.tistory.com/162)         
[ExceptionHandler 와 ControllerAdvice](https://tecoble.techcourse.co.kr/post/2021-05-10-controller_advice_exception_handler/)                                              
