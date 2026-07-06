---
categories: Project Redis
tags: [spring, Redis, Chat]
---

# Spring의 내장 브로커에서 Redis 브로커로 마이그레이션

내장 브로커에서 Redis 브로커로 전환한 이유는 크게 세 가지다.

<br>

**1. 확장성 및 메시지 처리량 향상**

Redis는 메모리 기반 데이터베이스로 데이터를 빠르게 읽고 쓸 수 있는 특징을 가지고 있다.

낮은 지연 시간을 제공하며 높은 메시지 처리량을 가능하게 해 더 많은 사용자나 요청을 처리할 수 있다.

<br>

**2. 간단한 설정과 운영**

Redis는 간단한 설정을 가지고 있어 초기 구축 및 운영이 간편하다.

<br>

**3. Pub/Sub 패턴 지원**

Redis는 Pub/Sub 패턴을 지원하여 다중 구독자가 하나의 메시지를 동시에 수신할 수 있도록 해준다.

이를 통해 실시간 이벤트 처리를 용이하게 구현할 수 있다.

<br><br>

> Redis Pub/Sub은 메시지 브로커 시스템으로
> 
> 다른 서비스나 애플리케이션 간에 이벤트 기반 통신을 단순화하기 위해 사용된다.
>
> WebSocket은 클라이언트와 서버 간에 실시간 양방향 통신을 제공하는 프로토콜이다.

<br><br>

## Redis pub/sub

Pub/Sub은 발행/구독 모델(Publish/Subscribe)을 기반으로 한 통신 방법으로  

topic(or channel)에 대해 관심 있는 구독자(Subscriber)들에게 메시지를 발행(Publish)하는 방식이다.

Subscriber는 여러 개의 채널을 구독할 수 있다. (유튜브 채널 구독과 비슷한 개념)

Publisher가 채널에 메시지를 보내면 그 채널을 구독하고 있는 Subscriber들에게 메시지가 전달된다.

메시지를 보낼 때 Publisher는 채널에 어떤 Subscriber가 있는지 알 필요가 없고 그냥 메시지를 전송만 하면 된다.

Publish/Subscribe 구조에서 사용되는 Queue를 일반적으로 Topic이라고 한다.

<br><br>

### 동작 과정

1. 발행자가 특정 주제를 선택하여 메시지를 생성하고 해당 topic의 채널에 메시지를 발행한다.

2. 해당 topic을 구독한 모든 구독자들은 채널을 구독하고 있기 때문에 메시지를 받을 준비가 되어있다.

3. 발행된 메시지는 해당 topic의 채널로 전달되며 구독자들은 채널에서 메시지를 수신한다.

4. 구독자들은 자신들이 관심을 갖는 특정 topic을 구독하므로 해당 topic에 관련된 메시지만을 수신하게 된다.

5. 새로운 구독자가 특정 topic을 구독하면 이후 발행되는 해당 topic의 메시지를 받게 된다.

<br><br><br>

## code

기존에 설정했던 내장 브로커는 제거했다.

```java
public class StompWebSocketConfig implements WebSocketMessageBrokerConfigurer {
    @Override
    public void configureMessageBroker(MessageBrokerRegistry config) {
        //config.enableSimpleBroker("/topic"); // sub
        config.setApplicationDestinationPrefixes("/app"); // pub
    }
}
```

<br><br>

### build.gradle

```
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

<br><br><br>

### RedisConfig

*추가된 코드만 작성

```java
@Configuration
public class RedisConfig {
    @Value("${spring.redis.host}")
    private String redisHost;
    @Value("${spring.redis.port}")
    private int redisPort;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(redisHost, redisPort);
    }

    @Bean
    public RedisTemplate<?, ?> redisTemplate() {
        RedisTemplate<?, ?> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory());
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new StringRedisSerializer());
        return redisTemplate;
    }

    @Bean
    public RedisTemplate<String, ChatMessage> chatMessageRedisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, ChatMessage> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return redisTemplate;
    }

    @Bean
    public RedisMessageListenerContainer redisMessageListenerContainer(RedisConnectionFactory connectionFactory) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory);
        return container;
    }
}
```

각 Bean의 역할은 다음과 같다.

- `redisConnectionFactory()` : Redis 서버와의 연결을 생성한다.

- `redisTemplate()` : Redis 데이터에 쉽게 접근하기 위한 기본 설정이다.

- `chatMessageRedisTemplate()` : ChatMessage 타입의 메시지를 직렬화하여 Redis에 저장하기 위한 설정이다.

- `redisMessageListenerContainer()` : Redis를 브로커로 사용하기 위한 핵심 설정이다.
  
  pub/sub은 항상 Redis에 발행된 데이터가 있는지 확인하고 있어야 하기 때문에
  
  이 컨테이너에 Listener를 등록해야 한다.

<br><br><br>

### Publisher(발행자)

```java
@Service
@RequiredArgsConstructor
public class RedisPublisher {

    private final RedisTemplate<String, ChatMessage> redisTemplate;

    public void publish(ChannelTopic topic, ChatMessage message) {
        redisTemplate.convertAndSend(topic.getTopic(), message);
    }
}
```

기존에는 `@MessageMapping`으로 메시지를 받는 메소드에서  

바로 `redisTemplate.convertAndSend()`를 호출했는데  

RedisPublisher 클래스를 분리하여 `@MessageMapping`에서 `publish()`를 호출하는 방식으로 변경했다.

역할을 분리하면 Publisher 로직을 독립적으로 테스트하거나 수정할 수 있어 유지보수가 편해진다.

<br><br><br>

### Subscriber(구독자)

```java
@Component
@RequiredArgsConstructor
public class RedisSubscriber implements MessageListener {
    private final ObjectMapper objectMapper;
    private final RedisTemplate<String, ChatMessage> redisTemplate;
    private final SimpMessageSendingOperations messagingTemplate;

    @Override
    public void onMessage(Message message, byte[] pattern) {
        try{
            String channel = new String(message.getChannel());
            String msg = redisTemplate.getStringSerializer().deserialize(message.getBody());
            ChatMessage chatMessage = objectMapper.readValue(msg, ChatMessage.class);
            log.info("chatMessage : " + chatMessage);

            if(chatMessage.getType().equals(MessageType.TALK)){
                messagingTemplate.convertAndSend(channel, chatMessage);
            }
        } catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

`onMessage()`는 Redis에서 메시지가 발행되면 자동으로 호출되는 메소드다.

메시지는 json 형태로 수신되기 때문에  

`objectMapper.readValue()`로 ChatMessage 객체로 역직렬화한 뒤 처리한다.

`messagingTemplate.convertAndSend()`를 통해 WebSocket 클라이언트에게 메시지를 전달한다.

> Controller에서 `SimpMessagingTemplate`으로 직접 보낼 수도 있다. (아래 코드 참고)
> 
> *client에 msg를 보낼 수 있고 Controller에서 따로 보낼 수도 있다.

<br><br>

#### 특정 채널의 구독자 수 조회

구독이 제대로 되었는지 확인하기 위해 작성했다.

```
implementation 'redis.clients:jedis:3.6.3'
```

```java
import redis.clients.jedis.Jedis;

try (Jedis jedis = new Jedis("localhost")) {
    Map<String, String> stringStringMap = jedis.pubsubNumSub(channel);
    log.info("구독자 수: " + stringStringMap);
}
```

<br><br><br>

### Controller

*redis와 관련 없는 코드들은 최대한 제거함

```java
@Controller
@RequiredArgsConstructor
@Slf4j
public class ChatController {
    private final SimpMessagingTemplate template;
    private final RedisMessageListenerContainer redisMessageListener;
    private final RedisPublisher redisPublisher;
    private final RedisSubscriber redisSubscriber;
    private Map<String, ChannelTopic> channels;

    @PostConstruct
    public void init(){
        channels = new HashMap<>();
    }

    @MessageMapping("/chat/sendMessage")
    public void sendMessage(@Payload ChatMessage chatMessage) {
        ChannelTopic channel = channels.get(chatMessage.getRoomId());
        redisPublisher.publish(channel, chatMessage.getMessage());
    }

    @MessageMapping("/chat/addUser")
    public void addUser(@Payload ChatMessage chatMessage, SimpMessageHeaderAccessor headerAccessor) {
        String roomId = chatMessage.getRoomId();
        ChannelTopic channel = new ChannelTopic(roomId);
        redisMessageListener.addMessageListener(redisSubscriber, channel);
        channels.put("/topic/public/"+ roomId, channel);

        chatMessage.setSender(user.getNickname());
        chatMessage.setType(ChatMessage.MessageType.JOIN);
        template.convertAndSend("/topic/public/" + roomId, chatMessage);
    }

    @MessageMapping("/disconnect")
    public void disConnect(@Payload DisconnectPayload disconnectPayload){
        String roomId = disconnectPayload.getRoomId();
        ChannelTopic channel = channels.get(roomId);
        redisMessageListener.removeMessageListener(redisSubscriber, channel);
        channels.remove(roomId);

        ChatMessage chatMessage = new ChatMessage();
        chatMessage.setType(ChatMessage.MessageType.LEAVE);
        chatMessage.setRoomId(roomId);
        template.convertAndSend("/topic/public/" + roomId, chatMessage);
    }
}
```

흐름을 정리하면 다음과 같다.

- `addUser()` : 채팅방 입장 시 해당 roomId로 ChannelTopic을 생성하고
  
  `redisMessageListener`에 구독자를 등록한다.

- `sendMessage()` : 메시지 전송 시 `redisPublisher.publish()`를 통해 Redis에 발행한다.

- `disConnect()` : 채팅방 퇴장 시 해당 채널의 구독자를 제거한다.

<br><br>

`RedisTemplate`과 `SimpMessagingTemplate`의 역할 차이는 다음과 같다.

- `RedisTemplate` : Redis 서버와 직접 상호작용하며 메시지를 Redis에 저장하고 발행한다.

- `SimpMessagingTemplate` : STOMP를 통해 WebSocket 클라이언트에게 메시지를 전송한다.

즉 Redis에 메시지를 저장하는 것과  

WebSocket 클라이언트에게 메시지를 전송하는 역할이 분리되어 있다.

<br><br><br>

### client에서 받기

client에서 제대로 값을 받는지 확인하기 위해 작성했다.

```js
let socket = new SockJS('/ws');

socket.onmessage = (event) => {
    const message = event.data;
    console.log('받은 메시지:', message)
    onMessageReceived(message)
};
```

data 타입은 String이고 ChatMessage 형식으로 보냈을 때 아래와 같이 전달받았다.

```
MESSAGE
destination:/topic/public/123
content-type:application/json
subscription:sub-0
message-id:wrqzxuoi-10
content-length:100

{"roomId":"123","type":"TALK","sender":"hi123","message":"hihi"}
```
