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
static은 메모리에 한 번만 할당되고, 프로그램이 종료될 때 해제된다. 

해당 변수를 클래스 레벨에서 공유하도록 한다.

*클래스 레벨: 해당 클래스에서 모든 인스턴스가 동일한 값을 공유하는 것

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

        System.out.println(v1.globalScope);  // 20 
        System.out.println(v2.globalScope);  // 30 

        v1.staticVal = 10;
        v2.staticVal = 20;

        System.out.println(v1.staticVal);  // 20 
        System.out.println(v2.staticVal);  // 20 
    }
}
```
`static`을 사용하면 모든 인스턴스가 같은 값을 공유한다.

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

## static의 장점
메모리 효율성 : 메모리에 한 번만 할당되어 여러 인스턴스가 공유하므로, 메모리 사용이 효율적이다.

공유 메모리 : static 변수를 사용하면 모든 인스턴스가 같은 값을 공유한다. 

객체 생성없이 클래스를 통해 메서드를 직접 호출할 수 있다.

<br><br><br>

## static과 final
`static final`을 함께 사용하면 클래스 레벨에서 공유되면서 변경할 수 없는 상수를 의미한다. 

```java
public static final double PI = 3.14;
```
<br><br>

### final 멤버 변수에 static을 사용하지 않는 경우
→ DI(Dependency Injection) 기법을 사용해 클래스 내부에 외부 클래스 의존성을 집어넣는 경우

bean을 주입하는 경우에는 static을 쓰면 안된다. (ex. `@Component`)    
```java
@Controller
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }
}
```               
DI(의존성 주입)의 목적은 유연성과 확장성이다.

각 객체는 독립적으로 관리되고 환경에 따라 다른 의존성을 주입받아야 할 수 있다.

static은 모든 인스턴스가 동일한 객체를 공유하므로 유연성이 떨어진다.
                  
→ 모든 사용자에게 동일한 객체가 적용되어 각자의 상태를 독립적으로 관리할 수 없게된다.

<br>

참고로 생성자를 만들어야 객체를 만들 수 있는데 

위에서 객체 생성 없이 바로 `클래스.메소드`를 사용할 수 있었던 이유는    

Spring IoC 컨테이너는 객체(Bean)를 생성하고 관리하며, 의존성을 주입해주기 때문이다. 

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

<br><br>

**Class Loader와 Runtime Data Area**

java 코드를 작성면 확장자가 java인 `*.java` 인 소스 파일을 생성한다. 

해당 java 파일들은 Java 컴파일러(javac)에 의해 `.class` 파일인 Byte Code로 컴파일된다. 

Class Loader는 이 바이트코드를 JVM의 메모리 영역인 Runtime Data Area에 적재한다.       

<br>

**Java Virtual Machine**
![image](https://user-images.githubusercontent.com/74857364/199482810-2735b380-87d7-487d-8437-bfcea406a043.png){: width="70%"}


<br>

`Runtime Data Area`은 `Method Area`, `Heap Area`, `Stack Area`, `PC register`, `Native Method Stack` 총 5가지로 구분된다.

- Method Area (Static Area): 클래스 정보, 상수, static 변수, final 변수 등 항상 메모리에 상주하는 영역

- Heap Area: new 키워드로 생성된 객체와 배열이 저장되는 영역

- Stack Area: 메서드 호출 시 사용되는 지역 변수, 파라미터, 리턴 값 등이 저장

- PC Register: 현재 실행 중인 JVM 명령의 주소를 저장

- Native Method Stack: 네이티브 메서드를 위한 스택
  
<br><br><br>

## static과 메모리 구조
Class Loader가 `.class` 파일을 적재하는 동안 static 키워드가 붙은 멤버를 발견하면

JVM은 이를 객체가 생성되지 않았더라도 Method Area에 즉시 메모리 할당을 한다.

→ static 멤버가 클래스에 소속된 변수이므로

static 변수나 메서드는 인스턴스와 무관하게 클래스 레벨에서 공유된다.

static 멤버는 모든 객체가 동일한 메모리 영역을 바라보며 이를 **클래스 변수** 또는 **클래스 메서드**라고 부른다.

```java
public class Counter {
    public static int count = 0; // static 변수
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
위 코드에서 count는 static 변수이기 때문에 c1, c2 두 객체가 같은 메모리 공간을 공유한다. 

따라서 c1이 생성되면서 증가된 값은 c2에서도 동일하게 반영된다.

<br><br>

같은 이유로 static 메서드 안에서 사용할 변수들은 메모리에 올라가는 순서 때문에 아래와 같은 코드는 불가능하다.                        
(스태틱 메서드 안에서는 인스턴스 변수 접근이 불가능하다)

```java      
public class Counter {
    public int count = 0; // 인스턴스 변수
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

위 코드에서 count는 인스턴스 변수이므로 static 메서드 `getCount()` 내에서 접근할 수 없어 에러가 발생한다. 

이를 해결하려면 count를 static 변수로 선언해야 한다.


<br><br><br>

```java
public class JvmStack {
	public static void main(String[] args) {
		add(); // static 메서드 호출
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
위 코드에서 모든 메서드가 static으로 선언되었다. 

static이 아닌 메서드는 JVM에서 메모리에 올라가는 타이밍이 달라져 호출할 수 없기 때문이다. 

따라서 메모리에 적재 시점을 고려해야 static을 올바르게 사용할 수 있다.

<br><br>  

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

### 무분별한 static 사용의 문제점
이러한 static의 특징들 때문에 메서드의 호출 시간이 짧아진다고 무분별한 static의 사용은 java에서 지양된다.

- 캡슐화 문제: static 변수는 객체 간에 공유되기 때문에 객체지향 프로그래밍의 캡슐화 원칙을 깨뜨릴 수 있다.

- 테스트의 복잡성: static 변수는 전역적으로 공유되므로, 독립적인 테스트가 어렵다.

- 오버라이딩 불가: static 메서드는 오버라이딩이 불가능해 코드의 재사용성이 떨어진다.

- 메모리 낭비: 프로그램이 종료되기 전까지 메모리에 남아 있어 자주 사용하지 않는 static 메서드나 변수는 메모리 낭비로 이어질 수 있다.

<br><br>

🐣 static을 사용하는 것이 좋을 때 : 자주 사용하는 객체, 생성 시간이 오래 걸리거나 메모리를 많이 사용하는 객체


<br><br><br>


REFERENCE                                           
- [final 을 쓰는 이유가 궁금합니다.](https://okky.kr/articles/239853)                    
- [03-11 형변환과 final](https://wikidocs.net/158529)                    
- [07-03 정적(static) 변수와 메소드](https://wikidocs.net/228)                  
- [Java에서 자주 보이는 Static이란 무엇일까?](https://honbabzone.com/java/java-static/)           
- [왜 자바에서 final 멤버 변수는 관례적으로 static을 붙일까?](https://djkeh.github.io/articles/Why-should-final-member-variables-be-conventionally-static-in-Java-kor/)                         
