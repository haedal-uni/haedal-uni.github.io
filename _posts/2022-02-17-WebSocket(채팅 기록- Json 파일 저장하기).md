---
categories: Project
tags: [spring, WebSocket, Chat]
---

# Json 파일 저장하기
## 문제
채팅 대화내용을 보여주기 위해서 처음에는 클라이언트에서 sessionStorage로 저장하려고 했는데 생각보다 쉽지 않았다.

- type이 "TALK"인 경우에만 저장하기                
→ 대화가 1번 이루어지면 저장이 되지 않고 2번 작성한 경우 제일 먼저 작성한 것만 저장이 된다. (이런 식으로 하나씩 저장이 안된다.)                                    
그래서 저장이 무조건 되게 if문을 추가해서 sessionStorage의 길이와 실제 작성한 채팅 횟수가 같지 않으면 다시 저장하게끔 설정했더니        
무한루프가 되어버렸다.                 

- for문으로 대화 저장된 값들을 가져오는데 순서가 뒤죽박죽이다.              
→ admin도 같이 sessionStorage를 저장하면서 뒤죽박죽으로 나오는 것 같다.          

<br>

그래서 서버에서 저장해서 채팅 기록을 띄워주는 방향으로 바꿨다.

db에 저장하는 것보다 파일에 저장하는게 더 효율적일 것 같아서 파일에 저장하는 방식으로 진행했다.

<br>

채팅을 띄워주는 형식이 json으로 받기 때문에 저장도 json형식으로 해야겠다고 생각해서 JSONObject를 사용했다.

그런데 문제가 생겼다. 아래와 같이 글을 작성하면 

![image](https://user-images.githubusercontent.com/74857364/216782488-30d8260f-7504-4a8e-9306-42b4033404bf.png)

파일에 제대로 저장이 되는것까지 확인됐다.

<br><br>

![image](https://user-images.githubusercontent.com/74857364/216782500-07c66f32-e6b9-4b74-96ce-a47b77981ef5.png)

오! 잘 되는건가?😃 싶었는데 글을 한번 더 써보고 문제를 발견했다.

<br><br>

![image](https://user-images.githubusercontent.com/74857364/216782540-fe5bd68f-9259-4038-b68d-7211185ee604.png)

구분자가 없다.😭 결과 이미지를 올린 블로그만 보니 다 하나씩만 하고 올려뒀다. 하나만 하고 올려둔건가🤔

<br><br>

*Unexpected token LEFT BRACE({) at position 95.*

라는 에러가 뜨는데 "{}" 끝나고 바로 "{" 로 시작되서 에러가 뜬 것 같다.

그래서 내가 생각한 방법은 배열 안에 json 방식을 넣는 것이었다.

`[ {}, {} ]` 이런 방식을 생각하고 JSONArray를 사용해서 JSONObject를 넣었다.

그랬더니...

`[{}] [{}]` 이런 형식으로 저장됐다. 하핳😅

![image](https://user-images.githubusercontent.com/74857364/217054305-699a05bd-5711-40a5-83a8-50ab1cd9b568.png)


<br><br><br>

예상보다 엄청난 삽질이 시작되었다. 그래도 결과가 나왔으니 이렇게 글을 정리하게 됐다. 😙

이게 맞는지는 모르겠으나 아래와 같이 작성안하면 내가 원하는 결과가 안나온다.

아무튼 원하는 결과가 나왔으니 정리해본다.

추가된 코드를 설명하다보니 기존에 작성된 코드는 생략되어있는 경우도 있다.

중간에 메소드 전체 코드가 궁금해지거나 다른 설명이 필요하다면 웹소켓 1편 or 2편을 보면 된다.

<br><br>

- 웹소켓 1편 : [기본](https://haedal-uni.github.io/posts/WebSocket/)               
- 웹소켓 2편 : [부가기능](https://haedal-uni.github.io/posts/WebSocket-+%EB%B6%80%EA%B0%80%EA%B8%B0%EB%8A%A5/)               

<br><br><br><br>

---

### setFile (client)
서버에서 `@MessageMapping`으로 메세지를 보낼 때 마다 실행되는 함수에 *setFile*을 실행시켰다.  

메세지를 보내면 바로 file로 저장이 된다.
```js
function sendMessage(event) {
    let chatMessage = {
        roomId: roomId, sender: username, message: messageInput.value, type: 'TALK'
    };
    setFile(chatMessage) // ⬅️ 
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
JSON은 JavaScript Object Notation 의 줄임말로 말 그대로 자바스크립트에서 객체를 표현하는 방법이다.

JAVA에서 JSON을 다룰 때 보통 google의 json-simple을 사용하는 것 같다.

maven을 사용하는 경우 pom.xml에 의존성을 설정하면 되고 그 외는 설치(직접 라이브러리 받아서 classpath 경로에 넣어두셈)를 하라고 하는데

나는 그냥 maven에 적힌 의존성 설정을 그대로 gradle에 적용하면 되지 않나 싶어서 그대로 적용했다.         
(설치하기 귀찮,,)          

```xml
<!-- https://mvnrepository.com/artifact/com.googlecode.json-simple/json-simple -->
<dependency>
    <groupId>com.googlecode.json-simple</groupId>
    <artifactId>json-simple</artifactId>
    <version>1.1.1</version>
</dependency>
```
위가 maven 설정인데 groupId를 보고 gradle에 적용했더니 잘 된다.

<br>

```.gradle
implementation 'com.googlecode.json-simple:json-simple:1.1.1'
```

<br><br>

대부분 maven만 적용 코드가 있고 나머지는 다운받으라고 해서 gradle도 다운받아야 하는건 아닌가 확신이 들지 않을 때 

gradle에 `implementation 'com.googlecode.json-simple:json-simple:1.1.1'` 코드를 없애고 import하면 아래와 같이 뜬다.

![image](https://user-images.githubusercontent.com/74857364/217088955-844a4013-2519-480e-9d0d-77b3c8b1252f.png)

```java
import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;
import org.json.simple.parser.ParseException;
```

그래서 gradle도 의존성만 추가하면 되는 듯하다.

<br><br>

json-simple에서는 java와 json의 값들을 아래와 같이 mapping 해서 사용한다.

Json value | JAVA class | 설명
|:---:|:---:|:---
배열 | java.util.List | JSON에서의 배열인 `[]` 표기법은 List로 표현. (실제로는 List를 구현한 **JSONArray**로 맵핑됨)
객체 | java.util.Map   | JSON에서의 객체인 '{}' 표기법은 key-value 형식인 Map으로 표현. (실제로는 Map을 구현한 **JSONObject**로 맵핑됨)


<br><br><br>

```java
private void saveFile(ChatMessage chatMessage) { // 파일을 저장하는 method
    JSONObject json = new JSONObject();
    json.put("roomId", chatMessage.getRoomId());
    json.put("type", chatMessage.getType().toString());
    json.put("sender", chatMessage.getSender());
    json.put("message", chatMessage.getMessage());

    Gson gson = new GsonBuilder().setPrettyPrinting().create();
    String json1 = gson.toJson(json);
    try { 
        FileWriter file = new FileWriter(chatUploadLocation + "/" + chatMessage.getRoomId() + ".txt", true);
        //뒤에 true 붙이면 덮어쓰기 가능
       
        File file1 = new File(chatUploadLocation + "/" + chatMessage.getSender() + "-" + chatMessage.getRoomId() + ".txt");
        
        if (file1.exists() && file1.length() == 0) {
            file.write(json1);
        } else {
            file.write("," + json1);
        }
        file.flush();
        file.close(); // 연결 끊기
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
☑️ FileOutputStream & FileWriter

파일을 저장할 때 보통 FileOutputStream 이랑 FileWriter를 쓰던데 

**FileWriter**는 문자 데이터를 파일에 쓰는 클래스이고 **FileOutputStream**은 바이트 데이터를 파일로 출력하는 클래스이다.

<br>

☑️ JSONObject

`JSONObject json = new JSONObject();`에 client에서 받아온 값을 넣으면 아래와 같이 저장된다.

{"sender": "ss", "type": "TALK", "message": "안녕하세요", "roomId": "3d41c3ed-8ddb-458d-a248-64fb9fb74e6f"}

<br>

☑️ 내용 덮어쓰기

`FileWriter file = new FileWriter("url", true)` 에서 true를 넘겨주면 원래 있던 파일의 내용에 덮어쓰게 된다.

<br>

☑️ 파일 저장 방식

`File file1 = new File(chatUploadLocation + "/" + chatMessage.getSender() + "-" + chatMessage.getRoomId() + ".txt");`

위 코드를 작성한 이유는 쉼표(,)를 넣기 위함이다.

파일이 없다면 처음 작성한 것이므로 처음에는 쉼표(,)를 작성하지 않고 그 뒤부터 json을 넣기 전에 쉼표(,)를 넣는다.


  
<br><br>

#### 저장하는 형식

코드에 따라서 이쁘게 파일을 작성할 수 있다. 아래 사진을 첨부했으니 차이를 알 수 있을 것이다.

나는 2번 방식을 사용했다.

**1.**
```java
try {
    FileWriter file = new FileWriter(chatUploadLocation + "/" + chatMessage.getRoomId() + ".txt", true);
    File file1 = new File(chatUploadLocation + "/" + chatMessage.getRoomId() + ".txt");
    if (file1.exists() && file1.length() == 0) {
        file.write(json.toJSONString());
    } else {
        file.write("," + json.toJSONString());
    }
    file.flush();
    file.close(); // 연결 끊기
} catch (IOException e) {
    e.printStackTrace();
}
```
![image](https://user-images.githubusercontent.com/74857364/217083003-d48b31a8-cec2-450b-b555-d9443d9348f0.png)

<br><br>

**2.**
```java
Gson gson = new GsonBuilder().setPrettyPrinting().create();
String json1 = gson.toJson(json);
try {
    FileWriter file = new FileWriter(chatUploadLocation + "/" + chatMessage.getRoomId() + ".txt", true);
    File file1 = new File(chatUploadLocation + "/" + chatMessage.getRoomId() + ".txt");
    if (file1.exists() && file1.length() == 0) {
        file.write(json1);
    } else {
        file.write("," + json1);
    }
    file.flush();
    file.close(); // 연결 끊기
} catch (IOException e) {
    e.printStackTrace();
}
```    

![image](https://user-images.githubusercontent.com/74857364/217083434-2408c49d-c40f-4a26-8f22-8cd4bed99194.png)

<br><br><br><br>

### getFile (server)
이 부분이 가장 오래 걸렸다.

어떻게 하면 값을 `[{}, {}]` 형식으로 가져올까 고민했다.

처음에는 FileReader를 사용해서 파일을 읽어온 다음 list를 생성해서 파일 전체 값을 넣으면 되지 않을까 싶었는데
```java
JSONParser parser = new JSONParser();
Object object = parser.parse(reader);
```
JSON 데이터를 넣어서 Object로 만들어주는 과정에서 오류가 났다.

`{}, {}` 형식으로 저장을 해서 콤마가 문제고 콤마를 없애면 "{"가 문제였다.

<br><br>

```
{
    "chatMessage" : [
        { 
        "name" : "hi"
        }
    ]
}
```
이런 식으로 저장해서 가져오는 방법도 있는데 꼭 저렇게 저장해야하나 싶어서 다른 방법을 찾았다.

<br><br>

찾아보니 Files 클래스를 이용하면, 텍스트 파일 내용 전체를 List나 배열, String에 쉽게 담을 수 있다고 한다.

*java.nio.file.Files 클래스는 Java 7 이후부터 사용할 수 있다.            
Files 클래스는 모두 static 메소드로 구성이 되어있다.        

<br><br>

그래서 이를 활용해서 한번에 파일을 읽고 list에 담았다. 

그런데.. 무슨 이유인지는 모르겠으나 서버에서는 분명 `[{}, {}]` 형태로 보내는데 client에서는 `{}, {}`로 받아버린다.😭

<br><br>

그래서 결국 아래와 같은 코드로 구현했다.

```java
public Object readFile(String roomId) {
    try {
        String str = Files.readString(Paths.get(chatUploadLocation + "/" + roomId + ".txt"));
        JSONParser parser = new JSONParser();
        Object obj = parser.parse("[" + str + "]");
        return obj;
    } catch (NoSuchFileException e){
        throw new FileNotFoundException();
    }catch (IOException | ParseException e) {
        e.printStackTrace();
        return null;
    }
}
```
☑️ FileNotFoundException은 CustomException이다. 

CustomException은 해당 글 [Custom Exception](https://haedal-uni.github.io/posts/Custom-Exception/)을 참고한다.

<br><br><br><br>

### getFile (client)
왜인지는 아직도 모르겠으나 for문이 무한루프로 계속 돌아간다.

length는 무한루프가 아닌데도 해당 for문을 여러 번 돈다.

그래서 isRun 변수로 무한루프를 막았다. 

```js
let isRun = false;
function getFile() {
    if (isRun == true){
        return;
    }
    isRun = true;
    $.ajax({
        type: "GET", url: `/room/enter/` + roomId + '/' + roomName, contentType: false, processData: false, success: function(response) {
            for (let i = 0; i < response.length; i++) {
                onMessageReceived(response[i])
            }
        }
    })
}

function onMessageReceived(payload) { // 메세지 받기
    let message = JSON.parse(payload.body);
}
```

onMessageReceived에서 값을 처리하기 위해선 [object Object] 로 보내야 하므로 

`JSON.stringify(response)`으로 작성을 하지 않았다.

<br>

onMessageReceived(response)에서 parse를 사용하는데 parse는 string 객체를 json 객체로 변환시켜준다.

하지만 나는 data를 body로 보내지도 않고 미리 json으로 저장을 했기 때문에 `JSON.parse(payload.body);` 코드를 사용할 필요가 없었다.

그래서 그냥 response를 보내면 SyntaxError 에러가 떴다. 이를 해결하기 위해 try ~ catch문을 사용했다.

```js
function onMessageReceived(payload) { // 메세지 받기
    let message;
    try {
        message = JSON.parse(payload.body);
    } catch (SyntaxError) {
        message = payload;
    }
}
```

- JSON.parse() : string 객체를 json 객체로 변환
- JSON.stringify() : json 객체를 String 객체로 변환

<br><br><br><br>

### 실행 화면

![AC_ 20230207-053150](https://user-images.githubusercontent.com/74857364/217095681-612beec0-57e6-45f0-994b-651c7b86fb4c.gif)

<br><br><br><br>

*reference*           
[파일 입출력 클래스: FileReader, FileWriter, FileInputStream, FileOutputStream](https://mainichibenkyo.tistory.com/19)             
[Javascript JSON.parse(), JSON.stringify() 사용하는법](https://ithub.tistory.com/54)               
[[JAVA] JAVA에서 JSON 데이터 다루기. GOOGLE의 JSON-SIMPLE 사용 방법](https://dololak.tistory.com/625)               
[[Java] 텍스트 파일 읽기 ( FileReader, BufferedReader, Scanner, Files )](https://hianna.tistory.com/587)              
[AJAX 중복요청, 중복클릭, 중복호출 막는 여러가지 방법들](https://coding-restaurant.tistory.com/227)                  
[]()                   
