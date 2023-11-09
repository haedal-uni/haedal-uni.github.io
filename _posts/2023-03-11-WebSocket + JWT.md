---
categories: Project WebSocket
tags: [spring, WebSocket, Chat, Security]
---

# WebSocket + JWT
관련 글          
- [Websocket](https://haedal-uni.github.io/posts/WebSocket/)                                    
- [Websocket + 부가기능](https://haedal-uni.github.io/posts/WebSocket-+%EB%B6%80%EA%B0%80%EA%B8%B0%EB%8A%A5/)                            
- [Websocket (채팅 기록 json 파일 저장하기)](https://haedal-uni.github.io/posts/WebSocket(%EC%B1%84%ED%8C%85-%EA%B8%B0%EB%A1%9D-Json-%ED%8C%8C%EC%9D%BC-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B8%B0)/)                               
- [Sse](https://haedal-uni.github.io/posts/SSE/)                      
- [Sse 문제점](https://haedal-uni.github.io/posts/SSE-%EB%AC%B8%EC%A0%9C%EC%A0%90/)                        
- [Websocket + jwt](https://haedal-uni.github.io/posts/WebSocket-+-JWT/) 👈🏻                          
- [Websocket test](https://haedal-uni.github.io/posts/WebSocket-Test/)                        
- [Jmh - 채팅 파일 refactoring](https://haedal-uni.github.io/posts/JMH/)           

<br><br> 

내가 채팅방을 작성하면서 security를 구현한 이유는 stomp를 사용하면 헤더에 token을 추가해 보안을 강화할 수 있다는 것

이제 코드로 작성해본다.

<br>

### WebSocketConfig
StompHandler가 Websocket 앞단에서 token을 체크할 수 있도록 interceptor로 설정한다.
```java
private final StompHandler stompHandler;

@Override
public void configureClientInboundChannel(ChannelRegistration registration){
    // jwt 토큰 검증을 위해 생성한 stompHandler를 인터셉터로 지정해준다.
    registration.interceptors(stompHandler);
}
```
<br><br><br>

### JwtTokenProvider
```java
// 기존에 작성한 코드 활용
// jwt token을 복호화 하여 이름을 얻는다.
public String getUsername(String token) {
    log.info("[getUsername] 토큰 기반 회원 구별 정보 추출");

    // 토큰을 생성할때 넣었던 sub 값 추출
    String info = getClaims(token).getBody().getSubject();
    log.info("[getUsername] 토큰 기반 회원 구별 정보 추출 완료");

    return info;
}

private Jws<Claims> getClaims(String jwt){
    try{
        return Jwts.parser().setSigningKey(secretKey).parseClaimsJws(jwt);
    }catch (SignatureException e){ // 잘못된 jwt signature
        log.error("Invalid JWT signature");
        throw e;
    } catch (MalformedJwtException e){
        log.error("Invalid JWT token");
        throw e;
    } catch (ExpiredJwtException e){ // jwt 만료
        log.error("Expired JWT token");
        throw e;
    } catch (UnsupportedJwtException e){ // 지원하지 않는 jwt
        log.error("Unsupported JWT token");
        throw e;
    } catch (IllegalArgumentException e){ // 잘못된 jwt 토큰
        log.error("JWT claims string is empty");
        throw e;
    }
}
```
`Jwts.parser().setSigningKey(secretKey).parseClaimsJws(jwt)` 을 사용하는 코드가 많아서 묶어서 사용

<br>

예시 출력

`getClaims(token)` : header={alg=HS256},body={sub=haedal, roles=USER, iat=1516239022, exp=1234567890},signature=ajslkdfjaldkfjaDSLFieo

![image](https://user-images.githubusercontent.com/74857364/224392770-dd5f8550-e18c-45f0-ba73-d7b4ffa46d24.png)

정리글 👉🏻 [Session과 jwt](https://haedal-uni.github.io/posts/Session%EA%B3%BC-JWT/)

<br><br><br>

### StompHandler
websocket 연결 시 요청 header의 jwt token 유효성을 검증하는 코드를 추가한다.

유효하지 않은 jwt 토큰이 세팅될 경우 websocket 연결을 하지 않고 예외처리 된다.

```java
@RequiredArgsConstructor
@Component
public class StompHandler implements ChannelInterceptor {
   private final JwtTokenProvider jwtTokenProvider;

   @Override
   public Message<?> preSend(Message<?> message, MessageChannel channel) {
      StompHeaderAccessor headerAccessor = StompHeaderAccessor.wrap(message);
    
      // websocket 연결 시 header의 jwt token 검증
      if (StompCommand.CONNECT.equals(headerAccessor.getCommand())) {
         jwtTokenProvider.validateToken(headerAccessor.getFirstNativeHeader("Authorization"));
      }
      return message;
   }
}
```
`headerAccessor.getNativeHeader()` : 지정된 네이티브 헤더가 있는 경우 모든 값을 반환

`headerAccessor.getFirstNativeHeader()` : 지정된 네이티브 헤더가 있는 경우 첫 번째 값을 반환

<br><br>

preSend() 메소드에서 클라이언트가 CONNECT할 때 헤더로 보낸 Authorization에 담긴 jwt Token을 검증하도록 한다.
```js
function connect(event) {
    let socket = new SockJS('/ws');
    stompClient = Stomp.over(socket);
    stompClient.connect({Authorization:token}, onConnected, onError);
}

```

<br><br><br>

### ChatController

```js
function sendMessage(event) {
    let messageContent = messageInput.value.trim();
    if (messageContent && stompClient) {
        let chatMessage = {
            roomId: roomId, sender: username, message: messageInput.value, type: 'TALK'
        };
        saveFile(chatMessage)
        stompClient.send("/app/chat/sendMessage", {Authorization:token}, JSON.stringify(chatMessage));
        messageInput.value = '';
    }
    event.preventDefault(); // 계속 바뀌는 것을 방지함
}
```

<br><br>

Authorization으로 header에 token값을 넣었으니 Controller에서 `@Header("Authorization")`라고 작성했다.   
```java
@Controller
@RequiredArgsConstructor
public class ChatController {
    private final JwtTokenProvider jwtTokenProvider;

    @MessageMapping("/chat/sendMessage")
    public void sendMessage(@Payload ChatMessage chatMessage, @Header("Authorization") String token) {
        String nickname = jwtTokenProvider.getUsername(token);
        template.convertAndSend("/topic/public/" + chatMessage.getRoomId(), chatMessage);
    }
}  
```
websocket을 통해 서버에 메세지가 send 되었을 때도 jwt token 유효성 검증이 필요하다.

위와 같이 회원 대화명(id)를 조회하는 코드를 삽입하여 유효성이 체크될 수 있도록 한다.   
    

<br><br><br><br>

*reference*          
[[Spring boot + React] STOMP로 실시간 채팅 구현하기 (3) - 사용자 인증 구현하기](https://velog.io/@dldmswjd322/Spring-boot-React-STOMP%EB%A1%9C-%EC%8B%A4%EC%8B%9C%EA%B0%84-%EC%B1%84%ED%8C%85-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-3-%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%9D%B8%EC%A6%9D-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0)               
[Spring Boot + STOMP + JWT Socket 인증하기](https://velog.io/@tlatldms/Spring-Boot-STOMP-JWT-Socket-%EC%9D%B8%EC%A6%9D%ED%95%98%EA%B8%B0)                           
[Spring websocket chatting server(4) - SpringSecurity+Jwt를 적용하여 보완강화하](https://www.daddyprogrammer.org/post/5072/spring-websocket-chatting-server-spring-security-jwt/)               

