---
categories: Error
tags: [error]
---
         
Unsatisfied dependency expressed through constructor parameter                            
                                             
available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {}                             

<br>

블로그를 보니 어노테이션 문제라는 것을 알 수 있었다.                      

<br>

나는 어노테이션을 명시한게 아니라 `@Service`와 연결된 파일들을 적는 private final에 model을 넣어서 에러가 뜬 것이었다.             

→ `@Service`가 아닌 model을 넣었으니 `@Service`라고 안적혀있는데? 하고 에러가 뜬 것 같다.                          

<br><br>

---

REFERENCE      

- [[스프링 Error]org.springframework.beans.factory.NoSuchBeanDefinitionException 에러메시지](https://sas-study.tistory.com/385)            


