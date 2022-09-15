## Controller

나는 Controller를 이전에 작성한 방법처럼 객체를 return했다.                                            
```java
    @PostMapping("/registry")
public Board saveBoard(@RequestBody BoardDto boardDto){
  return boardService.saveBoard(boardDto);
  }
```
<br>

그런데 내가 tdd를 하면서 본 영상에서는 아래 코드처럼 ResponseEntity<T>를 사용했다.                       

```java
    @PutMapping("/music/modification")
    public ResponseEntity<MusicResponseDto> putMusic(@RequestBody MusicDto musicDto) {
        MusicResponseDto musicResponseDto = musicService.modifyMusic(musicDto);
        return ResponseEntity.status(HttpStatus.OK).body(musicResponseDto);
    }
```
그래서 ResponseEntity에 대해 알아봤다.

<br>
<br>

#### ResponseEntity

**ResponseEntity** : 응답 자체의 독립된 값이나 표현 형태                       

Spring Framework에서 제공하는 클래스인 HttpEntity<T>를 상속받고 있으며, RestTemplate 및 @Controller 메서드에 사용하고 있다.                       

HttpEntity는 HTTP 요청(Request) 또는 응답(Response)에 해당하는 HttpHeader와 HttpBody를 포함하는 클래스이다.                       

HttpEntity 클래스를 상속받아 구현한 클래스가 RequestEntity, ResponseEntity 클래스이다.                       

ResponseEntity는 사용자의 HttpRequest에 대한 응답 데이터를 포함하는 클래스이다.

따라서 HttpStatus, HttpHeaders, HttpBody를 포함한다.

ResponseEntity는 StatusField를 가지고 있기 때문에 상태코드는 필수적으로 포함해줘야 한다.

<br>

##### 그런데 왜 dto를 사용할까?
```java
ResponseEntity<MusicResponseDto>
```
- 엔티티 내부 구현을 캡슐화할 수 있다.                       

엔티티란 도메인의 핵심 로직과 속성을 가지고 있고, 실제 DB의 테이블과 매칭되는 클래스이다.                                              

그렇기 때문에 엔티티가 getter와 setter를 갖게 된다면,                        
  controller와 같은 비즈니스 로직과 크게 상관없는 곳에서 자원의 속성이 실수로라도 변경될 수 있다.                       

또한 엔티티를 UI계층에 노출하는 것은 테이블 설계를 화면에 공개하는 것이나 다름없기 때문에                        
  보안상으로도 바람직하지 못한 구조가 된다.                       

따라서 엔티티의 내부 구현을 캡슐화하고 UI계층에 노출시키지 않아야하는 것은                        
  충분히 데이터 전달 역할로 DTO를 사용해야 할 이유로 볼 수 있다.                       

<br>


- 화면에 필요한 데이터를 선별할 수 있다.

애플리케이션이 확장되면 엔티티의 크기는 점차 커지게 된다. 엔티티의 크기만 커질까?

화면도 다양해지고, API 스펙도 더 많아질 것이다.

이때 요청과 응답으로 엔티티를 사용한다면, 요청하는 화면에 필요하지 않은 속성까지도 함께 보내지게 된다.                       

단순히 사용자의 이름만 보여주면 되는 상황에서 필요 이상으로 사용자가 가지고 있는 다른 속성들까지                        
  항상 데이터 전송에 참여하게 되는 것이다.                       

엔티티에서도 `@JsonIgnore`같은 어노테이션을 사용하면 화면으로 보내지 않을 속성을 지정할 수 있는데,                        
  이 역시 근본적인 해결책이 될 수는 없다.                       

<br>

- 순환참조를 예방할 수 있다.

양방향 참조된 엔티티를 컨트롤러에서 응답으로 return하게 되면, 엔티티가 참조하고 있는 객체는 지연 로딩되고,                       

로딩된 객체는 또 다시 본인이 참조하고 있는 객체를 호출하게 된다.                       

이렇게 서로 참조하는 객체를 계속 호출하면서 결국 무한 루프에 빠지게 되는 문제가 발생한다.                       

<br>

- validation 코드와 모델링 코드를 분리할 수 있다.

엔티티 클래스는 DB의 테이블과 매칭되는 필드가 속성으로 선언되어 있고, 복잡한 비즈니스 로직이 작성되어있는 곳이다.                       

그렇기 때문에, 속성에는 @Column, @JoinColumn , @ManyToOne, @OneToOne 등의 모델링을 위한 코드가 추가된다.                       

여기에 만약 @NotNull, @NotEmpty, @NotBlank 등과 같은 요청에 대한 값의 validation코드가 들어간다면                        
  엔티티 클래스는 더 복잡해지고 그만큼 가독성이 저하된다.

<br>
<br>

이 글을 읽고 entity로 코드를 작성했었는데                       

Comment를 새로 추가해서 Dto를 활용한 코드를 작성해봤다.

```java
기존 코드

BoardService → BoardDto → BoardService → BoardServiceImpl → BoardRepository → Board


이후 dto 사용 코드 추가

BoardService → CommentDto, RegistryDto → BoardService → BoardServiceImpl → CommentRepository → Comment
```


[코드 보기](https://github.com/juniorBoard/SpringBoot-Study/tree/main/ch6/Soyeon)

---

### 코드 뜯어보기
  
<br>
  
```java
@WebMvcTest(BoardController.class)
public class BoardControllerTest {
	@Autowired
	private MockMvc mockMvc;
	
	@MockBean
	BoardServiceImpl boardService;

	//http://localhost:8080/registry?boardId={boardId}
	@Test
	@DisplayName("GET test 해보기")
	void getRegistry() throws Exception {
		given(boardService.getBoard("23")).willReturn(new Board("23", "title", "main", "writer"));
		String boardId = "23";
		mockMvc.perform(get("/registry?boardId=" + boardId)).andExpect(status().isOk()).andExpect(jsonPath("$.boardId").exists())
				.andExpect(jsonPath("$.boardTitle").exists()).andExpect(jsonPath("$.boardMain").exists())
				.andExpect(jsonPath("$.boardWriter").exists()).andDo(print());
		verify(boardService).getBoard("23");
	}
}
```

<br><br>
  
***@WebMvcTest(BoardController.class)***

test 하고자 하는 class를 넣어주면 된다.

<br><br>

```java
// BoardController에서 잡고 있는 Bean 객체에 대해 Mock 형태의 객체를 설명해준다.                       
@MockBean BoardServiceImpl boardService;
```
본 코드의 Controller를 보면 BoardService를 자동으로 주입받고 있다.                       

<br><br>

```java
given(boardService.getBoard("23")).willReturn(
    new Board("23", "title", "main", "writer")
);
```

여기서 사용되는 것은 **@MockBean**으로 만든 boardService를 사용한다.

***given*** → 어떠한 상황이 주어지면                       

<br>

getBoard 자체가 Board를 return 해주는 메소드로 설정했기 때문에

willReturn 값도 Board 객체를 return 값으로 넘겨줘야 한다.

<br><br>

```java
String boardId = "23";
```
String boardId 필드 값 생성

<br><br>

```java
// http://localhost:8080/registry/{registryId}
// local test라서 앞부분을 생략시켰다.

        mockMvc.perform(get("/registry?boardId=" + boardId))
          .andExpect(status().isOk())
          .andExpect(jsonPath("$.boardId").exists())
          .andExpect(jsonPath("$.boardTitle").exists())
          .andExpect(jsonPath("$.boardMain").exists())
          .andExpect(jsonPath("$.boardWriter").exists())
          .andDo(print());
```
`perform()` 이라는 메소드를 사용해서 RestApi 테스트를 할 수 있는 하나의 환경을 만들어 준다.

<br>

`mockMvc.perform(get("/registry?boardId=" + boardId))`

MockHttpServletRequestBuilder 라는 곳에 있는 get이라는 메소드이다.

실제로 어떤 통신을 할 것인지에 대해서 정의를 해준다.

<br>

**andExpect** : 기대하는 값이 나왔는지 체크해볼 수 있는 메소드

`andExpect()` 메소드는 builder 구조를 가지고 있기 때문에 `.`을 구분해서 사용한다.

<br>

http request를 날리면 기본적으로 json 형태의 body 값을 받기 때문에                       

jsonPath를 사용해서 각각의 값들(boardId, boardTitle)이 존재 하는지 확인한다.

<br>
<br>
<br>
  
```java
verify(boardService).getBoard("23");
```
verify : 해당 객체의 메소드가 실행되었는지 체크해준다.
  
 
<br><br><br>

출처                       
[JPA PK String 관련 문의드립니다](https://www.inflearn.com/questions/531442)                       
[ResponseEntity란?](https://bimmm.tistory.com/34)                       
[요청과 응답으로 엔티티(Entity) 대신 DTO를 사용하자](https://tecoble.techcourse.co.kr/post/2020-08-31-dto-vs-entity/)                       
