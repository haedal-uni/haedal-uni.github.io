---
categories: spring
tags: [spring]
---
                       
# DAO와 MyBatis

**Mapper 파일의 저장경로 설정**          

Mapper란 MyBatis에서 SQL 문을 저장하는 존재를 말한다.            
Mapper를 저장할 폴더 mappers를 src/main/resources 에 추가해준다.       
[[Spring] 7.DAO 구현](https://kookyungmin.github.io/server/2018/08/13/spring_07/)

<br>
<br>

## DAO
**UserDAO**     
쿼리문에 던져주는 정보가 매개변수(`User user`) 부분이고 해당 정보를 토대로 쿼리에 다시 반환하는 값이 리턴 타입인 `UserA`부분이다.    
```java
UserA userInfo(User user)
```
<br>
<br>
          
## MyBatis 

### Mapper          
MyBatis의 namespace에 Mapper 인터페이스의 경로를 적어준다.           

**user.xml**
```sql
<mapper namespace="haedal.uni.UserDAO">
```


<br>
<br>

### 전해주는 값

SQL id에는 매핑하는 메소드의 이름을 지정한다. → DAO의 메소드 이름과 id값이 같아야 한다.         
```sql
<select id="userInfo" resultType="UserA">
```

<br>

`#` 기호가 코드 로직에서 담기는 부분이기 때문에 userId가 User에 있어야 한다.       

```sql
<select id="userInfo" resultType="UserA">
   select
      USERID 
      , USERNAME
   from TABLE_USER
   where
      USERID = #{userId}
</select>
```
```java
UserA userInfo(User user)
```

<br>

`#{ }`는 후에 삽입되어 대체될 값이다.          
ex. userId가 1로 정해지면 USERID = #{userId}는 USERID = 1이 된다.            
[[Spring] 7.DAO 구현](https://kookyungmin.github.io/server/2018/08/13/spring_07/)

<br><br>

### 반환 하는 값 
```java
UserA userInfo(User user)
```
UserA가 반환 값이다. sql에서는 resultType이 반환 값으로 적힌다.  

아래 코드를 보면 as(alias)가 추가되었다.                   
      
<br>

<details><summary>UserA에는 `private string username;` 과 `private User user;` 이 있고 User 에는 userId가 있다고 가정한다.</summary>
                    
  <br>
  
```java          
@Alias("UserA")
public class UserA {
    private String username;
    private User user;
}
```
                      
```java
@Alias("User")
public class User {
    private UserId userId;
} 
```  

<br>
  
  ---
  
</details>
                     
                      
                            
```sql
<select id="userInfo" resultType="UserA">
   select
      USERID as "USER.USERID"
      , USERNAME as "USERNAME"
   from TABLE_USER
   where
      USERID = #{userId}
</select>
```
resultType이 UserA이며, UserA에는 username이 있으므로 해당 변수만 적으면 되지만    

userId는 User에 있으므로 해당 경로에 맞게 적어준다.        

`변수 담기는 값` ***as*** `vo 변수이름(명칭)`
 
<br>
<br>

참고로 resultType에서 패키지 명(ex. haedal-uni.project.UserA)을 다 적어줄 필요가 없던 이유는       

***UserA*** 라는 model 객체에 `@Alias`를 추가했기 때문이다.               
