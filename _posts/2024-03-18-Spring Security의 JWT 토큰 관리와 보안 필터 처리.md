---
categories: Project
tags: [spring, Security]
---  

## Spring Security의 JWT 토큰 관리와 보안 필터 처리
log에 `[doFilterInternal] 유효한 JWT 토큰이 없습니다, uri: ` 라는 메시지가 표시되어 

해당 인증이 필요없는 CSS 및 JavaScript 파일에도 이 메시지가 표시되는 문제가 생겼다. 

<br>

```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
    }
}
```
<br>

back과 front를 분리할 경우, 각각의 서버는 독립적으로 동작하며 API 요청만을 처리하기 때문에 보안 필터를 적용할 필요가 없다. 

이로 인해 log에는 해당 메시지가 표시되지 않는다.

<br>

그러나 back과 front를 결합하여 하나의 서버로 실행할 경우, 보안 필터가 모든 요청에 대해 작동하게 된다. 

이로 인해 CSS 및 JavaScript 파일과 같이 인증이 필요 없는 자원에 대해서도 보안 필터가 적용되어 해당 메시지가 log에 표시된다.

<br>

해당 메시지가 나타나는 경우는 두 가지다.

- 현재 사용자의 인증 정보가 없을 때(`SecurityContextHolder.getContext().getAuthentication()`) 

- client에서 전송한 token이 없을 때 (`HttpServletRequest.getHeader("Authorization")`)

<br><br>

문제는 해당 인증이 필요없는 css, js까지 해당 log가 뜬다는 점이었다.

```java
public static final String[] VIEW_LIST = {
    "/css/**",
    "/js/**"
};

@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeRequests()
            .requestMatchers(VIEW_LIST).permitAll()
}
```
인증 여부와 관계 없이 접근을 허용 하는 `permitAll()`을 사용했는데도 css와 js에 인증을 요구하고 있었다.

<br><br><br><br>

### 코드 살펴보기

Spring Security에서는 기본적으로 세션 기반의 인증 방식을 사용하지만, 

여기서는 JWT(Json Web Token)를 사용한 인증 방식을 사용했다. 

이를 위해 기본적으로 제공되는 UsernamePasswordAuthenticationFilter를 사용하지 않고, 

커스텀 필터인 JwtAuthenticationFilter를 사용했다.

```java
http.addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider), UsernamePasswordAuthenticationFilter.class);
```
<br><br><br>

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/24eeb110-a769-448c-be50-5c6d37c8bd73)

JwtAuthenticationFilter는 인증 과정에서 JWT token을 검증하며,  

인증에 실패할 경우 JwtAuthenticationEntryPoint가 호출되어 일관된 인증 예외 응답을 반환한다. 

이를 통해 클라이언트에게 인증 실패에 대한 적절한 응답을 제공할 수 있다.

```java
// SecurityConfig
http.exceptionHandling()
        .authenticationEntryPoint(customAuthenticationEntryPoint)


// CustomAuthenticationEntryPoint
@Override
public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException, ServletException {
    log.info("[ commence ] : " + "인증 실패");
}
```
JwtAuthenticationFilter에서 JWT 인증이 실패할 경우, 

JwtAuthenticationEntryPoint가 동작하여 클라이언트에게 401 에러를 반환하게 된다.

<br><br><br><br>



### 해결 방법  
`permitAll()` 을 적용하면 CSS 및 JavaScript 파일에 대한 접근을 허용했지만, 

Spring Security의 필터 체인이 동작한다.  

<br>

URL에 대한 모든 사용자의 요청을 허용하는 메소드로써 인증 처리 결과를 무시하는 것이지만

Spring Security 의 필터 체인은 동작한다는 얘기다.

<br>

→ `permitAll()` 을 적용해도, 구성된 Spring Security의 필터 체인을 거친다.

따라서 필터 체인을 거치지 않게 하기 위해 ignoring을 작성했다.

<br><br>

```java
@Bean
public WebSecurityCustomizer webSecurityCustomizer() {
       return web -> {
           web.ignoring()
               .antMatchers(
                   "/css/**",
                   "/js/**"
                   );
       };
 }
```
WebSecurity 는 HttpSecurity 상위에 존재하며 WebSecurity의 ignoring에 API를 등록하면, 

Spring Security의 필터 체인을 무시하고 해당 URL에 대한 접근을 허용한다. 

하지만 이경우, Cross-Site Scripting,XSS 공격 등에 취약해진다.

<br>    

HttpSecurity 에서 `permitAll()` 은 인증 처리 결과을 무시하는 것이지 

Spring Security의 필터 체인이 적용은 정상적으로 된다.

<br>  

그래서 WebSecurity는 보안과 전혀 상관없는 로그인 페이지, 공개 페이지(어플리캐이션 소개 페이지 등), 

application의 health 체크를 하기위한 API에 사용하고, 그 이외에는 HttpSecurity의 `permitAll()` 을 사용하는 것이 좋다.

<br><br><br><br>

REFERENCE
- [[Spring Security] - SecurityConfig 클래스의 permitAll() 이 적용되지 않았던 이유](https://velog.io/@choidongkuen/Spring-Security-SecurityConfig-%ED%81%B4%EB%9E%98%EC%8A%A4%EC%9D%98-permitAll-%EC%9D%B4-%EC%A0%81%EC%9A%A9%EB%90%98%EC%A7%80-%EC%95%8A%EC%95%98%EB%8D%98-%EC%9D%B4%EC%9C%A0) 
