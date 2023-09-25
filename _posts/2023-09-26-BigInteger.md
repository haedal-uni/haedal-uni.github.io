---
categories: Coding Test
tags: [java, 코테]
---

## BigInteger
[코테](https://school.programmers.co.kr/learn/courses/30/lessons/181846) 문제를 풀다가 

여기에서 a와 b가 long으로도 변환이 안되는 큰 수로 인해 계산을 어떻게 해야할지 몰랐다.

a : "18446744073709551615",	b : "287346502836570928366",	result : "305793246910280479981"

a와 b를 long으로 변환을 시키면 NumberFormatException 오류가 떴다.

<br><br><br><br>

### BigInteger
[docs.oracle](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/math/BigInteger.html) 를 참고하여 코드를 작성했다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/e41e869f-173c-4eec-bde0-15437acc55f5)

```java
import java.math.*;
class Solution {
    public String solution(String a, String b){
        String answer = "";
        BigInteger aa = new BigInteger(a);
        BigInteger bb = new BigInteger(b);
        answer = String.valueOf(aa.add(bb));
        return answer;
    }
}
```

<br><br>

BigInteger는 long 형을 넘는 더 큰 범위의 정수를 다룰 때 사용하는 클래스로 사칙연산 대신 메소드를 사용해야한다.

참고로 BigInteger은 정수 , BigDecimal은 실수를 다룬다.

<br><br>

*reference*            
[[java] BigInteger,BigDecimal,long 큰 수의 표현-사용법](https://technote-mezza.tistory.com/104)
