---
categories: Project
tags: [API, study, project, test]
---

# Postman
`FetchType.Lazy`로 인한 오류를 해결하면서 응답받는 값을 체크하기 위해 Postman을 사용했다.

그런데 하나 문제가 있었고 이를 기록했다.

<br><br><br>

## 로그인 문제

체크해야할 부분은 Comment이기 때문에 "/Comment" GET Method를 테스트를 진행했다.

js에서 registryId(게시글 id)값을 주면 registryId에 해당하는 댓글들을 보여준다.

그런데 postman에 `http://localhost:8080/comment?idx=1`을 send하면 html만 떴다.

<br><br>

바로 "로그인" 문제였다.

해당 페이지는 로그인을 해야 해당 페이지로 들어갈 수 있다.

이 문제는 [stackoverflow](https://stackoverflow.com/questions/38783739/how-to-set-a-session-id-in-postman)에 있는 글을 보고 해결할 수 있었다.


<br><br><br>

## 쿠키 설정하기

![image](https://user-images.githubusercontent.com/74857364/206256650-24e9f6ee-6070-47f7-af65-ba5d6c20bb0e.png)

localhost에 쿠키 JSESSIONID 가 기본으로 세팅되어 있어서 value 값만 변경해줬다.

<br><br>

![image](https://user-images.githubusercontent.com/74857364/206257252-b0113907-184a-41a5-909b-5bce0a8b073a.png)

<br>

cookie에 저장된 JSESSIONID의 value를 복사해서 아래에 붙여넣기 했다.

<br>

![image](https://user-images.githubusercontent.com/74857364/206258191-8229ce09-a449-4bbe-b2e1-6c85dea7bd46.png)

<br><br><br>

### 쿠키 추가하기

페이지 내에서 nickname을 확인하기 위해 Session Storage에 넣어 진행했기 때문에 Session Storage에 저장한 nickname도 추가했다.

<br>

![image](https://user-images.githubusercontent.com/74857364/206259179-1bbb78f1-0f1a-4633-be28-b292038f8af1.png)

<br>

추가는 addCookie 눌러준후에 저장하는 방식 그대로 `nickname=haedal blog;`로 설정했다.

<br>

![image](https://user-images.githubusercontent.com/74857364/206261182-2e03b3dd-dce1-4bad-a2c9-a8eed88ceeaa.png)


![image](https://user-images.githubusercontent.com/74857364/206260553-c7585973-c8f6-4517-8852-0ac3241a0ebb.png)

<br><br><br>

해당 부분을 실행할 때는 로직을 수정할 때마다 프로그램 실행해서 

브라우저 켜고 로그인 후에 Cookies에서 JSESSIONID의 value를 복사해서 Postman에 설정된 JSESSIONID의 value를 수정했다.

그런데 그냥 Postman에 로그인 API를 실행한 후 위 처럼 로그인을 해야 실행되는 페이지를 테스트 하면 된다.

