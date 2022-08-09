---
categories: Error
tags: [error]
---
         
Unsatisfied dependency expressed through constructor parameter

available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {}


[1번](https://sg-moomin.tistory.com/entry/available-expected-at-least-1-bean-which-qualifies-as-autowire-candidate-Dependency-annotations-%EC%98%A4%EB%A5%98-%ED%95%B4%EA%B2%B0%ED%95%B4%EB%B3%B4%EA%B8%B0)          
[[스프링 Error]org.springframework.beans.factory.NoSuchBeanDefinitionException 에러메시지](https://sas-study.tistory.com/385)           

위 블로그를 보면 알겠지만 어노테이션 문제라는 것을 알 수 있다. 
ex) @Service를 명시안했다던지,,,

나는 어노테이션을 명시한게 아니라 @Service와 연결된 파일들을 적는 private final에 model을 넣어서 에러가 뜬 것이었다.            
→ @Service가 아닌 model을 넣었으니 @Service라고 안적혀있는데? 하고 에러가 뜬 것 같다.                         






