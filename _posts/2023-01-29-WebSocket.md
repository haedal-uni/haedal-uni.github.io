---
categories: Project
tags: [spring, WebSocket]
---

# WebSocket

## 기존에 작성했던 webSocket
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

### Handler

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

![image](https://user-images.githubusercontent.com/74857364/213742193-f8e7bc55-a971-452b-9999-abf619965594.png){: width="85%"}

<br><br><br>

이번엔 `@OnClose`와 handleTransportError에 대해서 살펴봤다. 

![image](https://user-images.githubusercontent.com/74857364/213743192-29403c2e-7504-4c94-bcb5-2aafc886d3b1.png){: width="50%"}

`@OnClose`와 session의 패키지가 같아서 session을 쓰면 @OnClose를 쓰는건가? 싶었다.

<br><br>

![image](https://user-images.githubusercontent.com/74857364/213743352-fb6a5b41-8e1c-41a2-8983-2e1afe63d102.png){: width="70%"}

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

## 이론 정리
### WebSocket이란?
Websocket을 이해하려면 먼저 http를 알아야한다.

![image](https://user-images.githubusercontent.com/74857364/216779693-d4c84460-422f-43f7-962d-a3c928a1f9ba.png)

*HTTP와 WebSocket이 같은 이미지에 있는 이유는 둘 다 protocol이기 때문이다. 

browser와 server는 http를 이용해서 소통할 수 있다. (인터넷 데이터 교환에서 필수 요소)

<br>

브라우저는 서버에서 http request(요청)을 보낸다.

서버는 해당 request를 보고 브라우저가 홈페이지 정보를 요구하는걸 확인 후 서버는 http response를 브라우저에 보낸다.

<br>

👧🏻(user) → http 요청 [데이터 보내줘] →  🖥️(서버)

👧🏻(user) ← 데이터 ← 🖥️(서버)

<br>

⭐ 서버가 브라우저의 요청(request)에 응답(response)하고 나면 브라우저 - 서버 간 통신은 끝나게 된다.

서버가 브라우저에게 데이터를 보낼 수 있는 것은 브라우저가 요청을 했을 때 뿐이다. 이 때문에 웹 소켓이 생겨났다. 

<br><br>

**WebSocket**

WebSocket 프로토콜은 서버-클라이언트 간에 단일 TCP 커넥션을 이용해서 양방향 통신을 제공한다.

request - response가 있는 것이 아니라 커넥션이 open - close 된 것이다.

ex) 전화 통화는 양방향이므로 나도 메세지를 보내고 받을 수 있고 상대방도 똑같이 할 수 있다.

또한 둘이 전화를 끊기 전까지는 전화 통화는 열려있다.

<br>

Spring Framework는 WebSocket API를 제공한다.

WebSocket 서버는 WebSocketHandler 인터페이스의 구현체를 통해서, 각 경로에 대한 핸들러를 구현할 수 있다.

뿐만 아니라, Message 형식에 따라 TextWebSocketHandler or BinaryWebSocketHandler 핸들러를 확장해 구현할 수도 있다.

![image](https://user-images.githubusercontent.com/74857364/216780090-693af2df-3c03-41d6-bca4-9741cb8bb07e.png)

```java
public abstract class AbstractoWebSocketHandler implements WebSocketHandler{
    public AbstractWebSocketHandler(){}
}
```
<br><br>

문자열 메시지 기반으로 진행하기 때문에 TextWebSocketHandler를 상속받아 메시지를 전달받는다.
```java
public class WebSocketHandler extends TextWebSocketHandler{}
```

위에서 처음에 작성했던 코드가 이에 해당한다.

<br><br><br>

### STOMP(Simple Text Oriented Messaging Protocol)
WebSocket 프로토콜은 두 가지 유형의 메시지를 정의하고 있지만, 그 메시지의 내용까지는 정의하고 있지 않다.

STOMP은 WebSocket 위에서 동작하는 프로토콜로써, 클라이언트와 서버가 전송할 메시지 유형, 형식, 내용들을 정의한다.

STOMP는 텍스트 지향 프로토콜이지만 Message Payload에는 Text 또는 Binary 데이터를 포함할 수도 있다.

STOMP는 TCP 또는 WebSocket 같은 양방향 네트워크 프로토콜 기반으로 동작한다.

STOMP (Simple Text Oriented Messaging Protocol)은 메세징 전송을 효율적으로 하기 위해 탄생한 프로토콜이고, 

기본적으로 pub / sub 구조로 되어있어 메세지를 전송하고 메세지를 받아 처리하는 부분이 확실히 정해져 있기 때문에 

개발자 입장에서 명확하게 인지하고 개발할 수 있는 이점이 있다. 

<br><br>

➡️ STOMP 프로토콜은 WebSocket 위에서 동작하는 프로토콜로써 

클라이언트와 서버가 전송할 메세지의 유형, 형식, 내용들을 정의하는 매커니즘이다.

또한 STOMP를 이용하면 메세지의 헤더에 값을 줄 수 있어 헤더 값을 기반으로 통신 시 인증 처리를 구현하는 것도 가능하다.

<br><br>

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

스프링에서 지원하는 STOMP을 사용하게 된다면, 스프링 WebSocket 애플리케이션은 STOMP Broker로 동작한다.

메시지를 @Controller의 메시지 핸들링하는 메서드로 라우팅하거나, 

Simple In-Memory Broker를 이용해서 Subscribe중인 다른 클라이언트들에게 메시지를 브로드캐스팅한다. 

Simple In-Memory Broker는 클라이언트의 Subscribe 정보를 자체적으로 메모리에 유지한다.

또한 RabbitMQ, ActiveMQ 같은 Message Broker를 이용해, Subscription(구독)을 관리하고 메세지를 브로드캐스팅할 수 있다.

<br>

👉🏻 WebSocket 기반으로 각 Connection(연결)마다 WebSocketHandler를 구현하는 것 보다 

`@Controller` 된 객체를 이용해 조직적으로 관리할 수 있다.

<br><br>

SUBSCRIBE `/topic/public/3d41c3ed-8ddb-458d`

```
>>> SUBSCRIBE
id:sub-0
destination:/topic/public/3d41c3ed-8ddb-458d
```
roomId가 3d41c3ed-8ddb-458d로 되어있는 형태인데 

만약 아래와 같이 roomId로 차이점을 주지않았다면 

채팅방이 여러 개를 만들 수 있어도 서로 공유하고 있는 채팅방이 될 것이다.

```
>>> SUBSCRIBE
id:sub-0
destination:/topic/public
```
<br><br>


**채팅방 입장**

```
>>> SEND
destination:/app/chat/addUser
content-length:77

{"roomId":"3d41c3ed-8ddb-458d","sender":"ss","type":"JOIN"}
```
<br><br>

**message**

SEND `app/chat/addUser`

```
>>> SEND
destination:/app/chat/sendMessage
content-length:92

{"roomId":"3d41c3ed-8ddb-458d","sender":"ss","message":"hi","type":"TALK"}
```
<br>

MESSAGE `/topic/public/3d41c3ed-8ddb-458d`

```
<<< MESSAGE
destination:/topic/public/3d41c3ed-8ddb-458d
content-type:application/json
subscription:sub-0
message-id:1taozffc-1
content-length:92

{"roomId":"3d41c3ed-8ddb-458d","type":"TALK","sender":"ss","message":"hi"}
```


<br><br>


**퇴장**
```
<<< MESSAGE
destination:/topic/public/3d41c3ed-8ddb-458d
content-type:application/json
subscription:sub-0
message-id:b0lxbjax-3
content-length:93

{"roomId":"3d41c3ed-8ddb-458d","type":"LEAVE","sender":"ss","message":null}
```

<br><br>


즉, 메세지는 STOMP의 "destination" 헤더를 기반으로 @Controller 객체의 `@MethodMapping` 메서드로 라우팅 된다.

STOMP의 "destination" 및 Message Type을 기반으로 메세지를 보호하기 위해 Spring Security를 사용할 수 있다.

<br><br>


**내장 메세지 브로커를 사용한 경우 컴포넌트 구성**

![image](https://user-images.githubusercontent.com/74857364/215541274-d405c685-004f-47d7-b6ae-549f7403dc61.png)

[공식문서](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html#websocket)


<br><br>

### STOM를 사용하는 이유
WebSocket은 통신 프로토콜 일뿐이다. 
 
특정 주제를 구독한 사용자에게만 메시지를 보내는 방법 또는 특정 사용자에게 메시지를 보내는 방법과 같은 내용은 정의하지 않는다. 
 
이러한 기능을 위해서는 STOMP가 필요하다.

<br><br><br>

### Message Broker란

Message Broker(메시지 브로커)는 Publisher(송신자)로부터 전달받은 메시지를 Subscriber(수신자)로 전달해주는 중간 역할이며 

응용 소프트웨어 간에 메시지를 교환할 수 있게 한다. 

이 때 메시지가 적재되는 공간을 Message Queue(메세지 큐)라고 하며 메시지의 그룹을 Topic(토픽)이라고 한다.

![image](https://user-images.githubusercontent.com/74857364/215319457-0adc8930-5c0f-41d6-beda-4430656b3c46.png)

메시지 브로커는 송신자가 보낸 메시지를 메시지 큐에 적재하고 이를 수신자가 받아서 사용하는 구조이다. 

이러한 구조를 Pulibsh/Subscribe(pub/sub) Pattern이라고 하며, Producer/Consumer Pattern 이라고도 한다.

메시지 브로커는 대표적으로 Apache Kafka, Redis, RabbitMQ, Celery 등이 있다. 

실시간 데이터를 처리할 때 DB에서 조회하는 것보다 메시지 브로커를 이용하여 처리하는 것이 성능이 뛰어나다는 것을 알 수 있는데 단점도 존재한다. 

DB를 사용하는 경우 Query를 이용하여 원하는 데이터만 필터링하여 조회할 수 있지만, 

메시지 브로커를 이용하면 Queue에 적재된 그대로 사용하기 때문에 불가능하다. 

따라서, 적재할 때 필터링된 데이터를 적재하던가 적재된 데이터를 Logstash를 이용하여 필터링해서 사용해야 한다. 

또한, 메시지 큐에 적재된 메시지는 주로 7일을 보관하기 때문에 장기간 보관해야하는 경우 별도의 저장소에 저장해야한다.

<br><br><br>

### 메세지 큐란?
메시지 지향 미들웨어(Message Oriented Middleware: MOM)는 

비동기 메시지를 사용하는 다른 응용프로그램 사이의 데이터 송수신을 의미하는데 

MOM을 구현한 시스템을 메시지큐(Message Queue:MQ)라 한다.

<br>

MOM(Message Oriented Middleware)를 구현한 솔루션으로 비동기 메시지를 사용하는 서비스들 사이에서 데이터를 교환해주는 역할을 한다.

Producer(sender)가 메시지를 큐에 전송하면 Consumer(receiver) 가 처리하는 방식으로, 

producer 와 consumer 에 message 프로세스가 추가되는 것이 특징이다.

MQ를 사용하면 메시지를 비동기로 요청을 처리하고 queue 에 저장하여 consumer 에게 병목을 줄여줄 수 있는 장점이 있다.

→ kafka, Apache ActiveMQ, rabbitMQ

<br><br>

#### kafka
분산형 스트리밍 플랫폼(A distributed streaming platform)이다.

(발행/구독: pub-sub은 메시지를 특정 수신자에게 직접적으로 보내주는 시스템이 아니고,

메시지를 받기를 원하는 사람이 해당 토픽(topic)을 구독함으로써 메시지를 읽어 올 수 있다.)

<br>

대용량 실시간 로그처리에 특화되어 설계된 메시징 시스템으로 

메시지를 메모리에 저장하는 기존 메시징 시스템과는 달리 파일에 저장을 하는데 

그로 인해 카프카를 재시작해도 메시지 유실 우려가 감소된다.

<br>

기본 메시징 시스템(rabbitMQ, ActiveMQ)에서는 브로커(Broker)가 컨슈머(consumer)에게 메시지를 push해 주는 방식인데, 

카프카는 컨슈머(Consumer)가 브로커(Broker)로부터 메시지를 직접 가져가는 PULL 방식으로 동작하기 때문에 

컨슈머는 자신의 처리 능력만큼의 메시지만 가져와 최적의 성능을 낼 수 있다.

![image](https://user-images.githubusercontent.com/74857364/216780860-296a119c-f43b-4659-9f64-2694918d9cc6.png)

<br><br>

#### Apache ActiveMQ
ActiveMQ는 JMS를 지원하는 클라이언트를 포함하는 브로커, 자바 뿐만 아니라 다양한 언어를이용하는시스템간의 통신을 할 수 있게 해준다.

클라이언트 간 메시지를 송수신 할 수 있는 오픈 소스 Broker(JMS 서버)다. 

<br>

**JMS란**

JMS 는 자바 기반의 MOM(메시지 지향 미들웨어) API 이며 둘 이상의 클라이언트 간의 메시지를 보낸다.

핵심 개념은 Message Broker 와 Destination 이다.  

- Message Broker : 목적지에 안전하게 메시지를 건네주는 중개자 역할.

- Destination: 목적지에 배달될 2가지 메시지 모델 QUEUE, TOPIC.

- Queue: Point to Point ( Consumer 는 메시지를 받기 위해 경쟁한다.)

- Topic: Publish to Subscribe.

<br><br>

**ActiveMQ 메세지 처리 구조**

기본적으로 Message를 생산하는 Producer, activeMQ Broker(Server), Message를 소비하는 Consumer로  구성되어 있다.

```
Producer → Broker → Consumer
```
Producer(생산자)가 message를 Queue/Topic에 넣어두면 Consumer가 message를 가져와 처리하는 방식


<br>

![image](https://user-images.githubusercontent.com/74857364/216781129-281687b2-1c4f-437e-b458-8e809cfe6a86.png)

- QUEUE 모델의 경우 메시지를 받는 Consumer가 다수일 때 연결된 순서로 메시지는 제공된다.

- TOPIC 모델의 경우 메시지를 받는 Consumer가 다수일 때 메시지는 모두에게 제공된다.

<br><br>

#### rabbitMQ
오픈소스 AMQP 브로커다.

JMS는 API, AMQP는 프로토콜이다. JMS는 메시지의 형식이 아닌 브로커와 통신하는 방법을 정의한다. 

또한 자바 애플리케이션에만 국한돼 있다.

AMQP는 브로커와 통신하는 방법에 대해서 논하지 않지만 메시지가 유선을 통해 큐에 어떻게 넣고 꺼내지는지에 대해 정의한다.

서로 다른 두 가지 애플리케이션이 있을 때,           
둘 다 자바면 JVMS를 통해 통신할 수 있지만 이중에 하나가 루비라면 JMS는 사용하지 못할것이다.

<br><br><br>

### 정리

**Websocket**

페이지의 refresh 없이 나 또는 다른 사람이 보낸 채팅을 받을 수 있어야 한다.

즉, 연결이 끊기지 않아야 한다.

<br>

**SockJS**

브라우저에서 Websocket을 지원하지 않거나, 네트워크 Proxy 제약 등으로 인한 Websocket을 사용할 수 없을 경우

fallback option을 제공하는데, 이는 SockJS Protocol에 기반으로 Websocket API를 사용할 수 있도록 한다.

<br>

**STOMP**

웹소켓만 사용했을 땐 직접 세션을 관리해서, 해당 세션으로 채팅 데이터를 전송해야했다면, 

STOMP를 사용함으로써 publish/subscribe (발행/구독) 구조로 간단하게 메세지를 선택적으로 수신할 수 있다.

<br><br><br>

## Code
[공식문서](https://spring.io/guides/gs/messaging-stomp-websocket/)

<br>

### ChatController
WebSocketConfig에서 "/app"로 시작하는 대상이 있는 클라이언트에서 보낸 모든 메시지는 

`@MessageMapping` 어노테이션이 달린 메서드로 라우팅 된다.

아래는 초반에 작성했던 코드이다. 
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

<br><br><br>

이후에 url에 roomId를 추가하면서 코드를 변경했다.
```java
@Controller
@RequiredArgsConstructor
public class ChatController {
    private final SimpMessagingTemplate template;

    @MessageMapping("/chat/sendMessage")
    public void sendMessage(@Payload ChatMessage chatMessage) {
        template.convertAndSend("/topic/public/"+ chatMessage.getRoomId(), chatMessage);
    }

    @MessageMapping("/chat/addUser")
    public void addUser(@Payload ChatMessage chatMessage, SimpMessageHeaderAccessor headerAccessor){
        headerAccessor.getSessionAttributes().put("username", chatMessage.getSender());
        headerAccessor.getSessionAttributes().put("roomId", chatMessage.getRoomId());
        template.convertAndSend("/topic/public/" + chatMessage.getRoomId(), chatMessage);
    }
}
```

client : 
```js
stompClient.subscribe('/topic/public/'+ roomId, onMessageReceived);
stompClient.send("/app/chat/addUser", {}, JSON.stringify({roomId: roomId, sender: username, type: 'JOIN'}))
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

```js
var socket = new SockJS('/ws');
```
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
destination:/topic/public/3d41c3ed-8ddb-458d
```

메시지 브로커는 특정 주제를 구독 한 연결된 모든 클라이언트에게 메시지를 broadcast 한다.

*브로드캐스팅은 송신 호스트가 전송한 데이터가 네트워크에 연결된 모든 호스트에 전송되는 방식을 의미

<br>

지금은 STOMP가 갖고있는(내장하고 있는) SimpleBroker 를 사용해 간단한 인 메모리 메시지 브로커를 활성화했다.

만일 이용자 수가 증가하여 처리해야하는 데이터가 많아진다면,      

내장되어있는 SimpleBroker는 철저하게 Spring Boot가 실행되는 (정확하게는 채팅 서버) 곳의 메모리를 잡아먹는다. 

따라서 다른 많은 비즈니스 로직과 채팅에 대한 부담까지 '하나의 서버'가 떠안게 된다.           

→ RabbitMQ 또는 ActiveMQ와 같은 다른 모든 기능을 갖춘 메시지 브로커를 사용하게 되면 채팅 관리 따로 빼서 서버의 부담을 줄일 수 있다.   

<br><br>

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
   stompClient.subscribe('/topic/public/'+ roomId, onMessageReceived);
   //(Object) subscribe(destination, callback, headers = {})
   // 목적지 "/topic/public"을 구독
   
   // send(path, header, message)로 메세지를 보낼 수 있음
   stompClient.send("/app/chat/addUser", {}, JSON.stringify({roomId: roomId, sender: username, type: 'JOIN'}))
   //(void) send(destination, headers = {}, body = '')
   // 목적지 "/app/chat/adduser"로 메세지를 보낸다.
}
```
예를 들어 'app/chat/addUser'로 클라이언트가 SEND 프레임을 보내면

```
>>> SEND
destination:/app/chat/addUser
content-length:77

{"roomId":"3d41c3ed-8ddb-458d","sender":"ss","type":"JOIN"}
```
@Controller에서는 `/app` desination prefix를 제외한 경로 `/chat/addUser`를 @MessageMapping하면 된다.

<br>

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
    String roomId = (String) headerAccessor.getSessionAttributes().get("roomId");
    if (username != null) {
        logger.info("User Disconnected : " + username);
        ChatMessage chatMessage = new ChatMessage();
        chatMessage.setType(ChatMessage.MessageType.LEAVE);
        chatMessage.setSender(username);
        chatMessage.setRoomId(roomId);
        sendingOperations.convertAndSend("/topic/public/" + roomId, chatMessage);
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

메시지를 보내고자 하는 위치에서 SimpMessagingTemplate 객체를 주입받아 위와 같이 사용을 해주면 된다.

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
    private String roomId; // 채팅방 아이디
    private MessageType type; // message type
    private String sender; // message 보내는 사람
    private String message; // 내용(message)
}
```

MessageType에 따라서 채팅형식이 달라지게 설정했다.

<br><br><br><br>

*reference*               
[Spring Websocket & STOMP](https://brunch.co.kr/@springboot/695#comment)                     
[WebSocket](https://velog.io/@koseungbin/WebSocket)        
[Spring WebSocket 소개](https://supawer0728.github.io/2018/03/30/spring-websocket/)                     
[[Spring Boot] WebSocket과 채팅 (3) - STOMP](https://dev-gorany.tistory.com/235)                     
[[Spring]Springboot + websocket 채팅[1]](https://ratseno.tistory.com/71)                     
[[Spring]Springboot + websocket 채팅[2]](https://ratseno.tistory.com/72)                     
[Spring websocket으로 간단 채팅 프로그램 만들기](https://rmcodestar.github.io/websocket/2019/02/11/spring-websocket/)                     
[[Spring] WebSocket 구현하기](https://hyeooona825.tistory.com/89)                      
[[Spring Boot] WebSocket STOMP 사용시 BroadCast 메시지 전달 방법](https://jinseongsoft.tistory.com/252)                       
[[Tech.] Message Broker란?](https://heodolf.tistory.com/49)                            
[[Spring Boot] WebSocket과 채팅 (4) - RabbitMQ](https://dev-gorany.tistory.com/325)       
[HTTP vs WebSocket 차이점](https://kbj96.tistory.com/46)      
[[메시지 지향 미들웨어:MOM] ActiveMQ, rabbitMQ, Kafka](https://gwonbookcase.tistory.com/49)         

Youtube           
[오늘의 테크용어 : 웹소켓이 뭐냐면](https://www.youtube.com/watch?v=yXPCg5eupGM)             
[WebRTC? WebSockets? 5분 개념정리!](https://www.youtube.com/watch?v=5EhsjtBE7I4)       
          
