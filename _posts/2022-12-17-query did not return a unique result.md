---
categories: Error
tags: [error]
---

# query did not return a unique result:

  
해당 오류는 Repository에서 Return 값을 Class로 받아 담을 수가 없어서 에러가 발생한 것이다.

Repository의 Return 타입을 Class에서 `List<Class>` 로 받아주면 해결된다.
  
```java
// Commnet findAllByRegistry_Idx(Long idx);
List<Comment> findAllByRegistry_Idx(Long idx);

```

<br><br>

*reference*                
[JPA query did not return a unique result 에러 해결방법](https://wakestand.tistory.com/943)
