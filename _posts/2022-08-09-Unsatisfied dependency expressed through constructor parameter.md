---

categories: Error
tags: [error, spring]
---------------------

## Unsatisfied dependency 에러

```text
Unsatisfied dependency expressed through constructor parameter

available: expected at least 1 bean which qualifies as autowire candidate.
Dependency annotations: {}
```

<br>

이 에러는 Spring이 생성자 주입을 하려고 했지만 주입할 수 있는 Bean을 찾지 못했을 때 발생한다.

<br>

이번 경우에는 `@Service` 클래스에서 생성자 주입을 위해 작성한 `private final` 필드에

Service가 아닌 Model 클래스를 넣어서 에러가 발생했다.

```java
@Service
@RequiredArgsConstructor
public class ExampleService {

    private final UserModel userModel; // 문제 발생 가능
}
```

<br>

`@RequiredArgsConstructor`를 사용하면 `final`이 붙은 필드를 생성자 파라미터로 만들어준다.

그러면 Spring은 `UserModel`을 자동으로 주입하려고 한다.

하지만 `UserModel`은 `@Service`, `@Component`, `@Repository` 같은 어노테이션으로

Spring Bean으로 등록된 클래스가 아니기 때문에 Spring이 주입할 대상을 찾지 못한다.

그래서 다음과 같은 의미의 에러가 발생한다.

> “생성자에 필요한 객체를 주입해야 하는데, 해당 타입의 Bean을 찾을 수 없습니다.”

<br>

### 잘못된 예시

```java
@Service
@RequiredArgsConstructor
public class ExampleService {

    private final UserModel userModel;
}
```

<br>

Model, DTO, Entity 같은 클래스는 보통 Spring이 주입해주는 Bean으로 사용하지 않는다.

필요할 때 직접 생성하거나, 메서드의 파라미터로 전달받아서 사용한다.

<br>

### 수정 예시

```java
@Service
@RequiredArgsConstructor
public class ExampleService {

    private final UserRepository userRepository;

    public void saveUser(UserModel userModel) {
        userRepository.save(userModel);
    }
}
```

<br>

정리하면 `private final`에 작성하는 생성자 주입 대상은

보통 `@Service`, `@Repository`, `@Component` 등으로 Spring Bean에 등록된 클래스여야 한다.

Model, DTO, Entity처럼 단순히 데이터를 담는 클래스는 생성자 주입 대상으로 넣지 않는 것이 좋다.
