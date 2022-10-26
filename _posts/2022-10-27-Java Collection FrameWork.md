---
categories: Study
tags: [Data-Structure, study, summary]
---

# Java Collection FrameWork
자바에서 컬렉션 프레임워크(collection framework)란 다수의 데이터를 쉽고 효과적으로 처리할 수 있는 표준화된 방법을 제공하는 클래스의 집합을 의미한다.

일정 타입의 데이터들이 모여 쉽게 가공 할 수 있도록 지원하는 자료구조들의 뼈대(기본 구조)

자바에서 제공하는 Collection은 크게 3가지 인터페이스로 나뉘어있다. 

크게 List, Queue, Set(집합), Map, Stack으로 나뉘어 있다. 그리고 각 분야별로 '구현' 된 것들이 있다.

![image](https://user-images.githubusercontent.com/74857364/198018798-dd135ee5-0baa-4b3b-868d-4edf6dbed4d9.png){: width="40%"}

![image](https://user-images.githubusercontent.com/74857364/198056704-e36b062b-acdf-4449-b675-3b5162565f65.png){: width="60%"}

* Iterable 이라 한다면 '반복 가능한' 이라는 정도로 이해하면 된다.

<br><br>

**Collection Interface 상위에 Iterable 이 있는 이유**        
Iterable 인터페이스를 쓰는 모든 클래스들은 기본적으로 for-each 문법을 쉽게 사용할 수 있다.             
→ 반복자로 구현되어 나오게 하는 것이다.

<br><br><br>

## 선형자료구조
선형 자료 구조란 요소가 일렬로 나열되어 있는 자료 구조를 말한다.

- [Array](https://haedal-uni.github.io/posts/Array/)
- [LinkedList](https://haedal-uni.github.io/posts/LinkedList/)
- Vector
- Stack
- Queue


<br><br>

## 비선형 자료구조
일렬로 나열된 것이 아닌, 각 요소가 여러 개의 요소와 연결 된 형태

- Graph
- Tree


<br><br><br>


출처               
[자바 [JAVA] - 자바 컬렉션 프레임워크 (Java Collections Framework)](https://st-lab.tistory.com/142)       
[컬렉션 프레임워크의 개념](http://www.tcpschool.com/java/java_collectionFramework_concept)              
[[Java] 자바 컬렉션 프레임워크(List, Set, Map) 총정리](https://coding-factory.tistory.com/550)                
