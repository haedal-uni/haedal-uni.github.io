---
categories: spring
tags: [spring]
---
         
## fn:escapeXml()이란           
XML마크업 문자로 인식될 문자열을 삭제한다.                    
[[JSTL] Functions Tag - fn:escapeXml() 사용하기](https://blog.naver.com/object0108/221199445521)

마크업이란 : ex) `<script>` `<div>` 

<br>

블로그를 보면 fn:escapeXml()이란 글들은 많지만 왜 문자열을 삭제하는 건지는 나와있지 않고 어떻게 사용하는지만 나와있다.

나는 fn:escapeXml()과 동일하게 쓰이는 `<c:out>`로 검색해서 그 이유를 찾았다.

### 왜 삭제할까?
- html이나 스크립트가 실행되어 위험하다. ⭐
- 엄격한 태그 규칙을 위해 사용한다.                
- 개행문자 파싱의 차이 때문에 사용한다.          
- 보안성 때문에 사용한다.            

[[JSP]<c:out>을 사용하는 이유](https://2ham-s.tistory.com/274)

<br>

### fn:escapeXml() 와 `<c:out>`                    
원래는 fn:escapeXml() 이나 `<c:out>` 둘 중에 아무거나 사용해도 된다.                    

role을 정해서 만들다 보면 어디서는 fn:escapeXml() 어디서는 `<c:out>` 를 사용하는 걸 볼 수 있다.     

다만 현재 사용하고 있는 프로젝트에서 어떤 방식으로 정하냐에 따라 fn:escapeXml()와 <c:out>를 같이 사용할 수 있다.          