---
categories: Study
tags: [spring, study, summary]
---

# @SpringBootTest와 @DataJpaTest 차이

연관관계 매핑 test를 하기 위해 어노테이션 설정을 했다.                

이 전에 실제 db가 아닌 h2로 테스트 하기 위해서 test용 application으로 설정했었다.          

`@TestPropertySource(locations = "/application.properties")`

또, @Transactional을 설정했는데 이를 한꺼번에 해주는 어노테이션이 있었다.

<br><br>

🐣 `@Transactional`
- `@Transactional`을 사용하는 이유는 트랜잭션 처리를 하기 위해서다.                
    - 일련의 작업들을 묶어서 하나의 단위로 처리            
    - 자동으로 트랜잭션을 걸어서 테스트를 실행해주고 끝날 때는 rollback 처리를 해준다.
    - [관련 글](https://tecoble.techcourse.co.kr/post/2021-05-25-transactional/)            
    
<br><br>

## @SpringBootTest
- full application config을 로드해서 통합 테스트를 진행하기 위한 어노테이션이다.

- ApplicationContext에 모든 Bean들을 등록한다.
  - SpringBoot 어플리케이션을 실행했을 때와 동일하게 컨테이너에 Bean들을 등록한다.

- 설정해놓은 config, context, components를 모두 로드한다.

- DataSource bean을 그대로 사용하기 때문에 in-memory, 로컬, 외부 상관 없이 DB를 사용해서 테스트가 실행된다.

- 테스트할 때마다 DB가 롤백되지 않기 때문에 @Transactional을 추가로 설정해줘야 한다.

<br><br><br>

## @DataJpaTest
- @DataJpaTest 어노테이션은 JPA 관련 테스트 설정만 로드한다.
  - Component Scan을 하지 않아 컨테이너에 @Component 빈들이 등록되지 않는다.

- JPA를 사용하여 데이터를 제대로 생성, 수정, 삭제하는지 등의 테스트가 가능하다.

- 기본적으로 in-memory embedded DB를 사용한다. (`Replace.ANY;`)

- `@Transactional`이 포함되어있다.

<br><br><br>

### @AutoConfigureTestDatabase
`@DataJpaTest`에 있는 `@AutoConfigureTestDatabase`에 `Replace.ANY;`를 통해서       

실제 db를 사용하지 않고 테스트 db로 테스트 할 수 있다. (기본적으로 내장된 임베디드 데이터베이스를 사용한다.)  

![image](https://user-images.githubusercontent.com/74857364/201484613-a8891fde-c417-40f5-b2d9-af411f1962e0.png)

<br><br>

Any 속성 외에도 AUTO_CONFIGURED와 None이 더 있다.

![image](https://user-images.githubusercontent.com/74857364/201484635-3265edbd-4405-4556-9ea4-2e3e35676777.png)

<br>

- ANY
  - 자동 구성, 수동 구성된 DataSource 빈 모두 교체함
- AUTO_CONFIGURED
  - 자동 구성된 DataSource 빈만 교체함
- NONE
  - 어플리케이션에 기본 값으로 설정된 DataSource를 교체하지 않음


<br><br>

**`@DataJpaTest`에서 in-memory 데이터베이스로 사용할 수 있는 db 유형**

![image](https://user-images.githubusercontent.com/74857364/201484324-0b8793b4-e3cc-41ca-9f53-bfd4e02b0baf.png)

EmbeddedConnection으로 제공하고 있는것은 H2, DERBY, HSQL을 제공하고 있다.

<br><br>

**실제 db를 사용하려면?**              

`@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)`를 작성한다.

<br><br><br>

## 정리
기존 프로젝트에서 @SpringBootTest를 사용한 test 코드를 작성했을 때

db를 h2로 사용했을 때는 문제가 없었으나 mysql로 변경하면서 

<br>

@SpringBootTest 어노테이션이 존재하는 클래스는 

테스트를 실행하면 @SpringBootApplication 어노테이션을 검색하고

@SpringBootApplication에서 설정된 값을 읽어와서, 테스트에서 사용하기 때문에 mysql로 연결이 되었다.        

<br>

테스트 코드에서는 기존 db대신 test용 임시 데이터로 사용해야 해서 

test용 `application.properties`에서 h2 설정을 하고 여러 어노테이션을 붙였었는데

Jpa 관련 테스트를 할 때는 @DataJpaTest를 사용하는게 더 좋고 @SpringBootTest 보다 실행 속도가 더 빠르다.

<br><br><br>

출처          
[Spring DB : DB 테스트](https://ojt90902.tistory.com/924)                         
[[Spring] @SpringBootTest vs @DataJpaTest](https://da-nyee.github.io/posts/spring-springboottest-vs-datajpatest/)                  
[[JPA] DataJpaTest: 테스트시에 실제 DB 사용하기](https://kangwoojin.github.io/programing/auto-configure-test-database/)          
[💥[@DataJpaTest] Error creating bean with name 'dataSource' 에러](https://velog.io/@chlwogur2/DataJpaTest-Error-creating-bean-with-name-dataSource-%EC%97%90%EB%9F%AC)                          
[스프링 부트 테스트 : @DataJpaTest](https://webcoding-start.tistory.com/20)                
[[Spring Boot] @DataJpaTest vs @SpringBootTest 비교](https://velog.io/@jwkim/spring-boot-datajpatest-springboottest)               


