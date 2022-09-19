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

- `@DataJpaTest` : Repository들에 대한 빈들을 등록하여 단위 테스트의 작성을 용이하게 한다.

테스트를 위해서는 테스트 컨텍스트를 잡아주어야 할텐데,               
`@DataJpaTest`는 내부적으로 @ExtendWith(SpringExtension.class) 어노테이션을 가지고 있다.

마찬가지로 @DataJpaTest에는 `@Transactional` 어노테이션이 있어서,       
테스트의 롤백 등을 위해 별도로 트랜잭션 어노테이션을 추가하지 않아도 된다.

<img src = https://user-images.githubusercontent.com/74857364/190093647-c6a4be88-4da9-49a7-b2a7-d82ca0ee89e5.png width="40%">

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

<br><br><br>


출처
[[Spring] TDD로 멤버십 등록 API 구현 예제 - (3/5)](https://mangkyu.tistory.com/184)
