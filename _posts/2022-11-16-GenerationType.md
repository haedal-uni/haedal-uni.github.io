---
categories: Study
tags: [DB, study, summary]
---

# GenerationType
test 코드를 실행하면서 생긴 문제들이 GenerationType과 관련되어있어 이를 정리해봤다.  

GenerationType 에는 **AUTO**, **IDENTITY**, **SEQUENCE**, **TABLE** 이 있다.   

<br><br>

### AUTO
id 값을 null로 하면 DB가 알아서 AUTO_INCREMENT 해준다.

기본 설정 값으로 각 데이터베이스에 따라 기본키를 자동으로 생성한다.

insert 쿼리 전에 select, update 쿼리가 실행된다.

<br><br>

**기본 키 제약 조건**
- null이 아니다.
- 유일하다.
- 변하면 안된다

<br><br>

방언에 따라 TABLE, IDENTITY, SEQUENCE 3개 중에 하나가 자동으로 지정된다.

AUTO로 설정 시
- MySQL : **`IDENTITY`**
  - Hibernate 5.0부터는 MySQL의 `AUTO`는 `IDENTITY`가 아닌 `TABLE`을 기본 시퀀스 전략으로 선택한다.
  - [관련 글](https://pomo0703.tistory.com/203)

- H2 : **`SEQUENCE`**

![image](https://user-images.githubusercontent.com/74857364/201987641-3000584b-e896-4c37-aeba-62d5f16fda2f.png)

<br><br><br>

#### 정리
이 전에 내가 Comment와 Registry를 AUTO로 설정했을 때 

내 버전은 5.0 이상인걸로 확인했으나 HIBERANTE SEQUENCE TABLE이 안나왔었다.

그래서 해당 글이 맞는지 확신을 못했었는데 [블로그 글](https://jojoldu.tistory.com/295)을 보고 왜 안나왔는지 이유를 알 수 있었다.

<br>

먼저 정리를 하자면 Spring Boot는 Hibernate의 id 생성 전략을 그대로 따라갈지 말지를 결정하는 `useNewIdGeneratorMappings` 설정이 있다.

Spring Boot 1.5에선 기본값이 false, Spring Boot 2.0부터는 true이다.

Hibernate 5.0부터 MySQL의 AUTO는 `IDENTITY`가 아닌 `TABLE`을 기본 시퀀스 전략으로 선택된다.

즉, 1.5에선 Hibernate 5를 쓰더라도 AUTO를 따라가지 않기 때문에 `IDENTITY`가 선택된다.

2.0에선 true이므로 Hibernate 5를 그대로 따라가기 때문에 TABLE이 선택된다.

<br><br>

만약 이를 해결하고 싶다면 2가지 방법이 있다.

이 중 나는 1번을 적용하고 있어서 `TABLE`이 아닌 `IDENTITY`가 되었던 것이다.

<br>

**1. application.properties에서 아래와 같이 false로 설정한다.**
```
# do not share id
spring.jpa.hibernate.use-new-id-generator-mappings= false
```
<br><br>

**2. @GeneratedValue의 전략을 `GenerationType.IDENTITY`로 지정한다**


<br><br><br>

### IDENTITY
기본키 생성을 db에게 위임하는 방식으로 id값을 따로 할당하지 않아도 

db가 자동으로 AUTO_INCREMENT를 하여 기본키를 생성해준다.

id 값을 null로 하면 DB가 알아서 AUTO_INCREMENT 해준다.

<br>   

JPA는 보통 영속성 컨텍스트에서 객체를 관리하다가 트랜잭션 커밋 시점에 INSERT 쿼리문을 실행한다.                               
 
하지만 **IDENTITY** 전략에서는 `EntityManager.persist()`를 하는 시점에 Insert SQL을 실행하여 DB에서 식별자를 조회해온다.           
그 이유는 영속성 컨텍스트는 1차 캐시에 PK와 객체를 가지고 관리를 하는데 기본키를 데이터베이스에게 위임했기 때문에                         
`EntityManager.persist()` 호출 하더라도 DB에 값을 넣기 전까지 기본키를 모르고 있기 때문에 관리가 되지 않기 때문이다.            

AUTO_INCREMENT는 DB에 INSERT 쿼리문을 실행 한 이후에 ID값을 알 수 있다.                         
영속성 관리 시점에서 1차캐시에 @Id값을 알 수 없다는 말이된다. (IDENTITY 전략은 AUTO_INCREMENT로 동작하기 때문)                     

따라서 IDENTITY 에서는 `EntityManager.persist()`를 하는 시점에 insert 쿼리를 실행해                       
DB에서 식별자를 조회하여 영속성 컨텍스트 1차 캐시에 값을 넣어주기 때문에 관리가 가능해진다.        

그렇기에 IDENTITY 에서는 지연쓰기가 제한된다 . (하지만 크게 성능 하락이 있거나 하지는 않다.)                      

<br><br><br>

### Sequence
DB Sequence Object를 사용할 때 쓰인다.

Current value를 서로 공유한다.

<br><br><br>

### Table
키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략

Hibernate_Sequence 생성


모든 데이터 베이스에서 사용할 수 있지만, 성능이 떨어진다.

<br><br><br>


## 문제 상황
### IDENTITY(EntityNotFoundException)

```java
    @Test
    void commentSave() {
        Registry registry = new Registry();
        registry.setIdx(3L);  // 👈🏻
        registry.setNickname("coco");
        registry.setTitle("안녕하세요");
        registry.setMain("hi");

        registryRepository.save(registry);

        Comment comment = new Comment();
        comment.setComment("❤️🧡💛💚💙💜🤎🖤");
        comment.setNickname("우헤헤");
        comment.setRegistryId(5L);
        comment.setRegistryNickname("pop");

        comment.setRegistry(registry);
        commentRepository.save(comment);

        Comment savedComment  = commentRepository.findById(1L).get();
        Registry savedRegistry = savedComment.getRegistry();

        Assertions.assertThat("coco").isEqualTo(savedRegistry.getNickname());
        Assertions.assertThat("❤️🧡💛💚💙💜🤎🖤").isEqualTo(savedComment.getComment());
    }
```

*nested exception is javax.persistence.EntityNotFoundException* 에러가 떴다.

<br>
 
`IDENTITY`로 설정 후 3L로 저장했을 때 3L을 갖고오지 못하는 이유는 **auto_increment**로 인해 1L로 저장이 된다.
 
<br>

🐣 해당 에러는 데이터를 저장할 때 @OneToOne, @OneToMany.. 등의 annotation이 선언되어 있을 경우에                      
매핑된 id값이 0인 경우에 오류가 발생하는 경우가 있다. id값을 null 처리를 하게 되면 조회 할 수 있다.           
→ idx 값 설정을 생략하라는 말     


<br><br><br>

### AUTO(값을 제대로 못가져 오는 상황)

pk를 1L로 저장을 하고 꺼내올 때 1L로 꺼내오려고 하면 에러가 뜨고 계속 2L로 가져오는 상황이 나타났다.

```java
@DataJpaTest
public class CommentandRegistryTest {
    @Test
    void commentSave() {
        Registry registry = new Registry();
        registry.setIdx(1L);
        registry.setNickname("coco");
        registry.setTitle("안녕하세요");
        registry.setMain("hi");

        registryRepository.save(registry);

        Comment comment = new Comment();
        comment.setIdx(1L);
        comment.setComment("❤️🧡💛💚💙💜🤎🖤");
        comment.setNickname("우헤헤");
        comment.setRegistryId(5L);
        comment.setRegistryNickname("pop");

        comment.setRegistry(registry);
        commentRepository.save(comment);


        System.out.println("comment.getComment() :   " + comment.getComment());
        Comment savedComment = commentRepository.findById(2L).get();  // ← 
        Registry savedRegistry = savedComment.getRegistry();

        System.out.println("savedComment :  " + savedComment);
        System.out.println("savedRegistry :   " + savedRegistry);
        System.out.println("(savedRegistry.getComments() :  " +savedRegistry.getComments());

    }
}
```

![image](https://user-images.githubusercontent.com/74857364/201512413-394d74ed-7671-4382-80df-4301510868e3.png)

<br><br>

Comment의 GenerationType을 `AUTO`로 설정했었는데 `IDENTITY`로 바꾸면서 정상적으로 1L로 값을 가져왔다.

<br>

여기서 @DataJpaTest를 사용했기 때문에(H2) `AUTO`는 `SEQUENCE`이다. 

그러므로 Current value로 값을 공유해서 먼저 저장한 것이 l, 이후에 저장한 것이 2로 나오게 된 것이다.

<br><br><br>


#### 문제 파악하기
왜 AUTO로 설정하면 값이 1L이 아닌 2L로 저장이 될까???

Registry와 Comment 설정을 AUTO(`@GeneratedValue(strategy = GenerationType.AUTO)`)로 하고

아래 코드를 작성한 후 출력시켰다.

```java
@Test
    void autoTest() {
        Registry registry1 = new Registry();
        registry1.setNickname("coco");
        registry1.setTitle("안녕하세요");
        registry1.setMain("hi");


        Comment comment1 = new Comment();
        comment1.setComment("❤️🧡💛💚💙💜🤎🖤");
        comment1.setNickname("우헤헤");
        comment1.setRegistryId(5L);
        comment1.setRegistryNickname("pop");
        comment1.setRegistry(registry1);

        Registry savedRegistry1 = registryRepository.save(registry1);
        Comment savedComment1 = commentRepository.save(comment1);

        System.out.printf("%s idx : %d\n", "comment", savedComment1.getIdx());
        System.out.printf("%s idx : %d\n", "registry", savedRegistry1.getIdx());
    }
```

![image](https://user-images.githubusercontent.com/74857364/201984124-f957d6b7-4e08-4d72-b34e-5535698ea95f.png)

둘다 1로 나올 것만 같았지만 결과는 1과 2로 나왔다. 

Current value로 값을 공유해서 먼저 저장한 것이 l, 이후에 저장한 것이 2로 저장한것이 아닐까 라는 생각으로

Comment와 Registry를 하나씩 더 추가해봤다.

<br><br>

![image](https://user-images.githubusercontent.com/74857364/201984707-29b27b0f-c737-40a4-9a2d-7f8b3df38bd2.png)

저장된 순서대로 값이 출력되고 있었고 idx 값을 공유하고 있다고 판단할 수 있었다.

<br><br>

H2 콘솔로 확인을 해봤다.

<br><br>

#### H2
**application**             

![image](https://user-images.githubusercontent.com/74857364/202108619-25e38910-7a6c-4477-b02b-b91f1025cd22.png)

<br>

**test**

![image](https://user-images.githubusercontent.com/74857364/202108293-96d84c80-3474-46b0-b6b0-37fa11c96312.png)

<br><br>

이번엔 Comment와 Registry를 `SEQUENCE`로 바꾸고 application을 실행해봤다.

![image](https://user-images.githubusercontent.com/74857364/202109919-19ce56bf-191f-4f41-bf09-609fba198a3a.png)

<br>

h2로 실행해보니 HIBERNATE_SEQUENCE가 생겼다. 
그런데 2개를 변경했는데 HIBERNATE_SEQUENCE는 하나만 생긴 것을 볼 수 있다.

*HIBERNATE_SEQUENCE 아래 SYSTEM_SEQUENCE는 기존에 있던 다른 클래스로 인해 생긴 것

<br><br>

여기서 몇 가지를 파악할 수 있었다. `AUTO`로 설정하면
- test 코드에서 Current value를 서로 공유하는 것을 볼 수 있었다.       
  - H2는 auto가 `SEQUENCE`이기 때문에 Current value를 공유하는 것을 볼 수 있다.

- application으로 실행하면 Syestem_Sequence가 떴고 test에서는 table hibernate_sequences가 생성되었다.


<br><br>

이번에는 `TABLE`로 변경해봤다. 

![image](https://user-images.githubusercontent.com/74857364/202111675-32ab209f-416e-490d-9a21-c3502c264f4b.png)

<br><br><br>

#### MySQL
그럼 MySQL도 TEST 해봐야 하지 않을까?

MySQL도 TEST 해본 결과 H2에서는 뜨던 SYSTEM_SEQUENCE가 뜨지 않고              
`SEQUENCE`로 설정하거나 `TABLE`로 설정했을 시 HIBERNATE SEQUENCE가 뜨는 것을 볼 수 있었다.      

![image](https://user-images.githubusercontent.com/74857364/202113679-fbd1884f-49f6-4545-8fdc-19c6a83e4dd2.png)

<br>

MySQL에는 시퀀스 기능이 없다고한다. 그래서 sequence전략 사용시 hibernate_sequence 테이블을 생성한다.  

이를 통해서 `SEQUENCE`와 `TABLE`의 차이는 HIBERANTE SEQUENCE TABLE을 만들지만                    
SEQUENCE는 Current value를 공유하고 table은 공유하지 않는다는 차이점도 파악할 수 있었다.                      

<br><br><br>

## 정리
`EntityManager.persist()`를 하는 시점에 기본키를 알면 `AUTO`, 모르면 `IDENTITY` 

<br>

- 어떤 db냐에 따라서 AUTO가 바뀐다.

  - 버전 차이에 따라서 AUTO 기본 값이 다르다.

- MySql, H2 DB인 경우 `SEQUENCE`일 때 Current value를 공유한다.


<br><br>

GenerationType을 잘 모르는 상황에서 `AUTO`로 작성했을 때 어떻게 돌아가는지 모르는 경우 위험하다.

따라서 직접 정해서 작성해주는 것이 좋은데 MySQL에서는 `SEQUENCE` 기능을 제공하지 않고 있다.

사실상 `TABLE`과 `IDENTITY`인데 TABLE을 생성하기 보다는 `IDENTITY`로 하는게 낫다고 생각한다.

그래서 `AUTO` 쓸 바에는 `IDENTITY` 쓴다.😂

<br><br><br>

출처         
[@GeneratedValue 전략](https://velog.io/@gudnr1451/GeneratedValue-%EC%A0%95%EB%A6%AC)               
[(JPA) JPA @Id GenerationType.AUTO, IDENTITY 차이](https://lion-king.tistory.com/entry/JPA-JPA-Id-GenerationTypeAUTO-IDENTITY)                   
[[JPA] @GeneratedValue AUTO와 IDENTITY 차이점](https://velog.io/@gkdud583/JPA-GeneratedValue-AUTO%EC%99%80-IDENTITY-%EC%B0%A8%EC%9D%B4%EC%A0%90)                
[[JPA] 기본키(PK) 매핑 방법 및 생성 전략](https://gmlwjd9405.github.io/2019/08/12/primary-key-mapping.html)            
[javax.persistence.EntityNotFoundException: Unable to find ... with id 0 에러](https://bkjeon1614.tistory.com/35)               
[엔티티 매핑](https://catsbi.oopy.io/f03397e5-a900-4f1c-956f-fc660a1ac3c5)          
[Spring Boot Data JPA 2.0 에서 id Auto_increment 문제 해결](https://jojoldu.tistory.com/295)              
