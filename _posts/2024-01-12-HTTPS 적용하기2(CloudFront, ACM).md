---
categories: Project Cloud
tags: [AWS]
---

# HTTPS 적용하기(CloudFront, ACM)

가장 먼저 도메인을 구매한다. (가비아)

여기서는 `www.domain.com`이라고 부르겠다.

<br><br>

## ACM
HTTPS 프로토콜을 위한 ACM을 생성한다. 

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/242263dc-dd62-4e8a-9a9a-125a953f0b3f)

주의할 점은 만들 때 지역을 버지니아 북부로 변경하고 만들어야 한다.
![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/bac3d56e-bbbd-4407-a0c7-c86d518ab3be)

서울과 같이 다른 지역으로 설정하면 계속 검증 대기중으로만 뜨게 된다.

<br><br>

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/32bc265b-4c45-498e-9515-0988d5bcd190)

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/2ea19a5d-4731-4448-8376-d6962e6ca810)

요청을 완료하면 검증 대기중인 인증서가 띄워진다. 

<br>

인증서를 클릭하면 CNAME이름과 CNAME 값이 띄워지는데 

이 값을 가비아 DNS 설정에 가서 레코드 수정 버튼을 누르고 호스트에 CNAME 이름을, 값/위치에는 CNAME 값을 넣으면 된다.

<br>

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/52599eff-d57d-4f0a-8e89-679bc5de1308)

<br>

여기서 CNAME 이름은 가장 처음에 나오는 `.` 부분까지만 작성하고 뒷 부분은 제거 한 후 작성한다. 

ex) `abcdefghijk.www.domain.com.` → `abcdefghijk` 부분만 호스트명에 작성  

<br><br><br><br><br>


## CloudFront 적용하기

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/d5fb099b-2f4c-41ab-9c5d-726b53e7f558)

원본 도메인 탭을 누르면 방금 만든 s3 버킷에 대한 도메인을 자동으로 띄워준다.      

해당 도메인을 클릭하지 말고 `S3 > 속성 > 정적 웹 사이트 호스팅` 을 보면 나오는 주소를 넣는다.

단, `http://` 부분은 빼고 넣는다. 

ex) `domain/s3-website.ap-northeast-2.amazonaws.com`  

<br>

만약 자동으로 띄워주는 도메인을 등록한다면 cloudfront를 실행했을 때 

`This XML file does not appear to have any style information associated with it. The document tree is shown below.` 

라는 에러를 볼 수 있다.

<br><br><br>
![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/64e1373f-8b88-4b5e-804b-e212b5e27262){: width="40%"}    


뷰어(Client)가 HTTP로 접근하면 HTTPS로 리다이렉트 하겠다는 설정이다.

<br><br><br>

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/eb261c5f-ee07-4bd4-bb5a-3ee4ad3d778b){: width="60%"}    

대체 도메인 이름(CNAME) - 선택사항에 구매한 도메인을 넣으면 된다. ex) `www.domain.com`     

사용자 정의 SSL 인증서 - 전에 생성한 SSL 인증서를 선택한다.
