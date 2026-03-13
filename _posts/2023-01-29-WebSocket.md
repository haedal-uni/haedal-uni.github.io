---
categories: Project Chat
tags: [spring, WebSocket, Chat]
---

# WebSocket

관련 글          
- [Websocket](https://haedal-uni.github.io/posts/WebSocket/)  👈🏻                               
- [Websocket + 부가기능](https://haedal-uni.github.io/posts/WebSocket-+%EB%B6%80%EA%B0%80%EA%B8%B0%EB%8A%A5/)                            
- [Websocket (채팅 기록 json 파일 저장하기)](https://haedal-uni.github.io/posts/WebSocket(%EC%B1%84%ED%8C%85-%EA%B8%B0%EB%A1%9D-Json-%ED%8C%8C%EC%9D%BC-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B8%B0)/)                               
- [Sse](https://haedal-uni.github.io/posts/SSE/)                                        
- [Websocket + jwt](https://haedal-uni.github.io/posts/WebSocket-+-JWT/)                         
- [Websocket test](https://haedal-uni.github.io/posts/WebSocket-Test/)                        
- [Jmh - 채팅 파일 refactoring](https://haedal-uni.github.io/posts/JMH/)           


<br><br>


## 기존에 작성했던 WebSocket

그동안 채팅 기능을 구현하기 위해 여러 글과 예제를 참고하며 코드를 작성했다.

이 과정에서 각각의 코드가 어떤 역할을 하는지 정확히 이해하지 못한 상태로 적용한 부분이 많았다.

이번에는 WebSocket을 처음부터 다시 공부하면서 구조와 개념을 정리하며 구현을 진행했다.

초기에는 어떤 글에서는 이 코드만 사용하고 다른 글에서는 전혀 다른 코드를 사용해서 혼란스러웠다.

내용이 길어져서 1편과 2편으로 나누어 작성했다.

<br><br><br>

### Handler 기반 WebSocket
WebSocket은 서버와 클라이언트 간의 양방향 통신을 가능하게 하는 프로토콜이다.

Spring에서는 WebSocket을 구현하기 위해 Handler 클래스를 사용할 수 있다.

Handler는 서버와 클라이언트 간 소켓 통신에서 사용할 메시지 처리 규칙을 정의한다.

WebSocket Handler를 구현할 경우 아래의 4가지 메서드를 오버라이드해야 한다.
- afterConnectionEstablished : WebSocket 연결이 성공했을 때 호출
- handleTextMessage : 메시지를 주고받을 때 호출
- afterConnectionClosed : 연결이 종료되었을 때 호출  
- handleTransportError : 통신 중 에러가 발생했을 때 호출

<br>

문자열 기반 메시지를 처리하기 위해 TextWebSocketHandler를 상속받아 사용한다.

```java
public class WebSocketHandler extends TextWebSocketHandler {
    private final Map<String, WebSocketSession> sessions = new ConcurrentHashMap<>();

    @Override
    public void afterConnectionEstablished(WebSocketSession session) {}

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) {}

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) {}

    @Override
    public void handleTransportError(WebSocketSession session, Throwable throwable) {}
}
```

<br><br><br>

### 어노테이션 기반 WebSocket (@OnOpen 방식)

WebSocket Handler는 `@OnOpen`, `@OnMessage`, `@OnClose`, `@OnError` 를 사용해서도 정의할 수 있다.  

WebSocketSession 대신 `javax.websocket.Session`을 사용한다.

```java
public class WebSocketHandler {
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
*나도 해당 방식을 사용했다.

<br><br>  

![image](https://user-images.githubusercontent.com/74857364/213742193-f8e7bc55-a971-452b-9999-abf619965594.png){: width="85%"}

WebSocketSession과 Session 모두 Closeable을 상속받고 있다.

WebSocketSession은 Spring에서 WebSocket 연결을 추상화한 세션 객체이다.


- afterConnectionEstablished (= `@OnOpen`)
- handleTextMessage (= `@OnMessage`)
- afterConnectionClosed (= `@OnClose`)
- handleTransportError (= `@OnError`) 

이 방식은 개발자 스타일에 따라 선택해서 사용하면 된다.

<br><br><br><br>  

## WebSocket 

WebSocket을 이해하기 위해서는 먼저 HTTP의 통신 방식을 이해해야 한다.

HTTP와 WebSocket은 모두 프로토콜이라는 공통점을 가진다.

브라우저와 서버는 HTTP를 통해 요청과 응답을 주고받는다.

브라우저가 요청을 보내면 서버는 그 요청에 대해서만 응답한다.

응답이 끝나면 서버와 브라우저의 연결은 종료된다.

서버는 브라우저의 요청 없이 먼저 데이터를 보낼 수 없다.

이러한 한계를 해결하기 위해 WebSocket이 등장했다.

<br><br><br>

### WebSocket의 특징

WebSocket은 단일 TCP 커넥션을 통해 서버와 클라이언트가 양방향 통신을 할 수 있는 프로토콜이다.

HTTP처럼 request-response 구조가 아니라 open-close 구조를 가진다.

연결이 열려 있는 동안 서버와 클라이언트는 자유롭게 메시지를 주고받을 수 있다.

전화 통화처럼 연결이 유지되는 동안 양쪽 모두 메시지를 보낼 수 있다.

<br>

Spring은 WebSocket API를 제공한다.

WebSocket 서버는 WebSocketHandler 인터페이스를 구현하여 각 경로에 대한 처리를 정의할 수 있다.

메시지 타입에 따라 TextWebSocketHandler 또는 BinaryWebSocketHandler를 상속해서 구현할 수 있다.

![image](https://user-images.githubusercontent.com/74857364/216780090-693af2df-3c03-41d6-bca4-9741cb8bb07e.png)

```java
public abstract class AbstractWebSocketHandler implements WebSocketHandler {
    public AbstractWebSocketHandler() {}
}
```

<br>

문자열 메시지를 사용하기 때문에 TextWebSocketHandler를 상속받는다.

```java
public class WebSocketHandler extends TextWebSocketHandler {}
```

<br><br><br><br>

## STOMP(Simple Text Oriented Messaging Protocol)

STOMP는 WebSocket 위에서 동작하는 메시징 프로토콜이다.  

클라이언트와 서버가 주고받는 메시지의 형식과 규칙을 정의한다.  

<br>

STOMP는 pub/sub 구조를 기반으로 동작한다.  

발행자(Publisher)는 메시지를 특정 수신자에게 직접 보내지 않는다.  

메시지는 브로커를 통해 전달되며 해당 Topic을 구독한 Subscriber만 메시지를 수신한다.  

<br>

채팅방에 비유하면 다음과 같다.  

채팅방 생성은 Topic 생성에 해당한다.  

채팅방 입장은 Topic 구독에 해당한다.  

채팅 메시지 전송은 Topic으로 메시지를 발행하는 것이다.  

<br>

pub/sub 구조는 비동기 메시징 패턴이다.  

발행자는 메시지를 보낸 후 결과를 기다리지 않고 자신의 작업을 계속 수행한다.  

수신자는 자신이 구독한 Topic의 메시지만 수신한다.  

<br>

Spring에서 STOMP를 사용하면 `@Controller` 기반으로 메시지를 처리할 수 있다.  

Simple In-Memory Broker를 사용하면 구독 정보를 메모리에 저장하고 메시지를 브로드캐스팅한다.  

Kafka, RabbitMQ, ActiveMQ 같은 외부 메시지 브로커를 연동할 수도 있다.  


<br><br><br>

### STOMP 메시지 구조

STOMP 메시지는 COMMAND, header, body로 구성된다.

```text
COMMAND
header:value

Body
```

COMMAND는 HTTP의 Method와 유사한 역할을 한다.

SEND는 메시지를 서버로 전송한다.

SUBSCRIBE는 특정 destination을 구독한다.

MESSAGE는 구독자에게 전달되는 메시지이다.

destination 헤더는 메시지가 전달될 경로나 구독 대상 경로를 의미한다.  

<br><br><br>

### STOMP와 Spring

Spring에서 STOMP를 사용하면 WebSocket 애플리케이션은 STOMP Broker처럼 동작한다.

메시지는 `@Controller`의 `@MessageMapping` 메서드로 라우팅된다.

또는 Simple In-Memory Broker를 통해 구독자들에게 브로드캐스트된다.

Redis, RabbitMQ, ActiveMQ 같은 외부 브로커도 사용할 수 있다.

<br>

이후에 Message Broker로 [Redis](https://haedal-uni.github.io/posts/Redis-pub&sub/) 를 적용하다가 [rabbitMQ]()로 변경했다.

<br>

WebSocketHandler를 직접 관리하는 방식보다 `@Controller` 기반 구조가 훨씬 조직적이다.  

<br><br><br>

### STOMP Subscribe 예시

```text
>>> SUBSCRIBE
id:sub-0
destination:/topic/public/3d41c3ed-8ddb-458d
```

<br>

roomId를 destination에 포함하지 않으면 모든 채팅방이 하나로 공유된다.

```text
>>> SUBSCRIBE
id:sub-0
destination:/topic/public
```

<br><br><br>

### 메시지 전송 흐름

**채팅방 입장**

```text
>>> SEND
destination:/app/chat/addUser
content-length:77

{"roomId":"3d41c3ed-8ddb-458d","sender":"ss","type":"JOIN"}
```

<br>

**채팅 메시지 전송**

```text
>>> SEND
destination:/app/chat/sendMessage
content-length:92

{"roomId":"3d41c3ed-8ddb-458d","sender":"ss","message":"hi","type":"TALK"}
```

<br>

broker를 통해 구독자에게 전달되는 메시지

```text
<<< MESSAGE
destination:/topic/public/3d41c3ed-8ddb-458d
content-type:application/json
subscription:sub-0

{"roomId":"3d41c3ed-8ddb-458d","type":"TALK","sender":"ss","message":"hi"}
```

<br><br><br>

## Message Broker와 Message Queue

Message Broker는 발신자로부터 받은 메시지를 적절한 수신자에게 전달하는 중간 컴포넌트이다.  

메시지가 적재되는 공간을 Message Queue라고 하고 메시지의 그룹을 Topic이라고 한다.  

대표적인 메시지 브로커로는 Kafka, Redis, RabbitMQ, ActiveMQ가 있다.  

실시간 처리에서는 DB 조회보다 메시지 브로커를 사용하는 것이 성능상 유리하다.  

다만 메시지는 필터링된 상태로 적재하거나 별도의 처리 로직이 필요하다.  

<br><br>

**내장 메세지 브로커를 사용한 경우 컴포넌트 구성**

![image](https://user-images.githubusercontent.com/74857364/215541274-d405c685-004f-47d7-b6ae-549f7403dc61.png)

[공식문서](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket)


<br><br><br>

### STOMP를 사용하는 이유

WebSocket은 통신 프로토콜일 뿐 메시지 라우팅 규칙을 제공하지 않는다.  

STOMP를 사용하면 메시지 타입을 정의하고 Controller 기반으로 처리할 수 있다.  

roomId를 destination에 포함시켜 채팅방 단위 메시지 처리를 쉽게 할 수 있다.  

<br><br><br><br>  

## Code
### Build
```
implementation 'org.springframework.boot:spring-boot-starter-websocket'
implementation 'org.webjars:sockjs-client:1.1.2' // 프론트단에서 사용하는 sockjs 라이브러리
implementation 'org.webjars:stomp-websocket:2.3.3-1'
implementation 'org.webjars.bower:bootstrap:4.3.1'
implementation 'org.webjars:jquery:3.5.1'
```

아래는 위 라이브러리를 설치해서 client에 적용한 코드이다.
```html
<link rel="stylesheet" href="/webjars/bootstrap/4.3.1/dist/css/bootstrap.min.css">
<script src="/webjars/jquery/jquery.min.js"></script>
<script src="/webjars/sockjs-client/sockjs.min.js"></script>
<script src="/webjars/stomp-websocket/stomp.min.js"></script>
<script src="/webjars/bootstrap/4.3.1/dist/js/bootstrap.min.js"></script>           
```

<br><br>

### WebSocket + STOMP Config

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

`@EnableWebSocketMessageBroker`는 STOMP 기반 WebSocket 서버를 활성화한다.  

<br><br>

endpoint 구성에 `withSockJS()`를 사용한다.

WebSocket은 HTML5 이후에 나왔기 때문에, HTML5 이전은 SockJS를 이용해서 

웹 소켓을 지원하지 않는 브라우저에 폴백 옵션을 활성화하는 데 사용된다.

*Fallback 이란? : 어떤 기능이 약해지거나 제대로 동작하지 않을 때, 이에 대처하는 기능 또는 동작

<br><br>

registerStompEndpoints는 클라이언트가 연결할 WebSocket 엔드포인트를 정의한다.  

```js
var socket = new SockJS('/ws');
```

<br><br>  

enableSimpleBroker는 메시지를 브로드캐스팅할 In-Memory Broker를 활성화한다.  

`/topic`, `/queue`로 시작하는 "destination" 헤더를 가진 메세지를 브로커로 라우팅

```
>>> SUBSCRIBE
id:sub-0
destination:/topic/public/3d41c3ed-8ddb-458d
```
<br><br>

setApplicationDestinationPrefixes는 `@MessageMapping`으로 라우팅될 경로 접두사를 지정한다.  

client가 메시지를 보낼 때 경로 맨앞에 `/app`이 붙어있으면 Broker로 보내진다.

client : 
```js
var socket = new SockJS('/ws');
stompClient = Stomp.over(socket);
stompClient.connect({}, onConnected, onError);

function onConnected() {
   stompClient.subscribe('/topic/public/'+ roomId, onMessageReceived);
   //(Object) subscribe(destination, callback, headers = {})  목적지 "/topic/public"을 구독
   
   // send(path, header, message)로 메세지를 보낼 수 있음
   stompClient.send("/app/chat/addUser", {}, JSON.stringify({roomId: roomId, sender: username, type: 'JOIN'}))
   //(void) send(destination, headers = {}, body = '') 목적지 "/app/chat/adduser"로 메세지를 보낸다.
}
```
예를 들어 'app/chat/addUser'로 클라이언트가 SEND 프레임을 보내면

```
>>> SEND
destination:/app/chat/addUser
content-length:77

{"roomId":"3d41c3ed-8ddb-458d","sender":"ss","type":"JOIN"}
```
`@Controller`에서는 `/app` desination prefix를 제외한 경로 `/chat/addUser`를 `@MessageMapping`하면 된다.

<br><br>
  
예시 2)
```js
var chatMessage = {
    roomId: roomId, sender: username, message: messageInput.value, type: 'TALK'
};
stompClient.send("/app/chat/sendMessage", {}, JSON.stringify(chatMessage));
```

```
<<< MESSAGE
destination:/topic/public/3d41c3ed-8ddb-458d
content-type:application/json
subscription:sub-0
message-id:1taozffc-1
content-length:92

{"roomId":"3d41c3ed-8ddb-458d","type":"TALK","sender":"ss","message":"hi"}
```

<br><br><br> 

### ChatController
WebSocketConfig에서 "/app"로 시작하는 대상이 있는 클라이언트에서 보낸 모든 메시지는 

`@MessageMapping`이 달린 메서드로 라우팅 된다.
```java
@Controller
@RequiredArgsConstructor
public class ChatController {
    private final SimpMessagingTemplate template;

    @MessageMapping("/chat/sendMessage")
    public void sendMessage(@Payload ChatMessage chatMessage) {
        template.convertAndSend("/topic/public/" + chatMessage.getRoomId(), chatMessage);
    }

    @MessageMapping("/chat/addUser")
    public void addUser(@Payload ChatMessage chatMessage, SimpMessageHeaderAccessor headerAccessor) {
        headerAccessor.getSessionAttributes().put("username", chatMessage.getSender());
        headerAccessor.getSessionAttributes().put("roomId", chatMessage.getRoomId());
        template.convertAndSend("/topic/public/" + chatMessage.getRoomId(), chatMessage);
    }
}
```

`@MessageMapping`은 클라이언트가 보낸 STOMP 메시지를 처리하는 메서드를 지정한다.  

convertAndSend는 객체를 Message로 변환하여 지정한 destination으로 전송한다.  

<br>

client : 
```js
stompClient.subscribe('/topic/public/'+ roomId, onMessageReceived);
stompClient.send("/app/chat/addUser", {}, JSON.stringify({roomId: roomId, sender: username, type: 'JOIN'}))
```

<br><br><br>

### EventListener

```java
@RequiredArgsConstructor
@Component
public class WebSocketEventListener {
    private final SimpMessageSendingOperations sendingOperations;

    @EventListener
    public void handleWebSocketDisconnectListener(SessionDisconnectEvent event) {
        StompHeaderAccessor headerAccessor = StompHeaderAccessor.wrap(event.getMessage());
        String username = (String) headerAccessor.getSessionAttributes().get("username");
        String roomId = (String) headerAccessor.getSessionAttributes().get("roomId");
        if (username != null) {
            ChatMessage chatMessage = new ChatMessage();
            chatMessage.setType(ChatMessage.MessageType.LEAVE);
            chatMessage.setSender(username);
            chatMessage.setRoomId(roomId);
            sendingOperations.convertAndSend("/topic/public/" + roomId, chatMessage);
        }
    }
}
```

EventListener를 사용하면 WebSocket 연결 및 종료 이벤트를 감지할 수 있다.  

사용자가 퇴장할 때 LEAVE 메시지를 브로드캐스팅할 수 있다.  

<br>

*SimpMessagingTemplate와 SimpMessageSendingOperations은 모두 Spring의 WebSocket 메시징을 처리하는 인터페이스

메시지를 보내고자 하는 위치에서 SimpMessagingTemplate 객체를 주입받아 위와 같이 사용을 해주면 된다.

<br><br><br>

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
    private String roomId;
    private MessageType type;
    private String sender;
    private String message;
}
```

MessageType에 따라 입장, 채팅, 퇴장 메시지를 구분하여 처리한다. 


<br><br><br><br>

## 정리

WebSocket은 실시간 양방향 통신을 위한 프로토콜이다.

SockJS는 WebSocket을 지원하지 않는 환경을 위한 fallback 옵션이다.

STOMP는 WebSocket 위에서 메시징 구조를 단순화해준다.

STOMP를 사용하면 세션을 직접 관리하지 않고 pub/sub 구조로 메시지를 처리할 수 있다.

Message Broker를 사용하면 서버 부하를 분산시킬 수 있다.


<br><br><br><br>

*reference*                                    
[WebSocket](https://velog.io/@koseungbin/WebSocket)        
[Spring WebSocket 소개](https://supawer0728.github.io/2018/03/30/spring-websocket/)                           
[[Tech.] Message Broker란?](https://heodolf.tistory.com/49)                      
