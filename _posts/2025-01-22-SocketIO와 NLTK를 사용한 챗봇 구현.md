---
categories: Eng-Project
tags: [log, Python, Flask, SocketIO, NLTK]
---

# Python-SocketIO와 NLTK를 사용한 챗봇 구현
Python-SocketIO를 활용하여 실시간 양방향 통신을 구현하고 NLTK를 사용한 간단한 챗봇 기능을 추가했다.

Spring과 WebSocket으로 채팅을 구현한 경험이 있어 이를 비교하면서 적용하니 이해가 더 쉬웠다.

Spring WebSocket 경험이 있다면 SocketIO을 이해하는 데 도움이 될 것이다.

<br>

![AC_ 20250126-172901](https://github.com/user-attachments/assets/c931e6c0-c66e-4694-b8c6-7f6c5b7e38d7)

<br><br>  

## 1. 기본 설정  
#### 가상환경  
```bash
python -m venv venv
source venv/Scripts/activate
deactivate
```

<br><br>

#### 라이브러리 설치  
```bash
pip install nltk
pip install python-socketio
pip install eventlet
```

<br><br><br><br>

## 2. Python-SocketIO      
Python-SocketIO는 Python 애플리케이션에서 Socket.IO 서버와 client를 쉽게 구현할 수 있게 해주는 라이브러리다. 

Socket.IO는 실시간 웹 애플리케이션을 위한 JavaScript 라이브러리 및 프로토콜로 

WebSocket을 기반으로 하면서 WebSocket이 제공하지 않는 기능들(ex: 자동 재연결, 방(broadcasting), room)을 추가로 지원한다.   

<br><br><br> 

### 주요 기능
- 양방향 통신 : 서버와 클라이언트 간의 실시간 양방향 통신을 구현할 수 있다.

- 네임스페이스 및 room 지원 : 메시지의 범위를 제한하기 위해 네임스페이스와 room을 사용할 수 있다.

- 다양한 클라이언트 지원 : JavaScript, iOS, Android 등 다양한 client와 통신이 가능하다.

- 비동기 서버 지원 : `asyncio`, `gevent`, `eventlet`과 같은 Python의 비동기 프레임워크와 통합하여 사용할 수 있다.

- 내장된 WSGI 애플리케이션 : 별도의 웹 서버 없이도 독립적으로 실행할 수 있는 WSGI 애플리케이션을 제공한다.

<br><br>
  
#### 네임스페이스와 Room   

내가 이해한대로 정리를 하자면  

네임스페이스 : Spring에서 WebSocket 경로를 지정하는 endpoint와 유사  
```java
registry.addEndpoint("/coco").setAllowedOriginPatterns("*").withSockJS();
registry.addEndpoint("/coco/chat").setAllowedOriginPatterns("*").withSockJS();
```
Room : Spring에서 UUID를 생성해 사용자별 roomId를 관리하는 방식과 유사

Python에서는 client를 그룹화해 개별 Room을 제공 (기본적으로 세션 ID(`sid`)를 사용하여 구분)  

<br><br><br> 

#### spring의 websocket과 차이점
**spring** : `@MessageMapping`을 통해 메시지 경로 연결
```js
client.send("/app/chat.sendMessage", {}, JSON.stringify(chatMessage));
```
```java
@MessageMapping("chat.sendMessage")
public void sendMessage(@Payload ChatMessage chatMessage) {
    rabbitTemplate.convertAndSend(sendExchange, "room." + roomId, chatMessage);
}
```

<br><br>

**Python-SocketIO** : 이벤트 이름으로 동작   
```js
socket.emit("my_message", { command: selectedCommand, word: message });  
```
```python
@sio.event
def my_message(sid, data):
    sio.emit('reply', {"response": response})
```
```py
socket.on("reply", function (data) {
})
```
<br><br>

Spring에서는 uuid를 생성해 roomId로 개별 채팅을 구분했지만 

Python에서는 기본 제공하는 세션 ID(`sid`)로 개별 채팅 가능했다. (별도의 설정x) 

`sio.emit('message', data, room=sid)`

<br><br>

spring websocket에서는 사용자가 채팅을 종료할 때 모달에서 진행되다보니 새로고침이 아닌

닫기 버튼이나 다른 화면을 클릭해 채팅 모달만 없어지게 하는 경우

종료 감지가 되지 않아 disconnect라고 client에서 값을 줘야 했는데

`Python-SocketIO`에서는 Client에서 따로 작성하지 않아도 disconnect가 처리되었다.    


<br><br><br><br>

### 구현 : Python-SocketIO
`socket.py`으로 client와 연결 후 `nltk_command.py`를 통해 사용자가 원하는 결과값을 제공하는 것으로 구현했다.   

```py            
src          
|__ socket.py # SocketIO 서버 구현                     
|__ nltk_command.py # NLTK 관련 명령 처리         
```

<br><br>

#### import ... from ... 
```python
import socketio
sio = socketio.Server(cors_allowed_origins="*")
app.wsgi_app = socketio.WSGIApp(sio, app.wsgi_app)

#########################################################

from socketio import Server, WSGIApp
sio = Server(cors_allowed_origins="*")
app.wsgi_app = WSGIApp(sio, app.wsgi_app)
```
`import ~` 라이브러리 전체를 가져온다. (모든 것을 사용할 수 있지만 필요 없는 항목까지 로드) 

`from ... import ...` : 라이브러리에서 특정 항목만 가져온다. (필요한 물건(ex. WSGIApp)만 꺼내 사용)

`from a import b` : 모듈 a에서 특정 항목 b만 가져온다.

*후자는 socketio 모듈 전체가 필요하지 않고 특정 클래스(Server, WSGIApp)만 사용해 from을 사용한 것.

<br><br><br> 

#### 이벤트 핸들링 
`@sio.on('event_name')` : 명시적으로 이벤트 이름 지정(이벤트 이름이 함수 이름과 다를 경우)  

`@sio.event` : 함수 이름을 이벤트 이름으로 자동 설정(이벤트 이름과 함수 이름이 동일할 경우)  

```py
@sio.on('message')
def handle_message(sid, data):
    print("Message received")

@sio.event
def connect(sid, environ):
    print(f"Client connected: {sid}")

@sio.event
def my_message(sid, data):
    sio.emit('reply', {"response": "Message Received"}, room=sid)

@sio.event
def disconnect(sid):
    print(f"Client disconnected: {sid}")
```
- 연결(`connect`): `sid`(client의 세션 ID)와 `environ`(환경 정보) 확인  
  - `environ` : client 연결과 관련된 HTTP 환경 정보 (IP 주소, 헤더 등)  

- 메시지 수신(`my_message`): 데이터 처리      

- 연결 종료(`disconnect`): 자동 감지

<br>

*`connect`, `disconnect`와 같이 이미 정의된 event의 파라미터는 필수로 작성해야한다.   

(`connect`는 연결된 클라이언트의 정보를 처리하기 위해 `sid`와 `environ`이 필요)   


<br><br><br> 

#### 비동기 서버 실행 
RuntimeError: You need to use the eventlet server

Flask의 기본 실행 방법인 `app.run()`을 사용했더니 오류가 떴다. 

<br>

Flask의 기본 실행 방식(`app.run()`)은 WebSocket 통신을 지원하지 않는다.   

→ `Eventlet`를 적용 (비동기 지원을 위해 이벤트 기반 웹 서버를 실행)

`eventlet.wsgi.server(eventlet.listen(('0.0.0.0', 5000)), app)`

<br><br><br> 

#### room
`sio.emit('reply', {"response": response}, room=sid)`  

Socket.IO는 자동으로 클라이언트를 세션 ID(`sid`)에 해당하는 기본 Room에 연결한다. (따로 설정x)   

따라서 `room=sid`로 특정 클라이언트에게만 메시지를 전송할 수 있다.   

<br><br>

Room을 별도로 생성하고 관리하려면 아래와 같이 구현하면 된다. 
```py 
room_id = generate_room_id()  # Room ID 생성(임의의 uuid 생성 함수)  
sio.enter_room(sid, room_id)  # Room에 join 
sio.emit('message', data, room=room_id)  # Room에 메시지 전송
```
<br>

room1은 챗봇, room2는 일반 채팅방과 같이 분리를 할 수 있다. 

```py
sio.emit('message', data, room='room1')  # 'room1'에만 전송
```

*client는 특정 room에 지속적으로 연결 되어야 하는 경우(ex. 채팅 기록 저장) roomId를 기억해야한다. 

<br>

나는 단순 챗봇이고 굳이 채팅 기록을 남겨야 할 이유가 없어서 `sid`를 활용하기로 했다.  

따라서 client에서 기억할 필요가 없다.   

<br><br><br> 

#### 전체 코드 

```python
# 기본 설정
import socketio
from flask import Flask
from src.nltk_command import nltk_command
import eventlet

sio = socketio.Server(cors_allowed_origins="*")  # 모든 origin에서 연결 허용
app = Flask(__name__)
app.wsgi_app = socketio.WSGIApp(sio, app.wsgi_app) # Flask와 WebSocket 기능을 결합

# 이벤트 핸들링
@sio.event
def connect(sid, environ): # client 연결
    print(f"environ : {environ}")
    print(f"Client connected: {sid}")

@sio.event
def my_message(sid, data):
    command = data.get("command")
    word = data.get("word")
    if command=="lemmatizer" :
        pos = data.get("pos")
        response = process_command(command, word, pos)
    else :
        response = process_command(command, word)

   # 특정 client에게 메시지 전송  
    sio.emit('reply', {"option" : command, "response": response}, room=sid)

@sio.event
def disconnect(sid): # client 연결 종료 
    print(f"Client disconnected: {sid}")

# 서버 실행 
if __name__ == '__main__':
    eventlet.wsgi.server(eventlet.listen(('0.0.0.0', 5000)), app)
```

<br><br> <br><br> 

---

## 3. nltk   
`n`: noun, 
`v`: verb, 
`a`: adjective, 
`s`: adjective satellite, 
`r`: adverb

```py
from nltk.corpus import wordnet as wn
from nltk.stem import WordNetLemmatizer
import json

# 동의어 
def get_synonyms(word):
    synsets = wn.synsets(word)
    synonyms = []
    for synset in synsets:
        synonyms.extend([lemma.name() for lemma in synset.lemmas()])
    return list(set(synonyms))  # 중복 제거

# 예문
def get_examples(word):
    synsets = wn.synsets(word)
    examples = []
    for synset in synsets:
        examples.extend(synset.examples())  # .examples()는 문자열을 반환
    return list(set(examples))

# 정의
def get_definition(word):
    synsets = wn.synsets(word)
    definitions = []
    for synset in synsets:
        definitions.append(synset.definition())  # 단일 문자열을 추가
    return list(set(definitions))

# 단어의 품사(동사, 명사 등)
def get_pos(word):
    synsets = wn.synsets(word)
    pos_tags = []
    for synset in synsets:
        pos_tags.append(synset.pos())  
    return list(set(pos_tags))

# 표제어 추출 
def get_lemmatizer(word, pos):
    lemmatizer = WordNetLemmatizer()
    return [lemmatizer.lemmatize(word, pos)]

def process_command(command, word, pos=None):
    global response
    if command == "synonyms":
        response = get_synonyms(word)
    elif command == "examples":
        response = get_examples(word)
    elif command == "definition":
        response = get_definition(word)
    elif command == "pos":
        response = get_pos(word)
    elif command == "lemmatizer" and pos:
        response = get_lemmatizer(word, pos)
    return json.dumps(response)
```

<br><br> 

```python
lemmas_list = []
for lemma in synset.lemmas():
    lemmas_list.append(lemma.name())

→ lemmas_list = [lemma.name() for lemma in synset.lemmas()]
``` 

`synonyms.extend([lemma.name() for lemma in synset.lemmas()])`

synonyms 리스트에 lemma.name()으로 생성된 리스트를 항목별로 추가한다.

<br><br><br> 

## 4. 문제 해결
#### append와 extend

- 동의어와 예문은 list 형태로 반환되므로 extend를 사용해 개별 요소를 추가

- 정의와 품사는 단일 문자열로 반환되므로 append 사용

<br>

`append`는 리스트에 하나의 객체(item)를 그대로 추가하고 `a.append([1, 2]) → a = [[1, 2]]`

`extend`는 리스트에 다른 요소를 개별적으로 추가한다. `a.extend([1, 2]) → a = [1, 2]`


```py
synonyms = []
synonyms.append(["example"])  # [['example']]
synonyms.extend(["example"]) # ['example']
```

<br>

따라서 동의어와 예문을 `append()`로 사용하게 된다면 `[["a"],["b"]]`와 같이 리스트 안에 리스트가 생기기 때문에

개별적으로 리스트에 추가하는 `extend()`를 사용했다. 

<br><br><br>  

#### TypeError: Object of type set is not JSON serializable
```py
import json

return json.dumps(response) 
```
반환 데이터를 `json.dumps()`로 해결    

<br><br><br>  

#### list와 `[]`  
```py
# 품사 알기
def get_lemmatizer(word, pos):
    lemmatizer = WordNetLemmatizer()
    return [lemmatizer.lemmatize(word, pos)]
```
처음엔 return값을 `list(lemmatizer.lemmatize(word, pos))`로 작성했더니 

hello를 입력했을 때 반환 값이 `["h", "e", "l", "l", "o"]` 로 출력되었다. 

따라서 hello로 출력하기 위해 `list()` 대신 `[]`를 사용해 `[lemmatizer.lemmatize(word, pos)]`로 작성했다.

<br>

`list()` 함수는 문자열을 문자 단위로 쪼개어 리스트로 변환한다. `hello → ['h', 'e', 'l', 'l', 'o']` 

`[]`는 원하는 값을 리스트의 요소로 직접 추가한다. `hello → ['hello']` 

참고로 `list()`로 작성하는 것보다 `[]`(대괄호)를 사용하는게 성능이 더 좋다고 한다.          

<br><br><br><br> 

---

**REFERENCE**             
wordnet                           
- [wordnet](https://github.com/nltk/wordnet)          
- [워드넷(wordnet)을 재밌게 활용하기](https://teteker.tistory.com/12)          
- [[Python] WordNet으로 각 단어별 연관어(연상어) 추출하기](https://sens.tistory.com/455)
- [02-03 어간 추출(Stemming) and 표제어 추출(Lemmatization)](https://wikidocs.net/21707)        
  
python    
- [[Python] list append()와 extend() 차이점](https://m.blog.naver.com/wideeyed/221541104629)              
- [Python, `[]`(대괄호)와 list()의 차이](https://conansjh20.tistory.com/79)

Socket.IO               
- [python-socketio: 파이썬 Socket.IO 서버 및 클라이언트](https://wikidocs.net/236800)
- [[Flask] 플라스크로 채팅 기능을 구현해보자](https://blog.naver.com/shino1025/222179697262)            
- [python-socketio로 Socket.io서버를 생성하고 Django와 통합하여 배포하기](https://velog.io/@stresszero/python-socketio)           
