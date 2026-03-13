---
categories: Project Chat
tags: [spring, WebSocket, Chat, Security]
---
# WebSocket + STOMP + 보안 강화하기

채팅과 같이 지속적인 연결을 유지하는 구조에서 한 번의 인증 실수가 전체 세션 보안에 영향을 줄 수 있다.

이 글에서는 WebSocket 환경에서 JWT를 활용하여 인증을 강화하는 방법을 정리했다.

<br><br><br>

## JWT(Json Web Token)란 

JWT는 웹에서 사용자 인증 정보를 안전하게 전달하기 위한 표준 방식이다.  

JWT는 Base64로 인코딩된 JSON 구조로 이루어져 있으며 사용자에 대한 정보와 함께 이를 검증할 수 있는 시그니처를 포함하고 있다.  

토큰은 서버에서 발급되며 서버만 알고 있는 비밀키를 기반으로 생성된다.  

따라서 클라이언트가 토큰을 위조하더라도 서버에서 시그니처를 검증하는 과정에서 위조 여부를 확인할 수 있다.

<br><br><br>

### JWT의 구조와 시그니처
![image](https://user-images.githubusercontent.com/74857364/234974933-691e69f0-fdfa-4085-8be7-a801a09164ec.png)

JWT는 헤더(Header), 페이로드(Payload), 시그니처(Signature) 총 세 부분으로 구성된다.  

시그니처는 헤더와 페이로드를 조합한 뒤 서버의 비밀키로 해시한 값이다.  

서버는 동일한 비밀키로 다시 계산하여 시그니처를 비교함으로써 토큰의 무결성을 검증한다.  

만약 토큰 내용이 중간에 변경되었다면 시그니처 값이 달라지므로 검증에 실패하게 된다.  

따라서 secretKey는 반드시 서버 내부에서만 관리되어야 하며 외부로 노출되면 안된다. 

<br><br><br>

### WebSocket에서 JWT 인증이 필요한 이유

일반적인 HTTP 요청은 요청이 올 때마다 인증을 수행한다.  

Spring Security에서는 FilterChain을 통해 모든 요청에 대해 JWT를 검증할 수 있다.  

하지만 WebSocket은 HTTP와 다르게 한 번 연결이 이루어지면 지속적인 연결을 유지하는 구조다.  

CONNECT 요청에서 인증이 완료되면 이후 메시지 전송에서는 동일한 연결을 계속 사용하게 된다.  

이 때문에 보통 WebSocket에서는 CONNECT 시점에 JWT를 검증하고 Authentication 객체를 생성하여 세션에 저장한다.

채팅 중간에 가로채는 공격을 대비하려면 메시지 처리 시점에서도 추가 검증을 수행하는 것이 안전하다.

<br><br><br>

## JwtTokenProvider를 이용한 토큰 검증

JWT 인증 처리를 위해 JwtTokenProvider 클래스를 작성하여 토큰 검증과 사용자 정보 가져오기

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

`validateToken()` 는 토큰이 변조되지 않았는지 만료되지 않았는지 등을 검사한다.  

`getUserFromToken()`는 토큰 내부의 사용자 정보를 추출하여 User 객체로 반환한다.  

<br><br><br>

## WebSocket에서 JWT 인증 처리하기

`@RequestHeader("Authorization")`

WebSocket에서는 HTTP의 `@RequestHeader`를 직접 사용할 수 없기 때문에 STOMP 헤더를 통해 Authorization 값을 전달한다. 


ChannelInterceptor를 사용하면 메시지가 서버에 도달하기 전에 가로채어 인증 검증을 수행할 수 있다.
    

<br><br><br>

### 1. WebSocketConfig에 인터셉터 등록하기

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
}
```

`configureClientInboundChannel()` 를 통해 client에서 들어오는 모든 STOMP 메시지에 대해 인터셉터를 적용할 수 있다.

<br><br><br>

### 2. JwtInterceptor 구현

```java
public class JwtInterceptor implements ChannelInterceptor {
    
    private JwtTokenProvider jwtTokenProvider;
    
    public JwtInterceptor(JwtTokenProvider jwtTokenProvider) {
        this.jwtTokenProvider = jwtTokenProvider;
    }
    
    @Override
    public Message<?> preSend(Message<?> message, MessageChannel channel) {
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
            
            User user = jwtTokenProvider.getUserFromToken(jwtToken);
            accessor.setUser(new UsernamePasswordAuthenticationToken(user, null, new ArrayList<>()));
        }
        
        return message;
    }
}
```

CONNECT 요청이 들어왔을 때 Authorization 헤더에서 JWT를 추출하여 검증한다.  

토큰이 유효하면 사용자 정보를 기반으로 Authentication 객체를 생성하여 WebSocket 세션에 등록한다.

<br><br><br>

### 3. StompHandler를 이용한 추가 검증

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
         
         if (!jwtTokenProvider.validateToken(accessToken)) {
             throw new BadCredentialsException("Invalid JWT token");
         }
      }
      return message;
   }
}
```

CONNECT 시점에 한 번 더 토큰을 검증하여 보안을 강화

<br><br><br>

### 4. client에서 토큰 전달하기

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

client는 STOMP 연결 시 Authorization 헤더에 JWT 토큰을 담아 서버로 전달한다.

<br><br><br>

### `@MessageMapping`에서 토큰 사용하기

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

`@Header`를 사용하면 STOMP 헤더에서 Authorization 값을 가져올 수 있다.  

`@Payload`는 메시지 본문 데이터를 객체로 매핑해주는 역할을 한다.

<br><br><br>

#### JwtInterceptor와 ChannelInterceptor

JwtInterceptor는 주로 HTTP 요청에서 JWT를 검증

ChannelInterceptor는 WebSocket과 STOMP 메시지를 가로채어 처리  

REST API와 WebSocket을 함께 사용하는 경우 두 방식 모두 사용할 수 있다.  

토큰 검증 로직이 중복되지 않도록 공통 로직을 별도의 클래스로 분리하는 것이 유지보수 측면에서 좋다. 

<br><br><br>

## 정리

WebSocket은 HTTP와 다른 프로토콜이기 때문에 기존의 doFilter 방식으로는 인증을 처리할 수 없었다.  

ChannelInterceptor를 사용하여 CONNECT 시점 또는 메시지 전송 시점에 JWT를 검증했다.  

JWT를 사용할 때 서버의 secretKey를 안전하게 관리해야한다. 
