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
### 단일 연결 리스트 
데이터와 다른 노드를 가리킬 주소 데이터를 담을 객체가 필요하다. → 노드(*node*)

![image](https://user-images.githubusercontent.com/74857364/196886282-6c3e0ee9-a076-47cd-990f-beafefae4de9.png){: width="30%"}         

사용자가 저장할 데이터는 data 변수에 담기고, reference 데이터(참조 데이터)는 다음에 연결할 노드를 가리키는 데이터가 담긴다.

위와같은 노드들이 여러개가 연결 되어있는 것을 연결 리스트, 즉 **LinkedList**라고 한다.

<br>

이렇게 연결된 노드들에서 '**삽입**'을 한다면 링크만 바꿔주면 되기 때문에 매우 편리하며,

반대로 '**삭제**'를 한다면 삭제 할 노드에 연결 된 이전노드에서 링크를 끊고 삭제할 노드의 다음 노드를 연결해주면 된다.

<br><br><br>

### Node 구현
가장 기본적인 데이터를 담을 Node 클래스를 먼저 구현해줘야한다.

```java
class Node<E> {
 
    E data;
    Node<E> next;  // 다음 노드객체를 가리키는 래퍼런스 변수
 
    Node(E data) {
        this.data = data;
        this.next = null;
    }
}
```
next 가 앞서 그림에서 보여주었던 Reference 변수다. 

다음 노드를 가리키는 변수이며, '노드 자체'를 가리키기 때문에 타입 또한 `Node<E>`타입으로 지정해주어야 한다.

그리고 위 노드를 이용하기 위한 단일 연결리스트(*Singly LinkedList*)를 구현하면 된다.

<br><br><br>

### List Interface
```java
public interface List<E> {

    boolean add(E value);

    void add(int index, E value);

    E remove(int index);

    boolean remove(Object value);

    E get(int index);

    void set(int index, E value);

    boolean contains(Object value);

    int indexOf(Object value);

    int size();

    boolean isEmpty();

    public void clear();
 
}
```
1. `boolean add(E value);`
    - 리스트에 요소를 추가한다.
    - @param : value 리스트에 추가할 요소
    - @return : 리스트에서 중복을 허용하지 않을 경우에 리스트에 이미 중복되는     
      요소가 있을 경우 {@code false}를 반환하고, 중복되는 원소가 없을경우 {@code true}를 반환

2. `void add(int index, E value);`
    - 리스트에 요소를 특정 위치에 추가한다. 
    - 특정 위치 및 이후의 요소들은 한 칸씩 뒤로 밀린다.
    - @param : index 리스트에 요소를 추가할 특정 위치 변수
    - @param : value 리스트에 추가할 요소


3. `E remove(int index);`
    -  리스트의 index 위치에 있는 요소를 삭제한다.
    -  @param : index 리스트에서 삭제 할 위치 변수
    -  @return : 삭제된 요소를 반환

4. `boolean remove(Object value);`
    - 리스트에서 특정 요소를 삭제합니다. 동일한 요소가 여러 개일 경우 가장 처음 발견한 요소만 삭제된다.
    - @param : value 리스트에서 삭제할 요소
    - @return : 리스트에 삭제할 요소가 없거나 삭제에 실패할 경우 {@code false}를 반환하고 삭제에 성공할 경우 {@code true}를 반환 

5. `E get(int index);`

    - 리스트에 있는 특정 위치의 요소를 반환한다.
    - @param : index 리스트에 접근할 위치 변수 
    - @return : 리스트의 index 위치에 있는 요소 반환 

6. `void set(int index, E value);`
    - 리스트에서 특정 위치에 있는 요소를 새 요소로 대체한다.
    - @param : index 리스트에 접근할 위치 변수 
    - @param : value 새로 대체할 요소 변수 

7. `boolean contains(Object value);`
    - 리스트에 특정 요소가 리스트에 있는지 여부를 확인한다.
    - @param : value 리스트에서 찾을 특정 요소 변수 
    - @return : 리스트에 특정 요소가 존재할 경우 {@code true}, 존재하지 않을 경우 {@code false}를 반환  

8. `int indexOf(Object value);`
    - 리스트에 특정 요소가 몇 번째 위치에 있는지를 반환한다.
    - @param : value 리스트에서 위치를 찾을 요소 변수  
    - @return : 리스트에서 처음으로 요소와 일치하는 위치를 반환, 만약 일치하는 요소가 없을경우 -1 을 반환  

9. `int size();`
    - 리스트에 있는 요소의 개수를 반환
    - @return : 리스트에 있는 요소 개수를 반환  

10. `boolean isEmpty();`
    - 리스트에 요소가 비어있는지를 반환한다.
    - @return : 리스트에 요소가 없을경우 {@code true}, 요소가 있을경우 {@code false}를 반환

11. `public void clear();`
    - 리스트에 있는 요소를 모두 삭제한다. 


<br><br><br>

### SingleLinked 클래스
List 인터페이스를 implements 해준다.

```java
public class SingleLinked<E> implements List<E> {
    private Node<E> head;  // 노드의 첫 부분
    private Node<E> tail;  // 노드의 마지막 부분
    private int size;  // 요소 개수

    // 생성자
    public SingleLinked() {
        this.head = null;
        this.tail = null;
        this.size = 0;
    }
}
```

- Node<E> head : 리스트의 가장 첫 노드를 가리키는 변수다.

- Node<E> tail : 리스트의 가장 마지막 노드를 가리키는 변수다.

- size : 리스트에 있는 요소의 개수(=연결 된 노드의 개수)

<br>
	
처음 단일 연결리스트를 생성 할 때에는 아무런 데이터가 없으므로  
	
head와 tail이 가리킬 노드가 없기에 null로 초기화 및 size는 0으로 초기화해준다.

<br><br><br>

### search 메소드 구현
단일 연결리스트이다보니 특정 위치의 데이터를 삽입, 삭제, 검색하기 위해서는 

처음 노드(head)부터 next변수를 통해 특정 위치까지 찾아가야 하기 때문에 `search()` 메소드를 작성한다.
	
```java
// 특정 위치의 노드를 반환하는 메소드
private Node<E> search(int index) {
		
    // 범위 밖(잘못된 위치)일 경우 예외 던지기 
    if(index < 0 || index >= size) {
        throw new IndexOutOfBoundsException();
    }
		
    Node<E> x = head;	// head가 기리키는 노드부터 시작 
		
    for (int i = 0; i < index; i++) {
    x = x.next;	// x노드의 다음 노드를 x에 저장한다
    }
    return x;
}
```

<br><br><br>

### add 메소드 구현
- 가장 앞부분에 추가 - addFirst(E value)

- 가장 마지막 부분에 추가 (기본값) - addLast(E value)

- 특정 위치에 추가 - add(int index, E value)

<br><br>
	
#### addFirst(E value) : 가장 앞부분에 추가
	           
![image](https://user-images.githubusercontent.com/74857364/197574950-15ddc988-d0bb-4cdf-8116-1acc4829cf16.png) 
	
```java
public void addFirst(E value) {
 
    Node<E> newNode = new Node<E>(value);  // 새 노드 생성
    newNode.next = head;  // 새 노드의 다음 노드로 head 노드를 연결
    head = newNode;  // head가 가리키는 노드를 새 노드로 변경
    size++;

    if (head.next == null) {
        tail = head;
    }
}
```
`if (head.next == null) {tail = head;}` 
	
다음에 가리킬 노드가 없는 경우(=데이터가 새 노드밖에 없는 경우)          
	
데이터가 한 개(새 노드)밖에 없으므로 새 노드는 처음 시작노드이자 마지막 노드다.         
	
즉 tail = head 다.
	
<br><br><br>
	
#### addLast(E value) :  가장 마지막 부분에 추가
	
![image](https://user-images.githubusercontent.com/74857364/197575132-01a308a4-449d-43a1-aa04-e47dfb4f5c3a.png)           
	
```java
public boolean add(E value) {
    addLast(value);
    return true;
}
 
public void addLast(E value) {
    Node<E> newNode = new Node<E>(value);  // 새 노드 생성 
 
    if (size == 0) {  // 처음 넣는 노드일 경우 addFisrt로 추가
        addFirst(value);
        return;
    }
 
	
    // 마지막 노드(tail)의 다음 노드(next)가 새 노드를 가리키도록 하고
    // tail이 가리키는 노드를 새 노드로 바꿔준다. 
    tail.next = newNode;
    tail = newNode;
    size++;
}
```
	
<br><br><br>
	
#### add(int index, E value) : 특정 위치에 추가         
	
![image](https://user-images.githubusercontent.com/74857364/197575223-a71113e3-cf85-46c7-8df6-2fcb0ad36ed1.png)          
	
```java
@Override
public void add(int index, E value) {
 
    // 잘못된 인덱스를 참조할 경우 예외 발생
    if (index > size || index < 0) {
        throw new IndexOutOfBoundsException();
    }
    // 추가하려는 index가 가장 앞에 추가하려는 경우 addFirst 호출 
    if (index == 0) {
        addFirst(value);
        return;
    }
    // 추가하려는 index가 마지막 위치일 경우 addLast 호출
    if (index == size) {
        addLast(value);
        return;
    }
		
    // 추가하려는 위치의 이전 노드 
    Node<E> prev_Node = search(index - 1);
	
    // 추가하려는 위치의 노드
    Node<E> next_Node = prev_Node.next;
 
    // 추가하려는 노드
    Node<E> newNode = new Node<E>(value);	
		

    // 이전 노드가 가리키는 노드를 끊은 뒤 새 노드로 변경해준다. 
    // 또한 새 노드가 가리키는 노드는 next_Node로 설정해준다. 
    prev_Node.next = null;
    prev_Node.next = newNode;
    newNode.next = next_Node;
    size++;
 
}
```

<br><br><br>
	
### remove 메소드 구현

- 가장 앞의 요소(head)를 삭제 - remove()

- 특정 index의 요소를 삭제 - remove(int index)

- 특정 요소를 삭제 - remove(Object value)
	
<br><br>

#### remove() : 처음 노드 삭제
	
remove() 는 '가장 앞에 있는 요소'를 제거하는 것이다. 
	
→ head 가 가리키는 요소만 없애주면 된다는 뜻
	
![image](https://user-images.githubusercontent.com/74857364/197545427-4ae910d3-da4b-4718-9941-4482ea60aa80.png)         

```java
public E remove() {

    Node<E> headNode = head;
 
    if (headNode == null)
        throw new NoSuchElementException();
		
    // 삭제된 노드를 반환하기 위한 임시 변수
    E element = headNode.data;
		
    // head의 다음 노드 
    Node<E> nextNode = head.next;
		
    // head 노드의 데이터들을 모두 삭제
    head.data = null;
    head.next = null;
		
    // head 가 다음 노드를 가리키도록 업데이트
    head = nextNode;
    size--;
		
    
    // 삭제된 요소가 리스트의 유일한 요소였을 경우 그 요소는 head 이자 tail이었으므로 
    // 삭제되면서 tail도 가리킬 요소가 없기 때문에 size가 0일경우 tail도 null로 변환
    if(size == 0) {
        tail = null;
    }
    return element;
}
```
리스트에 아무런 요소가 없는데 삭제를 시도하려는 경우 요소를 찾을 수 없기 때문에 
`NoSuchElementException()` 예외를 던져주었다.
	
<br><br><br>

#### remove(int index) 
	
remove(int index) 메소드는 사용자가 원하는 특정 위치(index)를 리스트에서 찾아서 삭제한다.	

*`add(int index, E value)` 와 정반대로 구현해주면 된다.
  
삭제하려는 노드의 이전 노드의 next 변수를 삭제하려는 노드의 다음 노드를 가리키도록 해주면 된다.
  
  
![image](https://user-images.githubusercontent.com/74857364/197572368-ab0616d2-8b01-46bc-84ad-2baac1f8f09a.png)                  

```java
@Override
public E remove(int index) {

    // 삭제하려는 노드가 첫 번째 원소일 경우 
    if (index == 0) {
        return remove();
    }
 
    // 잘못된 범위에 대한 예외 
    if (index >= size || index < 0) {
    throw new IndexOutOfBoundsException();
    }
		
    Node<E> prevNode = search(index - 1);  // 삭제할 노드의 이전 노드 
    Node<E> removedNode = prevNode.next;  // 삭제할 노드 
    Node<E> nextNode = removedNode.next;  // 삭제할 노드의 다음 노드 
 
    E element = removedNode.data;    // 삭제되는 노드의 데이터를 반환하기 위한 임시변수
	
    // 이전 노드가 가리키는 노드를 삭제하려는 노드의 다음노드로 변경 
    prevNode.next = nextNode;
 
    // 만약 삭제했던 노드가 마지막 노드라면 tail을 prevNode로 갱신
    if(prevNode.next == null) {
        tail = prevNode;
    }
	
    // 데이터 삭제 
    removedNode.next = null;
    removedNode.data = null;
    size--;
 
    return element;
}
```
*노드를 얻기 위해 기존에 작성했던 search() 메소드를 이용하면 쉽게 얻을 수 있다.

<br><br><br>

#### remove(Object value)

remove(Object value) 메소드는 사용자가 원하는 특정 요소(value)를 리스트에서 찾아서 삭제한다.

remove(int index) 메소드하고 동일한 메커니즘으로 작동한다. 

다만 고려해야 할 점은 '삭제하려는 요소가 존재하는지'를 염두해두어야 한다.          
예로들어 삭제하려는 요소를 찾지 못했을 경우 false를 반환해주고, 만약 찾았을 경우 remove(int index) 와 동일한 삭제 방식을 사용하면 된다.
  
  
![image](https://user-images.githubusercontent.com/74857364/197573015-3d9c3271-30cc-40a9-a685-fd8f6a083532.png)               

```java
@Override
public boolean remove(Object value) {
    
    Node<E> prevNode = head;
    boolean hasValue = false;
    Node<E> x = head;  // removedNode 
		
    // value 와 일치하는 노드를 찾는다.
    for (; x != null; x = x.next) {
        if (value.equals(x.data)) {
            hasValue = true;
            break;
            }
        prevNode = x;
    }
 
    // 일치하는 요소가 없을 경우 false 반환
    if(x == null) {
        return false;
    }
 
    // 만약 삭제하려는 노드가 head라면 기존 remove()를 사용 	
    if (x.equals(head)) {
        remove();
        return true;
    }
 
    else {
        // 이전 노드의 링크를 삭제하려는 노드의 다음 노드로 연결
        prevNode.next = x.next;
 
        // 만약 삭제했던 노드가 마지막 노드라면 tail을 prevNode로 갱신
        if(prevNode.next == null) {
            tail = prevNode;
        }
        x.data = null;
        x.next = null;
        size--;
        return true;
    }
}
```
	
<br><br><br>
	
### 부가기능

#### get(int index)
index 로 들어오는 값을 인덱스삼아 해당 위치에 있는 요소를 반환하는 메소드다.

이전에 `search()` 메소드를 구현해놓았기 때문에 이를 이용했다.
	
```java
@Override
public E get(int index) {
    return search(index).data;
}
```

🐣 search() 내부에서 잘못된 위치일 경우 예외를 던지기 때문에 따로 예외처리를 해줄 필요는 없다.

<br><br><br>
	
#### set(int index, E value)
	
set 메소드는 기존에 index에 위치한 데이터를 새로운 데이터(value)으로 '교체'하는 것이다. 
	
결과적으로 index에 위치한 데이터를 교체하는 것이기 때문에 마찬가지로 search() 메소드를 사용하여 노드를 찾아내고,              
	해당 노드의 데이터만 새로운 데이터로 변경해주면 된다.  
	
```java
@Override
public void set(int index, E value) {
 
    Node<E> replaceNode = search(index);
    replaceNode.data = null;
    replaceNode.data = value;
}
```
<br><br><br>
	
#### indexOf(Object value)
indexOf 메소드는 사용자가 찾고자 하는 요소(value)의 '위치(index)'를 반환하는 메소드다.
	
찾고자 하는 요소가 중복된다면 가장 먼저 마주치는 요소의 인덱스를 반환한다.
	
찾고자 하는 요소가 없다면요?" 이러한 경우 -1 을 반환한다.
	
<br>
	
⭐ 객체끼리 비교할 때는 동등연산자(`==`)가 아니라 반드시 `.equals()` 로 비교해야 한다. 

객체끼리 비교할 때 동등연산자를 쓰면 값을 비교하는 것이 아닌 주소를 비교하는 것이기 때문에 잘못된 결과를 초래한다.	
	
```java
@Override
public int indexOf(Object value) {
    int index = 0;
	
    for (Node<E> x = head; x != null; x = x.next) {
        if (value.equals(x.data)) {
            return index;
        }
        index++;
    }
	
    // 찾고자 하는 요소를 찾지 못했을 경우 -1 반환
    return -1;
}
```

<br><br><br>
	
#### contains(Object value) 
사용자가 찾고자 하는 요소(value)가 존재 하는지 안하는지를 반환하는 메소드다.

찾고자 하는 요소가 존재한다면 true를, 존재하지 않는다면 false를 반환한다.
	
해당 요소가 존재하는지를 '검사'한다는 기능은 같기 때문에 indexOf 메소드를 이용하여 

만약 음수가 아닌 수가 반환되었다면 요소가 존재한다는 뜻이고, 음수(-1)이 나왔다면 요소가 존재하지 않는다는 뜻이다.	
	
```java
@Override
public boolean contains(Object item) {
	return indexOf(item) >= 0;
}
```
		
<br><br><br>
	
#### size()
모든 리스트 자료구조는 기본적으로 동적 할당을 전제로 한다. 
	
그만큼 요소들을 삽입, 삭제가 많아지면 사용자가 리스트에 담긴 요소의 개수가 몇 개인지 기억하기 힘들다.
	
그렇기에 size 변수의 값을 반환해주는 메소드인 size()를 만들어 줄 필요가 있다.
	
```java
@Override
public int size() {
    return size;
}	
```
	
<br><br><br>
	
#### isEmpty()
현재 ArrayList에 요소가 단 하나도 존재하지 않고 비어있는지를 알려준다.	
	
리스트가 비어있을 경우 true를, 비어있지 않고 단 한개라도 요소가 존재 할 경우 false를 반환해주면 된다.      
→ size가 요소의 개수였으므로 `size == 0` 이면 *true*,  0 이 아니면 *false* 라는 것	
	
```java
@Override
public boolean isEmpty() {
    return size == 0;
}
```
	
<br><br><br> 
	
#### clear()	
모든 요소들을 비워버리는 작업	
	
리스트에 요소를 담아두었다가 초기화가 필요할 때 쓸 수 있는 유용한 존재다.        
또한 모든 요소를 비워버린다는 것은 요소가 0개라는 말로 size 또한 0으로 초기화해준다.

<br> 

이 때 그냥 객체 자체를 null 해주기 보다는 모든 노드를 하나하나 null 해주는 것이 자바의 가비지 컬렉터가 명시적으로 해당 메모리를 안쓴다고 인지하기 때문에 메모리 관리 효율 측면에서 조금이나마 더 좋다.	
	
```java
@Override
public void clear() {
    for (Node<E> x = head; x != null;) {
        Node<E> nextNode = x.next;
        x.data = null;
        x.next = null;
        x = nextNode;
    }
    head = tail = null;
    size = 0;
}
```
<br><br><br>	

구현한 단일 연결리스트의 경우 삽입, 삭제 과정에서 '링크'만 끊어주면 되기 때문에 매우 효율적이라는 것을 볼 수가 있다.

반대로 모든 자료를 인덱스가 아닌 head부터 연결되어 관리하기 때문에 색인(access)능력은 떨어진다.
	
<br><br><br>

출처             
[원형 연결 리스트(circular linked list), 이중 연결 리스트(double linked list)](https://codingsalon.tistory.com/m/44)             
[[자료구조] Array VS LinkedList](https://hee96-story.tistory.com/46)                    
[[자료구조] 연결 리스트의 종류](https://skytitan.tistory.com/45)                    
[[Data Structure] 연결리스트에 대해 알아보자(Linked List)](https://fomaios.tistory.com/entry/DataStructure-%EC%97%B0%EA%B2%B0%EB%A6%AC%EC%8A%A4%ED%8A%B8%EC%97%90-%EB%8C%80%ED%95%B4-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90Linked-List)         
[원형 연결리스트, 이중 연결리스트, 이중 원형 연결리스트 - 자료구조 기초](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=kiminhovator&logNo=220335487935)                  
[[자료구조] 연결리스트의 종류와 특성](https://chaezzing-fly-dev.tistory.com/16)              
코드 출처 → [자바 [JAVA] - Singly LinkedList (단일 연결리스트) 구현하기](https://st-lab.tistory.com/167)
