---
categories: Project JPA
tags: [JPA, study, project]
---

# 연관관계 적용 2 - Refactoring과 Builder 패턴 정리

연관관계 목차
- [연관관계](https://haedal-uni.github.io/posts/%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84/) 
- [연관관계 적용](https://haedal-uni.github.io/posts/%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-%EC%A0%81%EC%9A%A9/)
- [연관관계 적용2(refactoring)](https://haedal-uni.github.io/posts/%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-%EC%A0%81%EC%9A%A92(refactoring)/)  👈
- [연관관계 적용3(궁금증 해결하기)](https://haedal-uni.github.io/posts/%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-%EC%A0%81%EC%9A%A93(%EA%B6%81%EA%B8%88%EC%A6%9D-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0)/)
- [연관관계 적용4 정리(코드 + MySQL)](https://haedal-uni.github.io/posts/%EC%97%B0%EA%B4%80%EA%B4%80%EA%B3%84-%EC%A0%81%EC%9A%A94-%EC%A0%95%EB%A6%AC(%EC%BD%94%EB%93%9C-+-MySQL)/)

<br><br>

## 개요

이전 글에서 `Registry`와 `Comment`의 연관관계를 매핑하여 값을 세팅했다.  

연관관계가 적용되었기 때문에 매핑 이전에 사용하던 `registryId`와 `registryNickname` 를 제거하는 refactoring을 진행했다. 

- test 코드 수정

- 본 코드 수정
  - Controller
  - ServiceImpl
  - Repository
  - Entity (Builder 패턴 적용)
  - DTO

- front 수정
  - ajax data 수정 (put, delete)
  - form data 수정 (post)

<br><br><br><br>

## `registryId` 와 `registryNickname` 제거

### Comment

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
        if (this.registry != null) {
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

연관관계가 적용되었기 때문에 Comment는 이제 `registryId`나 `registryNickname`가 아닌 `Registry` 객체만 참조하면 된다.

<br><br> 

### CommentDto

```java
public class CommentDto {
    private String nickname;
    private String comment;
}
```

<br><br>

### Repository 수정

이제 Comment가 Registry 객체를 참조하므로 기존의 `registryId` 기반 메서드는 사용할 수 없다.  

Spring Data JPA의 네이밍 규칙을 이용해 연관 객체의 필드를 타고 조회하도록 수정했다.

```java
public interface CommentRepository extends JpaRepository<Comment, Long> {
    List<Comment> findAllByRegistry_Idx(Long idx);
}
```

<br><br>

## 문제 발생
리팩토링 후 실행했더니 아래와 같은 에러가 발생했다. 

`Cannot invoke "domain.Registry.getIdx()" because the return value of "domain.Comment.getRegistry()" is null`

<br>

Comment 객체에 Registry가 세팅되지 않은 상태에서 `comment.getRegistry().getIdx()`를 호출했기 때문이다.  

즉, Comment 저장 전에 반드시 Registry가 먼저 존재하고 해당 Registry가 Comment에 세팅되어야 한다.

아래 test 코드로 refactoring 해보면서 확인해봤다.

<br>

*`.get()` 대신 `.orElseThrow()` 사용을 권장          

`.orElseThrow()`는 값이 없을 경우 즉시 예외를 발생시켜 문제를 빠르게 파악할 수 있다.        

```java
Registry registry = registryRepository.findById(id).orElseThrow();
```

<br><br><br><br> 

### test 코드 

**1.** registryId, registryNickname 제거

**2.** Comment에 Registry 객체 직접 세팅

<br><br>

### 1. registryId, registryNickname 제거

```java
Comment comment = new Comment();
comment.setComment("❤️🧡💛💚💙💜🤎🖤");
comment.setNickname("우헤헤");
```

<br><br>

### 2. Comment에 Registry 넣기

```java
this.commentDto.setRegistryId(registry.getIdx());
this.commentDto.setRegistryNickname(registry.getNickname());

// 수정 후
this.commentDto.setRegistry(saveRegistry);
```

<br><br>

### post 테스트 수정
```java
assertEquals(comment.getRegistryNickname(), registry.getNickname());  // 수정 전

assertEquals(comment.getRegistry().getNickname(), registry.getNickname()); // 수정 후
```

<br><br>

### put 테스트 수정
```java
commentService.updateComment(comment.getIdx(), comment.getRegistryId(), commentDtoEdit, nowUser); // 수정 전

commentService.updateComment(comment.getIdx(), comment.getRegistry().getIdx(), commentDtoEdit, nowUser); // 수정 후
```

<br><br>

### delete 테스트 수정
```java
commentService.deleteComment(comment.getIdx(), comment.getRegistryId(), commentDto, nowUser); // 수정 전
 
commentService.deleteComment(comment.getIdx(), comment.getRegistry().getIdx(), commentDto, nowUser); // 수정 후
```

<br><br>

### registry 기준 댓글 조회 테스트 수정

```java
comment.setRegistry(saveRegistry);
comment1.setRegistry(saveRegistry);

List<Comment> results = commentRepository.findAllByRegistry_Idx(idx);
```

<br><br><br><br>

### null 문제 정리

Comment는 Registry의 기본키를 직접 알 수 없으며 `IDENTITY` 전략에서는 DB에 저장된 이후에야 id가 생성된다.  

따라서 반드시 Registry를 먼저 저장한 뒤 해당 Registry 객체를 Comment에 세팅하고 저장해야 한다.

<br>

post 요청은 form 방식으로 데이터를 받기 때문에 CommentDto가 Registry 객체를 받을 수 있어야 하고

put과 delete는 이미 DB에 데이터가 존재하므로 registryId만 검증용으로 사용하면 충분하다고 판단했다.

<br><br><br><br>

## Controller

```java
@PostMapping("/comment")
public Comment setComment(@ModelAttribute CommentDto commentDto) {
    return commentService.setComment(commentDto);
}
```

<br><br><br><br>

## post 처리 방식

### Comment

```java
public Comment(CommentDto commentDto) {
    this.nickname = commentDto.getNickname();
    this.comment = commentDto.getComment();
    this.registry = commentDto.getRegistry();
}
```

<br><br>

### CommentDto

```java
public class CommentDto {
    private String nickname;
    private String comment;
    private Registry registry;
}
```

<br><br>

### front form 수정

```js
let form_data = new FormData()
form_data.append("comment", $("#comment").val())
form_data.append("nickname", nickname)
form_data.append("registry", $("#RegistryId").html())
```
![image](https://user-images.githubusercontent.com/74857364/203247789-71a9ae84-8a61-445c-9a0f-710dfe37809a.png)

registry에 id를 넣었는데 어떻게 Registry 객체로 인식되는지는 [연관관계 적용3(궁금증 해결하기)]() 에서 다룬다.  

<br><br><br><br>

## Builder 패턴 정리

### 생성자 방식의 문제점

```java
Registry registry = new Registry("nemo", "hi", "nice");
```

이 방식은 파라미터가 많아질수록 가독성이 떨어지고 값의 의미를 한눈에 파악하기 어렵다.

<br><br>

### Builder 패턴 장점

**1.** 어떤 값이 무엇을 의미하는지 명확하다.

**2.** 파라미터 순서에 의존하지 않는다.

```java
Registry registry = Registry.builder()
        .nickname("coco")
        .title("hi")
        .main("hello")
        .build();
```

<br><br>

### Builder 적용 시 주의사항

**1.** Entity에는 `@Setter`를 사용하지 않는다.

**2.** 기본 생성자는 `@NoArgsConstructor(access = AccessLevel.PROTECTED)`로 변경

**3.** `@AllArgsConstructor`는 사용하지 않는다.

<br><br>

### Entity에 Builder 적용

```java
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
@Entity
public class Registry extends Timestamped {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long idx;

    private String nickname;
    private String title;
    private String main;

    @OneToMany(mappedBy = "registry")
    private List<Comment> comments = new ArrayList<>();

    @Builder
    public Registry(String nickname, String title, String main) {
        this.nickname = nickname;
        this.title = title;
        this.main = main;
    }
}
```

Builder를 클래스가 아닌 생성자에 적용한 이유는 `@AllArgsConstructor` 사용을 피하기 위함이다.

<br><br>

### Builder 적용 후 test 코드

```java
Registry registry = Registry.builder()
        .nickname("coco")
        .title("hi")
        .main("hello")
        .build();
```

<br><br>

### Setter 제거 후 Update 처리

Setter를 제거했기 때문에 필드 변경은 의미 있는 메서드로 처리한다.

```java
// Comment
public void updateComment(String comment) {
    this.comment = comment;
}

// ServiceImpl
comment.updateComment(commentDto.getComment());
```


<br><br><br>

---

REFERENCE                          
[빌더 패턴(Builder pattern)을 써야하는 이유, @Builder](https://pamyferret.tistory.com/67)         
[올바른 JPA Entity, @Builder 사용법](https://velog.io/@mooh2jj/%EC%98%AC%EB%B0%94%EB%A5%B8-%EC%97%94%ED%8B%B0%ED%8B%B0-Builder-%EC%82%AC%EC%9A%A9%EB%B2%95)



