---
categories: Project
tags: [spring, WebSocket, Chat, test]
---

## WebSocket Test
관련 글          
- [Websocket](https://haedal-uni.github.io/posts/WebSocket/)                                    
- [Websocket + 부가기능](https://haedal-uni.github.io/posts/WebSocket-+%EB%B6%80%EA%B0%80%EA%B8%B0%EB%8A%A5/)                             
- [Websocket (채팅 기록 json 파일 저장하기)](https://haedal-uni.github.io/posts/WebSocket(%EC%B1%84%ED%8C%85-%EA%B8%B0%EB%A1%9D-Json-%ED%8C%8C%EC%9D%BC-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B8%B0)/)                               
- [Sse](https://haedal-uni.github.io/posts/SSE/)                      
- [Sse 문제점](https://haedal-uni.github.io/posts/SSE-%EB%AC%B8%EC%A0%9C%EC%A0%90/)                        
- [Websocket + jwt](https://haedal-uni.github.io/posts/WebSocket-+-JWT/)                         
- [Websocket test](https://haedal-uni.github.io/posts/WebSocket-Test/) 👈🏻                       
- [Jmh - 채팅 파일 refactoring](https://haedal-uni.github.io/posts/JMH/)           

<br><br>  

채팅 애플리케이션에서 WebSocket을 이용해 메세지 송수신을 구현한 경우, 해당 기능을 테스트해볼 필요가 있다. 

이를 위해 테스트 코드를 작성할 수 있는데, 이 과정에서 Mockito 프레임워크를 활용할 수 있다.

Mockito는 객체의 행위를 검증하기 위한 프레임워크로, 테스트 코드 작성 시 객체를 가짜 객체로 만들어 원하는 대로 동작하도록 설정할 수 있다. 

즉, 테스트 코드에서 객체가 실제로 실행되지 않고, 미리 정해진 동작만 수행하도록 구현되는 것이다.

<br>

```java
class ChatControllerTest { // server → client
   @Mock
   private SimpMessagingTemplate template;
   /* ChatController의 sendMessage() 메소드에서 실제로 template.convertAndSend()가 호출되는지 확인하기 위해서
   SimpMessagingTemplate 인스턴스를 Mock 객체로 생성 */

   @InjectMocks
   private ChatController chatController;

   @BeforeEach
   void setUp() { // @ExtendWith(MockitoExtension.class)를 class에 붙이거나 해당 메소드를 쓰거나
      /* @Mock 어노테이션으로 선언된 필드들을 초기화 하기 위해서는 MockitoAnnotations 클래스의 openMocks() 메소드를 이용해야한다.
      이 코드는 모든 @Mock 어노테이션으로 선언된 필드를 초기화 하기 위해 작성*/
    
      MockitoAnnotations.openMocks(this);
      // this : ChatControllerTest 클래스의 인스턴스
      //@Mock 어노테이션을 통해 선언한 SimpMessagingTemplate 객체를 Mockito에서 제공하는 mock 객체로 만들기 위해서는 this 인스턴스를 전달
   }

   @Test
   void sendMessageTest() {
      // given
      ChatMessage chatMessage = new ChatMessage();
      chatMessage.setType(ChatMessage.MessageType.TALK);
      chatMessage.setMessage("message");
      chatMessage.setSender("sender");
      chatMessage.setRoomId("roomId");

      /* client -> server (없어도 되는 코드)
      StompHeaderAccessor headerAccessor = StompHeaderAccessor.create(StompCommand.SEND); 
      headerAccessor.setDestination("/app/chat/sendMessage");
      headerAccessor.setSessionId("session1"); //StompSession을 생성하면 내부에서는 자동으로 sessionId를 생성
      headerAccessor.setSessionAttributes(new HashMap<>());
    */

      // when
      chatController.sendMessage(chatMessage);

      // then
      verify(template).convertAndSend(eq("/topic/public/" + chatMessage.getRoomId() ), eq(chatMessage));
      // verify : 특정 객체가 특정 메소드를 특정 매개 변수로 호출되는지 확인
      // verify(template) : SimpMessagingTemplate 객체가 제공하는 메소드 중 하나가 호출되었는지 검증하는 용도
      // eq : Mockito에서 제공하는 메소드로, 객체를 비교할 때 사용
   }
}
```

WebSocket 통신에서 사용하는 SimpMessagingTemplate 객체를 테스트하기 위해, 

해당 객체를 가짜 객체로 만들어 Mockito를 이용해 동작을 검증할 수 있다. 

테스트 코드에서는 `@Mock` 어노테이션을 사용해 SimpMessagingTemplate을 가짜 객체로 만들어주고, 

`@InjectMocks` 어노테이션을 사용해 ChatController 객체에 가짜 객체를 주입한다.

<br>

또한, WebSocket으로 메세지를 송수신하는 경우, 송수신할 때의 메세지 형식도 매우 중요하다. 

이를 테스트하기 위해, 예상되는 메세지와 실제 전송되는 메세지를 검증할 필요가 있다. 

이를 위해 verify 메소드를 사용해 SimpMessagingTemplate 객체에서 전송된 메세지가 예상한 대로인지 검증할 수 있다.

<br>

테스트 코드에서는 `MockitoAnnotations.openMocks(this)` 코드를 사용해 테스트 클래스 내의 객체를 가짜 객체로 만들어 줄 수 있다. 

이 코드를 사용하면, `@BeforeEach` 어노테이션에서 가짜 객체를 생성하는 코드를 작성할 필요가 없어져 더욱 편리하다.

<br>

마지막으로, `@ExtendWith(MockitoExtension.class)` 어노테이션을 통해 JUnit5에서 Mockito를 사용할 수 있다. 

하지만 이미 `MockitoAnnotations.openMocks(this)` 코드를 사용했기 때문에 이 어노테이션을 사용할 필요가 없다.

<br>

`@ExtendWith(MockitoExtension.class)`를 작성하면 JUnit Jupiter의 기능을 확장할 수 있지만, 

MockitoExtension은 이미 기본으로 활성화되어 있기 때문에, 굳이 `@ExtendWith(MockitoExtension.class)`를 작성하지 않아도 된다.

`@ExtendWith(MockitoExtension.class)`를 작성하고 싶다면` @BeforeEach   void setUp() {}` 코드를 제거하면 된다.
