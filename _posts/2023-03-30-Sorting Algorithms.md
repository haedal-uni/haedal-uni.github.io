---
categories: Study
tags: [study, summary, Algorithm, Sorting Algorithms]
---

## 선택 정렬 : Selection Sort
**해당 순서에 원소를 넣을 위치는 이미 정해져 있고, 어떤 원소를 넣을지 선택한다**

목록의 정렬되지 않은 부분에서 가장 작은 요소를 반복적으로 선택하여 목록의 정렬된 부분으로 이동하여 작동

![image](https://user-images.githubusercontent.com/74857364/228463294-221e3104-2307-4bf0-b5d4-a8ca9eef0735.png){: width="50%"}

<br>

### process
**1.** 주어진 배열 중에 최소값을 찾는다.

**2.** 그 값을 맨 앞에 위치한 값과 교체한다.

**3.** 맨 처음 위치를 뺀 나머지 배열을 같은 방법으로 교체한다.

![image](https://github.com/GimunLee/tech-refrigerator/raw/master/Algorithm/resources/selection-sort-001.gif)

<br>

목록의 정렬되지 않은 부분에서 가장 작은 요소를 반복적으로 선택하고 정렬되지 않은 부분의 첫 번째 요소와 교체한다. 

전체 목록이 정렬될 때까지 목록의 정렬되지 않은 나머지 부분에 대해 이 프로세스가 반복된다.

반복할 때마다 정렬된 하위 배열 크기는 1씩 증가하고 정렬되지 않은 하위 배열 크기는 1씩 감소한다.

```java
void selectionSort(int[] arr) {
    int indexMin, temp;
    for (int i = 0; i < arr.length-1; i++) {        // 1. 위치(index)를 선택
        indexMin = i;
        for (int j = i + 1; j < arr.length; j++) {  // 2. i+1번째 원소부터 선택한 위치(index)의 값과 비교
            if (arr[j] < arr[indexMin]) { // 3.     
                indexMin = j;
            }
        }
        
        // 4. swap(arr[indexMin], arr[i])
        temp = arr[indexMin];
        arr[indexMin] = arr[i];
        arr[i] = temp;
  }
  System.out.println(Arrays.toString(arr));
}
```
**3.**  오름차순이므로 현재 선택한 자리에 있는 값보다 순회하고 있는 값이 작다면, 위치(index)를 갱신    

**4.** '2'번 반복문이 끝난 뒤에는 indexMin에 '1'번에서 선택한 위치(index)에 들어가야하는 값의 위치(index)를 갖고 있으므로 서로 교환(swap)

<br><br>

**ex) 1회전 결과 예시**
```java
void selectionSort(int[] arr) {
    int indexMin, temp;
    for (int i = 0; i < arr.length-1; i++) { 
        indexMin = i; // indexMin = 0
        for (int j = i + 1; j < arr.length; j++) {  
            if (arr[j] < arr[indexMin]) { // arr[3] = 3 < arr[0] = 9    
                indexMin = j; // indexMin = 3
            }
        }
        
        temp = arr[indexMin]; // temp = arr[3] → temp = 3
        arr[indexMin] = arr[i]; // arr[3] = arr[0] → arr[3] = 9
        arr[i] = temp; // arr[i] = 3; → arr[0] = 3;
  }
}
```

<br><br>

*reference*         
[선택 정렬(Selection Sort)](https://gyoogle.dev/blog/algorithm/Selection%20Sort.html)      
[Selection Sort Algorithm](https://www.geeksforgeeks.org/selection-sort/)           
[기본적인 정렬 알고리즘 (선택, 삽입, 버블)](https://jongmin92.github.io/2017/11/06/Algorithm/Concept/basic-sort/)             
[[알고리즘] 선택 정렬(selection sort)이란](https://gmlwjd9405.github.io/2018/05/06/algorithm-selection-sort.html)               

<br><br><br><br>

## 버블 정렬 : Bubble Sort
**선택 정렬과 같이 이미 해당 순서에 원소를 넣을 위치는 정해져 있고, 어떤 원소를 넣을지 선택한다**

선택 정렬과는 다르게 최대값을 찾고, 그 최대값을 맨 마지막 원소와 교환하는 과정에서 차이가 있다.

![image](https://user-images.githubusercontent.com/74857364/228463196-c0dc78a6-39bf-4e4f-821e-da2533d8a3d4.png)

<br>

### process
**1.** 1회전에 첫 번째 원소와 두 번째 원소를, 두 번째 원소와 세 번째 원소를, 세 번째 원소와 네 번째 원소를, … 

이런 식으로 (마지막-1)번째 원소와 마지막 원소를 비교하여 조건에 맞지 않는다면 서로 교환한다.

<br>

**2.** 1회전을 수행하고 나면 가장 큰 원소가 맨 뒤로 이동하므로 2회전에서는 맨 끝에 있는 원소는 정렬에서 제외되고, 

2회전을 수행하고 나면 끝에서 두 번째 원소까지는 정렬에서 제외된다. 

이렇게 정렬을 1회전 수행할 때마다 정렬에서 제외되는 데이터가 하나씩 늘어난다.

![image](https://github.com/GimunLee/tech-refrigerator/raw/master/Algorithm/resources/bubble-sort-001.gif)

<br><br>

### code
```java
void bubbleSort(int[] arr) {
    int temp = 0;
   for(int i = 0; i < arr.length; i++) {       // 1.
      for(int j= 1 ; j < arr.length-i; j++) { // 2.
         if(arr[j-1] > arr[j]) {             // 3.
         
         // swap(arr[j-1], arr[j])
            temp = arr[j-1];
            arr[j-1] = arr[j];
            arr[j] = temp;
         }
      }
   }
   System.out.println(Arrays.toString(arr));
}
```
**1.** 제외될 원소의 갯수를 의미한다. 1회전이 끝난 후, 배열의 마지막 위치에는 가장 큰 원소가 위치하기 때문에 하나씩 증가시켜준다.

<br>

**2.** 원소를 비교할 index를 뽑을 반복문이다.                     

j는 현재 원소를 가리키고, j-1은 이전 원소를 가리키게 되므로, j는 1부터 시작하게 된다.

<br>

**3.** 현재 가르키고 있는 두 원소의 대소를 비교한다.                     

해당 코드는 오름차순 정렬이므로 현재 원소보다 이전 원소가 더 크다면 이전 원소가 뒤로 가야하므로 서로 자리를 교환해준다.

<br><br>

**예시**
```java
void bubbleSort(int[] arr) {
   int temp = 0;
   for(int i = 0; i < arr.length; i++) {
   
      for(int j= 1 ; j < arr.length-i; j++) {
      
         if(arr[j-1] > arr[j]) { // arr[0] > arr[1] → 7 > 4
         
            temp = arr[j-1]; // temp = arr[0]  → temp = 7
            arr[j-1] = arr[j]; // arr[0] = arr[1]  → arr[0] = 4
            arr[j] = temp; // arr[1] = 7
         }
      }
   }
}
```

<br><br>

*reference*         
[기본적인 정렬 알고리즘 (선택, 삽입, 버블)](https://jongmin92.github.io/2017/11/06/Algorithm/Concept/basic-sort/)             
[거품 정렬(Bubble Sort)](https://gyoogle.dev/blog/algorithm/Bubble%20Sort.html)                  
[[알고리즘] 버블 정렬(bubble sort)이란](https://gmlwjd9405.github.io/2018/05/06/algorithm-bubble-sort.html)                       

<br><br><br><br>

## 삽입 정렬 : Insertion Sort
**매 순서마다 해당 원소를 삽입할 수 있는 위치를 찾아 해당 위치에 넣는다**

![image](https://user-images.githubusercontent.com/74857364/228463757-3316318a-10aa-44ed-bcb2-5534ab844c36.png)

<br>

### process
**1.** 정렬은 2번째 위치(index)의 값을 temp에 저장한다.

**2.** temp와 이전에 있는 원소들과 비교하며 삽입해나간다.

**3.** '1'번으로 돌아가 다음 위치(index)의 값을 temp에 저장하고, 반복한다

![image](https://github.com/GimunLee/tech-refrigerator/raw/master/Algorithm/resources/insertion-sort-001.gif)

<br><br>

### code
```java
void insertionSort(int[] arr) {
   for(int index = 1 ; index < arr.length ; index++){ // 1.
      int target = arr[index];
      int prev = index - 1;
      
      while( (prev >= 0) && (arr[prev] > target) ) {    // 2.
         arr[prev+1] = arr[prev]; // 이 전 원소를 한칸 씩 뒤로 미룬다.
         prev--;
      }
      
      arr[prev + 1] = target;                           // 3.
   }
   System.out.println(Arrays.toString(arr));
}
```
**1.** 첫 번째 원소 앞(왼쪽)에는 어떤 원소도 갖고 있지 않기 때문에, 두 번째 위치(index)부터 탐색을 시작한다.        

temp에 임시로 해당 위치(index) 값을 저장하고, prev에는 해당 위치(index)의 이전 위치(index)를 저장한다.             

<br>

**2.** 이전 위치(index)를 가리키는 prev가 음수가 되지 않고, 이전 위치(index)의 값이 '1'번에서 선택한 값보다 크다면,      

서로 값을 교환해주고 prev를 더 이전 위치(index)를 가리키도록 한다.

<br>

**3.** '2'번에서 반복문이 끝나고 난 뒤, prev에는 현재 temp 값보다 작은 값들 중 제일 큰 값의 위치(index) 를 가리키게 된다.        

따라서, (prev+1)에 temp 값을 삽입해준다.

<br><br>

**예시**
```java
void insertionSort(int[] arr) {
   for(int index = 1 ; index < arr.length ; index++){
      int target = arr[index]; // target = arr[1] → temp = 5
      int prev = index - 1; // prev = 0
      
      // target이 이전 원소보다 크기 전까지 반복
      while( (prev >= 0) && (arr[prev] > target) ) {
         arr[prev+1] = arr[prev]; // arr[1] = arr[0] → arr[1] = 8
         prev--;
      }
      
      /*
      위 반복문에서 탈출하는 경우 앞의 원소가 target보다 작다는 의미이므로
      target 원소는 prev번째 원소 뒤에 와야한다.
      그러므로 temp는 prev+1에 위치하게 된다.
      */
      arr[prev + 1] = temp;   // arr[1] = 5
   }
}
```
결과적으로 타겟 이전 원소가 타겟 숫자보다 크기 직전까지 모든 수를 뒤로 한 칸씩 밀어내는 것이다. 

<br><br>

*reference*         
[기본적인 정렬 알고리즘 (선택, 삽입, 버블)](https://jongmin92.github.io/2017/11/06/Algorithm/Concept/basic-sort/)             
[삽입 정렬(Insertion Sort)](https://gyoogle.dev/blog/algorithm/Insertion%20Sort.html)        
[[알고리즘] 삽입 정렬(insertion sort)이란](https://gmlwjd9405.github.io/2018/05/06/algorithm-insertion-sort.html)         
[자바 [JAVA] - 삽입 정렬 (Insertion Sort)](https://st-lab.tistory.com/179)                 

<br><br><br><br>

## 퀵 정렬 : Quick Sort
분할 정복(divide and conquer) 방법 을 통해 주어진 배열을 정렬한다.                 
*분할 정복 : 문제를 작은 2개의 문제로 분리하고 각각을 해결한 다음, 결과를 모아서 원래의 문제를 해결하는 전략이다.

![image](https://user-images.githubusercontent.com/74857364/228463577-ff581d74-6e6a-46e3-9cf4-4a464cc9fc06.png){: width="50%"}

<br>

하나의 리스트를 피벗(pivot)을 기준으로 두 개의 비균등한 크기로 분할하고 분할된 부분 리스트를 정렬한 다음, 

두 개의 정렬된 부분 리스트를 합하여 전체가 정렬된 리스트가 되게 하는 방법

![image](https://user-images.githubusercontent.com/74857364/228496914-1ab904d0-9339-44c6-b3d2-a058554e0f91.png){: width="50%"}

**분할(Divide)**: 입력 배열을 피벗을 기준으로 비균등하게 2개의 부분 배열              
(피벗을 중심으로 왼쪽: 피벗보다 작은 요소들, 오른쪽: 피벗보다 큰 요소들)로 분할한다.

**정복(Conquer)**: 부분 배열을 정렬한다.               
부분 배열의 크기가 충분히 작지 않으면 순환 호출을 이용하여 다시 분할 정복 방법을 적용한다.

**결합(Combine)**: 정렬된 부분 배열들을 하나의 배열에 합병한다.              
              
순환 호출이 한번 진행될 때마다 최소한 하나의 원소(피벗)는 최종적으로 위치가 정해지므로,               
이 알고리즘은 반드시 끝난다는 것을 보장할 수 있다.              

<br>

퀵 정렬에서 피벗을 기준으로 두 개의 리스트로 나누는 과정

![image](https://user-images.githubusercontent.com/74857364/228497722-d566317b-0045-45b3-a138-c397882fa29f.png){: width="80%"}

<br>

### process
**1.** 배열 가운데서 하나의 원소를 고른다. 이렇게 고른 원소를 피벗(pivot) 이라고 한다.   

<br>

**2.** 피벗 앞에는 피벗보다 값이 작은 모든 원소들이 오고, 

피벗 뒤에는 피벗보다 값이 큰 모든 원소들이 오도록 피벗을 기준으로 배열을 둘로 나눈다.                

이렇게 배열을 둘로 나누는 것을 분할(Divide) 이라고 한다. 분할을 마친 뒤에 피벗은 더 이상 움직이지 않는다.          

<br>

**3.** 분할된 두 개의 작은 배열에 대해 재귀(Recursion)적으로 이 과정을 반복한다.                    

![image](https://github.com/GimunLee/tech-refrigerator/raw/master/Algorithm/resources/quick-sort-001.gif)

<br><br>

### code

왼쪽 pivot 방식으로 작성했다. 그 외의 방법은 아래 첨부된 블로그 링크를 참고한다.          
[자바 [JAVA] - 퀵 정렬 (Quick Sort)](https://st-lab.tistory.com/250)             

<br>

#### 정복(Conquer)

부분 배열을 정렬한다. 

부분 배열의 크기가 충분히 작지 않으면 순환 호출을 이용하여 다시 분할 정복 방법을 적용한다.

- array : 정렬할 배열
- low : 현재 부분 배열의 왼쪽
- high : 현재 부분 배열의 오른쪽
   
```java
public void quickSort(int[] array, int low, int high) {
  // low가 high보다 크거나 같다면 정렬할 원소가 1개 이하이므로 정렬하지 않고 return한다.
    if(low >= high) return;
    
    
    // 정렬할 범위가 2개 이상의 데이터이면(list의 크기가 0 or 1이 아니면)
    // +)
    
    // 분할 
    int pivot = partition(array, low, high); 
    
    // 피벗은 제외한 2개의 부분 배열을 대상으로 순환 호출
    quickSort(array, low, pivot-1);  // 정복(Conquer) 1.
    quickSort(array, pivot+1, high); // 정복(Conquer) 2. 
}
```
**+)**   

피벗을 기준으로 요소들이 왼쪽과 오른쪽으로 약하게 정렬 된 상태로 만들어준 뒤

최종적으로 pivot의 위치를 얻는다.

그리고 나서 해당 피벗을 기준으로 왼쪽부분 리스트와 오른쪽 부분 리스트로 나누어 분할 정복을 해준다.

<br>

**1.** (low ~ 피벗 바로 앞) 앞쪽 부분 리스트 정렬

**2.** (피벗 바로 뒤 ~ high) 뒤쪽 부분 리스트 정렬

<br>

#### 분할

입력 배열을 피벗을 기준으로 비균등하게 2개의 부분 배열 

(피벗을 중심으로 왼쪽 : 피벗보다 작은 요소들, 오른쪽 : 피벗보다 큰 요소들) 로 분할한다.

<br>

'피벗'을 하나 설정하고 피벗보다 작은 값들은 왼쪽에, 큰 값들은 오른쪽에 치중하도록 하는 것이다. 

이 과정을 흔히 파티셔닝(Partitioning)이라고 한다.

- array : 정렬할 배열
- left : 현재 부분 배열의 왼쪽
- right : 현재 부분 배열의 오른쪽
```java
public int partition(int[] array, int left, int right) {
    /**
    // 최악의 경우, 개선 방법
    int mid = (left + right) / 2;
    swap(array, left, mid);
    */
    
    int pivot = array[left]; // 가장 왼쪽값을 피벗으로 설정
    int low = left, high = right;
    
    while(low < high) { // low가 high보다 작을 때 까지만 반복
    
        
        /*  high가 low보다 크면서 high의 요소가 pivot보다 작거나 같은 원소를 찾을 때 까지
         high를 감소시킨다. */
        while(low < high && pivot < array[high]) {
            high--;
        }
        
        /* low가 high보다 크면서 low의 요소가 pivot보다 큰 원소를 찾을 때 까지
        low를 증가시킨다. */ 
        while(low < high && pivot >= array[low]){
            low++;
        }
        
        // 교환 될 두 요소를 찾았으면 두 요소를 바꾼다.
        swap(array, low, high);
    }
    
    /* 마지막으로 맨 처음 pivot으로 설정했던 위치(a[left])의 원소와
    low가 가리키는 원소를 바꾼다. */
    array[left] = array[low];
    array[low] = pivot;
    
    // 두 요소가 교환되었다면 피벗이었던 요소는 low에 위치하므로 low를 반환한다.
    return low;
}
```
<br><br>

```java
public class QuickSort {
    public void quickSort(int[] array, int low, int high) {
        // low가 high보다 크거나 같다면 정렬할 원소가 1개 이하이므로 정렬하지 않고 return한다.
        if(low >= high){
            return;
        }

        // 분할
        int pivot = partition(array, low, high);

        // 피벗은 제외한 2개의 부분 배열을 대상으로 순환 호출
        quickSort(array, low, pivot-1);  // 정복(Conquer) 
        quickSort(array, pivot+1, high); // 정복(Conquer) 
        System.out.println(Arrays.toString(array));
    }

    public int partition(int[] array, int left, int right) {
        int pivot = array[left]; // 가장 왼쪽값을 피벗으로 설정
        int low = left, high = right;

        while(low < high) { // low가 high보다 작을 때 까지만 반복
            
            while(low < high && pivot < array[high]) {
                high--;
            }
            
            while(low < high && pivot >= array[low]){
                low++;
            }
            swap(array, low, high);
        }
        swap(array, left, low);
        return low;
    }

    private void swap(int[] a, int i, int j) {
        int temp = a[i];
        a[i] = a[j];
        a[j] = temp;
    }

    public static void main(String[] args) {
        int[] arr = {5, 3, 8, 4, 9, 1, 6, 2, 7};
        QuickSort quickSort = new QuickSort();
        quickSort.quickSort(arr, 0, arr.length-1);
    }
}
```

<br><br>

*reference*         
[퀵 정렬(Quick Sort)](https://gyoogle.dev/blog/algorithm/Quick%20Sort.html)         
[[알고리즘] 퀵 정렬(quick sort)이란](https://gmlwjd9405.github.io/2018/05/10/algorithm-quick-sort.html)    
[자바 [JAVA] - 퀵 정렬 (Quick Sort)](https://st-lab.tistory.com/250)              
[[알고리즘] 퀵정렬 (Quick Sort) Java Example](https://javaoop.tistory.com/8)        
