---
categories: Project Redis
tags: [spring, Redis]
---
# Spring의 내장 브로커에서 Redis 브로커로 마이그레이션
**1. 확장성 및 메시지 처리량 향상**

Redis는 메모리 기반 데이터베이스로, 데이터를 빠르게 읽고 쓸 수 있는 특징을 가지고 있다.

낮은 지연 시간을 제공하며, 높은 메시지 처리량을 가능하게 해 더 많은 사용자나 요청을 처리할 수 있다.

<br>

**2. 간단한 설정과 운영**

Redis는 간단한 설정을 가지고 있어 초기 구축 및 운영이 간편하다. 

<br>

**3. Pub/Sub 패턴 지원**

Redis는 Pub/Sub 패턴을 지원하여 다중 구독자가 하나의 메시지를 동시에 수신할 수 있도록 해준다. 

이를 통해 실시간 이벤트 처리를 용이하게 구현할 수 있다.

<br><br>

*Redis Pub/Sub은 메시지 브로커 시스템으로,            
다른 서비스나 애플리케이션 간에 이벤트 기반 통신을 단순화하기 위해 사용된다.        

WebSocket은 클라이언트와 서버 간에 실시간 양방향 통신을 제공하는 프로토콜이다.

<br><br>

## Redis pub/sub
Pub/Sub은 발행/구독 모델(Publish/Subscribe)을 기반으로 한 통신 방법으로,

topic(or channel)에 대해 관심 있는 구독자(Subscriber)들에게 메시지를 발행(Publish)하는 방식이다.

Subscriber는 여러 개의 채널을 구독할 수 있다. (유튜브와 비슷한 개념)

Publisher가 채널에 메시지를 보내면, 그 채널을 구독하고 있는 Subscriber들에게 메시지가 전달된다.

메시지를 보낼 때 Publisher는 채널에 어떤 Subscriber가 있는지 알 필요가 없이, 그냥 메시지를 전송만 하면 된다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/a3bc658f-ce54-430d-89f1-f4ad9e760928)

Publish/Subscribe 구조에서 사용되는 Queue를 일반적으로 Topic이라고 한다.

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
	@Value("${spring.redis.host}")
	private String redisHost;
	@Value("${spring.redis.port}")
	private int redisPort;

	@Bean
	public RedisConnectionFactory redisConnectionFactory() {
		return new LettuceConnectionFactory(redisHost, redisPort);
	}

	@Bean //Redis 데이터에 쉽게 접근하기 위한 코드
	public RedisTemplate<?, ?> redisTemplate() { //RedisTemplate 에 LettuceConnectionFactory 을 적용
		RedisTemplate<?, ?> redisTemplate = new RedisTemplate<>();
		redisTemplate.setConnectionFactory(redisConnectionFactory());
		redisTemplate.setKeySerializer(new StringRedisSerializer());//redisTemplate.setKeySerializer(new GenericToStringSerializer<>(Object.class));
		redisTemplate.setValueSerializer(new StringRedisSerializer());
		return redisTemplate;
	}

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
`redisMessageListenerContainer()`는 Redis를 브로커로 사용하기 위한 설정

나머지 메소드들은 Redis 자체의 기본 설정과 관련된 코드다.

<br><br><br>

### Publisher(발행자)
```java
@Service
@RequiredArgsConstructor
public class RedisPublisher { // 발행자(Publisher) 추가

    private final RedisTemplate<String, Object> redisTemplate;

    public void publish(ChannelTopic topic, ChatMessage message) {
        redisTemplate.convertAndSend(topic.getTopic(), message);
    }
}
```
기존에 `@MessageMapping`으로 메세지를 받는 메소드에서 바로 `redisTemplate.convertAndSend()` 해줬는데

지금은 RedisPublisher class를 추가해서 `@MessageMapping`에서 `publish()`를 호출한 후 실행된다.

<br><br><br>

### Subscriber(구독자)

```java
@Component
@RequiredArgsConstructor
public class RedisSubscriber implements MessageListener { // 구독자
    private final ObjectMapper objectMapper;
    private final RedisTemplate<String, ChatMessage> redisTemplate;
    private final SimpMessageSendingOperations messagingTemplate;

    @Override // Redis 메시지를 수신하면 호출되는 메소드
    public void onMessage(Message message, byte[] pattern) {
        try{
            // Redis로부터 수신된 메시지 처리 로직을 구현
            String channel = new String(message.getChannel());
            log.info("Received message from channel: " + channel);

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
☑️ `RedisTemplate<String, Object>`와 `SimpMessagingTemplate`

`RedisTemplate<String, Object>`은 Spring에서 Redis와 상호작용하기 위한 일반적인 Template class 다. 

Redis 서버와의 상호작용을 추상화하고, Redis에 데이터를 저장하고 조회하고 수정할 수 있는 메소드를 제공한다.  

이를 통해 Redis를 데이터 저장소로 사용할 수 있으며, 직렬화 및 역직렬화 메커니즘과 관련된 설정을 지원한다.   

Redis 서버와 직접적으로 상호작용하기 위해 사용된다.   

<br><br>

`SimpMessagingTemplate`은 Spring의 WebSocket 기능을 지원하는 클래스로, 클라이언트 간 실시간 메시징을 위해 사용된다. 

이는 STOMP (*Simple Text Oriented Messaging Protocol*)를 통해 WebSocket 연결을 통해 메시지를 보내고 받을 수 있도록 도와준다. 

주로 웹 소켓을 통해 클라이언트 간의 메시지 전달 및 실시간 통신에 사용한다.   

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
[🙈[Spring Redis] Spring Data Redis (2) - Redis pub sub🐵](https://victorydntmd.tistory.com/249)                   
[🗃️ REDIS의 PUB/SUB 기능 (채팅 / 구독 알림)](https://inpa.tistory.com/entry/REDIS-%F0%9F%93%9A-PUBSUB-%EA%B8%B0%EB%8A%A5-%EC%86%8C%EA%B0%9C-%EC%B1%84%ED%8C%85-%EA%B5%AC%EB%8F%85-%EC%95%8C%EB%A6%BC)           
