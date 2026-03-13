---
categories: Project
tags: [spring, study, project]
---

# DTO ↔ Entity 변환 

## DTO를 받는 Entity 생성자란?

```java
public Registry(RegistryDto registryDto) {
    this.title = registryDto.getTitle();
    this.main = registryDto.getMain();
    this.nickname = registryDto.getNickname();
}
```

위 코드는 RegistryDto를 인자로 받아 Registry(Entity)를 생성하는 생성자이다.

<br>

- `RegistryDto` 기반 생성자
- DTO → Entity 변환 생성자

라고 부를 수 있다.

<br><br>

그런데 왜 이 코드는 검색해보면 사용하지 않는 곳이 많았다. 

> Entity가 DTO를 의존하게 되기 때문이다.

<br><br>

## 왜 Entity가 DTO를 의존하면 안 될까?

일반적인 레이어 구조는 다음과 같다.

```
Controller
↓
Service
↓
Repository
↓
Entity (최하위)
```
- DTO는 Controller / Service 계층에서 사용
- Entity는 DB와 직접 매핑되는 가장 하위 계층

<br>
  
Entity 생성자가 DTO를 받는다면 Entity → DTO 의존 발생   

하위 레이어(Entity)가 상위 레이어(DTO)를 알고 있는 구조가 되면서 레이어 분리 원칙에 어긋난다

<br><br> 

### 해결 방법 : DTO가 Entity를 만들자

- Entity에서 DTO 받지 않기
- DTO에서 Entity를 생성

이때 사용하는 메서드가 바로 `toEntity()` 이다.

<br><br><br><br>   

## DTO → Entity : toEntity()

### Registry DTO

```java
public class RegistryDto {
    private String nickname;
    private String title;
    private String main;

    // dto → entity
    public Registry toEntity() {
        return Registry.builder()
                .nickname(nickname)
                .title(title)
                .main(main)
                .build();
    }
}
```

- DTO가 자신의 데이터를 사용해서 Entity를 직접 생성한다.
- Entity는 DTO의 존재를 모른다

<br><br> 

### Registry Entity (생성자 제거)

```java
public class Registry extends Timestamped {

    @OneToMany(mappedBy = "registry")
    @JsonIgnore
    private List<Comment> comments = new ArrayList<>();

    // RegistryDto를 받는 생성자 제거
//    public Registry(RegistryDto registryDto) {
//        this.title = registryDto.getTitle();
//        this.main = registryDto.getMain();
//        this.nickname = registryDto.getNickname();
//    }
}
```

<br><br> 

### Service 코드 변경

#### 변경 전

```java
public Registry setUpload(RegistryDto registryDto) {
    Registry registry = new Registry(registryDto);
    registryRepository.save(registry);
    return registry;
}
```

#### 변경 후

```java
public Registry setUpload(RegistryDto registryDto) {
    return registryRepository.save(registryDto.toEntity());
}
```


Service 코드가 단순해지고 책임 분리가 명확해졌다.

<br><br><br><br> 

## Comment
Comment도 동일하다.

### Comment DTO

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

### Comment Entity (DTO 생성자 제거)

```java
public class Comment extends Timestamped {

    @ManyToOne
    @JoinColumn(name = "registry_id", nullable = false)
    private Registry registry;

    // DTO 생성자 제거
//    public Comment(CommentDto commentDto) {
//        this.nickname = commentDto.getNickname();
//        this.comment = commentDto.getComment();
//        Long registryIdx = commentDto.getRegistryIdx();
//    }
}
```

<br><br> 

### Service 변경 전 → 후

#### 변경 전

```java
public Comment setComment(CommentDto commentDto) {
    Registry registryId = registryRepository.getById(commentDto.getRegistryIdx());

    Comment comment = Comment.builder()
            .comment(commentDto.getComment())
            .nickname(commentDto.getNickname())
            .registry(registryId)
            .build();

    commentRepository.save(comment);
    return comment;
}
```

#### 변경 후

```java
public Comment setComment(CommentDto commentDto) {
    Registry registry = registryRepository.getById(commentDto.getRegistryIdx());
    Comment comment = commentDto.toEntity();
    comment.setRegistry(registry);
    return commentRepository.save(comment);
}
```

<br><br><br><br> 

## 객체를 DTO에 넘기기

### DTO refactoring

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

### Service refactoring

```java
public Comment setComment(CommentDto commentDto) {
    Registry registry = registryRepository.findById(commentDto.getRegistryIdx())
            .orElseThrow();
    return commentRepository.save(commentDto.toEntity(registry));
}
```

Service가 비즈니스 흐름만 담당하게 된다.

<br><br><br><br>    

## Entity → DTO : of()

조회(Response)용 변환

<br>

**왜 Entity를 그대로 반환하면 안 될까?**

Entity에는 연관관계, 내부 컬럼, 민감한 정보가 모두 포함될 수 있다.

그래서 Response 전용 DTO를 사용한다.

<br><br>  

### Response DTO + of()

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
    public static ResRegistryDto of(Registry registry) {
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

<br><br> 

### of() 사용

```java
public ResRegistryDto getIdxRegistry(Long idx) {
    Registry registry = registryRepository.findById(idx)
            .orElseThrow(() -> new NullPointerException("해당 게시글 없음"));
    return ResRegistryDto.of(registry);
}
```

<br><br> 

#### of()를 쓰는 이유

- Entity 구조 변경 시
- DTO 변환 로직을 한 곳에서만 수정
- 유지보수에 매우 유리

<br><br><br><br>  

## toEntity vs of 정리

### DTO → Entity

```java
public Registry toEntity(User user) {
    return Registry.builder()
            .user(user)
            .title(title)
            .main(main)
            .build();
}
```

- 인스턴스 메서드
- 자기 자신의 필드 사용
- static x

<br><br> 

### Entity → DTO

```java
public static ResRegistryDto of(Registry registry) {
    return ResRegistryDto.builder()
            .idx(registry.getIdx())
            .title(registry.getTitle())
            .main(registry.getMain())
            .build();
}
```

- static 메서드
- Entity를 받아 변환
- 객체 생성 책임 명확

<br><br><br><br>  

## 요약

- 저장 / 수정 : DTO → Entity → `toEntity()`
- 조회 / 응답 : Entity → DTO → `of()`
- Entity는 DTO를 몰라야 한다


<br><br><br>

---

[pr - [refactoring] toEntity](https://github.com/dal-cho/adme/pull/117/files)            
[pr - ResponsetDto(of)](https://github.com/dal-cho/adme/pull/153/files)              


