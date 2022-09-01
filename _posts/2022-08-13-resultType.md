---
categories: Spring
tags: [spring, DAO]
---
         
## resultType이 int? 객체?
관련 글 : [Dao와 mybatis](https://haedal-uni.github.io/spring/2022/07/27/DAO%EC%99%80-MyBatis.html)             
               
<br>

```sql
select count(USER.USERID)
select avg(USER.USERSTAR)
```
resultType에서 select count는 int, select avg는 객체로 반환이 되는걸 보고                       
왜 하나는 int로 설정하고 왜 다른 하나는 객체로 설정을 할까 의문이 들었다.                       
                       
여기서 답은 "db"에 있다.                                              
`select count(USER.USERID)` : db에서 userId는 pk이다. → pk는 not null이다.                       

따라서 userId는 notNull이고 userStar은 null이 될 수 있어서 각각 0과 null로 뜬다.                        
USER.USERSTAR 컬럼에 데이터가 없으면 null로 나오기 때문에 계산을 할 수가 없고 무조건 에러가 뜰 수 밖에 없다.                                 
count는 없으면 0으로 뜬다.                        

그래서 각각 int와 객체로 설정을 하게 된 것이다.                       

<br>

### resultType = "int"            
카운트를 하는 경우에 resultType이 int 형식이 된다.*                    
객체면 조회하려는 sql 구문한테 DAO 랑 맞게 alias 써서 매칭시킨다. (대문자)                   
                   
*보통 카운트 처럼 조회하는 게 하나이고 다른곳에서 많이 사용하지 않는 경우는             
객체로 안 받고 그냥 int로 받는 경우가 있다.
