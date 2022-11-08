---
categories: Study
tags: [Data-Structure, study, summary]

---


# 연결 리스트

연결 리스트는 데이터를 감싼 노드를 포인터로 연결해서 공간적인 효율성을 극대화 시킨 자료 구조다.

삽입과 삭제가 `O(1)`이 걸리며 탐색에는 `O(n)`이 걸린다.

<br><br><br>

## 구조
연결 리스트는 아래와 같이 노드와 포인터로 이루어져 있고 다음 노드와 연결되어 있다.

![image](https://user-images.githubusercontent.com/74857364/196752267-a32b51c2-65cd-4667-8cf6-64a794aa3472.png){: width="55%"}             

- 자료의 주소 값으로 노드를 이용해 서로 연결되어 있는 구조를 갖는다.
  - → 메모리를 연속적으로 사용하지 않는다.
            
- 순차적 접근방식이다. → 특정 원소에 접근하기 위해서 처음부터 검색하면서 찾는다. 
                                
- 동적으로 삽입, 삭제가 편하다.

- 원소를 삽입할 경우
  - 맨 앞 , 맨 뒤 삽입은 위치를 찾지 않아도 되서 시간 복잡도 O(1)이다.
  - 중간 삽입은 이전 노드와 다음 노드의 위치를 알고 있는 경우 시간 복잡도는 O(1)이다.
  - 하지만 탐색을 해야하는 경우 시간 복잡도 O(n)이다.
   
- 원소를 삭제할 경우
  - 삽입과 마찬가지로 맨 앞, 맨 뒤 삭제는 시간 복잡도 O(1)이다.
  - 중간 삭제는 시간 복잡도 O(n) 또는 O(1)이다. (삽입과 같음)
   
- 특정 위치에 있는 원소에 바로 접근이 불가능하다. (주소를 바로 알 수 없기 때문)
  - 원하는 원소를 찾기 위해서 최소 한 번은 리스트를 순회해야하기 때문에 시간 복잡도는 O(n)이다.
   
- 메모리는 새로운 노드가 추가될 때 (Runtime) Heap 영역에 할당한다.


<br><br><br>

## 종류

prev 포인터와 next 포인터로 앞과 뒤의 노드를 연결시킨 것이 연결 리스트이며         

연결 리스트는 `싱글 연결 리스트`, `이중 연결 리스트`, `원형 이중 연결 리스트`가 있다.      

<br>

*맨 앞에 있는 노드를 헤드(head)라고 한다.                  

- **싱글** 연결 리스트 : *next* 포인터만 가진다.                             
- **이중** 연결 리스트 : *next* 포인터와 *prev* 포인터를 가진다.
- **원형 이중** 연결 리스트 : 이중 연결 리스트와 같지만 마지막 노드의 *next* 포인터가 헤드 노드를 가리키는 것을 말한다.

<br><br><br>

### 싱글 연결 리스트(Singly Linked List)
**자료들이 링크로 서로 연결되어 있는 형태**

![image](https://user-images.githubusercontent.com/74857364/196751267-af74923a-c845-4e9c-9bec-d235ce559021.png){: width="70%"}            

- 단방향 링크
- 이전 노드에 접근하기 위해선 첫 번째 노드부터 다시 순회해야한다.
- 각 노드 당 한 개의 포인터가 있고 포인터는 다음 노드의 위치를 가르킨다.
- 테일은 가장 마지막이므로 다음을 가리키는 포인터를 갖지 않는다.


<br><br><br>

### 원형 연결 리스트(Circular Linked List)
**가장 마지막 자료가 가장 처음 자료와 연결되어 있는 형태**

단일 연결 리스트(Single Linked List)는 마지막 노드가 null을 가리킨다. 

이 마지막 노드가 첫 번째 노드를 가리키게 하면 원형 연결 리스트가 된다.        

🐣 단일 연결 리스트의 테일에 포인터가 추가된 형태로 테일의 포인터는 헤더를 가르켜 원형이 되도록 한다. 

![image](https://user-images.githubusercontent.com/74857364/196751291-1036a092-b769-4d40-a3a3-bdc37d851ae2.png)      

- 단방향 링크
- 마지막 노드와 첫 번째 노드가 연결된 원형 구조
- 이전 노드에 접근하기 위해서 계속 한 방향으로만 순회하면 된다.

<br><br><br>

### 이중 연결 리스트(Doubly Linked List)
**앞 뒤 자료가 서로서로에게 양쪽으로 연결되어 있는 형태**

단일 연결 리스트는 포인터를 한 개 가지고 있어 다음 노드만 가리킬 수 있었다면             

이중 연결 리스트는 포인터를 두 개 가지고 있어 이전 노드와 다음 노드를 가리킨다.         

![image](https://user-images.githubusercontent.com/74857364/196751325-402b2865-8492-46c7-985a-4187c66ad9bd.png)  

- 양방향 링크
- 각 노드가 앞 뒤로 연결된다.
- 이전 노드에 직접 접근(Direct Access) 가능하다.

<br>

이중 연결 리스트는 앞에서부터 요소를 넣는 `push_front()`, 뒤에서부터 요소를 넣는 `push_back`,               

중간에 요소를 넣는 `insert()` 등의 함수가 있다.

<br><br><br>

#### 특성 비교          
**이전 노드에 대한 접근 연산**  

- **싱글** 연결 리스트 : 이전노드를 접근할수 있는 방법이 없다. 무조건 가장 처음 노드부터 검색해야한다.           
- **원형** 연결 리스트 : 계속 가다보면 나의 이전 노드의 값을 찾을 수 있다.              
- **이중** 연결 리스트 : 이중으로 연결되어 있기 때문에 바로 이전 노드를 접근 할 수 있다.     

<br><br><br>

### 원형 이중 연결 리스트
이중 연결 리스트와 원형 연결 리스트의 특성을 합친 개념이다.

<br>

인접한 두개의 노드는 서로의 위치를 알고 있으며,                 

맨 마지막 노드는 맨 앞의 노드인 HEAD 노드와 인접하게 되어 서로의 위치를 알게 된다.       

따라서 원형의 형태를 띄게 된다.    


![image](https://user-images.githubusercontent.com/74857364/196751343-8729ecea-f28e-4311-898f-d93358547db2.png){: width="70%"}         

- HEAD에서 부터 시작해서 시계방향 또는 시계 반대방향으로 순환이 가능하다.        


<br><br><br>

## code

[Singly LinkedList 코드 직접 구현해보기](https://haedal-uni.github.io/posts/LinkedList-%EC%BD%94%EB%93%9C-%EA%B5%AC%ED%98%84/)         

[J 팀원 코드 보기](https://github.com/juniorBoard/Data-Structure/tree/main/Jehyeop/Prac/DataStructurePrac/src/main/java/LinkedList)

[M 팀원 코드 보기](https://github.com/juniorBoard/Data-Structure/tree/main/Minwoo/list/linkedlist)

<br><br>

### Dal 클래스 생성
이후에 예시로 사용될 클래스를 작성했다.
```java
class Dal{
    private String name;
    private int age;

    public Dal(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

<br><br>

### LinkedList 선언
LinkedList 선언은 ArrayList선언 방식과 같다.   

다만 LinkedList에서는 초기의 크기를 미리 생성할수는 없다.  

<br>

```java
LinkedList list1 = new LinkedList();  // 타입 미설정 Object로 선언된다.
LinkedList<Integer> num = new LinkedList<Integer>();  // 타입설정 int타입만 사용가능
LinkedList<Integer> num2 = new LinkedList<>(); // new에서 타입 파라미터 생략가능
LinkedList<Integer> list2 = new LinkedList<Integer>(Arrays.asList(1,2)); // 생성시 값 추가
```

<br><br>

### LinkedList 값 추가

```java
LinkedList<Integer> addList = new LinkedList<Integer>();
addList.addFirst(5);  // 가장 앞에 데이터 추가
addList.addLast(17);  // 가장 뒤에 데이터 추가
addList.add(21);  // 데이터 추가
addList.add(1, 3);  // index 1에 데이터 3 추가
```

LinkedList에 값을 추가하는 방법은 여러 개가 있는데 대중적으로 add(index, value) 메소드를 사용한다.

index를 생략하면 가장 마지막에 데이터가 추가된다.

![image](https://user-images.githubusercontent.com/74857364/200551767-68bd80cd-1d2f-42f6-8ce2-c8ba2b24dfa9.png)

<br><br>

```java
LinkedList<Dal> dal = new LinkedList<Dal>();  // 타입설정 Dal 객체만 사용가능
Dal dal1 = new Dal("hae", 1);
dal.add(dal1);
dal.add(new Dal("dal", 2));
    
for(Dal d : dal) { // for문을 통한 전체출력
    System.out.println("dal 이름 전체 출력  :   " + d.name);
}
```

![image](https://user-images.githubusercontent.com/74857364/200551376-980fa7c1-33bd-4f15-9397-74e566791072.png)

<br><br>

### LinkedList 값 삭제
```java
LinkedList<Integer> removeList = new LinkedList<Integer>(Arrays.asList(1,2,3,4,5));
removeList.removeFirst(); // 가장 앞의 데이터 제거
removeList.removeLast(); // 가장 뒤의 데이터 제거
removeList.remove(); // 생략시 0번째 index제거
removeList.remove(1); // index 1 제거
removeList.clear(); // 모든 값 제거
```

<br><br>

### LinkedList 크기 구하기
```java
LinkedList<Integer> list = new LinkedList<Integer>(Arrays.asList(1,2,3));
System.out.println(list.size()); // list 크기 : 3
```

<br><br>

### LinkedList 값 출력
위에서 값을 출력 시키기 위해서 코드를 사용했었는데 이 코드만 다시 한번 보자면 아래와 같다.

```java
System.out.println(list.get(0)); // 0번째 index 출력
    
for(Integer i : list) { // for문을 통한 전체출력
    System.out.println(i);
}
    
Iterator<Integer> iter = list.iterator(); // Iterator 선언
while(iter.hasNext()){ // 다음값이 있는지 체크
    System.out.println(iter.next()); //값 출력
}
```
![image](https://user-images.githubusercontent.com/74857364/200555301-4af4239e-15c7-48a8-8140-98ae7d2b2b33.png)

![image](https://user-images.githubusercontent.com/74857364/200555166-ca467477-62b3-4b3f-8880-02257e21b308.png)


<br><br>

### LinkedList 값 검색
```java
System.out.println(list.contains(1)); // list에 1이 있는지 검색 : true
System.out.println(list.indexOf(1)); // 1이 있는 index반환 없으면 -1
```

<br><br><br>

출처             
[원형 연결 리스트(circular linked list), 이중 연결 리스트(double linked list)](https://codingsalon.tistory.com/m/44)             
[[자료구조] Array VS LinkedList](https://hee96-story.tistory.com/46)                    
[[자료구조] 연결 리스트의 종류](https://skytitan.tistory.com/45)                    
[[Data Structure] 연결리스트에 대해 알아보자(Linked List)](https://fomaios.tistory.com/entry/DataStructure-%EC%97%B0%EA%B2%B0%EB%A6%AC%EC%8A%A4%ED%8A%B8%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90Linked-List)         
[원형 연결리스트, 이중 연결리스트, 이중 원형 연결리스트 - 자료구조 기초](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=kiminhovator&logNo=220335487935)                  
[[자료구조] 연결리스트의 종류와 특성](https://chaezzing-fly-dev.tistory.com/16)               
코드 출처 → [[Java] 자바 LinkedList 사용법 & 예제 총정리](https://coding-factory.tistory.com/552)         
