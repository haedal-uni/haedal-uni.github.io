---
categories: Study
tags: [study, summary, 자료구조, 알고리즘, 강의]
---

# Binary Search(이진검색 알고리즘) & Linear Search(선형검색 알고리즘)
아래 강의를 보고 정리한 글이다.                        
[검색 알고리즘? 기초개념 잡아드림. 10분 순삭.](https://www.youtube.com/watch?v=WjIlVlmmNqs&list=RDCMUCUpJs89fSBXNolQGOYKn0YQ&index=1)

<br><br>

이 둘은 Search 알고리즘에 속해있다. 

<br>

🐣 다른 알고리즘으로는 Sorting이 있다. 정렬 알고리즘에서는 자료를 정리한다.

<br><br><br>

## Linear Search

처음부터 끝까지 검색한다.

배열이 커지면 커질수록 선형검색을 하는 시간이 길어진다.

이것을 Linear Time Complexity(선형 시간 복잡도)라고 한다.

→ input이 많으면 많을 수록 수행하는 시간 역시 선형적으로 증가한다

<br>

이것보다 발전된 방법이 Binary Search(이진 검색 알고리즘)이다.

<br><br><br>

## Binary Search
이진 검색에서 이진은 반으로 쪼개는 것을 의미한다.

선형검색과 달리 이진검색에서는 인덱스 0부터 (처음부터) 검색을 하지 않는다.

이진 검색에서는 **중간**에서 시작한다.

정 가운데에 있는 숫자가 우리 목표 숫자보다 큰지 작은지를 본다. (`n > target?`)

만약 목표보다 크다면 아이템의 왼쪽으로 갈 것이다.

![image](https://user-images.githubusercontent.com/74857364/201640805-1f547536-c5b9-4dbd-b753-40a66ddb703a.png)

<br>

input이 2배가 되어도 필요한 스텝은 2배가 되지 않는다.      

이진 검색은 매 스텝마다 절반의 아이템을 없애기 때문이다.    

10 items → 20 items        
3 steps  → 4 steps        

<br>

→ 우리의 자료가 2배가 되어도 작업을 수행하는데 필요한 스텝은 +1이 될 것이다.


<br><br><br>

### 장점
큰 사이즈의 데이터를 처리할 때 효과적이다.

<br><br>

### 단점
Binary Search는 모든 배열에 쓸 수 없고 정렬된 배열(Sorted Array)에서만 사용가능하다.

Linear Search는 어느 배열에서도 가능하지만 Binary Search는 아니다.

<br>

🐣 What is a Sorted Array? 배열이 순서대로 정렬된 경우를 뜻한다.

<br>

정렬된 배열에 아이템을 추가하는 것은 정렬이 안된 경우보다 시간이 더 걸리지만 

정렬된 배열에서 검색을 하는 것은 정렬이 안된 경우보다 시간이 훨씬 빠르다.

<br>

![AC_ 20221114-204247](https://user-images.githubusercontent.com/74857364/201651646-bb9e533f-6bc1-400f-bc55-41b650f827fe.gif)


넣어야 할 값보다 큰 값을 찾기 위해 아이템들을 하나하나 비교해야하고 

이를 통해 포지션을 찾고 나면 해당 아이템 오른쪽에 있는 모든 것을 옮겨야한다. (넣어야 할 값보다 큰 값의 왼쪽에 넣어야하므로)

하지만 이렇게 배열에 추가하고 정렬하는 것에 시간을 더 투자하면 검색을 하는 순간 빨라지게 된다.

<br><br><br>

## 정리

이진 검색은 거대한 배열을 다룰 때 효과적이다.

그러나 이진 검색을 하고 싶다면 배열을 정리해야한다.

검색을 많이 하는 상황이라면 정렬을 해야한다

하지만 배열 정렬을 하려면 아이템 추가시 시간이 소요된다.

이러한 관계를 인지해야한다.
