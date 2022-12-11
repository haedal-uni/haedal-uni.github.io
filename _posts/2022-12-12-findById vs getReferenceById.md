---
categories: Project
tags: [JPA, study, project, spring]
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

### LAZY 오류
LAZY를 사용하다보면 아래와 같은 오류를 볼 수 있다.

*No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor and no properties discovered to create BeanSerializer*

<br>

**LAZY** 옵션은 필요할때 조회를 해오는 옵션인데           

위 에러는 필요가 없으면 조회를 안해서 비어있는 객체를 serializer 하려고 해서 발생되는 문제이다.                 

<br>

대표적으로 3가지 해결방법이 있지만 3번을 추천한다. 나머지는 근본적인 해결방법이 아니다.                       
(*자세한 내용은 [Proxy]()에서 다룰 예정이다.)


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

