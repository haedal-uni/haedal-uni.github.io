---
categories: Project
tags: [spring, WebSocket]
---

# WebSocket

그동안 채팅 구현을 하기 위해 이것 저것 보면서 적용하느라 뭐가 뭔지도 모르고 쓴 경향이 있었다.

그러다보니 이번에 제대로 공부를 하면서 구현하려고 했고 초반에는 어디서는 이 코드만 쓰고 또 어디서는 이 코드만 쓰고

또 다른데는 내가 현재 작성했던 코드를 써서 뭐가 뭔지도 모르겠고 정리가 잘 안되는 느낌이었는데

여러 글들을 보면서 글을 정리하기 시작했고 그러다보니 어느정도 감을 잡게 되었다. (💡코드에 정답은 없다!)

하지만 내가 생각한 부분에 대한 검증을 받을 수 없기 때문에 맞는지는 모르겠고 추측만 하는 것도 있다.

*이 전에 작성한 코드들이 있어서 비교하면서 작성해봤다.

<br>

내용이 길어져서 1편과 2편으로 나눠서 작성했다.

<br><br>

아래는 기존에 작성했던 코드 일부이다. 현재는 handler를 사용하지 않는다.

<br>

## Handler

서버-클라이언트 소켓 통신에서 사용하는 메세지 스펙을 정의 한다.

웹소켓 핸들러 클래스는 아래와 같이 4개의 메소드를 Override 해야한다.

- afterConnectionEstablished : 웹소켓 연결 시
- handleTextMessage : 데이터 통신 시
- afterConnectionClosed : 웹소켓 연결 종료 시
- handleTransportError : 웹소켓 통신 에러 시

<br>

문자열 메시지 기반으로 테스트를 진행하기 때문에 **TextWebSocketHandler**를 상속받아 메시지를 전달받는다.

```java
public class WebSocketHandler extends TextWebSocketHandler {
    private final Map<String, WebSocketSession> sessions = new ConcurrentHashMap<>();

    @Override // 웹 소켓 연결
    public void afterConnectionEstablished(WebSocketSession session){}
     
    @Override // 양방향 데이터 통신
    protected void handleTextMessage(WebSocketSession session, TextMessage message){}

    @Override // 소켓 연결 종료
    public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus){}

    @Override // 소켓 통신 에러
    public void handleTransportError(WebSocketSession session, Throwable throwable){}
}
```

<br><br>

여기서 간단하게 기존에 작성했던 코드와 비교해봤다. (내용은 생략)


```java
public class WebSocketHandler extends TextWebSocketHandler{
    private static List<Session> sessionUsers = Collections.synchronizedList(new ArrayList<>());
    
    @OnOpen
    public void open(Session newUser) throws IOException {}

    @OnMessage
    public void onMessage(Session receiveSession, String msg) throws IOException {}
    
    @OnClose
    public void onClose(Session nowUser, CloseReason closeReason) {}
    
    @OnError
    public void onError(Session session, Throwable e) {}
}
```    
    
기존에 나는 `@OnOpen`, `@OnMessage`, `@OnClose`, `@OnError`를 사용했고 WebSocketSession대신 Session을 사용했다.

먼저 WebSocketSession과 Session 둘 다 Closeable을 상속받고 있었고 크게 차이점을 느끼진 못했다.

*WebSocketSession은 Spring에서 Websocket Connection이 맺어진 세션

<br>

![image](https://user-images.githubusercontent.com/74857364/213742193-f8e7bc55-a971-452b-9999-abf619965594.png){: width="70%"}

<br><br><br>

이번엔 `@OnClose`와 handleTransportError에 대해서 살펴봤다. 

![image](https://user-images.githubusercontent.com/74857364/213743192-29403c2e-7504-4c94-bcb5-2aafc886d3b1.png){: width="70%"}

`@OnClose`와 session의 패키지가 같아서 session을 쓰면 @OnClose를 쓰는건가? 싶었다.

<br><br>

![image](https://user-images.githubusercontent.com/74857364/213743352-fb6a5b41-8e1c-41a2-8983-2e1afe63d102.png){: width="60%"}

위 사진은 handleTransportError가 포함된 AbstractWebSocketHandler 추상클래스인데 

패키지명이 WebSocketSession과 같아서 뭔가 패키지 명이 같은 것을 쓰는 듯한 느낌이 들었다.

<br>

WebSocketSession ( = session)
- afterConnectionEstablished (= @OnOpen)
- handleTextMessage (= @OnMessage)
- afterConnectionClosed (= @OnClose)
- handleTransportError (= @OnError)

<br>

각자 스타일에 맞게 작성하면 될 것 같은 결론을 내렸다.

<br><br><br>

---

## STOMP
STOMP는 TCP 또는 WebSocket 같은 양방향 네트워크 프로토콜 기반으로 동작한다.

STOMP (Simple Text Oriented Messaging Protocol)은 메세징 전송을 효율적으로 하기 위해 탄생한 프로토콜이고, 

기본적으로 pub / sub 구조로 되어있어 메세지를 전송하고 메세지를 받아 처리하는 부분이 확실히 정해져 있기 때문에 

개발자 입장에서 명확하게 인지하고 개발할 수 있는 이점이 있다. 

<br>

➡️ STOMP 프로토콜은 WebSocket 위에서 동작하는 프로토콜로써 

클라이언트와 서버가 전송할 메세지의 유형, 형식, 내용들을 정의하는 매커니즘이다.

또한 STOMP를 이용하면 메세지의 헤더에 값을 줄 수 있어 헤더 값을 기반으로 통신 시 인증 처리를 구현하는 것도 가능하다.

<br>

우체통(Topic)이 있다면 집배원(Publisher)이 신문을 우체통에 배달하는 행위가 있고, 

우체통에 신문이 배달되는 것을 기다렸다가 빼서 보는 구독자(Subscriber)의 행위가 있다. 

이때 구독자는 다수가 될 수 있다. pub / sub 컨셉을 채팅방에 빗대면 다음과 같다.

- 채팅방 생성 : pub / sub 구현을 위한 Topic이 생성됨

- 채팅방 입장 : Topic 구독

- 채팅방에서 메세지를 송수신 : 해당 Topic으로 메세지를 송신(pub), 메세지를 수신(sub)

<br>

스프링은 메세지를 외부 Broker에게 전달하고, Broker는 WebSocket으로 연결된 클라이언트에게 메세지를 전달하는 구조가 된다.

<br><br>

Spring framework 및 Spring Security는 STOMP 를 사용하여 WebSocket만 사용할 때보다 더 다채로운 모델링을 할 수 있다.

Messaging Protocol을 만들고 메세지 형식을 커스터마이징 할 필요가 없다.

RabbitMQ, ActiveMQ 같은 Message Broker를 이용해, Subscription(구독)을 관리하고 메세지를 브로드캐스팅할 수 있다.

WebSocket 기반으로 각 Connection(연결)마다 WebSocketHandler를 구현하는 것 보다 

`@Controller` 된 객체를 이용해 조직적으로 관리할 수 있다.

<br>

```
>>> SUBSCRIBE
id:sub-0
destination:/topic/public
```
<br>


**채팅방 입장**
```
>>> SEND
destination:/app/chat/addUser
content-length:32

{"sender":"admin","type":"JOIN"}
```
<br>

**message**
```
>>> SEND
destination:/app/chat/sendMessage
content-length:60

{"sender":"admin","message":"안녕하세요","type":"TALK"}
```
```
<<< MESSAGE
destination:/topic/public
content-type:application/json
subscription:sub-0
message-id:4aus2gfz-40
content-length:60

{"type":"TALK","sender":"admin","message":"안녕하세요"}
```
<br>


**퇴장**
```
<<< MESSAGE
destination:/topic/public
content-type:application/json
subscription:sub-0
message-id:lsgtkyk2-32
content-length:48

{"type":"LEAVE","sender":"admin","message":null}
```

<br>


즉, 메세지는 STOMP의 "destination" 헤더를 기반으로 @Controller 객체의 `@MethodMapping` 메서드로 라우팅 된다.

STOMP의 "destination" 및 Message Type을 기반으로 메세지를 보호하기 위해 Spring Security를 사용할 수 있다.

<br><br>

### STOM를 사용하는 이유
WebSocket은 통신 프로토콜 일뿐이다. 
 
특정 주제를 구독한 사용자에게만 메시지를 보내는 방법 또는 특정 사용자에게 메시지를 보내는 방법과 같은 내용은 정의하지 않는다. 
 
이러한 기능을 위해서는 STOMP가 필요하다.

<br><br><br>

## Code
### ChatController
WebSocketConfig에서 "/app"로 시작하는 대상이 있는 클라이언트에서 보낸 모든 메시지는 

`@MessageMapping` 어노테이션이 달린 메서드로 라우팅 된다.

```java
@Controller
public class ChatController {
    @MessageMapping("/chat/sendMessage")
    @SendTo("/topic/public")
    public ChatMessage sendMessage(@Payload ChatMessage chatMessage) {
        return chatMessage;
    }

    @MessageMapping("/chat/addUser")
    @SendTo("/topic/public")
    public ChatMessage addUser(@Payload ChatMessage chatMessage, SimpMessageHeaderAccessor headerAccessor){
        headerAccessor.getSessionAttributes().put("username", chatMessage.getSender());
        return chatMessage;
    }
}
```
`@MessageMapping("/chat/sendMessage")` : 클라이언트에서 `/app/chat/message`로 메세지를 발행한다.

`@SendTo("/topic/public")` : `/topic/public` 에 구독중인 클라이언트에게 메세지를 보낸다.

<br>

특정 사용자가 "/chat/sendMessage"라는 경로로 메세지를 보내면 

"/topic/public" 라는 토픽을 구독하는 사용자들에게 메세지를 전달한다.

<br>

@SendTo → 1:n - 경로는 "/topic"으로 시작된다.

@SendToUser → 1:1 - 경로는 "/queue"로 시작한다.

<br>

client : 
```js
stompClient.subscribe('/topic/public', onMessageReceived);
stompClient.send("/app/chat/addUser", {}, JSON.stringify({sender: username, type: 'JOIN'}))
```

<br><br><br><br>

### Config
```java
@Configuration // 컨테이너 등록
@EnableWebSocket // 웹소켓 서버를 사용하도록 정의
public class WebSocketConfig implements WebSocketConfigurer {
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) { 
        registry.addEndpoint("/ws/chat").setAllowedOriginPatterns("*");
    }
}
```
`@Configuration` : 해당 클래스가 Bean의 설정을 할 것이다 라는 것을 말한다.

**`registerStompEndpoints()`** : Client에서 websocket연결할 때 사용할 API 경로를 설정해주는 메서드

<br>

🔽

**WebSocket SockJs 설정**
```java
@Configuration // 컨테이너 등록
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) { 
        registry.addEndpoint("/ws/chat").setAllowedOriginPatterns("*").withSockJS();
    }
}
```
클라이언트가 웹 소켓 서버에 연결하는 데 사용할 웹 소켓 엔드 포인트를 등록한다.

endpoint 구성에 `withSockJS()`를 사용한다.

SockJS는 웹 소켓을 지원하지 않는 브라우저에 폴백 옵션을 활성화하는 데 사용된다.

*Fallback 이란? : 어떤 기능이 약해지거나 제대로 동작하지 않을 때, 이에 대처하는 기능 또는 동작
  
<br>

🔽

**STOMP 사용**
```java
@Configuration 
@EnableWebSocketMessageBroker
public class WebSocketConfig implements WebSocketMessageBrokerConfigurer {
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) { 
        registry.addEndpoint("/ws/chat").setAllowedOriginPatterns("*").withSockJS();
    }
}
```

구현할 interface의 대상이 WebSocketMessageBrokerConfigurer로 바뀌었다.

→ 웹 소켓 연결을 구성하기 위한 메서드를 구현하고 제공한다.

<br>

```java
@Configuration
@EnableWebSocketMessageBroker
public class ChatConfig implements WebSocketMessageBrokerConfigurer {

    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/ws").setAllowedOriginPatterns("*").withSockJS();
    }

    @Override
    public void configureMessageBroker(MessageBrokerRegistry registry) {
        registry.enableSimpleBroker("/queue", "/topic");
        registry.setApplicationDestinationPrefixes("/app");
    }
}
```
☑️ **`@EnableWebSocketMessageBroker`** : Stomp를 사용하기위해 선언하는 어노테이션

WebSocket 서버를 활성화하는 데 사용된다.

<br>

☑️ `/ws` 는 WebSocket 또는 SockJS 클라이언트가 WebSocket Handshake로 커넥션을 생성할 경로이다.

<br>

☑️ ***configureMessageBroker()*** : 한 클라이언트에서 다른 클라이언트로 메시지를 라우팅 하는 데 사용될 메시지 브로커를 구성

<br>

☑️ ***.enableSimpleBroker("/queue", "/topic");*** : 해당 경로로 SimpleBroker를 등록한다. (메시지 받을 때 관련 경로 설정)

SimpleBroker는 해당하는 경로를 SUBSCRIBE하는 client에게 메시지를 전달하는 간단한 작업을 수행한다.

/topic, /queue로 시작하는 "destination" 헤더를 가진 메세지를 브로커로 라우팅한다.

주로 /queue는 1대1 메시징, /topic은 1대다 메시징일 때 주로 사용한다.

```
>>> SUBSCRIBE
id:sub-0
destination:/topic/public
```

메시지 브로커는 특정 주제를 구독 한 연결된 모든 클라이언트에게 메시지를 broadcast 한다.

*브로드캐스팅은 송신 호스트가 전송한 데이터가 네트워크에 연결된 모든 호스트에 전송되는 방식을 의미

<br>

간단한 인 메모리 메시지 브로커를 활성화했다.

그러나 RabbitMQ 또는 ActiveMQ와 같은 다른 모든 기능을 갖춘 메시지 브로커를 자유롭게 사용할 수 있다.

<br>

☑️ ***setApplicationDestinationPrefixes("/app");*** : client에서 SEND 요청을 처리한다. (메시지 보낼 때 관련 경로 설정)

클라이언트가 메시지를 보낼 때 경로 맨앞에 /app이 붙어있으면 Broker로 보내진다.

/app 경로로 시작하는 STOMP 메세지의 "destination" 헤더는 @Controller 객체의 @MessageMapping 메서드로 라우팅된다.

"/app" 시작되는 메시지가 message-handling methods으로 라우팅 되어야 한다는 것을 명시

<br>

client : 
```js
var socket = new SockJS('/ws');
stompClient = Stomp.over(socket);
stompClient.connect({}, onConnected, onError);

function onConnected() {
   stompClient.subscribe('/topic/public', onMessageReceived);
   //(Object) subscribe(destination, callback, headers = {})
   // 목적지 "/topic/public"을 구독
   
   // send(path, header, message)로 메세지를 보낼 수 있음
   stompClient.send("/app/chat/addUser", {}, JSON.stringify({sender: username, type: 'JOIN'}))
   //(void) send(destination, headers = {}, body = '')
   // 목적지 "/app/chat/adduser"로 메세지를 보낸다.
}
```
예를 들어 'app/chat/addUser'로 클라이언트가 SEND 프레임을 보내면

```
>>> SEND
destination:/app/chat/addUser
content-length:30

{"sender":"ggg","type":"JOIN"}
```
@Controller에서는 `/app` desination prefix를 제외한 경로 `/chat/addUser`를 @MessageMapping하면 된다.

<br>

예시 2)
```js
var chatMessage = {
   sender: username, message: messageInput.value, type: 'TALK'
};
stompClient.send("/app/chat/sendMessage", {}, JSON.stringify(chatMessage));
```

```
<<< MESSAGE
destination:/topic/public
content-type:application/json
subscription:sub-0
message-id:fdabeu5z-52
content-length:58

{"type":"TALK","sender":"ggg","message":"안녕하세요"}
```

<br><br><br><br>

### EventListener
```java
@RequiredArgsConstructor
@Component
public class WebSocketEventListener {
    private final SimpMessageSendingOperations sendingOperations;
    private static final Logger logger = LoggerFactory.getLogger(WebSocketEventListener.class);

    @EventListener
    public void handleWebSocketConnectListener(SessionConnectedEvent event) {
        logger.info("Received a new web socket connection  ");
    }

    @EventListener
    public void handleWebSocketDisconnectListener(SessionDisconnectEvent event) {
    StompHeaderAccessor headerAccessor = StompHeaderAccessor.wrap(event.getMessage());
    String username = (String) headerAccessor.getSessionAttributes().get("username");
    if (username != null) {
        logger.info("User Disconnected : " + username);
        ChatMessage chatMessage = new ChatMessage();
        chatMessage.setType(ChatMessage.MessageType.LEAVE);
        chatMessage.setSender(username);
        sendingOperations.convertAndSend("/topic/public", chatMessage);
        }
    }
}
```
event listner를 이용하여 소켓 연결(socket connect) 그리고 소켓 연결 끊기(disconnect) 이벤트를 수신하여

사용자가 채팅방을 참여(JOIN)하거나 떠날때(LEAVE)의 이벤트를 logging 하거나 broadcast 할 수 있다.

*`SimpMessageSendingOperations`와 `SimpMessagingTemplate` 둘 중 아무거나 써도 되는 듯 하다.          

→ 특정 Broker로 메세지를 전달(특정 사용자에게 메세지 전송)               

<br>

Server에서 Client로 특정 메시지를 BroadCast 해줘야 하는 상황에             

메시지를 보내고자 하는 위치에서 SimpMessagingTemplate 객체를 주입받아 와 같이 사용을 해주면 된다.

Client에서는 Topic을 subscribe 하고 있을 경우 Message를 받을 수 있게 된다.

<br><br><br><br>

### ChatMessage
```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class ChatMessage {
    public enum MessageType {
        JOIN, TALK, LEAVE
    }
    private MessageType type; // message type
    private String sender; // message 보내는 사람
    private String message; // 내용(message)
}
```

<br><br><br><br>

*reference*               
[Spring Websocket & STOMP](https://brunch.co.kr/@springboot/695#comment)                     
[WebSocket](https://velog.io/@koseungbin/WebSocket#sockjsclient)                     
[Spring WebSocket 소개](https://supawer0728.github.io/2018/03/30/spring-websocket/)                     
[[Spring Boot] WebSocket과 채팅 (3) - STOMP](https://dev-gorany.tistory.com/235)                     
[[Spring]Springboot + websocket 채팅[1]](https://ratseno.tistory.com/71)                     
[[Spring]Springboot + websocket 채팅[2]](https://ratseno.tistory.com/72)                     
[Spring websocket으로 간단 채팅 프로그램 만들기](https://rmcodestar.github.io/websocket/2019/02/11/spring-websocket/)                     
[[Spring] WebSocket 구현하](https://hyeooona825.tistory.com/89)                     
[[Spring Boot] WebSocket STOMP 사용시 BroadCast 메시지 전달 방법](https://jinseongsoft.tistory.com/252)                     
