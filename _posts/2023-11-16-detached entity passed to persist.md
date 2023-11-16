---
categories: Error
tags: [JPA, study, Java]
---

# detached entity passed to persist:

*detached entity passed to persist:~* 라는 에러가 떴다.

<br>

에러가 뜬 원인은 `@Entity`에서 `@Id`를

`@GeneratedValue(strategy = GenerationType.IDENTITY)`로 설정해두고

<br>

```java
Member member = new Member();
member.setId(10L);
member.setUsername("HELLO WORLD");
```
해당 객체의 id에 직접 값을 입력했기 때문에 뜨는 에러였다.   

<br><br>

```java
    @Id
    //@GeneratedValue(strategy = IDENTITY)
    private Long id;
```
위아 같이 `@GeneratedValue(strategy = GenerationType.IDENTITY)` 코드를 제거한 후 실행하면 에러가 뜨지 않는다.   

<br><br><br>   

*reference*        
[org.hibernate.PersistentObjectException: detached entity passed to persist 에러](https://velog.io/@dev-kmson/org.hibernate.PersistentObjectException-detached-entity-passed-to-persist-%EC%97%90%EB%9F%AC)
