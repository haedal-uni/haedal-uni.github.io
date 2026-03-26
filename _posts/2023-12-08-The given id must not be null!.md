---
categories: Error
tags: [Java]
---

# The given id must not be null!

The given id must not be null! 이라는 오류가 떴다.

결론만 말한다면 `@ModelAttribute` 대신 `@RequestBody`를 사용하면 된다.

<br>

추가적으로 request header에 

`data : JSON.stringify(form_data)`와 같이 json 형태로 데이터를 넘겨줘야 하며,

`'contentType': 'application/json'` 을 담아준다.

<br><br><br> 

### json 형태로 데이터를 넘겨주지 않는다면

front : 400 (Bad Request)          

back : *Resolved [org.springframework.http.converter.HttpMessageNotReadableException:*                      
*JSON parse error: Cannot deserialize value of type*        

과 같은 에러를 볼 수 있다.

<br><br><br>

### 'contentType': 'application/json' 을 header에 담아주지 않는다면

front : *415 (Unsupported Media Type)*

back : *Resolved [org.springframework.web.HttpMediaTypeNotSupportedException:*       
*Content type 'text/plain;charset=UTF-8' not supported]*


<br><br><br><br><br>

## @RequestBody
이제 왜 `@RequestBody`로 작성을 해야했는지 원인을 파악해본다.

그리고 왜 `@RequestBody`를 작성하면서 header의 data와 contentType을 작성해야했는지 

`@RequestBody`에 대해서 공부해보면 그 이유를 파악할 수 있다.

<br><br>

참고로 해당 [블로그](https://tecoble.techcourse.co.kr/post/2021-05-11-requestbody-modelattribute/)를 통해서 충분히 파악할 수 있으며 아래에 해당 글을 정리한 내용이다.      

<br><br>

`@RequestBody` 어노테이션의 역할은 클라이언트가 보내는 HTTP 요청 본문(JSON 및 XML 등)을 Java object로 변환하는 것이다.    

HTTP 요청 본문 데이터는 Spring에서 제공하는 **HttpMessageConverter**를 통해 타입에 맞는 객체로 변환된다.     

<br>

클라이언트에서 서버로 JSON 형식의 데이터를 전송할 때 `@RequestBody`를 사용하면, 

Spring은 해당 JSON 데이터를 Java 객체로 변환하는 과정에서 MappingJackson2HttpMessageConverter를 활용한다.

→ MessageConverter 중 MappingJackson2HttpMessageConverter를 사용한다.

<br><br>               

```java
public class MappingJackson2HttpMessageConverter extends AbstractJackson2HttpMessageConverter {
    // 생략
}
```
MappingJackson2HttpMessageConverter 클래스는 AbstractJackson2HttpMessageConverter 클래스를 상속받아 구현된 클래스다.

<br><br>

```java
public abstract class AbstractJackson2HttpMessageConverter extends AbstractGenericHttpMessageConverter<Object> {

    protected ObjectMapper defaultObjectMapper;

    private Object readJavaType(JavaType javaType, HttpInputMessage inputMessage) throws IOException {
        MediaType contentType = inputMessage.getHeaders().getContentType();
        Charset charset = getCharset(contentType);

        ObjectMapper objectMapper = selectObjectMapper(javaType.getRawClass(), contentType);
        Assert.state(objectMapper != null, "No ObjectMapper for " + javaType);

        boolean isUnicode = ENCODINGS.containsKey(charset.name()) ||
                "UTF-16".equals(charset.name()) ||
                "UTF-32".equals(charset.name());
        try {
            InputStream inputStream = StreamUtils.nonClosing(inputMessage.getBody());
            if (inputMessage instanceof MappingJacksonInputMessage) {
                Class<?> deserializationView = ((MappingJacksonInputMessage) inputMessage).getDeserializationView();
               // 역직렬화 뷰를 고려하여 ObjectReader 생성
                if (deserializationView != null) {
                    ObjectReader objectReader = objectMapper.readerWithView(deserializationView).forType(javaType);
                    if (isUnicode) {
                        return objectReader.readValue(inputStream);
                    }
                    else {
                        Reader reader = new InputStreamReader(inputStream, charset);
                        return objectReader.readValue(reader);
                    }
                }
            }

            // 역직렬화 뷰가 없는 경우 기본적인 역직렬화 수행
            if (isUnicode) {
                return objectMapper.readValue(inputStream, javaType);
            }
            else {
                Reader reader = new InputStreamReader(inputStream, charset);
                return objectMapper.readValue(reader, javaType);
            }
        }
        catch (InvalidDefinitionException ex) {
            throw new HttpMessageConversionException("Type definition error: " + ex.getType(), ex);
        }
        catch (JsonProcessingException ex) {
            throw new HttpMessageNotReadableException("JSON parse error: " + ex.getOriginalMessage(), ex, inputMessage);
        }
    }
}
```

내부적으로 ObjectMapper를 통해 클라이언트가 보낸 JSON 값을 Java 객체로 역직렬화하는 것을 알 수 있다.          

역직렬화란 생성자를 거치지 않고 리플렉션을 통해 객체를 구성하는 메커니즘이라고 이해하면 된다.    

역직렬화가 가능한 클래스들은 기본 생성자가 항상 필수다. 

👉🏻 역직렬화 과정에서 객체를 재구성하고 필드를 채우기 위해 기본 생성자가 필요하기 때문

<br>

따라서 `@RequestBody`에 사용하려는 Dto가 기본 생성자를 정의하지 않으면 데이터 바인딩에 실패한다.    

<br>

JSON으로 값을 보내기 위해서는 request header의 Content-Type을 application/json으로 설정하고, 

request body에는 실제 JSON 형식의 데이터를 포함해야 한다.           

<br><br><br>

💡 

직렬화(serialize)란 자바 언어에서 사용되는 Object 또는 Data를 

다른 컴퓨터의 자바 시스템에서도 사용 할 수 있도록 바이트 스트림(stream of bytes) 형태로 

연속적인(serial) 데이터로 변환하는 포맷 변환 기술을 일컫는다. 

역직렬화는(Deserialize)는 바이트로 변환된 데이터를 원래대로 자바 시스템의 Object 또는 Data로 변환하는 기술이다.

<br>

* 바이트 스트림 이란?

스트림은 클라이언트나 서버 간에 출발지 목적지로 입출력하기 위한 데이터가 흐르는 통로를 말한다.

자바는 스트림의 기본 단위를 바이트로 두고 있기 때문에, 

네트워크, 데이터베이스로 전송하기 위해 최소 단위인 바이트 스트림으로 변환하여 처리한다.

*Serialization* `Object → Stream of Bytes → [File, Data Base, Memory] → Stream of Bytes → Object`  *Deserialization*

<br>

[자바 직렬화(Serializable) - 완벽 마스터하기](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EC%A7%81%EB%A0%AC%ED%99%94Serializable-%EC%99%84%EB%B2%BD-%EB%A7%88%EC%8A%A4%ED%84%B0%ED%95%98%EA%B8%B0)

<br><br><br><br>

### How Jackson ObjectMapper Matches JSON Fields to Java Fields

기본적으로 Jackson은 JSON 필드의 이름을 Java 객체의 getter 및 setter 메소드와 일치시켜 

JSON 객체의 필드를 Java 객체의 필드에 매핑한다.    

Jackson은 getter 및 setter 메서드 이름의 "get" 및 "set" 부분을 제거하고 나머지 이름의 첫 번째 문자를 소문자로 변환한다.     

<br>

예를 들어, brand 라는 JSON 필드는 `getBrand()` 및 `setBrand()`라는 Java getter 및 setter 메서드와 일치한다.        

EngineNumber라는 JSON 필드는 `getEngineNumber()` 및 `setEngineNumber()`라는 getter 및 setter와 일치한다.   

JSON 객체 필드를 Java 객체 필드와 다른 방식으로 일치시켜야 하는 경우 

사용자 정의 직렬화 및 역직렬화를 사용하거나 많은 Jackson Annotations 중 일부를 사용해야 한다.       

<br>

<img width="941" height="267" alt="Image" src="/assets/img/posts/0da74268-66c1-4c11-9815-8b21921e68c3.png" /> 

[Jackson ObjectMapper](https://jenkov.com/tutorials/java-json/jackson-objectmapper.html)        


<br><br>

```java
@PostMapping("/comment")
public CommentResponseDto postComment(@RequestBody CommentRequestDto commentDto) {
    return commentService.postComment(commentDto);
}
```
따라서 `@RequestBody`를 사용할 `CommentRequestDto`는 해당 필드를 바인딩할 setter 혹은 getter가 있어야 한다.

또한, 역직렬화를 위해 기본 생성자는 필수다.

<br><br><br><br><br>

### 정리

Spring에서 HTTP 요청 본문 데이터를 처리하기 위해 *HttpMessageConverter* 인터페이스가 사용된다.      

*HttpMessageConverter*는 HTTP 요청과 응답의 메시지 변환을 담당하는 인터페이스로, 다양한 데이터 타입 간의 변환을 처리한다.          

<br>

*MappingJackson2HttpMessageConverter*는 *HttpMessageConverter* 인터페이스를 구현한 구체적인 클래스 중 하나로, 

JSON 데이터를 Java 객체로 역직렬화하거나 Java 객체를 JSON으로 직렬화하는 역할을 수행한다.        

내부적으로 *MappingJackson2HttpMessageConverter*는 Jackson 라이브러리의 ObjectMapper를 활용하여 

JSON 값을 Java 객체로 역직렬화한다.   

<br>

ObjectMapper를 통해 역직렬화가 이루어지므로 기본 생성자가 필수다.

ObjectMapper는 Java 객체의 필드와 JSON 데이터를 매핑할 때 getter와 setter를 활용한다.   

<br><br><br><br>

*reference*    
[@RequestBody vs @ModelAttribute](https://tecoble.techcourse.co.kr/post/2021-05-11-requestbody-modelattribute/)               
[[HTTP 에러] 415 Unsupported Media Type (@RequestBody 관련)](https://velog.io/@jyleedev/415-Unsupported-Media-Type)             
[[Spring] Content type 'text/plain;charset=UTF-8' not supported](https://developer-ping9.tistory.com/414)                


