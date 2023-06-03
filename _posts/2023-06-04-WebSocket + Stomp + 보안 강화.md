# WebSocket + Stomp + 보안 강화하기
WebSocket과 STOMP를 사용하여 실시간 통신을 구현하는 경우, 사용자 인증을 강화하여 보안성을 높이는 것이 중요하다.

이를 위해 JWT(Json Web Token) 토큰을 사용하여 사용자 인증을 처리하는 방법을 작성했다.

<br>

**JWT(Json Web Token) 토큰**

JWT(Json Web Token)는 웹에서 사용자 인증 정보를 전송하기 위한 표준 방식이다.

JWT는 Base64로 인코딩된 JSON 객체로 이루어져 있으며, 필요한 사용자 정보를 포함한다.

JWT는 서버에서 발급하며, 시그니처를 포함하여 유효성 검증이 가능하다.

<br>

Stomp를 사용하면 헤더에 token을 추가해 보안을 강화할 수 있다.

WebSocket 연결 시에만 JWT 인증 검증을 수행하는 것은 일반적인 방법이다. (WebSocket의 특성 때문) 

<br>

WebSocket은 일반 HTTP 요청과 달리, 클라이언트와 서버 사이에 지속적인 연결을 유지하고 있기 때문에,            

한 번 연결이 되면 해당 연결에서 보내는 모든 메시지는 이전에 검증된 인증 정보를 계속 사용한다.

`CONNECT` 요청에서 JWT 토큰을 검증하고, 유효한 토큰이면 User 객체를 추출하여 Authentication 객체를 생성한다. 

이후 WebSocket 연결에서는 해당 Authentication 정보를 사용하여 인증을 유지한다.

<br>

그러나 채팅 중간에 가로채는 공격은 여전히 가능하기 때문에, 

WebSocket 메시지를 보내거나 받을 때마다 해당 사용자의 인증을 다시 검증하는 것이 좋다. 

이를 위해서는 WebSocket 요청을 처리하는 Controller에서 인증 검증을 수행하는 Interceptor나 Filter를 사용하면 된다.

<br>

*일반적인 HTTP 요청의 경우에는 매 요청마다 JWT 인증 검증을 수행하는 것이 안전하다.                              
이때는 Spring Security의 FilterChain을 사용하여 JWT 인증 검증을 수행할 수 있다.                    
FilterChain을 사용하면 요청이 처리되기 전에 모든 요청에 대해 JWT 인증 검증을 수행하므로,                
누군가 요청을 가로채더라도 검증되지 않은 요청은 거절된다.        

<br><br><br>  

### JwtTokenProvider를 사용한 사용자 인증
JWT 인증 처리를 위해 JwtTokenProvider 클래스를 작성하여 토큰 검증과 사용자 정보 추출을 담당한다.

사용자 인증은 보안적인 측면에서 매우 중요한 요소 중 하나이다. 

유저의 아이디나 이메일과 같은 정보보다는 유저의 실제 인증을 확인하는 방식을 사용하는 것이 더 안전하다. 

따라서 JWT (Json Web Token) 토큰을 사용하여 사용자 인증을 확인하는 방식이 좋다.

JWT 토큰은 유효성 검증을 위한 시그니처가 포함되어 있어서, 시그니처를 확인하여 토큰의 유효성을 검증할 수 있다. 

<br>

**시그니처란?**

[관련 정리글](https://haedal-uni.github.io/posts/Session%EA%B3%BC-JWT/)      

JWT 토큰은 일련의 문자열로 이루어져 있으며, 헤더, 페이로드, 시그니처로 구성되어 있다.

시그니처는 토큰의 유효성을 검증하기 위한 값으로, 

토큰의 헤더와 페이로드를 조합한 후, 이를 서버에서 미리 정해진 비밀키를 이용하여 해시한 값이다.

이렇게 생성된 시그니처는 토큰의 끝에 추가되어 전체 토큰이 완성된다.

서버에서는 이 시그니처를 이용하여 토큰의 유효성을 검증한다.

만약 시그니처를 변경하거나 비밀키가 노출되면, 해당 토큰은 무효화되어야 한다.

따라서 시그니처는 JWT 토큰의 중요한 구성 요소 중 하나이며, 토큰의 무결성을 보장하기 위해 반드시 비밀키를 안전하게 관리해야 한다.

![image](https://user-images.githubusercontent.com/74857364/234974933-691e69f0-fdfa-4085-8be7-a801a09164ec.png)

<br>

이 방법은 JWT 토큰을 발급한 인증서버에서 시그니처를 생성하고 검증할 수 있도록 설계되어 있다. 

따라서, JWT 토큰을 활용하여 사용자 인증을 확인하는 것이 안전한 방법이다.

아래는 JWT 토큰 검증과 사용자 정보 가져오기를 위한 메서드이다.

```java
public boolean validateToken(String token) {
    try {
        Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token);
        return true;
    } catch (SignatureException ex) {
        logger.error("Invalid JWT signature");
    } catch (MalformedJwtException ex) {
        logger.error("Invalid JWT token");
    } catch (ExpiredJwtException ex) {
        logger.error("Expired JWT token");
    } catch (UnsupportedJwtException ex) {
        logger.error("Unsupported JWT token");
    } catch (IllegalArgumentException ex) {
        logger.error("JWT claims string is empty.");
    }
    return false;
}

public User getUserFromToken(String token) {
    Claims claims = Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token).getBody();
    String nickname = claims.getSubject();
    String role = claims.get("roles", String.class);
    UserRole userRole = UserRole.of(role);
    return User.builder()
            .nickname(nickname)
            .role(userRole)
            .build();
}

```
JwtTokenProvider 클래스에서는 토큰 검증과 사용자 정보 가져오기를 위한 메서드가 제공된다. 

이 메서드들을 사용하여 WebSocket 요청에서 인증 처리를 할 수 있다.

`validateToken()` 메서드는 입력받은 JWT 토큰의 유효성을 검증하고, 

`getUserFromToken()` 메서드는 JWT 토큰에서 추출한 사용자 정보를 반환한다.

<br>          

secretKey는 JWT를 생성하고 검증할 때 사용하는 시크릿 키다.   

이 값은 서버에서만 알고 있어야 하며, 노출되지 않도록 보안에 유의해야 한다.

<br>

WebSocket 연결을 요청하는 클라이언트는 JWT 토큰을 Authorization 헤더에 담아 요청한다. 

이 JWT 토큰을 검증하고, 인증된 사용자의 정보를 WebSocket 세션에 추가하는 과정이 필요하다. 

이를 위해 ChannelInterceptor를 사용할 수 있다.

<br><br><br>

## JWT 토큰을 이용한 인증 처리
* `@RequestHeader("Authorization")`는 채팅 인증이 아닌 HTTP 인증에서 사용할 수 있다.

<br>

**JwtInterceptor 와 ChannelInterceptor**

JwtInterceptor는 인증이 필요한 모든 요청에 대해 인증을 처리한다. 

 `@RequestHeader("Authorization")`을 추가하지 않고도 **JwtInterceptor**를 사용하면 **모든 컨트롤러**에서 인증 처리를 수행한다.

<br>

**ChannelInterceptor**는 **특정 채널**에 대해 처리를 수행한다. 

이를 이용하면 특정 컨트롤러에서 처리를 할 수 있다. 

예를 들어, WebSocket 연결에서 ChannelInterceptor를 사용하면 특정 채널에서 발생하는 메시지를 가로채고 처리할 수 있다.

따라서, **JwtInterceptor**와 **ChannelInterceptor**는 서로 다른 기능을 가지고 있으며, 사용 목적에 따라 선택하여 사용해야 한다.

<br>

### 1. JwtInterceptor를 이용한 JWT 토큰 검사
Spring WebSocket에서 JwtInterceptor를 이용하면 WebSocket 요청 시에 JWT 토큰을 검사할 수 있다.

JwtInterceptor는 WebSocket 요청이 처리되기 전에 요청의 header에서 JWT 토큰 값을 추출하고,

이를 검사하여 인증 처리를 수행한다.

JwtInterceptor를 사용하려면 다음과 같이 구현할 수 있다.
```java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    
    @Autowired
    private JwtTokenProvider jwtTokenProvider;
    
    @Override
    public void configureClientInboundChannel(ChannelRegistration registration) {
        registration.interceptors(new JwtInterceptor(jwtTokenProvider));
    }
    
    //... 생략 ...
}
```
위 코드에서 configureClientInboundChannel() 메소드에서 JwtInterceptor를 적용하고 있다.

JwtInterceptor는 JwtTokenProvider를 이용하여 JWT 토큰을 검사하고, 인증 정보를 얻어와 처리한다.

JwtInterceptor 클래스는 다음과 같이 구현할 수 있다.

<br>

```java
public class JwtInterceptor implements ChannelInterceptor {
    
    private JwtTokenProvider jwtTokenProvider;
    
    public JwtInterceptor(JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }
    
    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) throws AuthenticationException {
        StompHeaderAccessor accessor = MessageHeaderAccessor.getAccessor(message, StompHeaderAccessor.class);
        
        if (StompCommand.CONNECT.equals(accessor.getCommand())) {
            String token = accessor.getFirstNativeHeader("Authorization");
            
            if (token == null || !token.startsWith("Bearer ")) {
                throw new AuthenticationException("No JWT token found in request headers");
            }
            
            String jwtToken = token.substring(7);
            if (!jwtTokenProvider.validateToken(jwtToken)) {
                throw new AuthenticationException("JWT token is invalid");
            }
            
            Claims claims = jwtTokenProvider.getClaimsFromToken(jwtToken);
            accessor.setUser(new User(claims.getSubject(), true, new ArrayList<>()));
        }
        
        return message;
    }
}
```
<br><br><br>

### 2. ChannelInterceptor를 이용한 JWT 토큰 검증
Spring Boot를 사용하여 RESTful API를 개발하다보면, 인증 처리를 해야하는 경우가 있다. 

이때, JWT(JSON Web Token)은 많이 사용되는 인증 방식 중 하나이다.

Spring Security에서는 JWT 인증 처리를 위한 JwtInterceptor를 제공하고 있다.

하지만 JwtInterceptor를 사용하는 경우, 모든 컨트롤러에서 인증을 처리해야 하기 때문에 코드의 복잡도가 증가하게 된다.

이러한 문제점을 해결하기 위해 Spring에서는 ChannelInterceptor를 제공하고 있다.

<br>

ChannelInterceptor를 사용하면 특정 컨트롤러나 API에 대해서만 인증 처리를 할 수 있기 때문에 코드의 복잡도를 줄일 수 있다.

ChannelInterceptor는 STOMP 프로토콜에서 사용되며, WebSocket 연결 및 메시지 전송/수신 이벤트를 가로채고, 

메시지를 전송하기 전에 처리할 수 있는 기능을 제공한다.

<br><br><br>

### StompHandler
Spring Framework에서 WebSocket을 사용할 때, StompHandler 클래스를 사용하여 Header에 포함된 인증 정보를 검증하는 방법

WebSocket 연결 시 JWT 토큰을 확인하기 위해서는, ChannelInterceptorAdapter 내에서 JWT 토큰을 확인하는 코드를 작성해야 한다. 

이 코드는 ChannelInterceptorAdapter의 `preSend()` 메서드에서 처리할 수 있으며, 이 메서드는 클라이언트로부터 메시지가 전송되기 전에 호출된다. 

이 때 JWT 토큰을 확인하여 인증된 사용자인지 확인하고, 필요한 처리를 수행하면 된다.

<br>

#### 변경 전
```java
public class StompHandler implements ChannelInterceptor {
    private final JwtTokenProvider jwtTokenProvider;

    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) {
        StompHeaderAccessor headerAccessor = StompHeaderAccessor.wrap(message);

        if (StompCommand.CONNECT.equals(headerAccessor.getCommand())) {
            jwtTokenProvider.validateToken(headerAccessor.getFirstNativeHeader("Authorization"));
        }
        return message;
    }
}
```
<br><br>

#### 변경 후
기존 코드에서는 StompHandler 클래스에서 JwtTokenProvider 클래스의 `validateToken()` 메서드를 호출하여 인증 정보를 검증했지만, 

변경된 코드에서는 `getUserFromToken()` 메서드를 사용하여 유저 정보를 추출하고, 

이를 사용하여 UsernamePasswordAuthenticationToken 객체를 생성하여 SecurityContextHolder에 인증 정보를 저장하게 된다.

<br>

또, 예외 처리도 변경되었는데 이전 코드에서는 JwtTokenProvider 클래스에서 InvalidTokenException을 던졌지만

변경된 코드에서는 MissingAuthorizationException 또는 BadCredentialsException을 던져 예외 처리를 하게 된다.

이러한 변경 사항을 적용한 StompHandler 클래스의 코드는 아래와 같다.

```java
@RequiredArgsConstructor
@Component
@Slf4j
public class StompHandler implements ChannelInterceptor {
   // client가 connect할 떄 header로 보낸 Authorization에 담긴 jwt Token을 검증
   private final JwtTokenProvider jwtTokenProvider;

   @Override
   public Message<?> preSend(Message<?> message, MessageChannel channel) {
      StompHeaderAccessor headerAccessor = StompHeaderAccessor.wrap(message);

      if (StompCommand.CONNECT.equals(headerAccessor.getCommand())) {
         log.info("websocket 연결 - jwt token 검증");
         String accessToken = headerAccessor.getFirstNativeHeader("Authorization");
         User user = jwtTokenProvider.getUserFromToken(accessToken);
         if (user != null) {
            Authentication authentication = new UsernamePasswordAuthenticationToken(user, null, user.getAuthorities());
            SecurityContextHolder.getContext().setAuthentication(authentication);
         } else {
            throw new BadCredentialsException("Invalid JWT token");
         }
      }

      return message;
   }
}
```

이제 StompHandler 클래스를 사용하여 WebSocket을 보다 안전하게 사용할 수 있다.

<br>

```js
function connect() {
    let nickname = localStorage.getItem('wschat.sender');
    let token = localStorage.getItem('token');
    if (nickname) {
        let socket = new SockJS('/ws');
        stompClient = Stomp.over(socket);
        stompClient.connect({Authorization: token}, onConnected, onError);
    }
}
```

<br><br><br>

### WebSocket API에서 JWT 인증 구현하기

Spring에서 제공하는 ChannelInterceptor는 WebSocket을 통해 들어오는 요청을 인터셉트하고 처리하는 역할을 한다. 

이를 이용하여 WebSocket API를 개발할 때, 인증 처리와 관련된 기능을 구현할 수 있다.

이 경우에는 보통 WebSocket의 메시지 헤더(StompHeaderAccessor)를 활용하여 인증 정보를 전달다. 

그리고 이러한 인증 정보를 검증하는 작업은 ChannelInterceptor를 활용하여 수행할 수 있다.

```java
@MessageMapping("/chat/addUser")
public void addUser(@Payload ChatMessage chatMessage, SimpMessageHeaderAccessor headerAccessor) {
    String token = headerAccessor.getFirstNativeHeader("Authorization");
    String nickname = jwtTokenProvider.getNickname(token);
    chatMessage.setSender(nickname);
    chatMessage.setType(ChatMessage.MessageType.JOIN);
    chatService.connectUser("Connect", chatMessage.getRoomId(), chatMessage);
    template.convertAndSend("/topic/public/" + chatMessage.getRoomId(), chatMessage);
}
```

`@Header` 어노테이션은 STOMP 프로토콜에서 사용되는 어노테이션으로, 

STOMP 프로토콜 메시지의 header에서 값을 추출하여 메소드의 인자로 전달하는 역할을 한다. 

<br>

`@Payload` 어노테이션은 WebSocket API의 메시지 payload를 받아들인다는 것을 나타내는 어노테이션이다.      

이를 이용하여 WebSocket API의 메소드에 `@Header("Authorization")` 어노테이션을 사용하여  

HTTP 요청의 Authorization 헤더에서 JWT 토큰 값을 가져올 수 있다. 

그리고 이 값을 ChannelInterceptor를 사용하여 검증하고, 적절한 인증 정보를 얻어와서 채팅 메시지를 처리하면 된다.

<br><br>

```java
@MessageMapping("/chat/addUser")
public void addUser(@Payload ChatMessage chatMessage, @Header("Authorization") String token) {
    String nickname = jwtTokenProvider.getNickname(token);
    chatMessage.setSender(nickname);
    chatMessage.setType(ChatMessage.MessageType.JOIN);
    chatService.connectUser("Connect", chatMessage.getRoomId(), chatMessage);
    template.convertAndSend("/topic/public/" + chatMessage.getRoomId(), chatMessage);
}
```

`@Payload` 어노테이션은 ChatMessage 클래스의 인스턴스를 메시지 payload로 받아들인다는 것을 나타내는 어노테이션이다. 

따라서 ChatMessage 클래스에는 token 필드가 없으므로, 

해당 메소드에서는 `@Header("Authorization")` 어노테이션을 이용하여 HTTP 요청의 Authorization 헤더에서 JWT 토큰 값을 가져올 수 있다. 

그리고 이 값을 ChannelInterceptor를 사용하여 검증하고, 적절한 인증 정보를 얻어와서 채팅 메시지를 처리하면 된다.

<br><br><br>

JwtInterceptor와 ChannelInterceptor를 사용하는 방법 모두 HTTP 요청의 헤더에 담긴 토큰 값을 가져올 수 있다는 공통점이 있다. 

이렇게 가져온 토큰 값을 두 인터셉터에서 검사하여 인증 처리를 수행하고, 필요한 정보를 추출하여 해당하는 기능을 수행할 수 있다.

하지만 두 인터셉터는 기능적인 측면에서 차이가 있다. 

<br>

JwtInterceptor는 JWT 토큰 검증을 수행하고, 토큰이 유효한지 검사한 후 토큰 안에 포함된 정보를 추출하는 역할을 한다. 

ChannelInterceptor는 WebSocket 연결에서 사용하는 헤더 정보를 추출하여 검증하는 역할을 한다.

따라서, JwtInterceptor는 RESTful API를 사용하는 서버에서 주로 사용되며, 

ChannelInterceptor는 WebSocket 연결에서 사용되는 경우가 많다.

<br>

두 인터셉터 중에서 선택하는 것은 개발자의 취향이나 프로젝트 구조에 따라 다르다. 

RESTful API와 WebSocket을 함께 사용하는 경우라면 두 인터셉터를 모두 사용할 수 있다.

하지만 두 인터셉터를 모두 사용하는 경우에는 토큰 검증과 관련된 코드를 중복해서 작성해야 하기 때문에 

유지보수 측면에서 불편함이 있을 수 있다.

따라서, 두 인터셉터를 모두 사용하는 경우라면, 공통으로 사용되는 코드를 별도의 클래스로 분리하여 작성하고, 

이를 두 인터셉터에서 참조하도록 구현하는 것이 좋다. 

이렇게 구현하면 코드의 중복을 피할 수 있으며, 유지보수성이 높은 구조를 만들 수 있다.

<br><br><br>


참고

WebSocket 프로토콜은 서버와 클라이언트 사이의 양방향 통신을 제공하므로, 실시간으로 데이터를 주고받을 수 있다. 

HTTP 프로토콜은 클라이언트에서 서버로 요청을 보내고, 서버에서 클라이언트로 응답을 보내는 단방향 통신 방식이다. 

WebSocket과 HTTP는 서로 다른 프로토콜이므로, 서로 다른 방식으로 통신한다.

<br>

WebSocket의 경우 HTTP 프로토콜과는 다른 프로토콜을 사용하기 때문에, 

HTTP 요청에서 사용하는 doFilter와 WebSocket에서 사용하는 ChannelInterceptorAdapter는 서로 다른 방식으로 작동한다.

따라서, 만약 HTTP AJAX 호출에서 사용하는 doFilter로 JWT 토큰을 확인하도록 구현했다면

해당 코드는 WebSocket의 ChannelInterceptorAdapter와는 관련이 없다. 

<br>

WebSocket의 ChannelInterceptorAdapter는 HTTP 프로토콜이 아닌 다른 프로토콜을 사용하기 때문에

이 ChannelInterceptorAdapter를 사용하여 WebSocket 연결 시 JWT 토큰을 확인하도록 구현해야 한다.

따라서, WebSocket 연결 시 JWT 토큰을 확인하기 위해서는 

ChannelInterceptorAdapter 내에서 JWT 토큰을 확인하는 코드를 작성해야 한다. 

이 코드는 ChannelInterceptorAdapter의 preSend 메서드에서 처리할 수 있으며, 

이 메서드는 클라이언트로부터 메시지가 전송되기 전에 호출된다. 

이 때 JWT 토큰을 확인하여 인증된 사용자인지 확인하고, 필요한 처리를 수행하면 된다.       
