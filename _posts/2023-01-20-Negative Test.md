---
categories: Study
tags: [spring, test]
---

# Negative Test
[Mock Test](https://haedal-uni.github.io/posts/Mock-Test/) 코드 작성을 다 끝낸 후에 예외에 관한 test도 해봐야한다고 들었다.

<br><br>

## CommentService.java
```java
@Override
public CommentResponseDto postComment(CommentRequestDto commentDto) {
    Registry registry = registryRepository.getReferenceById(commentDto.getRegistryIdx());
    User user = userRepository.findByNickname(commentDto.getNickname()).orElseThrow(UserNotFoundException::new);
    Comment comment = commentDto.toEntity(registry, user);
    commentRepository.save(comment);
    return CommentResponseDto.of(comment);
}
```

<br><br>

## CommentServiceTest.java
```java
@Test
@DisplayName("저장 실패 _ user not found")
void save_user_not_found() {
    when(userRepository.findByNickname(anyString())).thenReturn(Optional.empty());
    CustomException e = assertThrows(UserNotFoundException.class, () ->
            commentService.postComment(commentDto));
    assertEquals(USER_NOT_FOUND, e.getErrorCode());
}
```
처음에는 `.thenThrow(new UserNotFoundException());` 로 예외에 관해서 처리를 하려고 했으나 수정했다.

<br>

`userRepository.findByNickname(commentDto.getNickname())` 자체가 Optonal.empty이다.

그래서 실제 로직의 orElseThrow 전에는 비어있는 값이어야 하므로 **thenReturn**으로 Optional.empty()를 작성했다.

<br><br>

`assertThrows(expectedType, executable)` : executable에서 expectedType의 예외가 발생했는지 확인하는 메서드

`commentService.postComment(commentDto)`를 실행해서 첫번째 인자인 예외 타입과 같은지(혹은 캐스팅이 가능한 상속 관계의 예외인지) 검사

<br><br>

`assertEquals(expect, actual)` : 객체 actual이 expect와 같은 값을 가지는지 확인하는 메서드

<br>

[pr - Negative Test](https://github.com/dal-cho/adme/pull/174/files)

<br><br><br><br>

## Test를 작성하면서
아래 글은 Negative Test와 직접적인 관련은 없지만 test를 작성하면서 코드에 다시 생각해볼 수 있었던 계기가 되어 정리해봤다.

이래서 test 코드는 정상 동작도 중요하지만 예외 test도 중요하다는 생각이 들게 되었다.

<br>

#### CommentService.java
```java
@Override
public CommentResponseDto postComment(CommentRequestDto commentDto) {
    Registry registry = registryRepository.getReferenceById(commentDto.getRegistryIdx());
    User user = userRepository.findByNickname(commentDto.getNickname()).orElseThrow(UserNotFoundException::new);
    Comment comment = commentDto.toEntity(registry, user);
    commentRepository.save(comment);
    return CommentResponseDto.of(comment);
}
```
본 코드로 돌아와서 *findByNickname*만 사용하는게 아니라 그 위 코드 *getReferenceById*로도 값을 찾는다.

`userRepository.findByNickname()`에 exception 처리가 되어있지만 

`registryRepository.getReferenceById()`에는 exception 처리가 되어있지 않다.

그래서 문득 예외상황에 대한 test 코드를 작성하면서 registry의 idx값도 존재하지 않을 수도 있을텐데? 싶었다.

ExceptionHandler로 처리하였고 이 부분은 [findById() vs getReferenceById()](https://haedal-uni.github.io/posts/findById-vs-getReferenceById/) 글에 작성해뒀다.


<br><br><br><br>

*reference*                      
[완벽정리! Junit5로 예외 테스트하는 방법](https://covenant.tistory.com/256)           
[JUnit 대표적 단정(Assert) 메서드, 라이프사이클(Lifecycle) 메서드](https://beststar-1.tistory.com/28)        
