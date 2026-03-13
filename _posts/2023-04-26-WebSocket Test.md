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
- [Websocket + jwt](https://haedal-uni.github.io/posts/WebSocket-+-JWT/)                         
- [Websocket test](https://haedal-uni.github.io/posts/WebSocket-Test/) 👈🏻                       
- [Jmh - 채팅 파일 refactoring](https://haedal-uni.github.io/posts/JMH/)           

<br><br>  

```java
class ChatControllerTest { // server → client
   @Mock
   private SimpMessagingTemplate template;
   /* ChatController의 sendMessage() 메소드에서 실제로 template.convertAndSend()가 호출되는지 확인하기 위해
   SimpMessagingTemplate 인스턴스를 Mock 객체로 생성 */

   @InjectMocks
   private ChatController chatController;

   @BeforeEach
   void setUp() {
      MockitoAnnotations.openMocks(this);
   }

   @Test
   void sendMessageTest() {
      // given
      ChatMessage chatMessage = new ChatMessage();
      chatMessage.setType(ChatMessage.MessageType.TALK);
      chatMessage.setMessage("message");
      chatMessage.setSender("sender");
      chatMessage.setRoomId("roomId");

      // when
      chatController.sendMessage(chatMessage);

      // then
      verify(template).convertAndSend(
         eq("/topic/public/" + chatMessage.getRoomId()),
         eq(chatMessage)
      );
   }
}
```

WebSocket 통신에서는 메시지를 클라이언트로 전송할 때 SimpMessagingTemplate 객체를 사용한다.

테스트 코드에서는 이 객체를 실제로 동작시키는 대신 Mockito의 `@Mock`을 사용해 가짜 객체로 생성할 수 있다.

또한 `@InjectMocks`을 사용하면 Mock 객체를 ChatController 내부에 자동으로 주입할 수 있다.

이를 통해 실제 메시지를 전송하지 않더라도 메시지 전송 로직이 올바르게 호출되는지 확인할 수 있다.

<br><br>

WebSocket 메시지 처리에서는 메시지가 어떤 경로(destination)로 어떤 내용으로 전송되는지가 매우 중요하다.

따라서 테스트 코드에서는 `verify()` 를 이용해

SimpMessagingTemplate의 `convertAndSend()`가 예상한 경로와 메시지로 호출되었는지 검증한다.

`verify(template)` → Mock 객체에서 특정 메소드가 호출되었는지 확인

`eq()` → Mockito에서 제공하는 메소드로 전달된 인자가 예상한 값과 동일한지 비교할 때 사용

<br><br>

또한 Mockito를 사용하기 위해서는 `@Mock`이 선언된 객체들을 초기화해야 한다.

이를 위해 `MockitoAnnotations.openMocks(this)` 를 사용한다.

이 코드는 테스트 클래스에서 선언된 모든 `@Mock` 객체를 초기화하여 Mock 객체로 생성하는 역할을 한다.

<br><br>

Mockito를 사용하는 또 다른 방법으로는

`@ExtendWith(MockitoExtension.class)` 을 사용하는 방식이 있다.

이 어노테이션을 사용하면 Mockito가 자동으로 초기화되기 때문에

`MockitoAnnotations.openMocks(this)` 를 따로 작성할 필요가 없다.

<br>

즉 두 방식 중 하나만 사용하면 된다.

`MockitoAnnotations.openMocks(this)` 사용 → `@BeforeEach`에서 Mock 초기화

`@ExtendWith(MockitoExtension.class)` 사용 → Mockito 자동 초기화

 → `@ExtendWith(MockitoExtension.class)`를 사용한다면 `@BeforeEach setUp()`은 제거해도 된다.
