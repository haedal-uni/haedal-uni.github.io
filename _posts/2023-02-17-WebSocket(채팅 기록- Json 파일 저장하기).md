---
categories: Project Chat
tags: [spring, WebSocket, Chat]
---

# Json 파일 저장하기
관련 글          
- [Websocket](https://haedal-uni.github.io/posts/WebSocket/)                                    
- [Websocket + 부가기능](https://haedal-uni.github.io/posts/WebSocket-+%EB%B6%80%EA%B0%80%EA%B8%B0%EB%8A%A5/)                                
- [Websocket (채팅 기록 json 파일 저장하기)](https://haedal-uni.github.io/posts/WebSocket(%EC%B1%84%ED%8C%85-%EA%B8%B0%EB%A1%9D-Json-%ED%8C%8C%EC%9D%BC-%EC%A0%80%EC%9E%A5%ED%95%98%EA%B8%B0)/) 👈🏻                              
- [Sse](https://haedal-uni.github.io/posts/SSE/)                                      
- [Websocket + jwt](https://haedal-uni.github.io/posts/WebSocket-+-JWT/)                         
- [Websocket test](https://haedal-uni.github.io/posts/WebSocket-Test/)                        
- [Jmh - 채팅 파일 refactoring](https://haedal-uni.github.io/posts/JMH/)           

<br><br>

## 문제

채팅 대화 내용을 보여주기 위해 처음에는 클라이언트에서 sessionStorage에 저장하는 방식을 사용하려고 했다.  

하지만 실제로 구현해보니 생각보다 문제가 많았다.  

<br>

type이 "TALK"인 경우에만 저장하도록 조건을 걸었다.  

→ 대화가 1번 이루어지면 저장이 되지 않고 2번 작성한 경우 가장 먼저 작성한 것만 저장되는 문제가 발생했다.  

→ 이런 식으로 하나씩 순서대로 저장되지 않았다.  

<br>

그래서 저장이 무조건 되도록 if문을 추가했다.  

sessionStorage의 길이와 실제 작성한 채팅 횟수가 같지 않으면 다시 저장하도록 설정했다.  

그 결과 무한 루프가 발생했다.  

<br>

for문으로 sessionStorage에 저장된 대화들을 가져왔다.  

→ 가져오는 순서가 뒤죽박죽이었다.  

→ admin도 sessionStorage에 같이 저장하면서 순서가 섞이는 것처럼 보였다.  

<br>

이 문제들로 인해 클라이언트에서 채팅 기록을 관리하는 방식은 한계가 있다고 판단했다.  

그래서 서버에서 채팅을 저장하고 다시 내려주는 방식으로 방향을 바꿨다.  

DB에 저장하는 방법도 있었지만 채팅 로그 특성상 파일로 저장하는 방식이 더 효율적이라고 생각했다.  

<br>

채팅을 내려줄 때 JSON 형식으로 데이터를 받기 때문에 저장도 JSON 형식으로 해야겠다고 판단했다.  

그래서 JSONObject를 사용했다.  

그런데 예상치 못한 문제가 발생했다.  

파일에는 아래와 같이 정상적으로 저장되는 것처럼 보였다.  

![image](https://user-images.githubusercontent.com/74857364/216782488-30d8260f-7504-4a8e-9306-42b4033404bf.png){: width="50%"}

<br>

![image](https://user-images.githubusercontent.com/74857364/216782500-07c66f32-e6b9-4b74-96ce-a47b77981ef5.png)

오! 잘 되는 건가? 😃 라고 생각했다.  

하지만 한 번 더 글을 작성하자 문제가 바로 드러났다.  

<br><br>

![image](https://user-images.githubusercontent.com/74857364/216782540-fe5bd68f-9259-4038-b68d-7211185ee604.png)

JSON 사이에 구분자가 없었다. 😭  

<br><br>

*Unexpected token LEFT BRACE({) at position 95.*  라는 에러가 발생했다.  

이 에러는 "{}"가 끝난 직후 바로 "{"가 시작되기 때문에 발생한 오류였다.  

그래서 JSON을 배열 안에 넣는 방식으로 저장해야겠다고 생각했다.  

즉 `[ {}, {} ]` 형태를 만들고 싶었다.  

그래서 JSONArray 안에 JSONObject를 넣는 방식을 시도했다.  

그 결과는 `[{}] [{}]` 형태로 저장되었다.  하핳 😅  

![image](https://user-images.githubusercontent.com/74857364/217054305-699a05bd-5711-40a5-83a8-50ab1cd9b568.png)

<br><br>

이때부터 예상보다 훨씬 큰 삽질이 시작되었다.  

그래도 결국 원하는 결과가 나와서 이렇게 정리하게 되었다. 😙  

이 방식이 정답인지는 모르겠지만 아래처럼 작성하지 않으면 내가 원하는 결과가 나오지 않았다.  

추가된 코드를 설명하다 보니 기존에 작성된 코드는 일부 생략되어 있다.  

전체 코드가 궁금하거나 흐름이 헷갈리면 웹소켓 1편이나 2편을 참고하면 된다.  

<br>

- 1편 : [기본](https://haedal-uni.github.io/posts/WebSocket/)               
- 2편 : [부가기능](https://haedal-uni.github.io/posts/WebSocket-+%EB%B6%80%EA%B0%80%EA%B8%B0%EB%8A%A5/)


<br><br><br><br>

---

### setFile (client)

서버에서 `@MessageMapping`으로 메시지를 보낼 때마다 실행되는 함수에서 setFile을 호출했다.  

메시지를 보내는 즉시 파일로 저장되도록 하기 위해서다.  

```js
function sendMessage(event) {
    let chatMessage = {
        roomId: roomId,
        sender: username,
        message: messageInput.value,
        type: 'TALK'
    };
    setFile(chatMessage);
    stompClient.send("/app/chat/sendMessage", {}, JSON.stringify(chatMessage));
}
```

```js
function setFile(chatMessage) {
    $.ajax({
        type: "POST",
        url: `/room/enter/` + roomId + '/' + roomName,
        data: JSON.stringify(chatMessage),
        contentType: 'application/json',
        processData: false,
        success: function(response) {
        }
    });
}
```

<br><br><br><br>

### setFile (server)

Java에서 JSON을 다룰 때는 보통 google의 json-simple 라이브러리를 사용한다.  

Maven을 사용하는 경우 pom.xml에 의존성을 추가하면 된다.  

나는 Gradle을 사용하고 있어서 Maven 설정을 그대로 Gradle에 적용했다.  

```xml
<dependency>
    <groupId>com.googlecode.json-simple</groupId>
    <artifactId>json-simple</artifactId>
    <version>1.1.1</version>
</dependency>
```

위는 Maven 설정이다.  

```gradle
implementation 'com.googlecode.json-simple:json-simple:1.1.1'
```

<br><br>

Gradle에 의존성을 추가하지 않고 import를 하면 에러가 발생한다.  

![image](https://user-images.githubusercontent.com/74857364/217088955-844a4013-2519-480e-9d0d-77b3c8b1252f.png)

<br>

```java
import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;
import org.json.simple.parser.ParseException;
```

Gradle도 Maven처럼 의존성만 추가하면 된다.  

<br><br>

```java
private void saveFile(ChatMessage chatMessage) {
    JSONObject json = new JSONObject();
    json.put("roomId", chatMessage.getRoomId());
    json.put("type", chatMessage.getType().toString());
    json.put("sender", chatMessage.getSender());
    json.put("message", chatMessage.getMessage());

    Gson gson = new GsonBuilder().setPrettyPrinting().create();
    String json1 = gson.toJson(json);

    try {
        FileWriter file = new FileWriter(chatUploadLocation + "/" + chatMessage.getRoomId() + ".txt", true);
        File file1 = new File(chatUploadLocation + "/" + chatMessage.getSender() + "-" + chatMessage.getRoomId() + ".txt");

        if (file1.exists() && file1.length() == 0) {
            file.write(json1);
        } else {
            file.write("," + json1);
        }

        file.flush();
        file.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

☑️ FileWriter는 문자 데이터를 파일에 쓰는 클래스이다.  

☑️ FileOutputStream은 바이트 데이터를 파일에 쓰는 클래스이다.  

☑️ JSONObject에 값을 넣으면 아래와 같은 형태로 저장된다.  

{"sender":"ss","type":"TALK","message":"안녕하세요","roomId":"uuid"}  

<br>

☑️ FileWriter 생성자에서 true를 주면 기존 파일에 이어서 쓴다. (`new FileWriter("url", true)`)

☑️ 파일이 비어 있는 경우에는 쉼표 없이 저장한다. 이미 내용이 있다면 JSON 앞에 쉼표를 붙인다.  

<br><br>

#### 저장 형식 비교

```java
// 1번 방식
file.write(json.toJSONString());
```
![image](https://user-images.githubusercontent.com/74857364/217083003-d48b31a8-cec2-450b-b555-d9443d9348f0.png)

<br>

```java
// 2번 방식 
file.write(gson.toJson(json));
```
![image](https://user-images.githubusercontent.com/74857364/217083434-2408c49d-c40f-4a26-8f22-8cd4bed99194.png)

<br>

나는 가독성이 좋아서 2번 방식을 사용했다가 채팅 내용이 많아질 수록 로딩하는데 시간이 오래 걸리므로

1번 방식으로 진행했다.

<br><br><br><br>

### getFile (server)

이 부분이 가장 오래 걸렸다.  

목표는 파일 내용을 `[{}, {}]` 형태로 만들어서 내려주는 것이었다.  

처음에는 FileReader로 파일을 읽은 뒤 JSONParser로 바로 파싱하려 했다.  

```java
JSONParser parser = new JSONParser();
Object object = parser.parse(reader);
```

하지만 저장된 형태가 `{},{}` 이다 보니 파싱 단계에서 오류가 발생했다.  

그래서 파일 전체를 문자열로 읽은 뒤 직접 대괄호를 붙이는 방식을 선택했다.  

```java
public Object readFile(String roomId) {
    try {
        String str = Files.readString(Paths.get(chatUploadLocation + "/" + roomId + ".txt"));
        JSONParser parser = new JSONParser();
        Object obj = parser.parse("[" + str + "]");
        return obj;
    } catch (NoSuchFileException e) {
        throw new FileNotFoundException();
    } catch (IOException | ParseException e) {
        e.printStackTrace();
        return null;
    }
}
```

Files 클래스를 이용하면 텍스트 파일 내용 전체를 List나 배열, String에 쉽게 담을 수 있다고 한다.

Files 클래스는 Java 7부터 사용 가능하다. (모든 메서드가 static) 

<br>

*FileNotFoundException은 CustomException

[Custom Exception](https://haedal-uni.github.io/posts/Custom-Exception/) 참고

<br><br><br><br>

### getFile (client)

클라이언트에서는 for문이 계속 여러 번 실행되는 문제가 있었다.  

length는 정상인데 for문만 반복 실행되었다.  

그래서 flag 변수를 사용해 무한 실행을 막았다.  

```js
let flag = false;

function getFile() {
    if (flag) {
        return;
    }
    flag = true;

    $.ajax({
        type: "GET",
        url: `/room/enter/` + roomId + '/' + roomName,
        success: function(response) {
            for (let i = 0; i < response.length; i++) {
                onMessageReceived(response[i]);
            }
        }
    });
}
```

```js
function onMessageReceived(payload) {
    let message;
    try {
        message = JSON.parse(payload.body);
    } catch (SyntaxError) {
        message = payload;
    }
}
```

JSON.parse : JSON 문자열 → JavaScript 객체  

JSON.stringify : JavaScript 객체 → JSON 문자열 

이미 JSON 형태로 내려받았기 때문에 parse를 그대로 쓰면 에러가 발생한다.  

그래서 try-catch로 분기 처리했다.  

<br><br><br><br>

### 실행 화면

![AC_ 20230207-053150](https://user-images.githubusercontent.com/74857364/217095681-612beec0-57e6-45f0-994b-651c7b86fb4c.gif)

<br><br><br><br>

*reference*           
[파일 입출력 클래스: FileReader, FileWriter, FileInputStream, FileOutputStream](https://mainichibenkyo.tistory.com/19)                                
[[Java] 텍스트 파일 읽기 ( FileReader, BufferedReader, Scanner, Files )](https://hianna.tistory.com/587)                                    
