---
categories: Project 성능테스트 
tags: [Monitoring]
---  

# Apache Jmeter

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/c87672db-11d2-4ee5-b41a-f1fb8f937848)

Download Release 클릭 후 나는 apache-jmeter-5.6.3.zip 다운받았다.

파일 압축을 해제 한 후 bin 파일에 들어가서  

ApacheJMeter.jar 파일을 실행하면 jmeter가 실행된다.

window인 나는 jmeter.bat을 실행했다.

<br><br><br><br><br> 

## JMeter plugin
[link](https://jmeter-plugins.org/)

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/3e1dffe1-8c13-4bff-bc3f-197c8630110e){: width="50%"} 

이렇게 검색해도 된다.   

<br><br><br> 

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/7cb95821-273a-4f54-9f47-d3c2d7a0c40f){: width="50%"} 

해당 이미지처럼 클릭해서 플러그인 매니저를 다운받는다.   

jmeter 압축해제 한 뒤 해당 파일을 jmeter 폴더의 lib/ext 안에 넣어준다.

<br><br><br> 

JMeter를 재시작 하면 아래와 같이 플러그인을 설치할 수 있게 메뉴가 생겼다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/f40cc445-d139-4557-a769-85233d0cd5db)

<br><br><br><br><br> 

## Http Test
### Thread 설정  
JMeter에서 Test Plan 우클릭 → add → Thread(Users) → Thread Group을 하여 Thread Group을 생성한다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/282b4e92-a4df-4071-b957-58055ca2bb2f)

1) Number of Threads (users) : 쓰레드 개수 → 사용자 수

<br><br>

2) Ramp-Up Period (in seconds) : 쓰레드 당 생성시간

    Number of Threads = 1000이고 Ramp-up = 10 일 때, 1000명의 유저를 생성하는데 10초가 걸린다.

    → 1초 동안 100명의 유저가 요청을 한다

    Ramp-up = 0 으로 설정하면, 동시 접속 자 수는 1000명이 된다.

<br><br> 

3) Loop Count : Thread당 수행할 테스트 횟수  

    Number of Threads = 1000 이고 Loop Count = 10 일 때, 1000명의 유저는 동일한 작업을 10번 수행하게 된다.

    따라서 총 1000 * 10 = 10000 번 수행되고, 총 요청 횟수로 생각할 수 있다.


<br><br><br><br><br> 

## WebSocket Test
WebSockeet Sampler 플러그인을 설치한다.  

options 탭 → Plugins Manager → Available Plugins 탭 클릭 → WebSocket Sampler by Peter Doornbosch를 체크

우측 하단의 Apply Changes and Restart JMeter버튼 클릭

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/1cf7a36e-6db5-48c2-a7a3-71dff8991621)

<br><br>

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/2cb82476-d959-4d51-b2ef-48c99de9883c){: width="60%"}  

**WebSocket Close**: WebSocket 연결을 닫는 작업을 테스트한다.     
    → WebSocket 연결의 안정성과 정상적인 종료 상황을 확인   

**WebSocket Open Connection**: WebSocket 서버에 연결을 연다.     
    → WebSocket 연결 설정이 올바른지 확인하고, 연결이 성공적으로 이루어지는지 확인

**WebSocket Ping/Pong**: WebSocket 통신 중에 Heartbeat 메시지를 보내고 받아서 연결의 활성 상태를 유지하는지 테스트한다.     
    → 서버와의 연결 유지 여부를 확인

**WebSocket Single Read Sampler**: WebSocket에서 메시지를 읽어오는 작업을 테스트한다.     
    → 서버로부터의 응답을 확인하거나 특정 이벤트를 감지

**WebSocket Single Write Sampler**: WebSocket으로 메시지를 전송하는 작업을 테스트한다.     
    ex) 채팅 메시지를 보내는 등의 작업 테스트    

**WebSocket Request-Response Sampler**: WebSocket을 통해 요청을 보내고, 서버로부터의 응답을 받는 작업을 테스트한다.     
    → 요청과 응답 간의 시간을 측정하고, 성능을 평가   
     
<br><br>

MessageMapping으로 채팅 메시지를 보내는 경우에는 WebSocket Single Write Sampler를 사용하면 될 것 같고 

EventListener를 통해 연결 및 종료 이벤트를 감지하는 경우에는 

각각 WebSocket Open Connection 및 WebSocket Close Sampler를 사용하면 될 것 같다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/1f08156f-244a-4dc5-b5ea-ef7d2af3073e){: width="60%"}  

<br><br><br><br><br> 

---

## test

### login
채팅을 하기 전 header에 token을 담기 위해서 login 먼저 진행했다.

<br><br> 

#### Thread Group
> Thread Group 안에 HTTP Request, Debug Sampler, View Results Tree 추가

> Thread Group > Add > Sampler > HTTP Request                 
> Thread Group > Add > Listener > View Results Tree           
> Thread Group > Sampler > Debug Sampler             

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/c3d75777-a309-4756-8287-9d6bf476cebe)

<br><br><br><br> 

#### HTTP Request
Body Data에 json형식으로 nickname과 password를 작성한다.

```js  
{
    "nickname" : "nick.123",
    "password" : "nick.123"
}
```
![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/18ae7be6-2437-4038-914c-3c92ae037d5d)

<br><br><br><br>   

#### HTTP Header Manager
> HTTP request > Add > Config Element > HTTP Header Manager

HTTP Header Manager에 Name = `Content-type`, Value = `application/json`를 추가한다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/45a6fdd8-008d-41c3-aeb8-a7a78de59b0b)

<br><br><br><br>   

#### View Results Tree
Response data > Response Body로 보면 아래와 같은 형식으로 응답이 온다.

`{"success":true, "role_check":"사용자","token":"token1234","username":"nick.123"}`

<br><br><br><br>  

#### Regular Expression Extractor
> HTTP request > Add > Post Processor > Regular Expression Extractor

응답이 온 값들 중 token만 값을 저장하려고 한다.

- Name of created variable : `token`
- Regular Excpression : `"token":"([^"]+)"`
- Template : `$1$`
- Match No. : `1`

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/08560422-58cf-4673-a5a1-b24550404897)

<br><br><br> 

token이 추출 된 것을 볼 수 있다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/4f75a8ab-abe5-4e17-959b-f500c4a17c21)

<br><br><br><br>  

#### BeanShell PostProcessor
> HTTP request > Add > Post Processor > BeanShell PostProcessor

Debug Sampler에서 추출한 token 값을 넣었다. 

```java
import org.apache.jmeter.protocol.http.control.Header;

sampler.getHeaderManager().add(new Header("Authorization",vars.get("token")));
```
![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/c0e7aa32-96ef-4b71-b742-5eef8be3e922)

<br><br><br><br> 

### 로그인 후의 http request
게시글 조회로 test를 해봤다.

> HTTP Request를 만들고 HTTP Header Manager에 token을 넣어준다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/0c3f3f5a-28fc-4e4b-b894-ce92194ad5c0)

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/bdf291d9-5e01-4fa6-9e07-77a6f9e37637)

<br><br><br><br><br> 
    
## Chat

전체 보기

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/592eca09-ad07-4776-80c0-44cf1dd9c650)

<br><br> 

참고로 모든 요청에 HTTP Header Manager를 생성하여 아래와 같이 작성했다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/11b8d7f3-b0f8-410a-b9fd-ef5a78737249)

*simpSessionId는 아래에서 설명 예정
    
<br><br><br>  

### WebSocket Open Connection

> Thread Group > Add > Sampler > WebSocket Open Connection

Stomp 프로토콜을 사용하여 통신하는 엔드포인트는 `/ws` 다.

채팅을 할 때 network로 해당 url을 살펴보면 아래와 같이 보여진다.

`ws://localhost:8080/ws/${유저 번호}/${세션 식별자}/websocket`

<br><br>   

이를 Server URL에 작성하면 ws, localhost, 8080으로 작성할 수 있다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/cc3f25c5-c24e-4318-aa2e-167e54b00ffd)

<br><br> 

여기서 Path는 `ws/${유저 번호}/${세션 식별자}/websocket`인데

유저번호를 3자릿수로, 세션 식별자를 8자릿수로 설정했다.

<br><br><br>  

### 변수 추가

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/eec29b51-a824-466e-968b-783ab56b2bb8)

<br><br> 

#### user number
> Thread Group > HTTP Request > Add > Config Element > Random Variable

100~999 랜덤 숫자

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/9e5a66fd-9e7c-4684-b334-12f6907a65b6)

<br><br><br> 

#### session
> Thread Group > HTTP Request > Add > Config Element > User Defined Variables  

a~z 중 랜덤 8자리

Name = `sessionId`, Value = `${__RandomString(8,abcdefghijklmnopqrstuvwxyz)}`
   
![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/71a83346-ad5a-42fe-b89e-a2b8b89feded)
           
<br><br> 

따라서 path는 `/ws/${num}/${sessionId}/websocket`로 설정할 수 있다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/f887d8a6-0f3a-4dfd-a4fa-d305bd720577)

<br><br>

작성 후에 실행하면 성공적으로 띄워지는 것을 볼 수 있다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/660846f5-866b-4ab2-9206-cc0541cb11cf)

<br>

이제 채팅을 연결 시켜봤으니 사용자가 해당 채팅을 연결 시켜보는 작업을 진행한다.


<br><br><br><br> 

### Send Connect : WebSocket Single Write Sampler
> Thread Group > Add > Sampler > WebSocket Single Write Sampler
  
![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/5694743c-57cc-450b-9157-a50be79444e0)

WebSocket Open Connection"으로 연결을 열어둔 상태에서 

"WebSocket Single Write Sampler"로 메시지를 보내기 때문에 use existing connection 적용   

Request data :`["CONNECTED\nversion:1.2\n\n\u0000"]`

<br>  

만약 config에서 heart-beat를 설정했다면 `["CONNECTED\nversion:1.2\nheart-beat:0,0\n\n\u0000"]`와 같이 작성


<br><br><br><br> 

### SEND SUBSCRIBE : WebSocket Single Write Sampler 
> Thread Group > Add > Sampler > WebSocket Single Write Sampler          

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/a19dcfb0-cac6-46c8-b57c-f5270c8b8ff7)
         
구독하는 url : `/topic/public/${roomId}`

Request data :  `["SUBSCRIBE\ndestination:/topic/public/room/'${roomId}'\nid:sub-0\n\n\u0000"]`

- `\n\n`: STOMP 프로토콜에서는 헤더와 본문을 구분하기 위해 빈 줄 (두 개의 연속된 개행 문자)을 사용

- `\u0000`: 이는 STOMP 프레임의 끝을 나타내는 NULL 문자

<br><br>

만약 id 값을 넣지 않고 `["SUBSCRIBE\ndestination:/topic/public/room/'${roomId}'\n\n\u0000"]` 와 같이 작성했다면

*No subscriptionId in GenericMessage* 라는 에러가 뜬다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/06357d18-8a21-4725-a961-e09979bdad8f)

<br><br> 
    
STOMP 프로토콜에서는 클라이언트가 구독(subscribe) 요청을 할 때 필수정인 정보를 전달한다. 

- destination : 구독할 대상의 목적지를 지정한다. (topic이나 queue의 이름)

- id : 클라이언트가 메시지를 수신할 때 사용할 subscriptionId를 지정한다. 

  서버가 클라이언트에게 할당하는 고유한 식별자로써
  
  클라이언트는 이 subscriptionId를 통해 메시지를 구독하고, 서버는 해당 subscriptionId를 통해 메시지를 전송한다.
  
<br> 

```
>>> SUBSCRIBE
id:sub-0
destination:/topic/public/3d41c3ed-8ddb-458d
```
[Websocket 정리 글](https://haedal-uni.github.io/posts/WebSocket/#pubsub-%EA%B5%AC%EC%A1%B0)을 참고한다.

<br><br><br><br>   

### 채팅방 입장(code)
```java
// ChatController
@MessageMapping("/chat/addUser")
public void addUser(@Payload ChatMessage chatMessage, SimpMessageHeaderAccessor headerAccessor) {
    // 일부 코드 생략 
    String sessionId = (String) headerAccessor.getHeader("simpSessionId");
    String token = headerAccessor.getFirstNativeHeader("Authorization");
    redisService.addSession(sessionId, token);
    log.info("[chat] addUser token 검사: " + user.getNickname());
    template.convertAndSend("/topic/public/" + roomId, chatMessage);
}

// WebSocketConfig  
@Override
public void configureMessageBroker(MessageBrokerRegistry config) {
    config.enableSimpleBroker("/topic");
    config.setApplicationDestinationPrefixes("/app");
}
```

작성해야 할 정보 : token, sessionId, roomId, MessageType, day, time

위 정보들을 request data에 담기 위해서 JSR223 Sampler를 사용했다.

<br><br> 

> Thread Group > Add > Sampler > JSR223 Sampler

*만약 JSR223 Sampler가 존재하지 않을 경우 Options의 Plugins Manager에서 설치하면 된다.  

```java
import java.util.UUID;
import java.util.Calendar;
import java.text.SimpleDateFormat;

// 현재 시간을 가져오기 위해 Calendar 객체 생성
Calendar cal = Calendar.getInstance();

// 월과 일을 가져옴 (월은 0부터 시작하므로 1을 더해줌)
int month = cal.get(Calendar.MONTH) + 1;
int day = cal.get(Calendar.DAY_OF_MONTH);

// SimpleDateFormat을 사용하여 원하는 형식으로 날짜를 포맷
SimpleDateFormat dateFormat = new SimpleDateFormat("MM/dd");
String formattedDate = dateFormat.format(cal.getTime());

// 시간과 분을 가져옴
int hour = cal.get(Calendar.HOUR_OF_DAY);
int minute = cal.get(Calendar.MINUTE);

// roomId 생성
String roomId = UUID.randomUUID().toString();

// 생성된 roomId를 JMeter 변수에 할당
vars.put("roomId", roomId);

// 가져온 월과 일을 JMeter 변수에 할당
vars.put("date", formattedDate); 

// 가져온 시간과 분을 JMeter 변수에 할당
vars.put("time", String.valueOf(hour) + ":" + String.valueOf(minute));
```

<br><br> 

Request data : 

```
["SEND\ndestination:/app/chat/addUser\nAuthorization:${token}\ncontent-type:application/json\n\n{\"roomId\":\"${roomId}\",\"type\":\"JOIN\",\"day\":\"${date}\",\"time\":\"${time}\"}\n\u0000"]
```

<br><br><br>  

```java

@MessageMapping("/chat/addUser")
public void addUser(@Payload ChatMessage chatMessage, SimpMessageHeaderAccessor headerAccessor) {
    // 코드 생략 
    log.info("[chat] addUser token 검사: " + user.getNickname());
}
```
사용자가 채팅방에 접속했을 때 띄워지는 log가 출력되었다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/b65cde2e-4d67-485d-9aec-6feea05ee144)


<br><br><br><br>

REFERENCE    
- [[JMeter] JMeter를 이용한 성능 테스트](https://12bme.tistory.com/503)
- [Apache JMeter란 무엇인가? (+ 사용 방법 with 성능 및 부하 테스트)](https://jaehoney.tistory.com/224)      
- [JMeter을 이용해서 웹서버 성능 테스트하기](https://www.youtube.com/watch?v=1AyxqIePusA)
- [JMeter 플러그인 설치 및 활용](https://sooo-9.tistory.com/37)
- [🙈Apache JMeter - HTTP, WebSocket 성능 테스트🐵](https://victorydntmd.tistory.com/267)
- [Jmeter jwt 동적으로 받기 (1탄)](https://velog.io/@dksdnjs830/jmeter)
- [[Spring Boot] Jmeter로 STOMP 부하 테스트 하기](https://velog.io/@mw310/Spring-Boot-Jmeter%EB%A1%9C-STOMP-%EB%B6%80%ED%95%98-%ED%85%8C%EC%8A%A4%ED%8A%B8-%ED%95%98%EA%B8%B0)     
