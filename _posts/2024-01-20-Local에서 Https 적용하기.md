---
categories: Project Chat
tags: [Chat, Network]
---

## 채팅 HTTPS 적용하기  
server와 client 간의 websocket 연결은 `HTTP 프로토콜`을 통해 이루어진다.  

***wss*** 란 https 처럼 ws 프로토콜에 데이터 보안을 위해 SSL을 적용한 프로토콜이다.

port는 옵션이지만 ws스키마는 기본적으로 80포트를, wss스키마는 443을 사용한다.  
   
<br><br><br>

# SSL 적용하기
자바는 두 가지의 인증서 형식을 지원한다

- PKCS12 (Public Key Cryptographic Standard #12) : 여러 인증서와 키를 포함할 수 있으며, 암호로 보호된 형식          
    (업계에서 널리 사용됨)

- JKS (Java KeyStore) : PKCS12와 유사하다. 독점 형식이며, Java 환경으로 제한된다.

<br><br>

빙법은 2가지가 있는데 나는 mkcert 프로그램을 이용했다.

<br><br><br>

## 1. PKCS12 
Spring Boot App에 SSL/HTTPS를 적용하기 위해서는 아래와 같은 절차를 거쳐야 한다.

1. SSL 인증서 얻기
2. application.yml 수정하기


실제로 배포하는 app은 정식 인증을 받은 SSL 인증서를 사용하지만 여기서는 Self Signed SSL 인증서를 사용한다.

<br><br>

PKCS12를 생성하는 명령어는 java의 keytool을 이용하는 것이므로, OS의 제약을 받지 않는다.

윈도우와 Linux모두 아래 명렁어를 사용해 키스토어를 생성할 수 있다.

<br><br>

### PKCS12 생성하기
- alias : 파일 별칭으로 자신이 원하는 이름으로 하면된다.
- keyalg : 키 알고리즘을 RSA로 설정한다.
- keystore : 개인키 파일

<br>

```bash
$ keytool -genkey -alias bns-ssl -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore keystore.p12 -validity 3650
```

- `alias bns-ssl`
    - key alias를 bns-ssl로 지정
- `keystore keystore.p12`
    - key store 이름을 keystore.p12로 지정

<br><br>

아래와 같이 원하는 값을 입력하면 된다.

<img width="1530" height="735" alt="Image" src="/assets/img/posts/4304bdf1-5c50-4884-a13a-87aa3e46b712.png" />

<br><br>

생성이 끝났다면 다음과 같이 파일이 프로젝트 디렉토리 안에서 생성되어야만 한다.

<img width="1530" height="365" alt="Image" src="/assets/img/posts/571d1b48-fb46-4464-9d25-d52b02b7aa08.png" /> 

<br><br>

이렇게 생성된 keystore을 프로젝트에 적용한다.

프로젝트의 application.yml 혹은 application.properties를 열어 다음을 입력한다.

```yml
spring:
  profiles: local # 로컬환경에서만 사용하기

# 위의 local 안적고 아래만 작성해도 됨  

server:
  ssl:
    enabled: true
    key-store: keystore.p12
    key-store-password: 123456
    key-store-type: PKCS12
    key-alias: bns-ssl
  port: 8443

# - - - - - - .properties - - - - - - - - 

server.ssl.enabled=true
server.ssl.key-store = keystore.p12
server.ssl.key-store-password=123456
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=bns-ssl
server.port=8443

```
프로젝트 디렉토리 내부에 있기 때문에 keystore의 절대 경로를 지정하지 않고 이름만 등록해도 된다.

설정을 완료 했다면 프로젝트를 실행해서 실제로 Https가 잘 적용되었는지 확인해본다.

<br><br>

브라우저에서 아래 주소를 입력한다.

```less
https://localhost:8443
```

정식 인증을 받은 SSL 인증서를 사용하지 않고 Self Signed SSL 인증서를 사용했기 때문에 연결이 차단된다. 

나중에 정식 인증을 받은 SSL 인증서를 사용하면 이 문제가 해결된다.

<br><br>

<img width="1530" height="881" alt="Image" src="/assets/img/posts/9b1e603f-7989-4edc-b71c-e56135c7b487.png" /> 


<img width="1530" height="1168" alt="Image" src="/assets/img/posts/4ca06f8a-ad00-4ca1-b983-b10f610f63a1.png" />  

localhost(안전하지 않음)으로 이동을 클릭하면 HTTPS가 적용된 사이트를 볼 수 있다.  

<br>

or 설정 → 개인정보 및 보안 → 인증서관리에 들어가서 

자신이 등록한 self SSL 인증서를 항상 신뢰함으로 체크해주면 된다.

<br><br>

### 오류
Failed to start bean 'webServerStartStop'; nested exception is org.springframework.boot.web.server.WebServerException: Unable to start embedded Tomcat server

Integrity check failed: java.security.NoSuchAlgorithmException: Algorithm HmacPBESHA256 not available

자꾸만 이 오류가 떴다.

<br>

그러다가 [[Spring Boot] embedded tomcat SSL 설정](https://sarc.io/index.php/java/1859-spring-boot-embedded-tomcat-ssl) 해당 블로그를 보면서 

spring boot 버전에 따라 2.x면 몇개를 더 작성해야하는 것을 보고 혹시 버전 차이로 인한 것인지 체크해보기 위해

spring boot를 3.x로 업그레이드 하면서 일부를 수정을 하고 실행을 해보니 해당 오류는 나타나지 않고 실행이 되었다. 

(2.x는 설정하는게 몇 개 부족했던 것 같다.)

<br>

관련 글 : [Spring boot 3.x적용](https://haedal-uni.github.io/posts/Spring-Boot-3.x%EC%A0%81%EC%9A%A9/)

<br><br><br><br>

## 2. mkcert 
mkcert 라는 프로그램을 이용하여 로컬환경에서 신뢰할 수 있는 인증서 만들 수 있다.(PKCS12형식만 지원)

<br>

### 윈도우 사용자 설치 (Ubuntu)  
[공식문서](https://github.com/FiloSottile/mkcert) 

Windows에서는 패키지 매니저인 [chocolately](https://chocolatey.org/)를 사용하여 `mkcert`를 설치할 수 있다.

<br><br><br>   

### choco 설치
powershell을 관리자 권한으로 실행한다. 

choco 설치 [공식문서](https://chocolatey.org/install)

<img width="1530" height="238" alt="Image" src="/assets/img/posts/6665958d-cf24-4da1-a7e0-214e3a884f69.png" /> 

```java
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

<br><br>

설치가 완료되면 설치가 제대로 되었는지 체크해본다.   

<img width="1530" height="161" alt="Image" src="/assets/img/posts/b47752a5-68f3-4788-9467-32cbbee29bd0.png" />

<br><br>

### 인증서 생성
로컬을 인증된 발급기관으로 추가한다.

```bash
$ mkcert -install
```
<br>

CA 생성 및 설치 이후 PKCS12 형식 인증서를 생성한다.

```bash
# 해당 커맨드가 입력된 위치에 localhost.p12라는 파일이 생성됨

$ mkcert -pkcs12 localhost
```
<img width="1530" height="606" alt="Image" src="/assets/img/posts/a6dc3e0c-25af-42c5-a6e1-44165544a946.png" /> 

<br><br>

**1.** application.properties 에서 관련 설정을 추가한다. (application.yml)

**2.** 생성된 인증서 (우분투 파일)를 애플리케이션 resource 폴더로 이동시킨다.

```yml   
# application.yml 일 경우
server:
  ssl:
    key-store: classpath:localhost.p12 # 인증서 경로 작성
    key-store-type: PKCS12 # 인증서 형식 작성
    key-store-password: changeit # 인증서 비밀번호를 작성 changeit은 설정하지 않았을 때의 기본값

# application.properties 일 경우
server.ssl.key-store=classpath:localhost.p12
server.ssl.key-store-type=PKCS12
server.ssl.key-store-password=changeit
```
<br>

실행시키면 아래와 같이 log가 찍힌다.   

<img width="1530" height="124" alt="Image" src="/assets/img/posts/a3c5392a-6622-4950-813f-d6c4094f9745.png" />

<br><br>   

실행 페이지

<img width="1530" height="800" alt="Image" src="/assets/img/posts/911a192a-b476-4981-a2e7-5da9ed798f14.png" />  

<br><br><br><br>

**REFERENCE**
- 1. PKCS12
  - [로컬 Spring Boot에 SSL 적용하기](https://jojoldu.tistory.com/350)
  - [[Spring Security] Spring Boot를 이용한 SSL/HTTPS 적용하기](https://velog.io/@code12/Spring-Security-SSLHTTPS-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)  
- 2. mkcert
  - [SEB_BE 60일차 - [인증/보안] 기초](https://velog.io/@subimm/SEBBE-60%EC%9D%BC%EC%B0%A8-%EC%9D%B8%EC%A6%9D%EB%B3%B4%EC%95%88-%EA%B8%B0%EC%B4%88)
  - [인증서 발급 및 Spring boot에서 HTTPS 서버 구현](https://e-room.tistory.com/135?category=975228)   
