---
categories: Project
tags: [spring, Redis, Cache]
---

## Redis pub/sub
Pub/Sub은 발행/구독 모델(Publish/Subscribe)을 기반으로 한 통신 방법으로, 

특정 주제(topic)에 대해 관심 있는 구독자(Subscriber)들에게 메시지를 발행(Publish)하는 방식이다.

<br><br>

### 동작 과정
**1.** 발행자가 특정 주제를 선택하여 메시지를 생성하고, 해당 topic의 채널에 메시지를 발행한다.

**2.** 해당 topic을 구독한 모든 구독자들은 채널을 구독하고 있기 때문에 메시지를 받을 준비가 되어있다.

**3.** 발행된 메시지는 해당 topic의 채널로 전달되며, 구독자들은 채널에서 메시지를 수신한다.

**4.** 구독자들은 자신들이 관심을 갖는 특정 topic을 구독하므로, 해당 topic에 관련된 메시지만을 수신하게 된다.

**5.** 새로운 구독자가 특정 topic를 구독하면, 이후 발행되는 해당 topic의 메시지를 받게 된다.

<br><br><br>

## code

### build.gradle
Redis 의존성 추가
```
implementation 'org.springframework.boot:spring-boot-starter-data-redis'
```

<br><br><br>

### RedisConfig
*추가된 코드만 작성

pub/sub은 항상 redis에 발행 된 데이터가 있는지 확인하고 있어야 하기 떄문에 Listener를 등록해야한다.
```java
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, Object> chatMessageRedisTemplate(RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(connectionFactory);
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        return redisTemplate;
    }
    @Bean // RedisMessageListenerContainer : Redis에서 발행되는 메시지를 수신하고 처리하기 위한 컨테이너
    public RedisMessageListenerContainer redisMessageListenerContainer(RedisConnectionFactory connectionFactory) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(connectionFactory); // 컨테이너와 Redis 서버 간의 연결을 설정
        return container;
    }
}
```

<br><br><br>

### Publisher(발행자)
```java
@Service
@RequiredArgsConstructor
public class RedisPublisher { // 발행자(Publisher) 추가

    private final RedisTemplate<String, Object> redisTemplate;

    public void publish(ChannelTopic topic, String message) {
        redisTemplate.convertAndSend(topic.getTopic(), message);
    }
}
```

*String 대신 ChatMessage로 작성할 경우 RedisSubscriber에서도 수정해야한다.

`public void publish(ChannelTopic topic, ChatMessage message) {}`

<br><br><br>

### Subscriber(구독자)

```java
@Component
@RequiredArgsConstructor
public class RedisSubscriber implements MessageListener { // 구독자
    private final ObjectMapper objectMapper;
    private final SimpMessageSendingOperations messagingTemplate;

    @Override // Redis 메시지를 수신하면 호출되는 메소드
    public void onMessage(Message message, byte[] pattern) {
        try{
            // Redis로부터 수신된 메시지 처리 로직을 구현
            String channel = new String(message.getChannel());
            String msg = new String(message.getBody(), StandardCharsets.UTF_8); // Convert message body to String
            JsonNode jsonNode = objectMapper.readTree(msg); // JSON 문자열을 파싱
            messagingTemplate.convertAndSend(channel, jsonNode);
        } catch (Exception e){
            e.printStackTrace();
        }
    }
}
```
*json 형태로 msg를 받는다.

`onMessage()`에서 `private final SimpMessageSendingOperations messagingTemplate;`를 통해

client에 msg를 보낼 수 있고 Controller에서 따로 보낼 수도 있다. (아래 코드 참고) 

<br><br>

#### 특정 채널의 구독자 수 조회
구독이 제대로 되었는지 확인해보고 싶어서 작성했다.

```build.gradle
implementation 'redis.clients:jedis:3.6.3'
```
<br>

```java
import redis.clients.jedis.Jedis;

try (Jedis jedis = new Jedis("localhost")) {
    // 특정 채널의 구독자 수 조회
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
    private  Map<String, ChannelTopic> channels;

    @PostConstruct
    public void init(){
        channels = new HashMap<>();
    }

    @MessageMapping("/chat/sendMessage")
    public void sendMessage(@Payload ChatMessage chatMessage) {
        ChannelTopic channel = channels.get(chatMessage.getRoomId());
        redisPublisher.publish(channel, chatMessage.getMessage());
        template.convertAndSend("/topic/public/" + chatMessage.getRoomId(), chatMessage);
    }

    @MessageMapping("/chat/addUser")
    public void addUser(@Payload ChatMessage chatMessage, SimpMessageHeaderAccessor headerAccessor) {
        String roomId = chatMessage.getRoomId();
        ChannelTopic channel = new ChannelTopic(roomId);
        redisMessageListener.addMessageListener(redisSubscriber, channel);
        channels.put(roomId, channel);

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
}
```
☑️ `RedisTemplate<String, Object>`이 아닌 `SimpMessagingTemplate`을 사용한 이유

**SimpMessagingTemplate**은 WebSocket을 통해 메시지를 바로 클라이언트에게 전달할 수 있기 때문에, 

채팅과 같은 실시간 통신에 적합하다.

**RedisTemplate**은 주로 분산 시스템이나 여러 서버 간의 메시지 전달, 데이터 공유, 이벤트 처리 등을 위해 사용된다.

<br>

→ client가 **SimpMessagingTemplate**을 통해 메시지를 받지 못했을 때 

**RedisTemplate**을 사용하여 메시지를 받도록 구현하면 좋을듯함

<br><br><br>

### client에서 받기
client에서 제대로 값을 받는지 체크하고 싶었다.

```js
let socket = new SockJS('/ws');

// redis로 응답 받기
socket.onmessage = (event) => {
    const message = event.data;
    console.log('받은 메시지:', message)
    onMessageReceived(message)
};
```
data 타입은 String이고 message 형식은 ChatMessage로 값을 보냈을 때 아래와 같이 전달받았다.

```
MESSAGE
destination:/topic/public/123
content-type:application/json
subscription:sub-0
message-id:wrqzxuoi-10
content-length:100

{"roomId":"123","type":"TALK","sender":"hi123","message":"hihi"}   
```

<br><br><br>


*reference*                  
[[Redis] 4.Spring boot에서 pub/sub 모델 사용하기](https://zkdlu.github.io/2020-12-29/redis04-spring-boot%EC%97%90%EC%84%9C-pub,sub-%EB%AA%A8%EB%8D%B8-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0/)             
