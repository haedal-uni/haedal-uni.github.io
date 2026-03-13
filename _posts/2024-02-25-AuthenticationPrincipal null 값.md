---
categories: Error  
tags: [Security, error]
---
       
# `@AuthenticationPrincipal`을 활용한 사용자 정보 주입
`@AuthenticationPrincipal`은 Spring Security에서 현재 인증된(principal) 사용자의 정보를 주입받을 때 사용하는 어노테이션이다.

이를 통해 컨트롤러나 서비스에서 현재 사용자의 정보에 쉽게 접근할 수 있다.  
         
<br><br><br>

## 문제
User가 UserDetails를 상속받은 엔티티에서 권한과 userName을 Override 했고

`getUsername()`을 통해 nickname을 반환하는 것으로 설정했다.  

```java
@Override
public Collection<? extends GrantedAuthority> getAuthorities() {
    List<GrantedAuthority> authorities = new ArrayList<>();
    authorities.add(new SimpleGrantedAuthority(this.role.name()));
    return authorities;
}

@JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
@Override
public String getUsername() {
    return this.nickname;
}
```

<br><br>

front에서 받아오는 nickname이 만약 사용자가 임의로 변경할 수 있다는 가정하에

back에서 로그인한 정보를 가지고 해당 nickname을 통해 채팅방을 만드는 코드다.

```java
@PostMapping("/chat")
public ChatRoomDto createRoom(@RequestBody String nickname, @AuthenticationPrincipal UserDetails userDetails) {
    if(nickname.equals(userDetails.getUsername())){
        return chatService.createRoom(nickname);
    }else{
        return chatService.createRoom(userDetails.getUsername());
    }
}
```
여기서 UserDetails 값이 null로 떠서 500 (Internal Server Error) 오류가 떴다.


<br><br><br><br>  

## 문제 해결하기     
`@AuthenticationPrincipal`는 

Spring Security의 **AuthenticationPrincipalArgumentResolver** 클래스를 통해 동작하며, 

SecurityContextHolder에 접근해서 값을 반환한다.  

<br><br>

```java
// AuthenticationPrincipalArgumentResolver.class   
public Object resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) {

    // SecurityContextHolder에서 Authentication 객체를 가져온다.  
    Authentication authentication = this.securityContextHolderStrategy.getContext().getAuthentication();

    if (authentication == null) {
        return null;
    } else {

        // 구현한 principal 객체가 각각 다르기 때문에 authentication.getPrincipal()를 사용하여 Object타입으로 받아준다.
        Object principal = authentication.getPrincipal();

        /* AuthenticationPrincipal 어노테이션에 사용한 parameter를 얻어내
        exprsssion 메서드를 통해 String으로 사용자가 인증에 사용하는 클래스의 이름을 알아낸다(ex. UserDetails)  */
        AuthenticationPrincipal annotation = (AuthenticationPrincipal)this.findMethodAnnotation(AuthenticationPrincipal.class, parameter);
        String expressionToParse = annotation.expression();

        if (StringUtils.hasLength(expressionToParse)) {


        /* 알아낸 클래스 이름과  스프링 컨테이너에서 Bean을 찾아주는 beanResolver, context에 설정한 값을 가지고
        인증시에 사용했던 클래스의 인스턴스를 만들어낸다.*/
            StandardEvaluationContext context = new StandardEvaluationContext();
            context.setRootObject(principal);
            context.setVariable("this", principal);
            context.setBeanResolver(this.beanResolver);
            Expression expression = this.parser.parseExpression(expressionToParse);
            principal = expression.getValue(context);
        }

        if (principal != null && !ClassUtils.isAssignable(parameter.getParameterType(), principal.getClass())) {
            if (annotation.errorOnInvalidType()) {
                throw new ClassCastException("" + principal + " is not assignable to " + parameter.getParameterType());
            } else {
                return null;
            }
        } else {
            return principal;
        }
    }
}
```

<br><br>

AuthenticationPrincipalArgumentResolver 클래스를 보면

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/0b514631-7a7c-4d2c-9a49-76a976befdb9)

**1.** `supportsParameter()`를 통해 `@AuthenticationPrincipal` 이 있는지 체크

**2.** `supportsParameter()`의 값이 true라면 `resolveArgument()`에서 파라미터에 값을 주입한다.

여기서 `resolveArgument()`의 반환값은 Authentication이다. 

Authentication 인터페이스는 Spring Security에서 인증 객체를 나타내는 인터페이스다. 

<br><br>

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/b804fb2d-4cde-40b9-bc18-8cd4ef5dfec5){: width="50%"}

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/a053f95f-1eee-4df6-a462-32ffe905d90c){: width="30%"}

Spring Security에서 AuthenticationFilter를 거쳐 사용자의 인증을 완료하면 

SecurityContextHolder에는 Authentication 인터페이스를 구현한 객체가 저장된다. 

대표적으로 사용되는 클래스는 UsernamePasswordAuthenticationToken다. 

<br>

따라서 사용자 정보를 가져오기 위해 `@AuthenticationPrincipal`을 사용할 때는 

UsernamePasswordAuthenticationToken을 사용하여 Authentication 객체를 설정해야 한다.   

<br><br>

UsernamePasswordAuthenticationToken은 Authentication 인터페이스를 구현한 클래스다. 

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/6ce3222d-3b07-4b7c-ad8a-ee53fb60e1e2){: width="50%"}      

<br>

따라서 SecurityContextHolder에 저장되는 것은 Authentication 인터페이스를 상속받은 객체다. 

<br><br>

### 문제 해결  
```java
UserDetails userDetails = userDetailService.loadUserByUsername(jwtProvider.getNickname(token));
UsernamePasswordAuthenticationToken auth =
        new UsernamePasswordAuthenticationToken(userDetails.getUsername(), null, userDetails.getAuthorities());
SecurityContextHolder.getContext().setAuthentication(auth);
```
UsernamePasswordAuthenticationToken의 첫 번째 매개변수는 principal을 나타내는데 

`@AuthenticationPrincipal`을 받는 객체가 UserDetails를 구현한 객체이므로 userDetails를 넣었어야 했다.

<br><br>

UsernamePasswordAuthenticationToken은 사용자의 인증 정보를 생성하고 설정하는 데 사용하고, 

SecurityContextHolder는 이를 저장하고 관리하는 역할을 한다.

<br><br><br><br>

## 결론
인증 객체를 저장하는 과정에서 username 정보만 포함시킨 문제를 수정했다.  

`@AuthenticationPrincipal`을 사용하여 Principal 정보를 가져올 때, 

UsernamePasswordAuthenticationToken의 Principal로 설정된 객체를 얻을 수 있다.

<br>

```java
UserDetails userDetails = userDetailService.loadUserByUsername(jwtProvider.getNickname(token));
UsernamePasswordAuthenticationToken auth =
        new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
SecurityContextHolder.getContext().setAuthentication(auth);
```
`@AuthenticationPrincipal` 을 사용하여 UserDetails 값을 가져올 때는 UserDetails 객체를 Principal로 설정해야 하므로

`userDetails.getUsername()` 대신 userDetails로 수정했다.

<br><br>

참고로 `SecurityContextHolder.getContext().getAuthentication()`의 값을 출력시켜도 알 수 있다.

수정 전)             
`UsernamePasswordAuthenticationToken [Principal=nickname, ~  // 이하 생략 ]`

수정 후)            
`UsernamePasswordAuthenticationToken [Principal=org.springframework.security.core.userdetails.User [Username=nickname, ~ // 이하 생략]`


<br><br><br><br>
         
REFERENCE           
- [@AuthenticationPrincipal 어노테이션](https://cantcoding.tistory.com/87)      
- [[프로젝트] @AuthenticationPrincipal에 null값 들어오는 문제 해결](https://devjem.tistory.com/70)   
