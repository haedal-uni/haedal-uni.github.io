---
categories: Project Overwatch
tags: [Cloudflare, Docker]
---

# Cloudflare Tunnel로 https 적용하기

기존에 Linux 서버에서 Docker로 배포를 하고 있었다.

여기에 https를 적용하기 위해 가비아에서 도메인을 사고 Cloudflare Tunnel을 활용했다.

<br><br><br><br>

## 1. 왜 https가 필요했나

http 주소(`http://서버 ip주소:포트번호`)로 서비스하던 동안 브라우저가 막는 기능이 있었다.

- 결과 카드를 휴대폰 공유창으로 보내는 "공유하기"
- 음성 입력
- 홈 화면에 앱처럼 추가(PWA)

셋 다 브라우저가 https에서만 허용하는 기능이라서 https가 없으면 동작하지 않는다.

<br><br><br><br>

## 2. 왜 Caddy가 아니라 Cloudflare Tunnel인가

https를 붙이는 방법으로 Caddy와 Cloudflare Tunnel을 비교했다.

Caddy는 인증서를 자동으로 받아 주는 웹 서버라서 https를 붙이기에 편하다.

그런데 인증서를 받으려면 "이 도메인이 내 것"이라는 걸 증명해야 하고, 그 방법이 서버의 80번이나 443번 포트를 쓰는 것이다.

내가 쓰는 서버는 여러 사람이 같이 쓰는 공유 서버다.

<br>

그러면 포트 대신 DNS로 증명하는 방법이 있는데, Caddy에는 가비아용 기능이 없었다.

그래서 Cloudflare Tunnel로 정했다.

<br>

| | Cloudflare Tunnel | Caddy |
|---|---|---|
| 돈 | 무료 | 무료 |
| 어디서 돌아가나 | 내 서버의 작은 프로그램이 Cloudflare로 연결을 건다. 사용자는 Cloudflare를 거쳐 들어온다 | 내 서버가 직접 사용자를 받는다 |
| 서버 포트 | 새로 열 포트가 없다 | 인증서를 받으려면 80 또는 443번이 필요하다 |
| 공유 서버 | 문제없다 | 80·443은 보통 다른 사람이 쓰고 있어서 안 된다 |
| 가비아 도메인 | 네임서버만 Cloudflare로 바꾸면 된다 | 80·443을 못 쓰면 DNS 방식으로 인증서를 받아야 하는데, Caddy에 가비아용 기능이 없다 |

<br>

"도메인을 서버 IP에 연결하고 쓰던 포트 번호로 접속하면 되는 거 아니야?"라는 생각도 했다.

그렇게 하면 `http://도메인:포트번호`까지는 된다.

하지만 이건 여전히 http라서 위에서 말한 기능들이 안 된다.

https에는 인증서가 필요한데, 쓰던 포트 번호만으로는 인증서를 받을 수 없다.

<br>

Cloudflare Tunnel은 이 문제를 이렇게 푼다.

- 인증서는 Cloudflare가 대신 가지고 있는다.
- 내 서버에서는 작은 프로그램(`cloudflared`)이 Cloudflare로 먼저 연결을 건다.
- 사용자가 도메인으로 들어오면 Cloudflare가 이 연결을 통해 내 서버의 쓰던 포트로 이어 준다.

그래서 새로 열 포트가 없고, 기존 서비스도 그대로 둔 채 https만 붙는다.

<br><br><br><br>

## 3. 준비

- Linux 서버에서 Docker로 돌고 있는 서비스
- 가비아에서 산 도메인
- Cloudflare 계정(무료)

돈이 드는 건 도메인값(1년에 1~2만 원 안팎)뿐이다.

Cloudflare 계정, 터널, 인증서는 모두 무료다.

<br><br><br><br>

## 4. 가비아 도메인을 Cloudflare에 연결

먼저 가비아에서 도메인을 산다.

구매 화면의 네임서버는 "가비아 네임서버 사용" 그대로 둔다.

Cloudflare 네임서버는 아직 모르기 때문에 아래에서 나중에 바꾼다.

<br>

Cloudflare에 로그인한 뒤 대시보드에서 `Add a domain`을 누르고 산 도메인을 입력한다.

요금제는 `Free`를 고른다.

<br>

이때 화면 아래에서 `Add record`가 아니라 `Continue` 버튼을 누르면 된다.

<img src="https://github.com/user-attachments/assets/ac7ca6a1-5bb2-46ab-b033-65aa809f8111" width="70%"/>

<br><br>

그러면 Cloudflare가 네임서버 두 개를 보여 준다.

(ex. `xx.ns.cloudflare.com`)

Cloudflare nameservers 부분의 이 두 개를 복사한다.

<img src="https://github.com/user-attachments/assets/31ea89da-a424-4190-a727-aac2c802651b" width="70%"/>

<br><br>

가비아로 가서 My가비아 → 도메인 관리 → 해당 도메인의 네임서버 설정에 들어간다.

1차, 2차 칸에 복사한 두 개를 붙이고 저장한다.

기존에 있던 네임서버는 제거한다.

<br>

Cloudflare 화면에서 도메인 상태가 `Active`가 되면 끝이다.

보통 몇 분에서 몇 시간, 길면 하루 걸린다.

Cloudflare가 메일로도 알려 준다.

<br><br><br><br>

## 5. 터널 만들기 (Cloudflare 웹 화면)

Cloudflare 대시보드에서 Zero Trust로 들어간다.

처음이면 팀 이름을 정하라고 하는데 아무 이름이나 넣고, 요금제는 Free를 고른다.

이때 결제 정보 입력 창이 나온다.

<br>

Networks → Tunnels → `Create a tunnel`을 누른다.

터널 종류는 `Cloudflared`를 고르고 이름을 정한다.

설치 안내 화면에서는 운영 체제를 `Docker`로 체크한다.

<br>

화면에 나온 명령어 안의 `--token` 뒤에 있는 긴 문자열을 복사해 둔다.

이 토큰은 비밀번호처럼 다뤄야 한다.

<br><br><br><br>

## 6. 서버에서 터널 켜기

배포한 서버 터미널에서 아래 명령어를 한 번 실행한다.

`<token 입력>` 자리에는 5번에서 복사한 문자열을 넣는다.

```bash
docker run -d --name overwatch-tunnel --restart unless-stopped --network host \
  cloudflare/cloudflared:latest tunnel --no-autoupdate run --token <token 입력>
```

<br>

- `--network host` : 이 컨테이너가 서버의 `localhost:포트번호`, 즉 기존에 돌던 Django에 바로 닿는다.

- `--restart unless-stopped` : 서버가 재부팅돼도 자동으로 다시 켜진다.

<br>

기존 컨테이너는 건드리지 않아도 되고 다시 띄울 필요도 없다. 

새 포트를 열지 않아서 다른 사람 서비스와 겹치지도 않는다.

<br>

Cloudflare 터널 화면에서 상태가 `HEALTHY`가 되면 연결된 것이다.

안 되면 서버에서 `docker logs overwatch-tunnel`로 확인한다.

<br><br><br><br>

## 7. 도메인 연결하기 (Route 추가)

방금 만든 터널에 들어가서 `Add a route`를 누르고 아래처럼 작성한다.

화면에 따라 메뉴 이름이 `Public Hostname` 또는 `Published application routes`로 나온다.

<img src="https://github.com/user-attachments/assets/a018adf3-cff1-46ac-b620-21bccaf95e7a" width="60%" />

<br><br> 

- Subdomain : 비워 둔다. 비우면 주소가 구매한 도메인 그대로가 된다. `coach`를 넣으면 `coach.구매한 도메인`이 된다.
- Domain : 구매한 도메인
- Service Type : `HTTP`
- URL : `localhost:docker포트번호`

URL에 쓰는 포트는 Docker로 서비스를 띄울 때 서버 쪽에 열어 둔 포트다.

`docker ps`의 PORTS가 `0.0.0.0:서버포트->8000/tcp` 같은 모양이면, 

서버의 그 포트로 들어온 요청이 컨테이너 안의 Django 8000번으로 간다는 뜻이다.

이 경우 URL에는 앞쪽의 서버 포트를 써서 `localhost:서버포트`로 적는다.

<br><br><br><br>

## 8. Django 설정 바꾸기

여기까지 하면 https로 접속은 되지만, 그대로는 채팅 전송이 막힌다.

Django의 `settings.py`를 아래처럼 바꿔야 한다.

```python
ALLOWED_HOSTS = ['구매한 도메인', '서버 ip주소', 'localhost', '127.0.0.1']

# 없으면 https 주소에서 채팅 전송이 403(CSRF 실패)으로 막힌다
CSRF_TRUSTED_ORIGINS = ['https://구매한 도메인']

# Cloudflare가 https로 받았다는 표시를 믿는다
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
```

<br>

각각 이런 역할을 한다.

- `ALLOWED_HOSTS` : Django가 받아 줄 주소 목록이다. 새 도메인을 넣지 않으면 그 주소로 들어온 요청을 거절한다.

- `CSRF_TRUSTED_ORIGINS` : https 주소에서 보낸 요청을 CSRF 검사에서 믿어 주는 목록이다. 없으면 채팅 전송이 403으로 막힌다.

- `SECURE_PROXY_SSL_HEADER` : 사용자는 https로 들어왔지만 Cloudflare에서 내 서버까지는 http로 온다.

    그래서 Cloudflare가 붙여 주는 "원래 https였다"는 표시를 믿게 한다.

<br>

Subdomain에 `coach`를 넣었다면 `ALLOWED_HOSTS`와 `CSRF_TRUSTED_ORIGINS` 두 곳 모두 `coach.구매한 도메인`으로 써야 한다.

<br>

바꾼 파일을 평소처럼 서버에 복사하고 컨테이너를 다시 시작한다.

<br><br><br><br>

## 9. 확인

1. `https://구매한 도메인` 접속 → 주소창에 자물쇠가 보이는지
2. 채팅을 하나 보내서 답이 오는지 (403이 나면 `CSRF_TRUSTED_ORIGINS` 철자를 확인한다)
3. 휴대폰으로 스탯창을 분석했을 때 결과 카드에 "공유하기" 버튼이 보이는지
4. 다른 경로(`/game/` 등)도 열리는지

<br><br><br><br>

## 10. 알아 둘 것

- 인증서 갱신은 Cloudflare가 알아서 한다. (따로 할 일X)

- 브라우저 localStorage에 쌓이는 기록(내 기록 같은 것)은 주소별로 따로 저장된다.

    예전 `http://서버 ip주소:포트번호`에서 쌓은 기록은 새 https 주소에서 보이지 않는다.

<br>

도메인을 사기 전에 미리 시험해 보고 싶으면 아래 명령어를 쓸 수 있다.

```bash
docker run --rm --network host cloudflare/cloudflared:latest tunnel --url http://localhost:포트번호
```

로그에 찍히는 `https://무작위단어.trycloudflare.com` 주소로 열린다.

켤 때마다 주소가 바뀌는 시험용이다.

채팅 전송까지 해 보려면 그 주소를 8번 settings에 넣어야 한다.

<br>

되돌리고 싶을 때는 서버에서 컨테이너를 지우고, Cloudflare 터널 화면에서 터널을 지우면 된다.

```bash
docker rm -f overwatch-tunnel
```

settings에 추가한 세 줄은 남겨 둬도 기존 http 접속에는 영향이 없다.
