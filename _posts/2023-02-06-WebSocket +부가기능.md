---
categories: Project Redis
tags: [spring, WebSocket, Chat]
---

# WebSocket
관련 글          
- [Websocket](https://haedal-uni.github.io/posts/WebSocket/)                                    
- [Websocket + 부가기능](https://haedal-uni.github.io/posts/WebSocket-+%EB%B6%80%EA%B0%80%EA%B8%B0%EB%8A%A5/)  👈🏻                            
- [Websocket (채팅 기록 json 파일 저장하기)](https://haedal-uni.github.io/posts/WebSocket(%EC%B1%84%ED%8C%85-%EA%B8%B0%EB%A1%9D-Json-%ED%8C%8C%EC%9D%BC-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B8%B0)/)                               
- [Sse](https://haedal-uni.github.io/posts/SSE/)                      
- [Sse 문제점](https://haedal-uni.github.io/posts/SSE-%EB%AC%B8%EC%A0%9C%EC%A0%90/)                        
- [Websocket + jwt](https://haedal-uni.github.io/posts/WebSocket-+-JWT/)                         
- [Websocket test](https://haedal-uni.github.io/posts/WebSocket-Test/)                        
- [Jmh - 채팅 파일 refactoring](https://haedal-uni.github.io/posts/JMH/)           


<br><br>

[1편](https://haedal-uni.github.io/posts/WebSocket/)에 이어서 채팅기능을 구현했으므로 고객센터를 구현하기 위해 admin과 user를 구분하여 작성했다.

<br>

## 부가적인 기능 추가
- 해당 채팅방을 만든 사람 즉, 본인과 관리자만 해당 채팅방 이용 하게하기
- 관리자만 채팅 리스트 띄우기(그 외는 본인이 만든 채팅방만 띄워주게하기)
- 채팅이 완료되면 5분뒤에 채팅방 삭제 시키기

<br>

삭제 버튼을 누르면 5분뒤에 삭제를 하기 위해서 ChatMessage의 MessageType에 DELETE를 추가했다.
```java
public enum MessageType {
    JOIN, TALK, LEAVE, DELETE
}
```

```js
stompClient.send("/app/chat/end-chat", {}, JSON.stringify({roomId: roomId, sender: username, type: 'DELETE'}))
```

<br>

전체코드는 해당 링크에 올려뒀다.  [github_Socket](https://github.com/haedal-uni/socket)

<br><br>

### ChatRoomController
```java
@Controller
@RequiredArgsConstructor
public class ChatRoomController {
    private final ChatServiceImpl chatService;

    // 채팅 리스트 화면
    @GetMapping("/room")
    public String rooms(Model model) {
        return "chat-list";
    }
    
    // 채팅방 입장 화면
    @GetMapping("/room/enter/{roomId}")
    public String roomDetail(Model model, @PathVariable String roomId){
        model.addAttribute("roomId", roomId);
        return "chat-room";
    }

    // 모든 채팅방 목록 반환
    @GetMapping("/rooms")
    @ResponseBody
    public List<ChatRoomDto> room() {
        return chatService.findAllRoom();
    }
    
    // 본인 채팅방
    @GetMapping("/room/one/{nickname}")
    @ResponseBody
    public ChatRoomDto roomOne(@PathVariable String nickname) {
        return chatService.roomOne(nickname);
    }

    // 채팅방 생성
    @PostMapping("/room")
    @ResponseBody
    public ChatRoomDto createRoom(@RequestBody String nickname){
        return chatService.createRoom(nickname);
    }

    // 완료된 채팅방 삭제하기
    @DeleteMapping("/room/one/{roomId}")
    @ResponseBody
    public void deleteRoom(@PathVariable String roomId){
        chatService.deleteRoom(roomId);
    }
    
   // 삭제 후 채팅방 재 접속 막기
    @GetMapping("/room/{roomId}")
    @ResponseBody
    public boolean getRoomInfo(@PathVariable String roomId) {
        return chatService.getRoomInfo(roomId);
    }
}
```

<br><br>

### Dto
#### ChatRoomDto
```java
@Getter
@Setter
@ToString
@Builder
@AllArgsConstructor
public class ChatRoomDto {
    private String roomId; // 채팅방 아이디
    private String roomName; // 채팅방 이름
    private String nickname;

    public ChatRoomDto() {
    }


    public static ChatRoomDto create(String name) {
        ChatRoomDto room = new ChatRoomDto();
        room.roomId = UUID.randomUUID().toString();
        room.roomName = name;
        return room;
    }

    public static ChatRoomDto of (Socket socket){
        return ChatRoomDto.builder()
            .roomId(socket.getRoomId())
            .nickname(socket.getNickname())
            .build();
    }
}
```
<br><br><br>

#### ChatRoomMap
```java
@Getter
@Setter
public class ChatRoomMap {
    private static ChatRoomMap chatRoomMap = new ChatRoomMap();
    private Map<String, ChatRoomDto> chatRooms = new LinkedHashMap<>();
    private ChatRoomMap(){}

    public static ChatRoomMap getInstance(){
        return chatRoomMap;
    }
}
```

<br><br><br>

### domain
```java
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Entity
@ToString
public class Socket {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "socket_id")
    private Long idx;

    @Column(nullable = false)
    private String nickname;

    @Column(nullable = false)
    private String roomId;

    public Socket(String roomId, String nickname){
        this.roomId = roomId;
        this.nickname = nickname;
    }
}
```
socket 코드를 작성하면서 db에 최대한 저장안하려고 했는데

채팅방에 들어갈 유저를 특정 유저로 정해놓고 해당 채팅방을 들어가게 하려면 db가 필요할 것 같았다.

db 내용은 최소한의 column으로 username과 roomId 만 설정했다.

roomName은 username으로 자동 설정되므로 추가하지 않았다.

<br>

**table 만들기**
```sql
CREATE TABLE IF NOT EXISTS socket (
    id bigint(5) NOT NULL AUTO_INCREMENT, # AUTO_INCREMENT : 자동으로 1씩 증가
    room_id varchar(255) NOT NULL,
    nickname varchar(255) NOT NULL,
    PRIMARY KEY (socket_id)
);
```

<br><br><br>

### Service
```java
@Slf4j
@RequiredArgsConstructor
@Service
public class ChatServiceImpl {
    private final ChatRepository chatRepository;

    //채팅방 불러오기
    public List<ChatRoomDto> findAllRoom() {
        List<ChatRoomDto> chatRoomDtos = new ArrayList<>();
        List<Socket> all = chatRepository.findAll();
        try {
            for(int i=0; i<all.size(); i++){
                chatRoomDtos.add(ChatRoomDto.of(all.get(i)));
            }
        } catch (NullPointerException e) {
            throw new RuntimeException("data 없음! ") ;
        }
        return chatRoomDtos;
    }

    //채팅방 하나 불러오기
    public boolean getRoomInfo(String roomId) {
        return chatRepository.existsByRoomId(roomId);
    }

    //채팅방 생성
    public ChatRoomDto createRoom(String nickname) {
        ChatRoomDto chatRoom = new ChatRoomDto();
        if (!chatRepository.existsByNickname(nickname)) {
            chatRoom = ChatRoomDto.create(nickname);
            ChatRoomMap.getInstance().getChatRooms().put(chatRoom.getRoomId(), chatRoom);
            Socket socket = new Socket(chatRoom.getRoomId(), nickname);
            log.info("Service socket :  " + socket);
            chatRepository.save(socket);
            return chatRoom;
        }
        else{
            Optional<Socket> byNickname = chatRepository.findByNickname(nickname);
            return ChatRoomDto.of(byNickname.get());
        }
    }

    public ChatRoomDto roomOne(String nickname){
        Optional<Socket> byNickname = chatRepository.findByNickname(nickname);
        return ChatRoomDto.of(byNickname.get());
    }

    public void deleteRoom(String roomId) throws IllegalStateException{
        Timer t = new Timer(true); 
        TimerTask task = new MyTimeTask(chatRepository, roomId);
        t.schedule(task,300000); 
        log.info("5분뒤에 삭제 됩니다.");
    }
}
```

**5분뒤에 삭제되는 채팅방 구현**

해당 코드는 단발성으로써 5분뒤에 삭제되는 로직이다. 

Timer : 실제 타이머의 기능을 수행하는 클래스

TimerTask : "Timer" 클래스가 수행되어야 할 내용을 작성하는 클래스 → run method 재정의 필수

`schedule()`은 타이머를 작동시키는 method이다.

<br><br><br>

### Time

```java
@Slf4j
@RequiredArgsConstructor
public class MyTimeTask extends TimerTask {
    private final ChatRepository chatRepository;
    private String roomId;
    public MyTimeTask(ChatRepository chatRepository, String roomId){
        this.chatRepository = chatRepository;
        this.roomId = roomId;
    }
    @Override
    public void run() {
        Socket socket = chatRepository.findByRoomId(roomId).orElseThrow(NullPointerException :: new);
        chatRepository.delete(socket);
        log.info("5분이 지나 삭제 되었습니다." + socket);
    }
}
```


<br><br><br>

### Repository
```java
public interface ChatRepository extends JpaRepository<Socket, Long> {
    boolean existsByNickname(String nickname);
    Optional<Socket> findByNickname(String nickname);
    boolean existsByRoomId(String roomId);
    Optional<Socket> findByRoomId(String roomId);
}
```

<br><br><br>

## 채팅방 gif
왼쪽 : 일반 user || 오른쪽 : admin

![AC_ 20230129-040520](https://user-images.githubusercontent.com/74857364/215287583-9a8c0b4a-f7a5-4ece-9479-73cdc804cdb8.gif)

<br><br><br>

*reference*         
[Java 를 활용한 Timer구현하기](https://hanna97.tistory.com/entry/Java-%EB%A5%BC-%ED%99%9C%EC%9A%A9%ED%95%9C-Timer%EA%B5%AC%ED%98%84-1)             
[[Spring]Springboot + websocket 채팅[1]](https://ratseno.tistory.com/71)                       
[[Spring]Springboot + websocket 채팅[2]](https://ratseno.tistory.com/72)             


