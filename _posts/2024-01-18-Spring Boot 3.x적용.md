---
categories: Intellij
tags: [Intellij]
---

# 11 → 17 변경
Java 버전은 17인데 프로젝트의 JDK는 11로 되어있었고 spring boot 버전은 2.x였다.

그래서 spring boot는 3.x로 프로젝트 JDK는 17로 통일 시키기로 했다.

<br><br>

### build.gradle

```
plugins {
    id 'org.springframework.boot' version '3.0.1'
}
sourceCompatibility = '17'
```
sourceCompatibility를 17 이상으로 변경한다.  

<br><br><br>

### Preferences/Settings
Preferences/Settings → Build, Execution, Deployment → Build Tools → Gradle

Gradle의 JVM이 Java 17이상인지 확인한다.

<img width="1192" height="606" alt="Image" src="/assets/img/posts/bba9dbbe-b75d-45ef-8b49-22daefeb583d.png" />  

<br><br>

Preferences/Settings → Build, Execution, Deployment → Compiler → Java Compiler

<img width="907" height="377" alt="Image" src="/assets/img/posts/1c0f5794-e15a-4f4b-8a38-38ac4d482df6.png" /> 


<br><br><br>    

### Project SDK
File → Project Structure로 이동. 프로젝트 SDK의 버전을 확인후 Java 17이상으로 설정한다.  

#### Project
<img width="1102" height="271" alt="Image" src="/assets/img/posts/cf736d1c-389f-45cc-bf20-70bc367d9a4d.png" />  

<br>

### Modules
<img width="950" height="163" alt="Image" src="/assets/img/posts/a8e391e4-3cfe-4513-ae97-bea5436f4f22.png" /> 

<br><br><br><br>

---

## 참고(적용하면서 생겨난 오류들 해결하기)
버전을 업그레이드 하면서 기존 프로젝트에 나타난 문제들을 정리했다.

<br>  

### import javax.* 오류
javax.* 패키지 부분에서 빨간 줄이 떴다.

이클립스 재단으로 이관되면서 Jakarta EE 라고 하고 패키지는 jakarta.* 로 명칭이 변경했다고 한다.

JavaEE 이름들이 JakartaEE 이름으로 변경되었고 패키지 이름로 변경되었다.

- Java Servlet(javax.servlet) → Jakarta Servlet(jakarta.servlet)
- Java Message Servie (javax.jms) → Jakarta Messaging (jakart.jms)
- JPA:Java Persistence API (javax.persistence) → Jakarta Persistence(jakarta.persistence)
- JTA:Java Transaction API (javax.transaction) → Jakarta Transaction(jakarta.transaction)

<br><br><br>

### `@PostConstruct`
Java 9 이상부터는 찾을 수 없다고 뜬다.

`@PostConstruct`를 사용할 수 없기 때문에 build.gradle에 종속성을 추가해주면 된다.

`implementation 'javax.annotation:javax.annotation-api:1.3.2'`

<br>

다른 방법으로는 Spring bean interface 사용이 있다.

Spring framework 의 경우 InitializingBean, DisposableBean 을 구현하여 @PostConstruct @PreDestroy 대체가 가능하다고 한다.

이 부분은 아래 reference에 첨부된 링크를 참고해서 보면 될듯하다.     

<br><br><br>

### `antMatchers()`
`antMatchers()`를 사용할 수 없었고 antMatchers 대신 `requestMatchers()`를 사용했다.   

<br><br><br>

### application.properties
```
# do not share id
spring.jpa.hibernate.use-new-id-generator-mappings= false
```
위와 같이 사용했었는데 Spring Boot 3.x 부터 제거 되었다고 한다.

spring.jpa.hibernate.use-new-id-generator-mappings= false 이 부분은 설정파일에서 제거하면 된다. 

<br><br>

<img width="726" height="72" alt="Image" src="/assets/img/posts/825821ca-c55f-4f46-8864-b5b36bd32848.png" /> 

spring.redis.host는 제거되었다고 해서 아래와 같이 변경했다.   
```
spring.redis.host = local
spring.redis.port = 6379
↓ 
spring.data.redis.host = local
spring.data.redis.port = 6379
```

<br><br><br>

### SLF4J
APPLICATION을 실행하려 보니 

SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".            
SLF4J: Defaulting to no-operation (NOP) logger implementation          

라는 에러가 떴다.

<br>

build.gradle에서
```
implementation 'org.slf4j:slf4j-api:1.7.36'
```
를 작성했었는데 해당 부분을 제거하고 아래로 작성했다.   

```
implementation 'org.slf4j:slf4j-simple:1.7.36'
```

<br>

이 부분에 대해 검색하다보면 2개를 작성하라는 글이 많은데 이럴 경우 오류가 뜬다.

slf4j-log4j12 또는 slf4j-simple 중 하나가 있어야 하며 **둘 다 작성하면 안된다.**

나는 후자로 작성했다.    

<br><br><br><br>

**REFERENCE**

- java 17 적용
  - [Spring Boot 3.x 실행이 안될 경우 (feat. IntelliJ)](https://jojoldu.tistory.com/698)
- javax
  - [Java EE에서 Jakarta EE로의 전환](https://www.samsungsds.com/kr/insights/java_jakarta.html)
- @PostConstruct
  - [I can't use @PostConstruct and @PostDestroy with Java 11](https://stackoverflow.com/questions/52701459/i-cant-use-postconstruct-and-postdestroy-with-java-11)
  - [[Spring] Java @PostConstruct @PreDestroy @Resource 찾을 수 없음](https://devbible.tistory.com/461)
- antMatchers
  - [Cannot resolve method 'antMatchers()' in AuthorizationManagerRequestMatcherRegistry](https://stackoverflow.com/questions/74753700/cannot-resolve-method-antmatchers-in-authorizationmanagerrequestmatcherregis)
- application.properties
  - [use-new-id-generator-mappings](https://www.inflearn.com/questions/905887/use-new-id-generator-mappings)
- SLF4J
  - [SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder"](https://stackoverflow.com/questions/7421612/slf4j-failed-to-load-class-org-slf4j-impl-staticloggerbinder)    
