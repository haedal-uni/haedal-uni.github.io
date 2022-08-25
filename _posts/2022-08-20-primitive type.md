---
categories: Spring
tags: [spring]
---
         
## primitive type
*리팩토링을 하던 중 공부하게 된 내용                         

SQL에 `#` 태그에 appleId와 ball이 적혀있다.                        
```sql
APPLEID = #{appleId}
<if test='ball != null'>
	and BALL = #{ball}
</if>
```
코드를 작성하면서 ServiceImpl과 DAO를 연결시키기 위해서 SQL의 `#` 태그를 보고                        

ServiceImpl에 `#` 태그에 해당하는 값이 있는 Apple 객체를 파라미터로 넣어주자 appleId는 인식이 되었다.                        

해당 `#` 태그는 2개 존재해서 또 다른 Ball 객체를 추가로 파라미터에 넣어줘야 했는데                        

Apple 객체와 Ball 객체를 같이 넣어주니깐 SQL에서 인식을 하지 못했다.                        

```java
ReturnType method (Apple apple, Ball ball)
```

기존에 작성된 코드는 ***@Param***으로 어떤 값을 사용할지 명시를 해서 2개의 값 모두 인식이 되었다.                        

```java
@Param("appleId") String appleId
```

그래서 궁금증이 생겼다.                        


<br>

내가 여태까지 코드를 짜면서 늘 파라미터로 객체를 주고 있었는데                        

다른 코드들을 살펴보면 객체가 아니라 ***@Param***으로 명시를 해서 주고 있는 코드들이 몇 개 보였다.                        

그리고 파라미터 값이 하나여도 ***@Param***으로 명시를 해주는 코드도 있는데 왜 이런건지 의문이 들었다.                        

<br>

이 경우를 ***primitive type*** 을 사용한다고 표현한다.                        

파라미터는 객체를 사용하거나 primitive를 사용하건 상관없으나 가급적 **객체**를 사용하는 것을 권장한다.                        

객체건 primitive이건 파라메터가 2개이상인 경우에는 ***@Param***을 통해 parameter를 구분해서 사용한다.                        

<br><br>

### @param 사용 이유                        
`public void gitBlog(@Param("haedal") String githubName);`
                        
이렇게 @Param 어노테이션을 붙이면 본인이 원하는 명으로 mapper에서 사용할 수 있다.
                        
위와 같은 경우는 `#{haedal}` 이다.                        
<br>

파라미터가 두 개 일 경우 MyBatis에서 인식을 못하기 때문에 ***@param***을 사용해야한다.             
           
어노테이션을 쓰지 않아도 mapper에서 `#{paramNum}` 이라던지, `#{githubName}` 으로 파라미터 명을 적으면 사용이 가능하다.                        
```sql
String gitBlog(String githubId);                        
where
   USERID = #{githubId}
```
                        

[[Spring] @param 사용이유](https://popo015.tistory.com/99)                              