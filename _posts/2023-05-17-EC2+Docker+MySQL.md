---
categories: Study Cloud
tags: [study, summary, Docker, AWS, EC2]
---

관련 글          
- [Ec2+docker+mysql](https://haedal-uni.github.io/posts/EC2+Docker+MySQL/) 👈🏻                                        
- [Cicd 수동](https://haedal-uni.github.io/posts/CICD-%EC%88%98%EB%8F%99/)                                                                   
- [Git actions](https://haedal-uni.github.io/posts/Git-Actions/)                      
- [배포 자동화](https://haedal-uni.github.io/posts/%EB%B0%B0%ED%8F%AC-%EC%9E%90%EB%8F%99%ED%99%94/)                           
- [Nginx](https://haedal-uni.github.io/posts/NGINX/)                                                  

<br>

스터디 - 배포 과정 정리

<br>

# EC2 + Docker + MySQL

## EC2 생성하기
- ubuntu 클릭
  
- 스팟 인스턴스 클릭


<img width="808" height="270" alt="Image" src="https://github.com/user-attachments/assets/ae0337a3-c372-471a-95a6-3c51e98ea069" />

<br>

<img width="465" height="171" alt="Image" src="https://github.com/user-attachments/assets/9fcab3cb-e90e-46cf-8246-3de709a97fb3" />

<br><br>

## Spot Instance

인스턴스는 AWS의 여유 리소스 또는 EC2 사용자가 사용하지 않는 리소스를 활용하여 제공된다.

여기서 인스턴스란 내가 AWS에서 생성한 서버 한 대를 의미한다.  

EC2에서 생성한 가상 서버 1대를 인스턴스라고 부른다.

Spot 인스턴스는 이런 서버 자원 중에서 "남는 자원"을 할인된 가격으로 사용하는 방식이다.

이러한 인스턴스는 경매 방식으로 가격이 결정되며 현재의 수요와 공급 상황에 따라 가격이 변동한다.

그래서 일반 인스턴스보다 저렴하다.

단점은 AWS에서 해당 자원이 다시 필요해지면 내 인스턴스가 중단될 수 있다.

<br>

그래서 테스트 서버, 잠깐 사용하는 서버에는 적합하지만

항상 켜져 있어야 하는 서비스에는 적합하지 않을 수 있다.

<br><br><br><br>

## 인스턴스 연결하기

EC2를 생성했다면 이제 서버에 접속 한다.

접속 방법은 SSH를 사용한다.

<br><br> 

### 키 페어 위치로 이동

EC2 생성 시 `.pem` 파일을 다운로드한다.

이 파일은 서버에 접속하기 위한 개인 키이다.

해당 파일이 없으면 해당 EC2 인스턴스에 접속할 수 없다.

나는 Linux 전용 폴더에 넣어두었다.

<br><br>

### keypair 파일의 접근 권한 변경
`{키 페어 이름}.pem`을 저장한 위치로 경로를 이동시킨 후 아래 명령어를 입력한다.

```bash
chmod 400 {키 페어 이름}.pem
```

<br>

Ubuntu에서는 sudo를 붙여야 실행되었다.

```bash
sudo chmod 400 {키 페어 이름}.pem
```

chmod는 파일 또는 디렉토리의 권한을 변경하는 명령어다.

400은 권한 설정을 의미한다.

- 파일 소유자 → 읽기만 가능

- 그 외 사용자 → 아무 권한 없음

<br>

AWS는 보안상 pem 파일이 다른 사용자에게 노출되지 않도록 요구하기 때문에

EC2 접속용 키 파일은 400으로 설정하는 것이 권장된다.

<br><br><br>  

### SSH를 사용하여 EC2 인스턴스에 접속

```bash
ssh -i {키 페어 주소} ubuntu@ipv4주소
```

<br>

나는 키페어 주소를 경로까지 적지 않고 키페어 이름만 작성해야 실행되었다.

```bash
sudo ssh -i test.pem ubuntu@{ipv4주소}
```

- `ssh` : 원격 접속 명령어
- `-i` : 사용할 개인 키 파일 지정
- `ubuntu` : EC2에 접속할 사용자 계정
- `ipv4주소` : EC2의 공개 IP 주소

이 명령어를 실행하면 내 컴퓨터에서 EC2 서버로 접속하게 된다.

<br><br><br> 

### SSH란?

SSH는 Secure Shell의 약자다.

네트워크 상에서 원격으로 컴퓨터에 안전하게 접속하기 위한 프로토콜이다.

여기서 프로토콜이란 컴퓨터끼리 통신할 때의 규칙이라고 이해하면 된다.

SSH는 기본적으로 22번 포트를 사용한다. (프로그램들을 구분하기 위해 사용하는 번호)

- 22 → SSH
- 3306 → MySQL
- 80 → 웹 서버

이런 식으로 프로그램마다 port가 다르다.

EC2에서 SSH 접속을 하려면 보안 그룹에서 22번 포트를 열어두어야 한다.

<br><br><br> 

#### 만약 22번이 아니라 다른 포트를 사용했다면?

```bash
ssh -i {키 페어 주소} -p 500 ubuntu@ipv4주소
```

`-p` 옵션은 접속할 포트를 지정하는 옵션이다.

예를 들어 500번 포트로 SSH를 열었다면 -p 500을 붙여야 접속할 수 있다.

<br><br><br><br>   

## Docker 설치하기
[Docker docs](https://docs.docker.com/engine/install/ubuntu/)            

Docker는 프로그램을 컨테이너라는 독립된 환경에서 실행하는 기술이다.

컨테이너란 하나의 격리된 실행 공간이다.

서버 안에 작은 서버를 하나 더 만드는 개념으로 이해하면 쉽다.

<br><br><br> 

### apt 저장소 사용

HTTPS를 통해 리포지토리를 사용할 수 있도록  

패키지 인덱스를 업데이트하고 필요한 패키지를 설치한다.

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```

<br>

패키지 인덱스 업데이트란 설치 가능한 프로그램 목록을 최신 상태로 갱신하는 것이다.

<br><br><br> 

### Docker 공식 GPG 키 추가

```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
 
Docker가 공식 사이트에서 제공한 파일이 맞는지 검증

<br><br><br> 

### repository 설정

```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Docker 저장소를 Ubuntu에 등록 

<br><br><br> 

### Docker Engine 설치

```bash
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

<br><br>

설치 확인

```bash
sudo docker -v
```

버전이 출력되면 정상 설치된 것이다.

<br><br><br><br>   

## MySQL 설치하기
[Docker hub](https://hub.docker.com/_/mysql)

### MySQL 인스턴스 시작

```bash
docker run --name ${some-mysql} -e MYSQL_ROOT_PASSWORD=${my-secret-pw} -d mysql:tag
```

- some-mysql → 컨테이너 이름

- my-secret-pw → root 비밀번호

- tag → MySQL 버전

<br><br>

```bash
docker run -p 3306:3306 --name chat -e MYSQL_ROOT_PASSWORD=1234 -d mysql:latest
```

-p 옵션에서 앞의 3306은 EC2 포트이고 뒤의 3306은 컨테이너 내부 포트이다.

<br><br>

```bash
docker run -p 1111:2222 --name hello -e MYSQL_ROOT_PASSWORD=1234 -d mysql:latest
```

- `-p 1111:2222` : EC2 1111번 포트, 컨테이너 2222번 포트

외부에서 1111로 접속하면 컨테이너의 2222로 전달된다.

이 과정을 포트 포워딩이라고 한다.

- `--name hello` : 실행되는 컨테이너의 이름 "hello"로 지정

<br><br><br> 

### Port Forwarding이란?

포트 포워딩은 한 포트로 들어온 요청을 다른 포트로 전달하는 기술이다.

외부 → EC2 1111  

→ 컨테이너 2222  

→ MySQL 실행

이 과정을 통해 컨테이너 내부 서비스에 접근할 수 있다.

<br><br><br> 

## Spring Boot와 DB 연결

MySQL이 EC2에서 실행 중이므로 프로젝트에서 해당 DB에 연결한다.

<img width="987" height="835" alt="Image" src="https://github.com/user-attachments/assets/660d280f-2b03-4ae4-8dd8-6e570286bc72" /> 

user와 password는 MySQL 실행 시 설정한 값

<br><br><br> 

### Edit Configuration
<img width="901" height="326" alt="Image" src="https://github.com/user-attachments/assets/b8e2232f-88cf-4fcd-9fe7-f462b4f9d155" /> 

Edit Configuration에서 Spring Boot 설정을 수정한다.

`jdbc:mysql://{EC2공개주소}:{host port}/스키마 이름`

ex) `jdbc:mysql://1.2.3.4:1111/chat`

Active Profiles에 쓰는 것이 아니라 Environment Variables에 작성해야 한다.

<br><br><br> 

### 인바운드 규칙 설정
<img width="1823" height="567" alt="Image" src="https://github.com/user-attachments/assets/170035bc-abff-4e6c-9263-de73b93b72e4" />

MySQL 포트를 열어줘야 한다.

보안 그룹에서 해당 포트를 인바운드 규칙으로 추가한다.

DB는 반드시 내 IP만 접근 가능하도록 설정한다. (보안상 모든 IP 허용X)

<img width="171" height="210" alt="Image" src="https://github.com/user-attachments/assets/5056a4c0-6f3d-478f-837a-f7822a6a7395" />

<br><br><br>

### 스키마 생성 후 실행

chat 스키마 생성 후 run 하면 정상 동작한다.

<img width="1080" height="304" alt="Image" src="https://github.com/user-attachments/assets/7cce6083-5940-447e-a8cf-4691991c6166" /> 

application.properties에 `spring.jpa.hibernate.ddl-auto=create`를 추가하면 실행과 동시에 테이블이 생성된다.

<br><br><br><br>

---

*reference*         
[[AWS, Docker] aws ec2 instance에 docker, mysql 설치하기](https://velog.io/@coastby/AWS-Docker-aws-ec2-instance%EC%97%90-docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0)                

