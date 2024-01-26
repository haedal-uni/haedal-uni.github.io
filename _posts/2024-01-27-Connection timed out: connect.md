---
categories: Error  
tags: [Cloud, 배포, error]
---

# Connection timed out: connect. 

DB를 연결하려고 보니 Connection timed out: connect. 에러가 떴다.

[이 글](https://realsalmon.tistory.com/3)을 보고 EC2의 보안그룹을 체크해봤다.      

<br>

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/e72013c6-33af-4613-b940-1052e2a7150f)

<br>

유형 : MYSQL, 포트 범위 : 330으로 설정은 했으나 소스 부분에서 팀원이 "내 IP"만 허용해둬서 나는 접근을 할 수가 없었다.

그래서 0.0.0.0/0으로 설정을 해뒀더니 해결되었다. (모든 IP에서 접속 허용)      
