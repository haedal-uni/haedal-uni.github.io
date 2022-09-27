---
categories: Study
tags: [spring, study, summary]
---

싱글톤 생성 과정을 정리하면서 final과 static에 대해서 정리해봤다.                   

<br>

## Singleton
### 생성 과정
예제로 든 싱글톤은 Thread Safe 하지는 않다.                  

<br>

#### 1. final로 써서 초기화 시켜주는 방법
1. 자기 자신을 `private static final`으로 선언                  
2. public으로 *getInstance*를 선언해서 이 메소드를 통해서만 조회하도록 허용                  
3. 생성자를 private 으로 선언해서 외부에서 *new* 키워드를 사용한 객체 생성을 못하게 막는다                  

```java
class Singleton{
  private static final Singleton instance = new Singleton();

  private Singleton(){
  }

  public static Singleton getInstance(){
    return instance;
  }
}

public class SingletonTest {
  public static void main(String[] args) {
    Singleton singleton1 = Singleton.getInstance();
    Singleton singleton2 = Singleton.getInstance();
    System.out.println(singleton1);
    System.out.println(singleton2);
  }
}
```
![image](https://user-images.githubusercontent.com/74857364/192307430-e9683fee-a5f4-48cf-a436-101bf78744c5.png)

<br>

#### 2. final을 쓰지 않고 null로 체크해서 작성하는 방법
```java
class Singleton{
	private static Singleton instance;

	private Singleton(){
	}

	public static Singleton getInstance(){
		if (instance == null) {
			instance = new Singleton();
		}
		return instance;
	}
}

public class SingletonTest {
	public static void main(String[] args) {
		Singleton singleton1 = Singleton.getInstance();
		Singleton singleton2 = Singleton.getInstance();
		System.out.println(singleton1);
		System.out.println(singleton2);
	}
}
```

final을 사용하는 1번 같은 경우에는 초기화를 해줘야 한다.                  

final이 상수이기 때문에 한번 초기화 시켜줘야한다.(*상수란 변하지 않고 항상 일정한 값)

<br>

2번은 final을 사용하지 않아서 초기화를 시켜주지 않은 것이다.

<br>

`private Singleton(){}`을 생성해야지 `new Singleton()`을 만들 수 있다.            

→ 생성자를 호출해야지 객체를 만들 수 있다.                  

<br>

`new 객체();` : 객체 생성 == 인스턴스 생성                  

객체를 변수에 처음 담는게 초기화이다.                  
                  
<br>
<br>
<br>

---

## final
자료형에 값을 단 한번만 설정할수 있게 강제하는 키워드이다.

즉, 값을 한번 설정하면 그 값을 다시 설정할 수 없다는 말이다.

<br>
<br>

### final 쓰는 이유?
final 을 선언함으로 인해 상수라는 개념을 가지게 된다.

상수라는 것은 절대 변하지않는 값을 뜻한다.                  

<br><br>

예를 하나 들어본다.

`private final UserService userService;`                  

위 코드에서는 final로 UserSerivce를 두고 있는데 이는 서비스 고정 의미를 담고 있다.                  

UserService를 쓰려고 했는데 BoardService가 오면 안되니깐 이를 고정하는 것이다.
                  
<br><br>

또 하나 예를 들어본다.

`private final UserService userService;`

`userService.setName("name")`으로 값을 지정하는데 final은 값을 지정 못하지 않을까?                  

`클래스.set메소드`는 메소드 사정이지 클래스 사정이 아니다.                           

Service에 final왔으니깐 set 하면 안되는거 아닌가?라는 질문에 관한 답은              

final 이 제한을 두는 건 userService 의 대한 변화를 제한하는 것이다.

즉 `userService.setName();` 같은 메서드 호출은 userSerivce 에 영향을 주는 로직이 아니기 때문에 관련이 없다.

<br><br>

### 사용 방법              

`final + 초기화`                  

*final*이 있으면 객체 선언이 필수이고                  

*final*이 없으면 객체 선언 자유이지만 쓸 경우에는 초기화가 필수이다.                  

<br>

초기화가 변수를 선언하고 값을 넣어준다. 그리고 값 변경x                  

사용하기 전에 final 초기화 코드를 작성하면 된다.                

초기화 코드 작성 전에 해당 코드를 사용하면 컴파일 에러가 뜬다.                  

<br>

**초기화 하지 않았을 때**

![image](https://user-images.githubusercontent.com/74857364/192318430-f659ec4c-51fe-48d1-abbd-c858f2c93b57.png)

<br>

**초기화 했을 때**

![image](https://user-images.githubusercontent.com/74857364/192318592-e6ad197f-7e4c-4da7-b292-9d0220652c63.png)


<br>
<br>
<br>

---

## static
static 키워드를 붙이면 자바는 메모리 할당을 딱 한번만 하게 되어 메모리 사용에 이점이 있다.                  

static을 사용하는 또 한가지 이유로 **공유**개념을 들 수 있다.

static 으로 설정하면 같은 곳의 메모리 주소만을 바라보기 때문에 static 변수의 값을 공유하게 되는 것이다.

또한, 객체 생성없이 클래스를 통해 메서드를 직접 호출할 수 있다.

<br><br>

### static 쓰는 경우?
객체를 여러 번 쓸 상황이 생길 때

<br>

final 과 같은 예를 들어본다.

Service를 활용해서 여러 번 사용할 것이기 때문에 static을 사용하면 될까?
                  
`private final UserService userService;`                  

먼저 빈을 주입하는 경우에는 static을 쓰면 안된다. (ex. *@Component*)                      

*static을 쓰면 객체를 생성안해도 되는데 bean 주입하는 경우에는 static을 쓰면 안된다.                  

<br>

여기서 userService.setName("이름") 으로 객체 생성 없이 바로 사용할 수 있던거 아닌가? 라는 생각이 든다면 🙅🏻‍♀️                  
                  
그런 경우 `@RequiredArgsConstructor` 같은 어노테이션이 있는지 확인해본다.  

생성자를 만들어야 객체를 만들 수 있는데 위에서 객체 생성 없이 바로 `클래스.메소드`를 사용할 수 있었던 이유는      

`@RequiredArgsConstructor` 같은 생성자 어노테이션이 객체 생성을 생략해주는 것이다. 

<br>

**어노테이션 작성x**
```java
public class UserController {
  private final UserService userService;
  
  public UserController(UserService userService) {
    this.userService = userService;
  }

}
```

<br>

**어노테이션 작성**
```java
@RequiredArgsConstructor
public class UserController {
  private final UserService userService;
}
```

<br><br>

#### 접근제어자
static이 붙은 [메소드 or 변수]를 쓸 때 다른 파일에 있으면                  
패키지랑 사용하고자 하는 [메소드 or 변수] 의 접근제어자를 본다.              

패키지가 일단 *public*이어야 [메소드 or 변수] 의 접근제어자를 보겠쥬?                  

<br><br>

#### 예시
[점프 투 자바 - 03 정적(static) 변수와 메소드] 에서
```java
class Counter  {
    int count = 0;
    Counter() {
        this.count++;
        System.out.println(this.count);
    }
}
```
⬇️
```java
class Counter  {
    static int count = 0;
    Counter() {
        count++;  // count는 더이상 객체변수가 아니므로 this를 제거하는 것이 좋다.
        System.out.println(count);  // this 제거
    }
}
```
위 코드와 아래 코드의 차이점은 **static 유무**이다.

아래 코드에서 static을 붙이면서 this를 뺐다. 왜일까?

그 객체만의 것이 아니니깐 this를 뺀 것이다.

c1.count, c2.count 가 아니라 counter의 count이므로

공유 count이기 때문에 counter 전체를 아우르는 것이어서 this를 없앤 것이다.

<br><br>

출처                                  
[final 을 쓰는 이유가 궁금합니다.](https://okky.kr/articles/239853)                  
[03-11 형변환과 final](https://wikidocs.net/158529)                  
[07-03 정적(static) 변수와 메소드](https://wikidocs.net/228)                  
