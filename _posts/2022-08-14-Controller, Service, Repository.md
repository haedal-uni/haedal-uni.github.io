---
categories: Spring
tags: [spring]
---

## Controller
해당 요청 url에 따라 적절한 view와 mapping 처리                           

<br>

### @Controller
API와 view를 동시에 사용하는 경우에 사용                  
대신 API 서비스로 사용하는 경우는 `@ResponseBody`를 사용하여 객체를 반환한다.                  
view(화면) return이 주목적                  

<br>

### @RestController                  
view가 필요없는 API만 지원하는 서비스에서 사용 (Spring 4.0.1부터 제공)                  
`@RequestMapping` 메서드가 기본적으로 `@ResponseBody` 의미를 가정한다.     

data(json, xml 등) return이 주목적: return ResponseEntity                  
즉, `@RestController` = `@Controller` + `@ResponseBody`                  

<br>

---

## Service
비즈니스 로직          

<br>

### Service interface와 ServiceImpl로 나누는 이유

#### 1. AOP

`Spring AOP`는 빈 등록 시 사용자의 특정 메서드 호출 시점에 AOP를 수행하는 `Proxy Bean`을 생성해주며   

크게 2가지 프록시 객체 생성 방법이 존재한다.

<br>

`JDK Dinamic Proxy`는 프록시 객체 생성 시 인터페이스 존재가 필수적이고 

`CGLib`은 프록시 객체 생성 시 인터페이스가 존재하지 않아도 클래스 기반으로 프록시 객체를 생성할 수 있다.             

<br>

옛날 spring framework에서는 spring AOP 사용 시 `CGLib`의 여러 문제점으로 인해 `JDK Dinamic Proxy`을 사용하는 것을 권장했다.

**CGLib 문제점**
- `net.sf.cglib.proxy.Enhancer` 의존성 추가
- `default` 생성자가 무조건 필요함
- 타깃의 생성자 두 번 호출

<br>

`JDK Dinamic Proxy`는 인터페이스를 기반으로 프록시 객체를 생성한다.              

반드시 인터페이스가 존재해야 프록시 객체를 정상적으로 만들어 낼 수 있었다.            

하지만 요즘 spring boot는 인터페이스를 구현한 클래스임에도 프록시 객체 생성 시 `CGLib`를 사용한다.                

<br>

spring 3.2 이후부터는 `CGLib`의 위에서 언급한 문제점이 외부 라이브러리의 도움을 받아 개선이 이루어졌다고 판단되어       

spring core 패키지에 포함되었고 spring boot 사용 시 인터페이스를 구현한 클래스여도             

`JDK Dynamic Proxy`보다 성능이 좋은 `CGLib`를 디폴트(`spring.aop.proxy-target-class=true`)로 사용하여 프록시 객체를 생성한다.                 

만약 직접 `JDK Dynamic Proxy`와 `CGLib`를 활용한 프록시 객체 생성을 눈으로 확인해보고 싶다면 아래와 같이 입력한다.
```java
// application.properties
spring.aop.proxy-target-class=false // CGLib 사용하지 않음

spring.aop.proxy-target-class=true // CGLib 사용 - 디폴트)
````
<br><br><br>

#### 2. OOP
객체 간의 결합도를 낮추어 변화에 유연한 개발을 하기 위해서이다. 

하나의 인터페이스를 구현하는 여러 구현체가 있고 기능에 따라 적절한 구현체가 들어가서 다형성을 주기 위함이다. 

또 하나의 인터페이스만 바라보니 의존관계도 줄일 수 있다.

<br><br>

**OCP(개방 폐쇄 원칙)**

기존의 코드는 잘 변경하지 않으면서도 확장은 쉽게 할 수 있어야 한다.

<br>

**DIP(의존관계 역전의 원칙)** 

자신보다 변하기 쉬운 것에 의존 하던 것을 추상화된 인터페이스나 상위 클래스를 두어 변하기 쉬운 것의 변화에 영향받지 않게 하는 원칙

→ 구현 클래스에 의존하지 말고, 인터페이스에 의존하라는 뜻

<br><br>

**어떨 때 나눠서 사용할까?**

구현체를 2개 이상 갖게 될 때 인터페이스를 두는 것이 바람직하다.

일반적으로 카드는 결제가 있으면 반드시 취소 기능도 있어야 한다. 

인터페이스가 ‘카드 결제’라는 하나의 책임을 가질지라도 카드 결제와 관련된 다양한 메소드가 존재할 수 있음을 확인할 수 있다. 

<br><br>

### 트랜잭션

트랜잭션은 데이터베이스의 상태를 변환시키는 하나의 논리적 기능을 수행하기 위한 작업의 단위 또는            
한꺼번에 수행되어야할 일련의 연산들을 의미한다.

<br>

트랜잭션은 작업의 완전성을 보장해준다. 즉, 논리적인 작업 셋을 모두 완벽하게 처리하거나 또는 처리하지 못할 경우에는        

원 상태로 복구해서 작업의 일부만 적용되는 현상이 발생하지 않게 만들어주는 기능이다.             

사용자의 입장에서는 작업의 논리적 단위로 이해를 할 수 있고 시스템의 입장에서는 데이터들을 접근 또는 변경하는 프로그램의 단위가 된다.            

트랜잭션은 SELECT, UPDATE, INSERT, DELETE와 같은 연산을 수행하여 데이터베이스의 상태를 변화시키는 작업의 단위다.

![image](https://user-images.githubusercontent.com/74857364/203323080-e5d4acc3-6422-4fbe-b153-87c53a155835.png){: width="70%"}

<br><br>

**스프링에서 트랜잭션 처리 방법**

스프링에서는 트랜잭션 처리를 지원하는데                      
그 중 어노테이션 방식으로 `@Transactional`을 선언하여 사용하는 방법이 일반적이며, 선언적 트랜잭션이라 부른다.

클래스, 메서드위에 `@Transactional` 이 추가되면, 이 클래스에 트랜잭션 기능이 적용된 프록시 객체가 생성된다.

이 프록시 객체는 `@Transactional`이 포함된 메소드가 호출 될 경우, PlatformTransactionManager를 사용하여     
트랜잭션을 시작하고, 정상 여부에 따라 Commit 또는 Rollback 한다.
      
<br>

트랜잭션을 중구난방으로 적용하는 것은 좋지 않다. 

대신 특정 계층의 경계를 트랜잭션 경계와 일치시키는 것이 좋은데,                 
일반적으로 비지니스 로직을 담고 있는 **서비스** 계층의 메소드와 결합시키는 것이 좋다.          

왜냐하면 데이터 저장 계층으로부터 읽어온 데이터를 사용하고 변경하는 등의 작업을 하는 곳이 서비스 계층이기 때문이다. 
      
<br>

트랜잭션 사용에 대해 잘 정리 된 글이 있어 참고한다.               
[[Java]@Transactional Annotation 알고 쓰자](https://velog.io/@kdhyo/JavaTransactional-Annotation-%EC%95%8C%EA%B3%A0-%EC%93%B0%EC%9E%90-26her30h)              

<br>

---

## Repository      
Repository는 Entity에 의해 생성된 데이터베이스 테이블에 접근하는 메서드들(ex. findAll, save 등)을 사용하기 위한 interface다. 

데이터 처리를 위해서는 테이블에 어떤 값을 넣거나 값을 조회하는 등의 CRUD(Create, Read, Update, Delete)가 필요하다.

이 때 이러한 CRUD를 어떻게 처리할지 정의하는 계층이 바로 Repository다.

* DB에 접근하는 코드                                                   

<br>
  
### Spring Data JPA
Entity만으로는 데이터베이스에 데이터를 저장하거나 조회 할 수 없다. 

데이터 처리를 위해서는 실제 데이터베이스와 연동하는 JPA Repository가 필요하다.

<br>

#### JPA (Java Persistence API)                        
JPA : 애플리케이션을 객체지향 언어로 개발하고 관계형 DB 로 관리한 다음         
객체-관계형간의 차이를 해결하기 위해 JPA를 사용                          
JPA는 자바 진영에서 ORM 기술 표준으로 Application 과 JDBC 사이에서 동작한다.                         
                                                
![image](https://user-images.githubusercontent.com/74857364/184530243-fb94480f-e7bd-4d8e-b5e7-897e9d302451.png)                           

- ORM (Object Relational Mapper) : 직접 SQL을 작성하지 않고도 객체지향 방식으로 DB에 접근                        
- JDBC (Java Database Connectivity) : Java에서 DB에 접속할 수 있도록 하는 자바 API                        

<br>

#### Hibernate         
Hibernate는 JPA의 대표적인 프레임워크이다.                                                 
JPA는 Interface의 모음이며 이를 구현한 구현체로                        
**Hibernate**, **EclipseLink**, **DataNucleus** 등 여러 ORM framework가 존재하는데 주로 **Hibernate를** 사용한다.                                                

즉, JPA 핵심인 **EntityManagerFactory**, **EntityManager**, **EntityTransaction** Interface를                         
Hibernate에서 Interface로 상속받아 Impl로 구현했다.     

<br>

#### Spring Data JPA                        
Spring Data JPA 란 JPA를 추상화시킨 Repository Interface를 제공하여                            
개발자가 JPA를 더 편하게 사용할 수 있게 하는 모듈이다.    

![image](https://user-images.githubusercontent.com/74857364/184530292-544f47bd-fb38-4e27-a40e-8f9475ca36c3.png)                        
                        
Spring Data JPA를 사용하지 않는다면 클래스에 `@Repository` annotation 을 작성하고       
JPA를 적용한 다음 EntityManager의 API 를 직접 호출해야 entity CRUD 가 처리된다.                                       
                            
JpaRepository에는 EntityManager가 포함되어 있기 때문에 직접 작성하지 않아도 내부에서 자동으로 호출된다.                                    
또한, `@Repository` annotation 작성하지 않아도 spring data JPA가 알아서 Bean으로 등록해준다.                  

<br>

**작성 방법**

```java
public interface RegistryRepository extends JpaRepository<Registry, Long> {
}
```
RegistryRepository는 repository로 만들기 위해서 JpaRepository를 상속했다.

JpaRepository를 상속할 때는 제네릭스 타입으로 `<Registry, Long>` 처럼 

Repository의 대상이 되는 Entity의 타입(Registry)과 해당 엔티티의 PK의 속성 타입(Long)을 지정해야 한다.

<br>                        

##### 정리

![image](https://user-images.githubusercontent.com/74857364/190387991-877abdbc-d2e3-4a6e-8268-0df7a589ae59.png){: width="50%"}   
    
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

## Dto, Repository, Entity
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

## DAO, MODEL
DAO(repository 대신 DAO 사용), model(JPA를 사용안해서 @Entity가 없다.)           
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
[리포지터리](https://wikidocs.net/160890)                          

[[Spring] Transactional 정리 및 예제](https://goddaehee.tistory.com/167)                     
[[개발자 블로그] Spring에서 Service ServiceImpl 사용해야하는지](https://jeonyoungho.github.io/posts/spring%EC%97%90%EC%84%9C-Service-ServiceImpl%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%B4%EC%95%BC%ED%95%98%EB%82%98/)                              
[서비스 구현 시 인터페이스를 구현하는 형태로 하는 이유 (spring AOP)](https://velog.io/@suhongkim98/%EC%84%9C%EB%B9%84%EC%8A%A4-%EA%B5%AC%ED%98%84-%EC%8B%9C-%EC%9D%B8%ED%84%B0%ED%8E%98%EC%9D%B4%EC%8A%A4%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0-spring-AOP)                    
[[DB] 트랜잭션(Transaction)이란? ACID란?](https://code-lab1.tistory.com/51)                                  
[[Spring] Spring에서 트랜잭션의 사용법 - (3/3)](https://mangkyu.tistory.com/170)         
