# Spring Security: 인증과 권한 관리
back과 front를 분리하면서 해당 권한에 해당하는 api 주소를 back에 맞게 변경했다.

그리고 해당 주소 중 관리자만 사용할 수 있게 권한을 Security에 설정했다.  

<br>

문득 이 권한은 로그인한 사용자 정보를 얻기 위해 

`SecurityContextHolder.getContext().setAuthentication(authentication);`로 값을 저장하면 

`@AuthenticationPrincipal UserDetails userDetails`를 작성했을 때 값을 가져오는 것처럼

SecurityContextHolder에 저장된 사용자 정보를 사용하는걸까 생각이 들었다.  

<br><br>

## Spring Security

Spring Security는 Java 기반의 애플리케이션에서 보안과 인증을 처리하기 위한 프레임워크로 

REST API 및 서비스를 보호하기 위한 다양한 보안 기능을 제공하며 

주요 기능으로 인증(Authentication)과 인가(Authorization)가 있다. 

<br><br>

* 인증(Authentication) : 해당 사용자가 본인이 맞는지 확인하는 과정

  인증은 사용자의 신원을 확인하는 과정으로, 사용자가 제공한 자격 증명(예: id와 pw)을 검증하여 사용자를 식별한다.

<br><br>

* 인가(Authorization) : 해당 사용자가 요청하는 자원을 실행할 수 있는 권한이 있는가를 확인하는 과정

  인가는 인증된 사용자에게 특정 리소스 또는 기능에 대한 액세스 권한을 부여하는 과정이다.

<br>

Spring Security는 기본적으로 인증 절차를 거친 후에 인가 절차를 진행하며, 

인가 과정에서 해당 리소스에 접근 권한이 있는지 확인하게 된다

<br>

Spring Security는 세분화된 권한 관리를 지원하여, 역할 기반의 접근 제어를 구현할 수 있다. 

이를 통해 애플리케이션의 리소스에 대한 접근 제한을 설정할 수 있다.

<br><br><br>  

Spring Security는 '인증'과 '권한'에 대한 부분을 Filter 흐름에 따라 처리하고 있다. 

**Filter**는 Dispatcher Servlet으로 가기 전에 적용되므로 가장 먼저 URL 요청을 받지만, (웹 컨테이너에서 관리)

**Interceptor**는 Dispatcher와 Controller 사이에 위치한다는 점에서 적용 시기의 차이가 있다. (스프링 컨테이너에서 관리)

<br>
 
>Client (request) → Filter → DispatcherServlet → Interceptor → Controller

(실제로 Interceptor가 Controller로 요청을 위임하는 것은 아님, Interceptor를 거쳐서 가는 것)


<br><br><br><br>  

### Spring Security 설정
Spring Security 설정은 주로 `configure()` 메소드를 사용하여 이루어진다.     

여기서 URL 경로에 대한 인가 규칙을 설정하고, 특정 URL 경로를 허용하거나 인증을 요구하는 설정을 할 수 있다.


설정|   설명
|:--:|:--:| 
`http.authorizeRequests()` | URL 경로에 대한 인가 규칙을 설정한다.
`.antMatchers().permitAll()`   | 특정 URL 경로를 인증 없이 허용한다.
`.anyRequest().authenticated()`   | 모든 요청에 대해 인증을 요구한다.
`.hasRole()`   | 특정 역할을 가진 사용자만 접근을 허용한다.
`.formLogin()`   | 폼 기반 로그인을 활성화한다.
`.loginPage()`   | 로그인 페이지의 경로를 지정한다.
`.defaultSuccessUrl()`   | 로그인 성공 후 이동할 기본 URL을 설정한다.
`.logout()`   | 로그아웃을 처리하는 설정을 추가한다.   
`.logoutUrl()`   | 로그아웃 URL을 지정한다.
`.logoutSuccessUrl()`   | 로그아웃 성공 후 이동할 URL을 설정한다.
`.csrf()`   | CSRF(Cross-Site Request Forgery) 공격 방어 설정을 활성화한다.
`.sessionManagement()`   | 세션 관리를 설정한다.
`.sessionCreationPolicy()`   | 세션 생성 정책을 설정한다.
                 
<br><br><br><br>

### ROLE
```java
    http.authorizeRequests()
            .antMatchers("/hello").hasAuthority(UserRole.ADMIN.name())
```
hasRole을 사용하게 된다면 UserRole 클래스에 USER 대신 ROLE_USER로 저장을 하고

`.antMatchers("/hello").hasRole(hasRole("USER"))` 로 작성해야한다. 

<br><br> 

hasRole을 찾아보면 prefix가 붙어있다.  

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/de2d31a9-fd83-4fe2-b5e0-3607a7f36154)

<br>

별도로 설정을 하지 않는이상 prefix는 "ROLE_" 이라는 것을 알 수 있다. 

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/b03946a0-d598-434e-8ea5-7804279a3524)

<br>

`hasAuthority("ROLE_USER")` == `hasRole("USER")`

<br><br>

참고로 더 디테일하고 좁은 범위의 url이 위에 오고 넓은 범위의 url이 아래(하단)에 위치해야한다.

```java
    http.authorizeRequests()
            .antMatchers("/hello/morning").hasAuthority(UserRole.ADMIN.name())
            .antMatchers("/hello/**").hasAuthority(UserRole.ADMIN.name())
```

<br><br><br><br>

### UserDetailsService(사용자 정보 조회)  

Spring Security는 사용자 정보를 조회하기 위해 UserDetailsService를 제공한다. 

이 Interface는 사용자의 username을 기반으로 사용자 정보를 조회하고, UserDetails 객체를 반환한다.

```java
public class CustomUserService implements UserDetailsService {
   @Override
    public UserDetails loadUserByUsername(String nickname) throws UsernameNotFoundException {
        return userRepository.findByNickname(nickname)
                .map(this::buildUserDetails)
                .orElseThrow(() -> new UsernameNotFoundException("User not found with username: " + nickname));
    }
}
```
nickname과 pw로 로그인하기 때문에 파라미터에 nickname을 작성했다.

<br><br>

#### `loadUserByUsername()`에서 하는 일
- nickname을 가지고 사용자 정보를 조회하여 UserDetails 객체로 변환하여 반환  

- 사용자의 Role과 권한(Privilege)을 조회하여, SimpleGrantedAuthority 목록을 authorities에 세팅한다.
  
- Authentication 내부 principal 객체에 UserDetails 객체가 저장된다.

- Authentication 내부 authorities 객체에 사용자의 Role과 권한(Privilege) 정보가 저장된다.

- UserDetails에 authorities가 세팅되어 있어야, API별 role이나 권한 체크를 진행할 수 있다.


<br><br><br><br>

### GrantedAuthority(권한 관리)  
- USER : 사용자 정보를 포함하는 entity 객체로 UserDetails의 구현체

- UserDetails : 인증된 핵심 사용자 정보 (권한, 비밀번호, 사용자명, 각종 상태)를 제공하기 위한 interface이다.

<br>

Spring Security는 사용자의 권한을 GrantedAuthority를 통해 관리한다. 

이는 현재 사용자(Principal)가 가지고 있는 권한을 의미하며, 사용자의 Role과 권한(Privilege) 정보를 제공한다.

```java
public class User implements UserDetails { 
    Collection<? extends GrantedAuthority> getAuthorities()
}
```
Authentication 클래스에 `getAuthorities()` 메소드를 통하여, 인증받은 사용자의 authorities를 조회할 수 있다.

<br>
 
도메인 별로 구체적인 권한 체크가 필요한 경우에는 GrantedAuthority로 관리하지 않고, 

각 API 별로 비지니스 권한을 체크한다.

<br><br><br><br>

### SecurityContextHolder
SecurityContextHolder는 SecurityContext를 보관하는 저장소다. 

이를 통해 현재 사용자의 인증 정보를 관리하고, Authentication 인스턴스를 저장한다.

*Authentication에는 principal, credentials, authorities가 저장된다.

<br><br>

따라서 `getAuthorities()` 메소드를 통하여 가져오는 권한은 

SecurityContextHolder에 저장된 사용자 정보를 활용하여 가져오는 것과 동일한 공간에서 가져온다고 볼 수 있다.


<br><br><br><br>  

**REFERENCE**
- [[Spring Boot] Spring Security 권한 설정 및 사용 방법](https://cocococo.tistory.com/entry/Spring-Boot-Spring-Security-%EA%B6%8C%ED%95%9C-%EC%84%A4%EC%A0%95-%EB%B0%8F-%EC%82%AC%EC%9A%A9-%EB%B0%A9%EB%B2%95)  
- [Spring Security - 2. Role과 권한(Privilege)](https://gregor77.github.io/2021/04/21/spring-security-02/)
- [Spring Security의 구조(Architecture) 및 처리 과정 알아보기](https://dev-coco.tistory.com/174)  
