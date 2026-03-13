---
categories: Study
tags: [DB, study, summary]
---
# GenerationType 정리 (실제 테스트 기반)

test 코드를 작성하면서 발생한 여러 문제들이 

GenerationType 설정 문제라는 것을 알게 되었고 직접 테스트하면서 정리한 내용이다.

<br><br>

## GenerationType 종류
JPA의 기본키 생성

- AUTO
- IDENTITY
- SEQUENCE
- TABLE

<br><br>

### 기본 키 제약 조건
- null 이 아니어야 한다
- 유일해야 한다
- 변경되면 안 된다

<br><br><br>

### AUTO

- 기본 설정값
- 데이터베이스 방언(dialect)에 따라 전략을 자동 선택한다
- 내부적으로 `TABLE`, `IDENTITY`, `SEQUENCE` 중 하나로 결정된다

<br><br>

#### DB별 AUTO 전략

- MySQL : `TABLE`(Hibernate 5.0 이상 기준)
- H2 : `SEQUENCE`

<br>

즉, AUTO는 환경에 따라 다르게 동작한다.   

<br><br><br>

#### Spring Boot + Hibernate 버전 이슈

Spring Boot에는 Hibernate의 ID 생성 전략을 그대로 따를지 여부를 결정하는 옵션이 있다.

```java
private void applyNewIdGeneratorMappings(Map<String, Object> result) {
	if (this.useNewIdGeneratorMappings != null) {
		result.put(AvailableSettings.USE_NEW_ID_GENERATOR_MAPPINGS, this.useNewIdGeneratorMappings.toString());
	}
	else if (!result.containsKey(AvailableSettings.USE_NEW_ID_GENERATOR_MAPPINGS)) {
		result.put(AvailableSettings.USE_NEW_ID_GENERATOR_MAPPINGS, "true");
	}
}
```

기본값  
- Spring Boot 1.5 :  `false`
- Spring Boot 2.0 이상 : `true`

<br>

이 설정 때문에 다음과 같은 차이가 발생한다.

- Spring Boot 1.5 : Hibernate 5를 써도 AUTO가 IDENTITY처럼 동작
- Spring Boot 2.0 이상 : Hibernate 5의 AUTO 정책을 그대로 따름 (`TABLE`)

<br><br>

#### 해결 방법
**1.** Hibernate 설정을 false로 변경

```properties
spring.jpa.hibernate.use-new-id-generator-mappings=false
```

<br>

**2.** GenerationType을 명시적으로 지정

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

<br>

나는 2번 방식을 사용하고 있었기 때문에 AUTO가 아닌 IDENTITY처럼 동작했던 것이다.

<br><br><br>

### IDENTITY
- 기본키 생성을 DB에게 위임
- MySQL의 `AUTO_INCREMENT` 방식
- id 값을 직접 세팅하지 않는다

<br>

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
```

<br><br>

#### 동작 방식
JPA는 보통 영속성 컨텍스트에서 객체를 관리하다가 트랜잭션 커밋 시점에 INSERT 쿼리문을 실행한다.

하지만 IDENTITY 전략에서는 `EntityManager.persist()`를 하는 시점에 

Insert SQL을 실행하여 DB에서 식별자를 조회해온다.

그 이유는 영속성 컨텍스트가 객체를 관리하려면 PK 값을 알아야 하기 때문이다.

<br>

AUTO_INCREMENT는 INSERT 이후에야 PK를 알 수 있기 때문에 JPA는 즉시 INSERT를 실행해서 PK를 가져온다.

<br>

그 결과 지연 쓰기(write-behind)가 제한된다. 

*하지만 실무에서 큰 성능 문제는 거의 없음   

<br><br><br>

## 문제 상황 1  
### IDENTITY + EntityNotFoundException

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

    comment.setRegistry(registry);
    commentRepository.save(comment);

    Comment savedComment = commentRepository.findById(1L).get();
    Registry savedRegistry = savedComment.getRegistry();

    Assertions.assertThat(savedRegistry.getNickname()).isEqualTo("coco");
}
```

nested exception is javax.persistence.EntityNotFoundException 에러 발생  

<br><br>

#### 원인  

- IDENTITY 에서는 PK를 DB가 자동 생성
- `setIdx(3L)`로 값을 넣어도 실제 DB에는 1L로 저장된다 (AUTO_INCREMENT)

<br>

엔티티에는 3L, DB에는 1L로 영속성 컨텍스트와 DB 상태 불일치 발생


연관관계가 걸린 상태에서 존재하지 않는 ID를 참조하려 하면서 예외 발생


→ IDENTITY 사용 시 PK 직접 세팅 X, idx 값은 null 상태로 두어야 한다

<br><br><br>

## 문제 상황 2  
### AUTO + 값이 1이 아닌 2로 조회됨

```java
@DataJpaTest
class CommentAndRegistryTest {

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

        comment.setRegistry(registry);
        commentRepository.save(comment);

        Comment savedComment = commentRepository.findById(2L).get(); // 👈🏻
        Registry savedRegistry = savedComment.getRegistry();
    }
}
```

<br>

pk를 1L로 저장했다고 생각했는데 조회는 2L로 해야 정상 동작

<br><br>

#### 원인 추론

- `@DataJpaTest` → H2 사용
- H2에서 AUTO = SEQUENCE
- SEQUENCE는 Current Value를 공유

<br>

- Registry 저장 → 1
- Comment 저장 → 2

<br><br><br>

### Test
#### AUTO 전략에서 정말 값을 공유하는지 확인

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
    comment1.setRegistry(registry1);

    Registry savedRegistry = registryRepository.save(registry1);
    Comment savedComment = commentRepository.save(comment1);

    System.out.println("registry idx : " + savedRegistry.getIdx());
    System.out.println("comment idx  : " + savedComment.getIdx());
}
```

<br>

#### 결과
registry → 1, comment → 2

→ 같은 시퀀스를 공유하고 있었다. 

<br><br><br>

### H2 / MySQL 확인

- H2
  - AUTO, SEQUENCE → HIBERNATE_SEQUENCE 생성
  - 여러 엔티티가 하나의 시퀀스 공유

- MySQL
  - SEQUENCE 미지원
  - SEQUENCE / TABLE 전략 사용 시 hibernate_sequence 테이블 생성

<br>

차이점
- SEQUENCE → Current Value 공유
- TABLE → 테이블은 공유하지만 값은 엔티티별 관리

<br><br><br>

## 정리
- AUTO는 환경 / 버전 / DB에 따라 동작이 다르다. 
- 테스트 환경(H2)과 운영(MySQL)에서 결과가 달라질 수 있다. 

<br>

GenerationType은 명시적으로 지정하고 MySQL 환경에서는 AUTO 쓸 바에는 IDENTITY 쓴다 😂

<br><br><br>

REFERENCE    
                      
[[JPA] 기본키(PK) 매핑 방법 및 생성 전략](https://gmlwjd9405.github.io/2019/08/12/primary-key-mapping.html)               
[엔티티 매핑](https://catsbi.oopy.io/f03397e5-a900-4f1c-956f-fc660a1ac3c5)           
[Spring Boot Data JPA 2.0 에서 id Auto_increment 문제 해결](https://jojoldu.tistory.com/295)                           

