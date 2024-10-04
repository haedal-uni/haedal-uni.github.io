---
categories: Study
tags: [spring, study, summary]
---

싱글톤 생성 과정을 정리하면서 final과 static에 대해서 정리해봤다.                   

<br>

# Singleton
## 생성 과정
예제로 든 싱글톤은 Thread Safe 하지는 않다.                  

<br>

### 1. final로 써서 초기화 시켜주는 방법
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

### 2. final을 쓰지 않고 null로 체크해서 작성하는 방법
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

# final
final은 값을 한 번만 설정할 수 있도록 강제하는 키워드다. 

즉, 한 번 초기화된 값은 다시 변경할 수 없다.

<br><br>

## final의 특징
- 기본 자료형(int, String 등)
  
  final로 선언되면 값 자체를 변경할 수 없다.

```java
final String str = "하이";
str = "바이"; // 불가능 : 참조 변경이 불가능
str.charAt(0); // 가능 : 객체의 메서드 호출은 참조 변경이 아님

// 객체의 메서드 호출이 str = "바이"; 처럼 객체 값을 변경하는게 아니니까
```
메서드 호출은 객체의 내부 상태를 읽는 작업이므로 가능하다.

<br>

- 객체
  final로 선언된 객체의 참조는 변경할 수 없지만 객체 내부 상태는 변경할 수 있다.

```java
public class A {
    private int x = 4;

    public void setX(int x) {
        this.x = x;
    }

    public int getX() {
        return x;
    }
}

public class Service {
    private final A a = new A(); // a는 한 번 초기화되면 다른 객체로 재할당할 수 없음

    public void updateX() {
        a.setX(8); // a의 내부 상태는 변경 가능
    }

    public int getX() {
        return a.getX(); // 변경된 값 확인 가능
    }
}
```
a 객체의 참조는 변경할 수 없지만 객체 내부의 x 값은 변경 가능하다.

따라서 final은 상수가 아니라 한 번만 초기화가 가능하다고 보면 된다.

*상수 : 변경되지 않는 값   

<br><br>

### 예시

`private final UserService userService;`                  

위 코드에서는 final로 UserSerivce를 두고 있는데 이는 userService 참조를 고정한다는 의미를 담고 있다.                  

다른 서비스(예: BoardService)로 userService를 대체할 수 없지만 userService 내부 상태는 변할 수 있다.
                  
<br><br>

**final이 제한하는 것은 참조의 변경이지 객체 내부 상태의 변경은 아니다.**

```java
userService.setName("name"); // 가능: 메서드 호출로 내부 상태는 변경 가능
```

`userService.setName("name")`으로 값을 지정하는데 final은 값을 지정 못하지 않을까?                  

`클래스.set메소드`는 메소드 사정이지 클래스 사정이 아니다.                           

Service에 final왔으니깐 set 하면 안되는거 아닌가?라는 질문에 관한 답은              

final 이 제한을 두는 건 userService 의 대한 변화를 제한하는 것이다.

즉 `userService.setName();` 같은 메서드 호출은 userSerivce 에 영향을 주는 로직이 아니기 때문에 관련이 없다.


<br><br>

## 사용 방법              

`final + 초기화`                  

**final**로 선언된 변수는 반드시 한 번만 초기화되어야 한다.             

초기화는 선언과 동시에 하거나, 생성자에서 초기화할 수 있다.
```java
public class Example {
    private final int x; // final이므로 반드시 초기화 필요

    public Example(int value) {
        this.x = value; // 생성자에서 초기화 가능
    }
}
```
<br>       

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

# static
`static`을 사용하면 메모리에 한 번 할당되어 프로그램이 종료될 때 해제되는 것을 의미한다.

<br>

클래스 안에서 static 키워드가 붙는 경우는 2가지가 존재한다.

1. static 변수 혹은 정적 변수(static 변수 = 정적 변수 = 클래스 변수 = 공용 변수)
2. static 메서드 혹은 정적 메서드

🐣 공유 메모리(공유 변수)라고 생각하면 이해가 쉽다.

<br>

```java
public class Study {
    static int staticVal = 7;
    int globalScope = 10;

    public static void main(String[] args) {
        Study v1 = new Study();
        Study v2 = new Study();
        v1.globalScope = 20;
        v2.globalScope = 30;

        System.out.println(v1.globalScope);  // 20 이 출력된다.
        System.out.println(v2.globalScope);  // 30 이 출력된다.

        // 기존 설정 : static int staticVal = 7;
        v1.staticVal = 10;
        v2.staticVal = 20;

        System.out.println(v1.staticVal);  // 20 이 출력된다.
        System.out.println(v2.staticVal);  // 20 이 출력된다.
    }
}
```
<br><br>

#### 변수 앞에 static 키워드가 붙는 케이스
```java
public static double pi = 3.14
```

<br><br>

#### 메서드 앞에 static 키워드가 붙는 케이스
```java
public static int plus ( int x , int y ){
     return x + y; 
} 
```

<br><br><br>

## static 쓰는 경우?
static 키워드를 붙이면 자바는 메모리 할당을 딱 한번만 하게 되어 메모리 사용에 이점이 있다.                  

static을 사용하는 또 한가지 이유로 **공유**개념을 들 수 있다.

static 으로 설정하면 같은 곳의 메모리 주소만을 바라보기 때문에 static 변수의 값을 공유하게 되는 것이다.

또한, 객체 생성없이 클래스를 통해 메서드를 직접 호출할 수 있다.

<br><br>

**객체를 여러 번 쓸 상황이 생길 때**

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

<br><br><br>

### 접근제어자
static이 붙은 [메소드 or 변수]를 쓸 때 다른 파일에 있으면                  
패키지랑 사용하고자 하는 [메소드 or 변수] 의 접근제어자를 본다.              

패키지가 일단 *public*이어야 [메소드 or 변수] 의 접근제어자를 보겠쥬?                  

<br><br><br>

### 예시
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

<br><br><br>

## JVM 메모리 구조
JVM은 크게 `Garbage collector`, `Execution Engine`, `Class Loader`, `Runtime Data Area` 4가지 영역으로 나누어진다.

이 중에서 static을 이해하는 데 필요한 `Class Loader`와 `Runtime Data Area`(메모리 영역)에 관해 작성했다.

<br>

우리가 코드를 작성한다면 확장자가 java인 `*.java` 파일들을 만든다. 

해당 java 파일들은 Java 컴파일러(javac)에 의해 `.class` 파일인 JAVA Byte Code로 컴파일된다. 

이렇게 컴파일된 바이트 코드들은 `Class Loader`가 메모리가 할당된 `Runtime Data Area`으로 코드들을 적재시킨다.         

<br>

**Java Virtual Machine**
![image](https://user-images.githubusercontent.com/74857364/199482810-2735b380-87d7-487d-8437-bfcea406a043.png){: width="70%"}


<br>

`Runtime Data Area`은 `Method Area`, `Heap Area`, `Stack Area`, `PC register`, `Native Method Stack` 총 5가지로 구분된다.

이 중에서 static을 이해하는 데 필요한 3가지의 영역 중 하나인 `Method Area(Static Area)`은 초기 로드 필요한 정보들           
즉 필요한 패키지 클래스, 인터페이스, 상수, static변수, final 변수, 클래스 멤버 변수 등 로드된 후 메모리에 항상 상주하고 있는 영역이다.          

`Stack Area`는 클래스 안 메서드 실행 시 해당 영역이 할당되며 메서드에서 직접 사용할 지역 변수, 파라미터, 리턴 값, 참조 변수일 경우 주소 값들이 저장된다. 

`Heap Area`은 메서드 안에서 사용되는 객체들을 위한 영역으로 new를 통해 생성된 객체, 배열, immutal 객체 등의 메모리와 값이 저장된다.

<br><br><br>

## static과 메모리 구조

클래스 로더가 .class파일을 탐색 중 static 키워드를 보는 순간 객체가 생성되지 않아도 항상 메모리를 할당해야 하는 멤버로 보고                
`Method Area(Static Area)`에 메모리를 할당한다.

그래서 static 키워드가 붙은 멤버들은 객체(인스턴스)에 소속된 변수가 아니라 클래스에 소속된 변수이기 때문에                    
**클래스 변수** 혹은 **클래스 메서드**라고도 부른다. 

**new**를 통해 객체를 생성하면 각 인스턴스는 서로 독립적이지만 이러한 특징 때문에           
static 키워드가 붙은 멤버들은 모든 객체가 메모리 영역을 공유하기에 공통으로 같은 영역을 바라보기에 아래와 같은 코드가 가능하다.

```java
public class Counter {
    public static int count = 0;
    Counter() {
        this.count++;
        System.out.println(this.count);
    }
    public static void main(String[] args) {
        Counter c1 = new Counter();
        Counter c2 = new Counter();
    }
}
```

<br><br>

같은 이유로 static 메서드 안에서 사용할 변수들은 메모리에 올라가는 순서 때문에 아래와 같은 코드는 불가능하다.                        
(스태틱 메서드 안에서는 인스턴스 변수 접근이 불가능하다)

```java
public class Counter {
    public int count = 0;
    Counter() {
        this.count++;
    }
    public static int getCount() {
        return count; // 에러 발생
    }
    public static void main(String[] args) {
        Counter c1 = new Counter();
        Counter c2 = new Counter();
	System.out.println(Counter.getCount());
    }
}
```

static키워드를 만난 순간 메모리에 적재시켜야 하는데 count 변수에 대해 선언 및 메모리가 할당되어 있지 않아 에러가 발생한다. 

이를 해결하기 위해서는 count변수를 static변수로 만든다면 메모리 로드 시점에 count변수에 대한 선언이 존재하여 에러가 발생하지 않는다.


<br><br><br>

```java
public class JvmStack {
	public static void main(String[] args) {
		add();
	}

	public static void add(){
		minus();
		mul();
	}

	public static void minus(){
		System.out.println("minus");
	}
	public static void mul(){
		System.out.println("mul");
	}
}
```
static이 아닌 변수나 메소드는 static이 jvm에 올라가는 타이밍에 아직 jvm에 올라가지 않았기 때문에                    
사용할 수가 없으므로 관련 메소드는 모두 static으로 작성해줘야한다.

하지만 무분별한 static 사용은 지양한다.

<br>

따라서 아래와 같이 인스턴스 생성으로 작성한다.

```java
public class JvmStack {

	public static void main(String[] args) {
		JvmStack jvmStack = new JvmStack();
		jvmStack.add();
	}

	public void add(){
		mul();
		minus();
	}

	public void minus(){
		System.out.println("minus");
	}
	public void mul(){
		System.out.println("mul");
	}
}
```

<br><br><br>

## 이슈
이러한 static의 특징들 때문에 메서드의 호출 시간이 짧아진다고 무분별한 static의 사용은 java에서 지양된다.

- static 변수는 글로벌 변수에 가까우므로 글로벌 변수는 인스턴스 변수보다 테스트가 까다로워진다.

- static 변수는 객체지향 프로그램의 원칙인 각 객체의 데이터들이 캡슐화되어야 한다는 원칙에 어긋나며                    
   static 변수를 공유한 순간 서로에 영향을 주게 되어 어떤 사이드 이펙트가 발생할지 모른다.
   
- 오버라이딩을 할 수 없으므로 코드의 재사용성이 떨어진다.

- 프로그램이 종료되기 전에 항상 메모리에 상주하고 있어 자주 사용하지 않는 매서드가 누적된다면 GC에 수거되지 못하므로 오히려 메모리 낭비가 발생한다.

<br><br>

🐣 static을 사용하는 것이 좋을 때 : 자주 사용하는 객체 + 만드는데 오래 걸리고 메모리를 많이 사용하는 객체


<br><br><br>


출처                                  
[final 을 쓰는 이유가 궁금합니다.](https://okky.kr/articles/239853)                   
[03-11 형변환과 final](https://wikidocs.net/158529)                   
[07-03 정적(static) 변수와 메소드](https://wikidocs.net/228)                 
[Java에서 자주 보이는 Static이란 무엇일까?](https://honbabzone.com/java/java-static/)          
