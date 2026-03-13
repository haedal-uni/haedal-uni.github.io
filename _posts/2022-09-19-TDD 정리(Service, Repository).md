---
categories: Spring
tags: [spring, TDD, test]
---

## Service

```java
Mockito.when(boardRepository.save(new Board("3","title","main","writer")))
        .thenReturn(new Board("3","title","main","writer"));
```

객체를 새로 생성을 여러번 함으로써 메모리를 많이 차지하므로

변수를 생성해서 하나만 생성하게 한다.

```java
Board board = new Board("3", "title", "main", "writer");

//given
Mockito.when(boardRepository.save(board))
        .thenReturn(board);
```

<br><br>

Service test 코드 정리 마지막 내용으로 아래 post test 코드를 본다.

```java
@Test
public void postBoard() {
    Board board = new Board("3", "title", "main", "writer");

    //given
    Mockito.when(boardRepository.save(board))
            .thenReturn(board);

    Board boardSave = boardService.saveBoard(new BoardDto("3", "title", "main", "writer"));

    Assertions.assertEquals(boardSave.getBoardId(), "3");
    Assertions.assertEquals(boardSave.getBoardTitle(), "title");
    Assertions.assertEquals(boardSave.getBoardMain(), "main");
    Assertions.assertEquals(boardSave.getBoardWriter(), "writer");

    verify(boardRepository).save(new Board("3", "title", "main", "writer"));
}
```

<br>

Service 테스트 이므로 boardService로 적는다.        

(`boardRepository.save(new BoardDto("3", "title", "main", "writer"))` ❌ )

```java
Board boardSave = boardService.saveBoard(new BoardDto("3", "title", "main", "writer"));
```

<br><br>

---

## Repository
`@DataJpaTest`는 JPA 관련 컴포넌트만 빠르게 테스트할 때 사용한다.

`@Transactional`이 기본 적용되어 테스트 끝나면 자동 롤백된다. 

![image](/assets/img/posts/193873417-6ac08742-1904-41a6-b766-682940a5568c.png)

<br><br>

```java
public interface Repository extends JpaRepository<Entity, PK>
```

PK를 String으로 하기 위해 PK에 @Id를 사용했다.

```java
@Id
@NotNull
private String boardId;
```

<br>

⬇️

```java
public interface BoardRepository extends JpaRepository<Board, String> {
}
```
