---
categories: Project
tags: [DB, study, project]
---

# 연관관계 적용2

이전([연관관계 적용](https://haedal-uni.github.io/posts/%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-%EC%A0%81%EC%9A%A9/))에 `Registry`와 `Comment` 연관관계 매핑을 해서 값을 세팅했다.

이제 refactoring을 통해서 매핑 하기 전에 작성한 `registryId`와 `registryNickname`은 필요없으므로 코드를 수정하기로 했다.     

*[pr 👉🏻 연관관계 refactoring](https://github.com/dal-cho/adme/pull/111/files)   

<br>

- test 코드 수정

- 본 코드 수정
  - Controller
  - ServiceImpl
  - Repository
  - Entity(builder() 적용)
  - Dto

- front 수정
  - ajax data 수정 (put, delete)
  - form data 수정 (post)

<br><br><br>

## `registryId`와 `registryNickname` 제거

**Entity**
```java
@Setter
@Getter
@NoArgsConstructor
@Entity
public class Comment extends Timestamped {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "comment_id")
    private Long idx;

    @Column(nullable = false)
    private String nickname;

    @Column(nullable = false)
    private String comment;

    @ManyToOne
    @JoinColumn(name = "registry_id")
    private Registry registry;

    public void setRegistry(Registry registry) {
        if(this.registry != null) {
            this.registry.getComments().remove(this);
        }
        this.registry = registry;

        if (!registry.getComments().contains(this)) {
            registry.addComment(this);
        }
    }

    public Comment(CommentDto commentDto) {
        this.nickname = commentDto.getNickname();
        this.comment = commentDto.getComment();
    }
}
```

<br><br>

**CommentDto**
```java
public class CommentDto {
    private String nickname;
    private String comment;
}
```

<br><br>

**Repository**

더 이상 `registryId`를 사용할 수 없으니 메소드명을 수정한다. 

```java
public interface CommentRepository extends JpaRepository<Comment, Long> {
    List<Comment> findAllByRegistryId(Long idx);
}
```
⬇️
```java
public interface CommentRepository extends JpaRepository<Comment, Long> {
    List<Comment> findAllByRegistry_Idx(Long idx);
}
```

<br><br>

이대로 실행을 하면 문제가 없을 것 같지만 에러가 뜬다.   

<br><br>

## 문제 코드
CommentServiceImpl의 post 코드에서                 

`Registry registry = registryRepository.findById(comment.getRegistry().getIdx()).get();`     

해당 부분이 에러가 떴다. 

<br>

*Cannot invoke "domain.Registry.getIdx()" because the return value of "domain.Comment.getRegistry()" is null* 

왜 null이 나오는건지 잘 파악이 안되서 test 코드부터 refactoring을 해보면서 어떻게 작성을 해야할지 감을 잡기로 했다.

<br><br><br>

## test 코드
test 코드에서는 2가지 위주로 수정을 진행했다.

1. registryId와 registryNickname을 제거한다.
2. Comment에 Registry를 넣어준다.

<br><br>

#### 1. registryId, registryNickname 제거
```java
Comment comment = new Comment();
comment.setComment("❤️🧡💛💚💙💜🤎🖤");
comment.setNickname("우헤헤");
//comment.setRegistryId(5L);
//comment.setRegistryNickname("pop");
```

<br><br>

#### 2. Comment에 Registry를 넣어준다.
<br>

**수정 전**
```java
    @BeforeEach
    void beforeEach() {
        SignupRequestDto userDto = new SignupRequestDto("test1", "test1", "d","d","d");
        User user = userService.registerUser(userDto);
        this.nowUser = new UserDetailsImpl(user);

        // 게시글
        RegistryDto registryDto = new RegistryDto("test1","타이틀","본문"); 
        Registry saveRegistry = new Registry(registryDto);
        this.registry = registryRepository.save(saveRegistry);

        // 댓글
        this.commentDto = new CommentDto();
        this.commentDto.setComment("comment");
        this.commentDto.setNickname(nowUser.getUsername());
        this.commentDto.setRegistryId(registry.getIdx());
        this.commentDto.setRegistryNickname(registry.getNickname()); // 작성자
    }
```

<br>

**수정 후**
```java
    @BeforeEach
    void beforeEach() {
        SignupRequestDto userDto = new SignupRequestDto("test1", "test1", "d","d","d");
        User user = userService.registerUser(userDto);
        this.nowUser = new UserDetailsImpl(user);

        // 게시글
        RegistryDto registryDto = new RegistryDto("test1","타이틀","본문");
        Registry saveRegistry = new Registry(registryDto);
        this.registry = registryRepository.save(saveRegistry);

        // 댓글
        this.commentDto = new CommentDto();
        this.commentDto.setComment("comment");
        this.commentDto.setNickname(nowUser.getUsername());
        this.commentDto.setRegistry(saveRegistry); // 👈🏻
    }
```

<br><br><br>

#### post 수정 전
```java
    @Test
    void saveComment() throws IOException {
        // given

        // when
        Comment comment = commentService.setComment(commentDto);

        // then
        Comment commentTest = commentRepository.findById(comment.getIdx()).orElseThrow(
                () -> new NullPointerException("comment 생성 x")
        );

        assertEquals("comment의 id값이 일치하는지 확인", comment.getIdx(), commentTest.getIdx());
        assertEquals("comment의 nickname이 일치하는지 확인", comment.getRegistryNickname(), registry.getNickname());
    }
```
<br>

#### post 수정 후
```java
    @Test
    void saveComment() throws IOException {
        // given

        // when
        Comment comment = commentService.setComment(commentDto);

        // then
        Comment commentTest = commentRepository.findById(comment.getIdx()).orElseThrow(
                () -> new NullPointerException("comment 생성 x")
        );

        assertEquals("comment의 id값이 일치하는지 확인", comment.getIdx(), commentTest.getIdx());
        assertEquals("comment의 nickname이 일치하는지 확인", comment.getRegistry().getNickname(), registry.getNickname()); // 👈🏻
    }
```
<br><br><br>

#### put 수정 전
```java
    @Test
    @DisplayName("comment 수정")
    void updateComment() throws IOException {
        Comment comment = commentService.setComment(commentDto);
        CommentDto commentDtoEdit = new CommentDto();
        commentDto.setComment("comment-edit");

        //when
        Comment commentTest = commentService.updateComment(comment.getIdx(), comment.getRegistryId(), commentDtoEdit, nowUser);

        //then
        assertEquals("Comment Id 값이 일치하는지 확인.", comment.getIdx(), commentTest.getIdx());
        assertEquals("Comment 내용이 업데이트 되었는지 확인", commentDtoEdit.getComment(), commentTest.getComment());
    }
```
<br>

#### put 수정 후
```java
    @Test
    @DisplayName("comment 수정")
    void updateComment() throws IOException {
        Comment comment = commentService.setComment(commentDto);
        CommentDto commentDtoEdit = new CommentDto();
        commentDto.setComment("comment-edit");

        //when
        Comment commentTest = commentService.updateComment(comment.getIdx(), comment.getRegistry().getIdx(), commentDtoEdit, nowUser); // 👈🏻

        //then
        assertEquals("Comment Id 값이 일치하는지 확인.", comment.getIdx(), commentTest.getIdx());
        assertEquals("Comment 내용이 업데이트 되었는지 확인", commentDtoEdit.getComment(), commentTest.getComment());
    }
```
<br><br><br>

#### delete 수정 전
```java
    @Test
    @DisplayName("comment 삭제 성공")
    void deleteComment() throws IOException {
        // given
        Comment comment = commentService.setComment(commentDto);

        //when
        commentService.deleteComment(comment.getIdx(), comment.getRegistryId(), commentDto, nowUser);

        // then
        Optional<Comment> commentTest = commentRepository.findById(comment.getIdx());
        if (commentTest.isPresent())
            throw new IllegalArgumentException("Comment 가 정상적으로 삭제되지 않았습니다.");
        else
            assertEquals("Comment 가 비어있다.", Optional.empty(), commentTest);
    }
```
<br>

#### delete 수정 후
```java
    @Test
    @DisplayName("comment 삭제 성공")
    void deleteComment() throws IOException {
        // given
        Comment comment = commentService.setComment(commentDto);

        //when
        commentService.deleteComment(comment.getIdx(), comment.getRegistry().getIdx(), commentDto, nowUser); // 👈🏻

        // then
        Optional<Comment> commentTest = commentRepository.findById(comment.getIdx());
        if (commentTest.isPresent())
            throw new IllegalArgumentException("Comment 가 정상적으로 삭제되지 않았습니다.");
        else
            assertEquals("Comment 가 비어있다.", Optional.empty(), commentTest);
    }
```
<br><br><br>

#### id가 1인 댓글 _ 수정 전
```java
    @BeforeEach
    void beforeEach() {
        commentDto = new CommentDto("commentNickname", "testComment", 1L,"registryNickname");
        comment = new Comment(commentDto);
    }
    
    @Test
    @DisplayName("id가 1인 댓글")
    void showComment() throws IOException {
        //given
        RegistryDto registry = new RegistryDto();
        registry.setTitle("첫 번째");
        registry.setMain("1");
        registry.setNickname("nickname");


        CommentDto comment = new CommentDto();
        comment.setComment("funfun");
        comment.setNickname("hh");
        comment.setRegistryId(1L);
        comment.setRegistryNickname("nickname");

        CommentDto comment1 = new CommentDto();
        comment1.setComment("wow");
        comment1.setNickname("hh");
        comment1.setRegistryId(1L);
        comment1.setRegistryNickname("nickname");

        //when
        Registry saveRegistry = registryRepository.save(new Registry(registry));
        Comment saveComment = commentRepository.save(new Comment(comment));
        Comment saveComment1 = commentRepository.save(new Comment(comment1));

        //then
        Long idx = saveRegistry.getIdx();
        List<Comment> results = commentRepository.findAllByRegistryId(idx);
        assertThat(saveComment.getComment()).isEqualTo(results.get(0).getComment());
        assertThat(saveComment1.getComment()).isEqualTo(results.get(1).getComment());
    }
```
<br>

#### id가 1인 댓글 _ 수정 후
```java
    @BeforeEach
    void beforeEach() {
        Registry registry = new Registry();  // 👈
        registry.setNickname("registryNickname");
        registry.setTitle("registryTitle");
        registry.setMain("registryMain");
        
        commentDto = new CommentDto("commentNickname","comment", registry);
        comment = new Comment(commentDto);
    }
    
    @Test
    @DisplayName("id가 1인 댓글")
    void showComment() throws IOException {
        //given
        RegistryDto registry = new RegistryDto();
        registry.setTitle("첫 번째");
        registry.setMain("1");
        registry.setNickname("nickname");


        CommentDto comment = new CommentDto();
        comment.setComment("funfun");
        comment.setNickname("hh");

        CommentDto comment1 = new CommentDto();
        comment1.setComment("wow");
        comment1.setNickname("hh");

        //when
        Registry saveRegistry = registryRepository.save(new Registry(registry));
        comment.setRegistry(saveRegistry); // 👈🏻
        comment1.setRegistry(saveRegistry); // 👈🏻
        Comment saveComment = commentRepository.save(new Comment(comment));
        Comment saveComment1 = commentRepository.save(new Comment(comment1));

        //then
        Long idx = saveRegistry.getIdx();
        List<Comment> results = commentRepository.findAllByRegistry_Idx(idx); // 👈🏻
        assertThat(saveComment.getComment()).isEqualTo(results.get(0).getComment());
        assertThat(saveComment1.getComment()).isEqualTo(results.get(1).getComment());
    }
```

<br><br><br>

## null 해결하기
test 코드를 먼저 수정하다보니 Comment의 값을 저장하기 전에 Registry를 먼저 저장한 후에        

`registry.idx()`의 값을 가져오고 해당 값을 Comment에 set하여 저장을 해야지 제대로 db에 저장이 될텐데      

Registry의 값을 넣어주지 않아서 에러가 뜨는 것 같다라는 생각이 들었다.                  

(* `IDENTITY`에 의해서 id는 저장이 된 후에 알 수 있으므로 DB에 값을 넣기 전까지는 기본키를 모른다.)       

<br>

기존에 작성한 Controller를 보면 post는 form 형태로 데이터를 받기 때문에 dto에 Registry값이 있어야 하고

put과 delete는 기존 db가 있기 때문에 오류를 체크하는 정도로만 registry를 활용하기 때문에             

해당 파라미터로 받는 registryId만 있으면 될 것 같다라고 판단, fornt만 수정하고 기존 코드를 유지하기로 했다.          

<br>

**Controller**
```java
    @PostMapping("/comment")
    public Comment setComment(@ModelAttribute CommentDto commentDto) {
        return commentService.setComment(commentDto);
    }

    @GetMapping("/comment")
    public List<Comment> getComment(@RequestParam Long idx){
        return commentService.getComment(idx);
    }

    @PutMapping("/comment/{commentId}/registry/{registryId}")
    public Comment updateComment(@PathVariable Long commentId, @PathVariable Long registryId,
                                 @RequestBody CommentDto commentDto,
                                 @AuthenticationPrincipal UserDetailsImpl userDetails) throws AccessDeniedException {
        return commentService.updateComment(commentId, registryId, commentDto, userDetails);
    }

    @DeleteMapping("/comment/{commentId}/registry/{registryId}")
    public void deleteComment(@PathVariable Long commentId, @PathVariable Long registryId,
                              @RequestBody CommentDto commentDto,
                              @AuthenticationPrincipal UserDetailsImpl userDetails)throws AccessDeniedException {
        commentService.deleteComment(commentId, registryId, commentDto, userDetails);
    }
```
<br><br><br>

### post
값을 받기 위해서는 dto에 registry를 받을 수 있어야 한다. 

따라서 Comment와 CommentDto를 수정한다.

<br>

**Comment**
```java
public Comment(CommentDto commentDto) {
    this.nickname = commentDto.getNickname();
    this.comment = commentDto.getComment();
    this.registry = commentDto.getRegistry(); // 👈🏻
}
```

<br>

**CommentDto**
```java
public class CommentDto {
    private String nickname;
    private String comment;
    private Registry registry; // 👈🏻
}
```
<br>

front에서 form을 보낼 때 dto와 변수 명이 같아야 한다.

registry로 받아야 하므로 registryId를 아래와 같이 수정했다.

```js
let form_data = new FormData()
form_data.append("comment", $("#comment").val())
form_data.append("nickname", nickname)
form_data.append("registry",$("#RegistryId").html()) // 👈🏻
```

test 해보니 post가 잘 되었다. 근데 registry에 registryId를 넣었는데 어떻게 id인지 아는걸까?

![image](https://user-images.githubusercontent.com/74857364/203247789-71a9ae84-8a61-445c-9a0f-710dfe37809a.png)

<br>

이 부분은 [연관관계 적용3(궁금증 해결하기)](https://haedal-uni.github.io/posts/%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-%EC%A0%81%EC%9A%A93(%EA%B6%81%EA%B8%88%EC%A6%9D-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0)/) 에서 다뤘다.

<br><br><br>

### put, delete

```js
let RegistryComment = {
    nickname: nickname,
    comment: comment,
    registryId: registryId,
    registryNickname: registryNickname
}
```

⬇️ 위 형태로 data에 보냈었는데 이제는 아래와 같이 보내는 것으로 수정했다.           

```js
let RegistryComment = {
    nickname: nickname,
    comment: comment
}
```
<br><br><br>

## @Builder 

보통 생성자를 통해 객체를 생성하는데 아래와 같이 작성한다. 
```java
Registry registry = new Registry("nemo", "hi", "nice");
```


하지만 이에 단점들이 있어 객체를 생성하는 별도 builder를 두는 방법이 있다. 이를 builder pattern이라고 한다.

<br><br>

**1. 생성자 파라미터가 많을 경우 가독성이 좋지 않다.**

위 코드를 보면 nemo가 어떤 것을 의미하는지 알 수가 없다.

builder pattern으로 구현하면 각 값들의 이름이 함수로 setting이 되어 각각 무슨 값을 의미하는지 알 수 있다.

→ 생성자로 설정해야하는 값이 많을 경우에는 builder pattern을 쓰는 것이 가독성이 좋다.

```java
Registry registry = Registry.builder()
        .nickname("coco")
        .title("hi")
        .main("hello")
        .build();
```

<br><br>

**2. 어떤 값을 먼저 설정하던 상관없다.**

생성자의 경우는 정해진 파라미터 순서대로 꼭 값을 넣어줘야한다. 

순서를 무시하고 값을 넣으면 에러가 발생하거나 엉뚱한데 값이 들어갈 수 있다.

하지만 builder pattern은 빌더의 필드 이름으로 값을 설정하기 때문에 순서에 종속적이지 않다.

그냥 쓰이는 곳에서 어떤 필드를 먼저 설정해야하는지 굳이 순서를 생각할 필요 없이 편하게 설정하면 된다.

<br><br>

### @Builder 적용
builder pattern을 사용하려면 먼저 3가지 문제를 알아야 한다.

<br>

#### 1. @Setter를 사용하지 않는다.
Setter는 그 의도가 분명하지 않고 객체를 언제든지 변경할 수 있는 상태가 되어서 객체의 안전성이 보장받기 힘들다. 

특히 엔티티에서는 `@Setter`를 사용 시 해당 변경 가능성이 어디서 누구에 의해 발생했는지 추적하기가 힘들어진다.

때문에 값 변경이 필요한 경우 의미 있는 메소드를 생성하여 이를 사용하는 것이 좋다. 

<br><br>

#### 2. @NoArgsConstructor(access = AccessLevel.PROTECTED)로 변경한다.

기본 생성자(NoArgsConstructor)의 접근 제어를 PROCTECTED 로 설정하면 아무런 값도 갖지 않는 의미 없는 객체의 생성을 막게 된다. 

즉 무분별한 객체 생성에 대해 한번 더 체크할 수 있다.

```java
Registry registry = new Registry(); //컴파일 에러 발생
```
`@Builder`를 사용하는 방법은 총 2가지다.

1) 클래스에 @Builder를 붙이기
2) 생성자에 @Builder를 붙이기

아래 3번을 보고 1번과 2번 중 어느 것이 나을지 확인해본다.

<br><br>

#### 3. `@AllArgsConstructor`는 쓰지 않는다.

클래스 레벨에서 `@Builder`와 `@NoArgsConstructor`를 함께 쓰면 오류가 발생한다.

이를 해결하려면 모든 필드를 가지는 생성자를 만들어주어야 하는데 `@AllArgsConstructor`도 같이 써주게 된다.

하지만 `@AllArgsConstructor` 는 위험하다.

클래스에 존재하는 모든 필드에 대한 생성자를 자동으로 생성하는데,         
인스턴스 멤버의 선언 순서에 영향을 받기 때문에 변수의 순서를 바꾸면          
생성자의 입력 값 순서도 바뀌게 되어 검출되지 않는 치명적인 오류를 발생시킬 수 있다.                 

<br>

그래서 `@Builder`를 사용하는 방법 2(생성자에 `@Builder`를 붙이기)를 사용해서

`@AllArgsConstructor`를 쓰는 일이 없도록 한다.

<br><br><br>

### 프로젝트에 실제 적용시키기 
#### Entity
```java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Entity
@ToString
public class Registry extends Timestamped {
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Id
    @Column(name = "registry_id")
    private Long idx;

    @Column(nullable = false)
    private String nickname;

    @Column(nullable = false)
    private String title;

    @Column(nullable = false)
    private String main;

    @OneToMany(mappedBy = "registry")
    @JsonIgnore
    private List<Comment> comments = new ArrayList<>();

    public Registry(long idx, String nickname, String title, String main) {
        super();
    }

    public void addComment(Comment comment) {
        this.comments.add(comment);
        if(comment.getRegistry() != this) {
            comment.setRegistry(this);
        }
    }

    public Registry(RegistryDto registryDto) {
        this.title = registryDto.getTitle();
        this.main = registryDto.getMain();
        this.nickname = registryDto.getNickname();
    }

    @Builder
    public Registry(String nickname, String title, String main) {
        this.nickname = nickname;
        this.title = title;
        this.main = main;
    }
}
```
🐣 클래스가 아닌 생성자에 `@Builder`를 추가한 이유
<br>

클래스에 `@Builder`를 쓰면 초기화가 안되서 에러가 뜨는데            

`@AllArgsConstructor`는 지양해야하기 때문에 클래스가 아닌 **생성자**에 넣었다.              

<br>

`@AllArgsConstructor`는 클래스에 존재하는 모든 필드에 대한 생성자를 자동으로 생성하는데,               

인스턴스 멤버의 선언 순서에 영향을 받기 때문에 두 변수의 순서를 바꾸면

생성자의 입력 값 순서도 바뀌게 되어 검출되지 않는 치명적인 오류를 발생시킬 수 있다.                    
             
<br><br>

#### test 코드
```java
@Test
void registryTest() throws Exception {
//    Registry registry = new Registry();
//    registry.setTitle("hi");
//    registry.setMain("hello");
//    registry.setNickname("testCodeId");

    //given
    Registry registry = Registry.builder()
            .nickname("coco")
            .title("hi")
            .main("hello")
            .build();

    //when
    registryRepository.save(registry);

    //then
    Assertions.assertThat("hi").isEqualTo(registry.getTitle());
    Assertions.assertThat("hello").isEqualTo(registry.getMain());
}
```
<br><br><br>

Comment.java에서 @Builder를 썼더니 Service에서 comment를 Update(C,R, **U**, D)하는 로직에 에러가 떴다.

@Setter를 지우면서 `comment.setComment()`가 되지 않았던 것이다.

```java
// Setter 메소드 선언 방법
public void setFieldName(타입 fieldName){
  this.fieldName = fieldName;
}

// Getter 메소드 선언 방법
public 리턴 타입 getFieldName(){
  return fieldname;
}
```

그래서 해당 메소드가 Comment(객체) 중 comment(필드 명)만 담고 있는것을 확인하여 Comment.java와 ServiceImpl 코드를 수정했다.

Update 하는 부분이므로 updateComment()로 작성했다.

<br><br>

**Comment**
```java
public void updateComment(String comment) {
    this.comment = comment;
}
```

<br>

**CommentServiceImpl**

`comment.updateComment(commentDto.getComment());`

<br><br><br>

출처         
[빌더 패턴(Builder pattern)을 써야하는 이유, @Builder](https://pamyferret.tistory.com/67)         
[올바른 JPA Entity, @Builder 사용법](https://velog.io/@mooh2jj/%EC%98%AC%EB%B0%94%EB%A5%B8-%EC%97%94%ED%8B%B0%ED%8B%B0-Builder-%EC%82%AC%EC%9A%A9%EB%B2%95)
