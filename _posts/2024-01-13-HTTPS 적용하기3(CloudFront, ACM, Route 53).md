---
categories: Project Cloud
tags: [AWS]
---

# HTTPS 적용하기(CloudFront, ACM, Route 53)

가장 먼저 도메인을 구매한다. (가비아)

여기서는 `www.domain.com`이라고 부르겠다.

<br><br>

## ACM
HTTPS 프로토콜을 위한 ACM을 생성한다. 

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/242263dc-dd62-4e8a-9a9a-125a953f0b3f){: width="50%"}   

주의할 점은 만들 때 지역을 "버지니아 북부" 로 변경하고 만들어야 한다.
![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/bac3d56e-bbbd-4407-a0c7-c86d518ab3be)

서울과 같이 다른 지역으로 설정하면 계속 검증 대기중으로만 뜨게 된다.

<br><br>

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/32bc265b-4c45-498e-9515-0988d5bcd190)

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/2ea19a5d-4731-4448-8376-d6962e6ca810){: width="60%"} 

요청을 완료하면 검증 대기중인 인증서가 띄워진다. 

<br><br><br><br><br>


## CloudFront 적용하기

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/d5fb099b-2f4c-41ab-9c5d-726b53e7f558){: width="60%"}  

원본 도메인 탭을 누르면 방금 만든 s3 버킷에 대한 도메인을 자동으로 띄워준다.      

해당 도메인을 클릭하지 말고 `S3 > 속성 > 정적 웹 사이트 호스팅` 을 보면 나오는 주소를 넣는다.

단, `http://` 부분은 빼고 넣는다. 

ex) `domain/s3-website.ap-northeast-2.amazonaws.com`  

<br><br>   

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/ead48711-cc15-4da0-ad21-48ad3f801033){: width="60%"}   

<br><br>

만약 자동으로 띄워주는 도메인을 등록한다면 cloudfront를 실행했을 때 

`This XML file does not appear to have any style information associated with it. The document tree is shown below.` 

라는 에러를 볼 수 있다.

<br><br><br>

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/64e1373f-8b88-4b5e-804b-e212b5e27262){: width="50%"}     

뷰어(Client)가 HTTP로 접근하면 HTTPS로 리다이렉트 하겠다는 설정이다.

<br><br><br>

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/eb261c5f-ee07-4bd4-bb5a-3ee4ad3d778b){: width="70%"}    

대체 도메인 이름(CNAME) - 선택사항에 구매한 도메인을 넣으면 된다. ex) `www.domain.com`     

사용자 정의 SSL 인증서 - 전에 생성한 SSL 인증서를 선택한다.      

<br><br>

*사용자 정의 SSL 인증서가 띄워지려면 ACM 상태가 발급됨으로 표시되어야 한다.                

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/893b0a76-8be3-4c2f-8fee-84e0fb76cfdd){: width="15%"} 

만약 가비아에 레코드 추가를 했는데 발급됨으로 변경되지 않았다면 

1. 보통 바로 적용이 되지 않고 짧으면 15분 길면 몇시간까지 기다려야 발급됨으로 표시된다.
2. ACM에서 지역이 버지니아 북부로 되어있는지 확인한다.

<br><br><br>

### 접속 확인
배포 된 CloudFront의 도메인 이름을 들어가보면 https로 웹 서버에 접속된 것을 확인 할 수 있다.

이제 구매한 도메인으로 접속하게끔 Route 53을 활용해본다.

<br><br>

---

<br><br>

## Route 53   
![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/8394f204-ff10-4e64-8027-0766cee3bffc){: width="70%"}        

<br><br>

구매한 도메인을 작성하고 나머지는 default 후 호스팅 영역 생성 버튼 클릭

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/477a13d3-5ecb-4853-a028-a15c971dbc22){: width="70%"}  

<br><br>

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/61dcc039-2bad-4d86-9065-d76a4e70346f)

레코드는 현재 NS, SOA 2개의 레코드가 존재한다.   

<br>

만약 레코드가 NS, SOA, CNAME 3개의 레코드가 존재한다면 

ACM에서 Route 53 레코드 생성 버튼을 눌러 자동으로 생성된 것이다.     

<br><br>

레코드 생성을 눌러 아래와 같이 작성한다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/09ea2054-4d85-4da2-9970-3a13eacd29d2){: width="70%"}  

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/9c0950d7-a13d-440b-81b1-c7080732a7f6)

- 레코드 이름 : www
- 레코드 유형 : CNAME
- 값 : CloudFront 배포 도메인 이름을 작성하면 된다.

<br><br><br>   

### 가비아 설정
가비아 도메인의 네임서버를 NS를 클릭하면 나오는 값 4개로 바꿔야 한다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/d9f0a9cd-9fe1-4f27-8712-f71013e00b06){: width="40%"}  


가비아 홈페이지 > 이용 중인 서비스 > 도메인 > (내가 구입한 도메인) 관리 로 들어간다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/66033c66-b33f-4a9e-b667-2998fa73151e)

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/6b1e8cc1-e00c-4559-a5f6-778522a3b95d)

위 호스트명에 NS 레코드의 라우팅 4개를 넣어주면 된다. (마지막에 `.`이 안들어가야 한다.)   
