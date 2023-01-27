---
categories: Project
tags: [spring, study, project]
---

```java
public Registry(RegistryDto registryDto) {
    this.title = registryDto.getTitle();
    this.main = registryDto.getMain();
    this.nickname = registryDto.getNickname();
}
```

위 코드를 뭐라고 부를까? **RegisttyDto**에 대한 생성자라고 부를 수 있다.

그런데 이 부분에 대한 코드들을 다른 블로그에서 찾기 어려웠다.

그 이유는 바로 해당 코드를 쓰지 않는다는 것이다.

<br><br><br>

## toEntity()
엔티티에 **registryDto**에 대한 생성자를 쓰면 안된다.

하위 레이어를 의존하는 것을 기본으로 하기 때문이다.

그리고 해당 블로그들이 공통적으로 쓰는 것은 `toEntity()`였다.

<br><br>

**Controller** > **Service** > **Repository**

Entity는 최하위이기 때문에 최하위에서 dto를 사용하기 보다는

dto에서 Entity를 사용하는 것이 낫다라고 판단하는 것이다.

<br>

dto에서 entity로 변경할 때 `toEntity()`를 사용한다.

→ post, update에서 주로 쓴다.

<br>

🐣 entity를 보여주기보다 보안적으로 dto로 해서 보여주는게 낫다.

<br><br><br>

## 프로젝트에 적용하기
### Registry(게시글)
#### DTO (코드 추가)
```java
public class RegistryDto {
    private String nickname;
    private String title;
    private String main;

    // dto → entity
    public Registry toEntity(){
        return Registry.builder()
                .nickname(nickname)
                .title(title)
                .main(main)
                .build();
    }
}
```
<br><br>
        
#### Entity (코드 제거)
```java
public class Registry extends Timestamped {
    // 코드 생략
    @OneToMany(mappedBy = "registry")
    @JsonIgnore
    private List<Comment> comments = new ArrayList<>();


       // 코드 제거
//    public Registry(RegistryDto registryDto) {
//        this.title = registryDto.getTitle();
//        this.main = registryDto.getMain();
//        this.nickname = registryDto.getNickname();
//    }
}
```
<br><br>

#### ServiceImpl
```java
public Registry setUpload(RegistryDto registryDto) throws IOException {
    Registry registry = new Registry(registryDto);
    registryRepository.save(registry);
    return registry;
}
```
🔽 아래 코드로 변경
```java
public Registry setUpload(RegistryDto registryDto) throws IOException {
    return registryRepository.save(registryDto.toEntity());
}
```

<br><br><br>

### Comment(댓글)
#### DTO (코드 추가)
```java
public class CommentDto {
    private String nickname;
    private String comment;
    private Long registryIdx;

    // dto → entity
    public Comment toEntity() {
        return Comment.builder()
                .nickname(nickname)
                .comment(comment)
                .build();
    }
}
```

<br><br>

#### Entity (코드 제거)
```java
public class Comment extends Timestamped {
    // 코드 생략
    @ManyToOne
    @JoinColumn(name = "registry_id", nullable = false)
    private Registry registry;

       // 코드 제거
//    public Comment(CommentDto commentDto) {
//        this.nickname = commentDto.getNickname();
//        this.comment = commentDto.getComment();
//        Long registryIdx = commentDto.getRegistryIdx();
//    }

}
```
<br><br>

#### ServiceImpl
```java
    public Comment setComment(CommentDto commentDto) {
        Registry registryId = registryRepository.getById(commentDto.getRegistryIdx());
        Comment comment = Comment.builder()
                .comment(commentDto.getComment())
                .nickname(commentDto.getNickname())
                .registry(registryId)
                .build();
        // 연관관계 매핑
        Registry registry = registryRepository.findById(comment.getRegistry().getIdx()).get();
        comment.setRegistry(registry);
        commentRepository.save(comment);
        return comment;
    }
```
🔽 아래 코드로 변경

```java
    public Comment setComment(CommentDto commentDto) {
        Registry registry = registryRepository.getById(commentDto.getRegistryIdx());
        Comment comment = commentDto.toEntity();
        comment.setRegistry(registry);
        Comment save = commentRepository.save(comment);
        return save;
    }
```

<br><br><br>

### refactoring
위에서 수정한 코드에서 refactoring을 진행했다. 👉🏻 객체 넣기

#### DTO
```java
public class CommentDto {
    private String nickname;
    private String comment;
    private Long registryIdx;

    // dto → entity
    public Comment toEntity() {
        return Comment.builder()
                .nickname(nickname)
                .comment(comment)
                .build();
    }
}
```
🔽 refactoring
```java
public Comment toEntity(Registry registry) {
    return Comment.builder()
            .nickname(nickname)
            .comment(comment)
            .registry(registry)
            .build();
}
```

<br><br>

#### ServiceImpl
```java
    public Comment setComment(CommentDto commentDto) {
        Registry registry = registryRepository.getById(commentDto.getRegistryIdx());
        Comment comment = commentDto.toEntity();
        comment.setRegistry(registry);
        Comment save = commentRepository.save(comment);
        return save;
    }
```
🔽 refactoring
```java
public Comment setComment(CommentDto commentDto) {
    Registry registry = registryRepository.findById(commentDto.getRegistryIdx()).orElseThrow();
    Comment comment = commentDto.toEntity(registry);
    Comment save = commentRepository.save(comment);
    return save;
}
```

<br><br><br>

---


## of
이번엔 toEntity의 반대인 entity → dto를 작성해봤다.

아까는 dto에서 entity로 변경할 때 toEntity()를 사용했다면 이번에는 entity에서 dto로 변경하기 위해 of를 활용했다.

<br>

🐣 주로 responseDto에서 활용을 하는 것 같다.

<br><br>

### of 작성하기
```java
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class ResRegistryDto {
    private Long idx;
    private String title;
    private String main;
    private LocalDateTime createdAt;
    private LocalDateTime modifiedAt;
    private String nickname;

    // entity → dto
    public static ResRegistryDto of(Registry registry){
        return ResRegistryDto.builder()
                .idx(registry.getIdx())
                .title(registry.getTitle())
                .main(registry.getMain())
                .createdAt(registry.getCreatedAt())
                .modifiedAt(registry.getModifiedAt())
                .nickname(registry.getUser().getNickname())
                .build();
    }
}
```

이렇게 of를 작성한 이유는 entity를 보호하기 위해서이다.

entity로 쓰면 db 내용을 전부 들여다 볼수 있기 때문에 객체 자체 대신 풀어써서 사용했다.

<br><br>

### of 활용하기
```java
public ResRegistryDto getIdxRegistry(Long idx) throws NullPointerException {
    Registry getIdxRegistry = registryRepository.findById(idx).orElseThrow(
            () -> new NullPointerException("해당 게시글 없음")
    );
    return ResRegistryDto.of(getIdxRegistry);
}
```

of를 사용하므로써 유지보수에 좋다.

변수가 중간에 제거될 수도 있고 변수 명이 바뀔 때 

of를 사용하지 않는다면 해당 로직을 찾아서 일일히 변경해야하는데 of를 사용하면 한번에 수정이 가능하다.

ex) of를 사용하지 않는다면 더 이상 title을 사용하지 않을 때 ResRegistryDto의 title을 쓰는 코드를 찾아서 일일히 수정해야한다.

<br><br>

### toEntity와 of
```java
@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
public class RegistryDto {
    private String title;
    private String main;
    private Long userIdx;

    // dto → entity
    public Registry toEntity(User user){
        return Registry.builder()
                .user(user)
                .title(title)
                .main(main)
                .build();
    }
}
```

<br>

```java
@Getter
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class ResRegistryDto {
    private Long idx;
    private String title;
    private String main;
    private LocalDateTime createdAt;
    private LocalDateTime modifiedAt;
    private String nickname;

    // entity → dto
    public static ResRegistryDto of(Registry registry){
        return ResRegistryDto.builder()
                .idx(registry.getIdx())
                .title(registry.getTitle())
                .main(registry.getMain())
                .createdAt(registry.getCreatedAt())
                .modifiedAt(registry.getModifiedAt())
                .nickname(registry.getUser().getNickname())
                .build();
    }
}
```

toEntity()는 static을 붙이지 않았고 of는 static을 붙였다.

toEntity()에서 title과 main은 자기 객체에 있는 값을 활용한다.

특정 필드에 접근 하기 때문에 static을 사용할 수 없다.


<br><br><br>

[pr - [refactoring] toEntity](https://github.com/dal-cho/adme/pull/117/files)

[pr - ResponsetDto(of)](https://github.com/dal-cho/adme/pull/153/files)


