---
categories: Analysis
tags: [Analysis]
---

획득을 사용자 획득과 트래픽 획득 두 종류로 구분한다.

사용자 획득은 사람 단위이며 트래픽 획득은 세션 단위다. 

<br><br>

ex) 한 명의 사람이 총 5번 방문했다면 

사용자 획득은 1(명의 사람), 트래픽 획득은 5(번의 방문)가 된다. 

<br><br>

사용자가 웹사이트에 방문하여 세션이 시작되면 session_start 이벤트를 실행, 

해당 사용자가 처음 방문한 신규 사용자라면 first_visit 이벤트도 함께 실행시킨다.

<br><br>

→ 신규 사용자 유입 분석만 집중적으로 필요하다면 사용자 획득 보고서

전체 사용자의 전반적인 유입 분석이 필요하다면 트래픽 획득 보고서를 보면 된다.   

<br>

*참고로 데이터는 임의로 작성해 정리한 것도 있으며 기존 데이터를 활용해 정리한 것도 있다. 

<br><br><br><br>

---

### 1. 웹사이트 사용자 유입 분석 
전체 사용자의 전반적인 유입 분석이 가능한 [트래픽 획득] 탭 클릭 

<img src="https://github.com/user-attachments/assets/8a110c51-13ad-4ce4-8fad-77baae532cad" width="30%">   

<br><br><br><br>          

<세션 기본 채널 그룹▼> 으로 초기 측정기준이 설정되어있다.  

웹사이트에 사용자들의 가장 많은 유입처를 확인하기 위해 사용자 측정항목의 데이터를 확인

<br>                   

 
세션 기본 채널 그룹 |
:---: |
Organic Search |
Direct |
Referral |
Paid Search |
Organic Social |
Paid Social |
Display |


<br><br>  

Organic Search(다양한 검색 엔진에서의 자연 검색)의 사용자가 가장 많다. 

검색광고(Paid Search)와 디스플레이 광고(Display), SNS 플랫폼(Organic Social)도 있다는 것을 알 수 있다.   

<br><br>   

하지만 세션 기본 채널 그룹으로는 어떤 검색 엔진에서의 자연 검색 유입이 많은지 등 자세한 정보를 알기 어렵다. 

측정기준을 <세션 소스/매체▼> 로 변경했다.

<br>

<img src="https://github.com/user-attachments/assets/66e8ff6f-be9a-4196-956c-c09797322ff8" width="50%">    

<br><br><br>   
 
세션 기본 채널 그룹 | 세션 수  
:---: | :---: 
google / organic | 47.6% 
(direct) / (none) | 36.9%
github.com / referral | 11.7% 
google / cpc | 2.2%
FBIG / 1 | 1.3%
naver / organic | 0.3%

<br>

google / organic → 구글에서의 자연 검색이 가장 많다.

google / cpc를 통해 광고 역시 구글 광고의 유입이 가장 많은 것을 확인할 수 있다.

결론 : 해당 웹사이트의 사용자들은 구글에서의 자연 검색(google / organic)으로의 유입이 가장 많다.

<br><br><br>    

<img src="https://github.com/user-attachments/assets/507cbcfa-3632-492b-a544-2f829a62e96a" />

<br>         

github를 통한 유입은 전체 세션 중 11.7%로 적은 편이지만 참여율은 82%로 다른 유입 경로 대비 가장 높다.

google / organic(58%)이나 (direct) / (none)(72%)의 참여율은 github보다 낮다. 

이는 github에서 유입된 사용자가 다른 유입에 비해 관심도가 높다라고 볼 수 있다.  

<br><br><br><br>

----

### 2. 시간대에 따른 유입 분석 
- 전체 사용자에 대한 데이터가 필요 → [ 트래픽 획득 ] 탭 클릭   
- 측정 기준 <세션 소스 / 매체 ▼> 로 변경    
- 가장 상단에 <필터추가+> 클릭    
- 측정 기준
  - 세션 소스/매체
  - 검색 유형 : 알아서 체크('다음과 정확하게 일치'로 체크함)
  - 값 : google / organic (google / cpc 등 분석하고 싶은 부분 체크) 
- 보조 측정기준 <+> 클릭해서 시간 체크

<img src="https://github.com/user-attachments/assets/1e5c4cca-97a8-459a-b090-f7392c1235cf" width="30%">    

<br><br><br>    

<img src="https://github.com/user-attachments/assets/20efa034-425e-409b-9e1f-962fe9cf7af6" width="65%">      

```py
import matplotlib.pyplot as plt

ratio = [10.73, 9.75, 9.75, 9.46, 7.49, 6.21, 5.79, 40.82]
labels = ['17', '14', '15', '16', '11', '13', '18', 'else']

plt.pie(ratio, labels=labels, autopct='%.1f%%')
plt.show()
```
<img width="389" height="389" alt="image" src="https://github.com/user-attachments/assets/10130300-2148-4b1c-8cef-43875a8de285" />

'17', '14', '15', '16' 순으로 나와 오후 시간대에 유입이 많은 것 같았지만 

퍼센트로 확인해보면 시간대 별 큰 차이가 있어보이지는 않는다. 

<br><br><br>   

<img width="1314" height="497" alt="image" src="https://github.com/user-attachments/assets/9aeda717-9020-418f-8226-a40ffa34707a" />

참여 세션이란 세션이 10초 이상 지속되었거나 

주요 이벤트가 1회 이상 발생했거나, 페이지 또는 화면 조회수가 2회 이상인 세션을 의미한다. 

[[GA4] 참여도 개요 보고서](https://support.google.com/analytics/answer/13391283?hl=ko#zippy=%2C%ED%99%9C%EC%84%B1-%EC%82%AC%EC%9A%A9%EC%9E%90%EB%8B%B9-%EC%B0%B8%EC%97%AC-%EC%84%B8%EC%85%98%EC%88%98)

<br><br>

검색해서 들어온 사람들의 참여율은 50% 수준이다.

전체적으로 약 절반가량의 세션에서 사용자가 10초 이내에 이탈하거나 추가 이벤트 없이 종료된 것으로 확인된다.

<br>

전체적으로 이탈률이 높은 것인지 특정 글에서 높은 것인지 추가로 확인해봤다.

방문자 수가 적은(10 이하) 글을 제외하고 가장 높은 수치는 76%의 참여율을 보였다.

참여율이 높지 않은 것으로 보아 대부분의 참여율이 50~60%에 미치는 것으로 보여진다.

<br><br><br><br>

---

REFERENCE

[구글 애널리틱스 4 유입 분석의 첫 단계: 획득 보고서 보기](https://osoma.kr/blog/ga4-acquisition-report/)
