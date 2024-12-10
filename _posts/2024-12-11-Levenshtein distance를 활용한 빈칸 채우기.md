---
categories: Eng-Project
tags: [log]
---

# Levenshtein distance를 활용한 빈칸 채우기  
빈칸 채우기 문제는 예문에서 특정 단어를 빈칸으로 대체해 퀴즈를 구성하는 방식이다.

그러나 문제 단어의 변형이 예문에 포함될 경우 

기존 로직으로는 이를 처리하지 못해 예문이 퀴즈로 인식되지 않고 그대로 출력되는 문제가 발생했다.

<br>

예를 들어 문제 단어가 `run`일 때 예문에 `running`이 포함되어 있으면 기존 로직으로는 이를 처리하지 못했다.  

<br>

![AC_ 20241210-041827](https://github.com/user-attachments/assets/56937a76-7be1-4745-8e89-864fe6d6f01b)

<br><br>

이를 해결하기 위해 Levenshtein Distance를 활용해 단어 간 유사도를 측정하고

가장 유사한 단어를 빈칸으로 대체하는 방식을 적용했다.  

<br><br><br><br>

## Levenshtein Distance(편집 거리 알고리즘) 란?
**Levenshtein Distance**는 두 문자열 간의 최소 편집 거리를 계산해 유사도를 측정하는 알고리즘이다.    

편집 거리는 한 문자열을 다른 문자열로 변환하는 데 필요한 최소 작업 횟수를 의미한다. 

작업은 다음 세 가지로 구성된다. 

- 삽입: 문자 추가

- 삭제: 문자 제거

- 교체: 문자 변경

이 세 가지 작업의 비용을 계산하여 최소값을 도출한다.   

<br><br><br>

### 계산 방법 

- 삽입 : table[`i-1`][`j`] + 1

- 삭제 : table[`i`][`j-1`] + 1 

- 교체 : 

  문자가 같을 경우 : table[`i-1`][`j-1`] + 0

  문자가 다를 경우 : table[`i-1`][`j-1`] + 1

<br>    

`table[i][j]`= min(삽입 비용,삭제 비용,교체 비용)

테이블의 마지막 셀인 `table[length(a)][length(b)]` 값이 두 문자열의 편집 거리를 나타낸다.     


<br><br><br><br>  


### 예제 

**1.** 글자가 서로 동일하면 대각선 값을 가져온다

**2.** 변경이 필요하면 대각선 값에서 + 1을 한다.

**3.** 삽입이 필요하면 위의 값에서 +1을 한다.

**4.** 삭제가 필요하면 왼쪽 값에서 +1을 한다.

**5.** 1~4의 경우에서 최소값을 가져온다.

<br><br> 

#### 1. 편집 거리 계산
hello와 hi를 예시로 들었다. 

↓ 초기 테이블 설정

|  |  | h | i |
| --- | --- | --- | --- |
|  | 0 | 1 | 2 |
| h | 1 |  |  |
| e | 2 |  |  |
| l | 3 |  |  |
| l | 4 |  |  |
| o | 5 |  |  |

<br><br><br>

**계산 과정**

**1.** `table[1][1]` : `h`와 `h`는 서로 같으므로 비용은 0이다.

- 삽입 비용: `table[0][1]` + 1 = 2

- 삭제 비용: `table[1][0]` + 1 = 2

- 교체 비용: `table[0][0]` + 0 = 0

→ 최솟값 : 0

|  |  | **`h`** | i |
| --- | --- | --- | --- |
|  | 0 | 1 | 2 |
| **`h`** | 1 | 0 |  |
| e | 2 |  |  |
| l | 3 |  |  |
| l | 4 |  |  |
| o | 5 |  |  |

<br><br>

**2.** `table[1][2]` : `h`와 `i`는 서로 다르므로 비용 1

- 삽입 비용: `table[0][2]` + 1 = 3

- 삭제 비용: `table[1][1]` + 1 = 1

- 교체 비용: `table[0][1]` + 1 = 2  

→ 최솟값 : 1

|  |  | h | **`i`** |
| --- | --- | --- | --- |
|  | 0 | 1 | 2 |
| **`h`** | 1 | 0 | 1 |
| e | 2 |  |  |
| l | 3 |  |  |
| l | 4 |  |  |
| o | 5 |  |  |

<br><br>

**3.** `table[2][1]` : `e`와 `h`는 서로 다르므로 비용 1

- 교체 비용: `table[1][0]` + 1 = 2

- 삽입 비용: `table[1][1]` + 1 = 1

- 삭제 비용: `table[2][0]` + 1 = 3

→ 최솟값:  1

|  |  | **`h`** | i |
| --- | --- | --- | --- |
|  | 0 | 1 | 2 |
| h | 1 | 0 | 1 |
| **`e`** | 2 | 1 |  |
| l | 3 |  |  |
| l | 4 |  |  |
| o | 5 |  |  |

<br><br>
    
**최종 테이블 결과**

|  |  | h | i |
| --- | --- | --- | --- |
|  | 0 | 1 | 2 |
| h | 1 | 0 | 1 |
| e | 2 | 1 | 1 |
| l | 3 | 2 | 2 |
| l | 4 | 3 | 3 |
| o | 5 | 4 | 4 |

<br>

최종 값은 `table[5][2]` = 4이다. → `hello`와 `hi`의 편집 거리는 4이다. 

<br>

위 전체 결과는 아래 코드를 통해서도 확인할 수 있다.  

```java
public class Main {
    public static void main(String[] args) throws IOException {
        StringBuilder sb = new StringBuilder();
        String a = "hello";
        String b = "hi";
        int[][] table = new int[a.length()+1][b.length()+1];

       // 테이블 초기화 
        for(int i=1; i<=a.length(); i++){
            table[i][0] = i;
        }
        for(int i=1; i<=b.length(); i++){
            table[0][i] = i;
        }

        // 거리 계산 
        for(int i=1; i<=a.length(); i++){
            for(int j=1; j<=b.length(); j++){
                int insert = table[i-1][j]+1;
                int delete = table[i][j-1]+1;
                int replace = (a.charAt(i-1) == b.charAt(j-1) ? 0:1) + table[i-1][j-1];
                table[i][j] = Math.min(insert, Math.min(delete, replace));
                sb.append(table[i][j]).append(" ");
            }
            sb.append("\n");
        }
        System.out.print(sb);
    }
}
```

<br><br><br><br>

## Code 
### 1. 단어 유사도 측정
두 문자열의 편집 거리를 계산한다.  
```js
function getLevenshteinDistance(a, b) {
    // 테이블 초기화
    const table = Array.from({ length: a.length + 1 }, () => Array(b.length + 1).fill(0));

    // 기본 설정 
    for (let i = 1; i <= a.length; i++) {
        table[i][0] = i;
    }
    for (let j = 1; j <= b.length; j++) {
        table[0][j] = j;
    }

    // 편집 거리 계산
    for (let i = 1; i <= a.length; i++) {
        for (let j = 1; j <= b.length; j++) {
            const insert = table[i - 1][j] + 1;
            const del = table[i][j - 1] + 1;
            const replace = (a[i - 1] === b[j - 1] ? 0 : 1) + table[i - 1][j - 1];
            table[i][j] = Math.min(insert, del, replace);
        }
    }
    // 결과 반환
    return table[a.length][b.length];
}
```

<br><br>

### 2. 가장 유사한 단어 찾기 
가장 작은 값을 저장해서 가장 유사한 단어를 return 시킨다.
```js
function similarWord(card, word){
    let arr = card.sentence.split(" ");
    let similar;
    let min = 999;
    let x = 0;
    for(let i=0; i<arr.length; i++){
        x = getLevenshteinDistance(word, arr[i]);
        if(x<min){
            min = x;
            similar = arr[i];
        }
    }
    return similar;
}
```

<br><br>

### 3. 단어 변환
문제 단어가 예문에 포함되어 있다면 그대로 사용하고 아니라면 유사도를 기준으로 변환한다.

```js
if(card.sentence.indexOf(card.word)!==-1){
    s_word = card.word
}else{
    s_word = similarWord(card, card.word);
}
```

<br><br>

코드 적용 후 예문에 포함된 단어의 변형도 문제로 출제 되는 것을 확인할 수 있다. 

![AC_ 20241210-042244](https://github.com/user-attachments/assets/fe56ad2e-a594-4207-9caa-f937a564f1d5)

<br><br><br><br>

**REFERENCE**    
Levenshtein distance      
- [[Algorithm] 문장의 유사도 분석 - 편집 거리 알고리즘 (Levenshtein Distance)](https://jino-dev-diary.tistory.com/20)
- [편집거리 알고리즘 Levenshtein Distance(Edit Distance Algorithm)](https://madplay.github.io/post/levenshtein-distance-edit-distance)       
    
Array       
- [Array.from을 통한 배열의 초기화](https://velog.io/@teihong93/Array.from%EC%9D%84-%ED%86%B5%ED%95%9C-%EB%B0%B0%EC%97%B4%EC%9D%98-%EC%B4%88%EA%B8%B0%ED%99%94)                 
