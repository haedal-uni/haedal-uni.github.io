---
categories: Spring
tags: [spring]
---

### Controller
해당 요청 url에 따라 적절한 view와 mapping 처리                  
`@Autowired` Service를 통해 service의 method를 이용                  

<br>

#### @Controller
API와 view를 동시에 사용하는 경우에 사용                  
대신 API 서비스로 사용하는 경우는 `@ResponseBody`를 사용하여 객체를 반환한다.                  
view(화면) return이 주목적                  

<br>

#### @RestController                  
view가 필요없는 API만 지원하는 서비스에서 사용 (Spring 4.0.1부터 제공)                  
`@RequestMapping` 메서드가 기본적으로 `@ResponseBody` 의미를 가정한다.     

data(json, xml 등) return이 주목적: return ResponseEntity                  
즉, `@RestController` = `@Controller` + `@ResponseBody`                  

<br>

---

### Service
비즈니스 로직          

`@Autowired` Repository를 통해 repository의 method를 이용            

<br>

---

### Repository           
DB에 접근하는 코드                                                   

<br>
  
#### Spring Data JPA                        
##### JPA (Java Persistence API)                        
JPA : 애플리케이션을 객체지향 언어로 개발하고 관계형 DB 로 관리한 다음         
객체-관계형간의 차이를 해결하기 위해 JPA를 사용                          
JPA는 자바 진영에서 ORM 기술 표준으로 Application 과 JDBC 사이에서 동작한다.                         
                                                
![image](https://user-images.githubusercontent.com/74857364/184530243-fb94480f-e7bd-4d8e-b5e7-897e9d302451.png)                        

- ORM (Object Relational Mapper) : 직접 SQL을 작성하지 않고도 객체지향 방식으로 DB에 접근                        
- JDBC (Java Database Connectivity) : Java에서 DB에 접속할 수 있도록 하는 자바 API                        

<br>

##### Hibernate         
Hibernate는 JPA의 대표적인 프레임워크이다.                                                 
JPA는 Interface의 모음이며 이를 구현한 구현체로                        
**Hibernate**, **EclipseLink**, **DataNucleus** 등 여러 ORM framework가 존재하는데               
주로 **Hibernate를** 사용한다.                                                

즉, JPA 핵심인 **EntityManagerFactory**, **EntityManager**, **EntityTransaction** Interface를                         
Hibernate에서 Interface로 상속받아 Impl로 구현했다.     

<br>

##### Spring Data JPA                        
Spring Data JPA 란 JPA를 추상화시킨 Repository Interface를 제공하여                            
개발자가 JPA를 더 편하게 사용할 수 있게 하는 모듈이다.    

![image](https://user-images.githubusercontent.com/74857364/184530292-544f47bd-fb38-4e27-a40e-8f9475ca36c3.png)                        
                        
Spring Data JPA를 사용하지 않는다면 클래스에 `@Repository` annotation 을 작성하고       
JPA를 적용한 다음 EntityManager의 API 를 직접 호출해야 entity CRUD 가 처리된다.                                       

Spring Data JPA 의 **Repository Interface (SimpleJpaRepository)** 에는                         
EntityManager가 포함되어 있기 때문에 직접 작성하지 않아도 내부에서 자동으로 호출된다.                                      

또한, `@Repository` annotation 작성하지 않아도 spring data JPA가 알아서 Bean으로 등록해준다.                                            

<br>                        

###### 정리

<img src = https://user-images.githubusercontent.com/74857364/190387502-016baab9-180b-4ab8-aa5f-f80e0846f080.png width="40%">      
                         
                    
- JPA : ORM 기술 표준으로 채택된 인터페이스의 모음 ( = 기술 명세서 )

- Spring Data JPA 란 JPA를 추상화시킨 Repository Interface를 제공하여                  
  개발자가 JPA를 더 편하게 사용할 수 있게 하는 모듈   
→ JPA를 추상화 시킨게 Repository Interface

- ORM을 구현하기 위한 인터페이스(JPA) != JPA를 추상화 시킨 Repository Interface(Spring Data JPA)

- JPA 구현체가 Hibernate(ORM framework)

- JPA(인터페이스)를 구현해둔 Hibernate(ORM)                  
Repository인터페이스를 Spring Data JPA가 JPA화하고(JPA로 번역해준다) 그 JPA를 Hibernate로 실행하는 것 같다.        

- JPA를 추상화한게 Spring Data JPA이고 Spring Data JPA가 Repository를 제공                  
 
<br>

*추상화 : 불필요한 것을 지우고 핵심을 남겨둔다. (추상화 라는 단어가 어렵다면? → 반대로 생각하면 구체화)                   
                  
*인터페이스 : 완전한 추상화를 제공한다.                  

<br>

---

![image](https://user-images.githubusercontent.com/74857364/184529203-eda462c3-ab64-42e2-9b9e-81b71d5b83ac.png)

<br>

### dto, Repository, Entity
***Dto*** : 로직을 갖고 있지 않는 순수한 데이터 객체이며, getter/setter 메서드만을 갖는다.  

*Vo : VO는 DTO와 동일한 개념이지만 read only 속성을 갖는다.                         
VO는 특정한 비즈니스 값을 담는 객체이고, DTO는 Layer간의 통신 용도로 오고가는 객체를 말한다.        

<br>

***Repository*** : 실제로 DB에 접근하는 객체                 
Entity에 의해 생성된 DB에 접근하는 메서드 ex. findAll() 들을 사용하기 위한 인터페이스                 
    
<br>
    
***Entity*** : 실제 DB의 테이블과 매칭될 클래스 (Entity는 DB 테이블과 1대1로 대응 되는 객체)        
model에 `@Entity`를 넣어줌          
*`@Entity`가 붙은 클래스는 JPA가 관리하는 클래스       

Client ←(dto) → Controller ←(dto) → Service ←(dto) → Repository ← Entity → DB

<br>

---

### DAO, MODEL
dao(repository 대신 dao사용), model(JPA를 사용안해서 @Entity가 없다.)           
*model에 `@entity`를 넣어주면 entity라고 부르고 안붙으면 model이라 부르는 것 같다.          

<br>
          
***DAO*** : 실제 DB에 접근하는 객체, Service와 DB를 연결하는 고리의 역할          
`@Repository` : DAO를 인식시켜주기 위한 설정          
해당 클래스가 DAO라는 것을 알리기 위해서 `@Repository`라고 적어준다.          

**Client** ↔ **Controller** ↔ **Service** ↔ **DAO** ↔ **DB**          

Service에서 DAO를 호출하여 DB에 접근한다.          

mapper안에 정의된 xml 파일에는 요청한 정보를 처리하기 위한 SQL들이 있다.          
```sql
<mapper namespace="project.dao.PracticeDAO">
    <select id = "DAO 메소드 이름" resultType = "">
    </select>
```

<br>   
<br>                              

출처                           
[스프링 부트 : 기본 개념 1) Entity, Repository 개념](https://whitepro.tistory.com/265)                    
[[JPA] 엔티티 매핑 방법 (Entity Mapping)](https://gmlwjd9405.github.io/2019/08/11/entity-mapping.html)                   
[Entitiy 와 DTO 의 분리](https://velog.io/@boo105/Entitiy-%EC%99%80-DTO-%EC%9D%98-%EB%B6%84%EB%A6%AC)                  
[[DAO] DAO, DTO, Entity Class의 차이](https://gmlwjd9405.github.io/2018/12/25/difference-dao-dto-entity.html)                  
[[Spring - 어노테이션(Annotation) ] @Repository → DAO 인식, @Service → Service 인식](https://whitekeyboard.tistory.com/178)                  
[Spring MVC 디렉토리 구조 및 실행순서 (controller, service, dao, view )](https://it-sunny-333.tistory.com/129)                  
[스프링 부트 : 기본 개념 1) Entity, Repository 개념](https://whitepro.tistory.com/265)                    
[JPA와 Spring Data JPA의 차이](https://velog.io/@evelyn82ny/JPA-vs-Spring-Data-JPA)                        
[Java에서 추상화 란 무엇인가 – 예제로 배우기](https://ko.myservername.com/what-is-abstraction-java-learn-with-examples)                      
[[OOP] 추상화(Abstraciton)란?](https://steady-coding.tistory.com/453)
