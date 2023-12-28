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
## ec2 생성하기

point
- ubuntu 클릭
- 스팟 인스턴스 클릭

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/35fe007f-f9f1-49ed-8447-a3bfd5625138)

<br>

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/831ca840-c925-4825-8523-541b85c9a56a)


<br><br>

#### Spot Instance
인스턴스는 AWS의 여유 리소스 또는 EC2 사용자가 사용하지 않는 리소스를 활용하여 제공된다. 

이러한 인스턴스는 경매 방식으로 가격이 결정되며, 현재의 수요와 공급 상황에 따라 가격이 변동한다.

Spot 인스턴스는 비교적 저렴한 가격으로 사용할 수 있어서 비용을 절감할 수 있다. 

그러나 가격은 변동하기 때문에 인스턴스가 중단될 수 있으며, 중단될 경우에는 사전에 알림을 받고 인스턴스를 종료해야 한다. 

Spot 인스턴스는 특정 시간 동안 일시적으로 사용 가능한 리소스이기 때문에 중요한 데이터나 지속적인 작업에는 적합하지 않을 수 있다.

Spot 인스턴스는 비용 효율적인 방법으로 EC2 인스턴스를 실행하고자 하는 사용자에게 좋은 선택일 수 있다. 

그러나 임시적인 리소스로 제공되므로 인스턴스 중단에 대비하고 적절한 처리 방법을 고려해야한다.

<br><br><br>

## 인스턴스 연결하기
{키 페어 이름}.pem을 저장한 위치로 경로를 이동시킨 후 아래 명령어를 입력한다.  

나는 Linux 전용 폴더에 넣어뒀다.

<br>

**keypair 파일의 접근 권한을 변경**
```bash
$ chmod 400 {키 페어 이름}.pem
```
ubuntu에서는 chmod 전에 sudo를 적어줘야 실행이 되었다. (`sudo chmod 400 {키 페어 이름}.pem`)

chmod는 파일 또는 디렉토리의 권한을 변경하는 명령어이며, 400은 권한 설정을 나타낸다. 

400은 해당 파일을 소유한 사용자에게 읽기 권한만 부여하고, 그 외의 사용자에 대해서는 권한을 제한하는 설정이다.

일반적으로 AWS에서는 EC2 인스턴스에 연결하기 위해 사용되는 keypair 파일을 400으로 설정하여 보안을 강화하는 것이 권장된다.  

<br><br>

**SSH를 사용하여 Amazon EC2 인스턴스에 접속**
```bash
$ ssh -i {키 페어 주소} ubuntu@ipv4주소
```
나는 키페어 주소를 입력하지 않고 키페어 이름만 작성해야 실행이 되었다. ex) `sudo ssh -i chat.pem ubuntu@{ipv4주소}`

`-i {키 페어 주소}`는 접속에 사용할 개인 키 파일의 경로를 지정하는 옵션이다. 

해당 개인 키는 키 페어 생성 시에 다운로드한 .pem 파일이어야 한다.

`ubuntu@IPv4`주소는 SSH로 접속할 EC2 인스턴스의 사용자 이름과 IPv4 주소를 지정하는 부분이다. 

여기서 ubuntu는 EC2 인스턴스에 접속할 사용자 계정이며, IPv4주소는 EC2 인스턴스의 공개 IPv4 주소이다.

위의 명령어를 실행하면 개인 키를 사용하여 SSH 연결이 설정되고, 지정한 사용자 이름으로 EC2 인스턴스에 로그인할 수 있게 된다. 

이후에는 SSH 세션 내에서 해당 인스턴스를 제어하고 명령을 실행할 수 있다.

<br><br>

### SSH란?
SSH(Secure Shell)는 네트워크 상에서 안전하게 원격으로 컴퓨터에 접속하고 통신하기 위한 프로토콜 및 프로그램이다. 

SSH를 사용하면 인터넷을 통해 암호화된 연결을 통해 원격 시스템에 로그인하고 명령을 실행하거나 파일을 전송할 수 있다.

SSH는 기본적으로 22번 포트를 사용한다. Amazon EC2에서도 기본적으로 SSH 접속을 위해 22번 포트를 사용한다.

EC2 인스턴스를 생성할 때 보안 그룹(Security Group)이라는 설정을 지정할 수 있다. 

보안 그룹은 인스턴스에 대한 네트워크 트래픽을 제어하는 가상 방화벽 역할을 한다. 

기본적으로 SSH 접속을 허용하기 위해 보안 그룹 구성에 22번 포트를 개방해야 한다.

따라서, SSH를 사용하여 EC2 인스턴스에 접속할 때는 일반적으로 SSH 클라이언트에서 22번 포트를 목적지 포트로 지정하게 된다. 

이렇게 지정된 포트를 통해 SSH 연결이 수립되어 EC2 인스턴스에 접속할 수 있다.

<br><br>

**만약 22번 포트가 아닌 500같이 다른 번호로 설정했다면?**

`--port` 옵션을 사용하여 목적지 포트를 지정해야 한다.

```bash
ssh -i {키 페어 주소} -p 500 ubuntu@ipv4주소
```
`-p 500`은 SSH 접속 시 목적지 포트를 500으로 지정하는 옵션이다. 

실제로 사용하려는 EC2 인스턴스의 포트 번호에 맞게 -p 옵션을 사용하여 목적지 포트를 지정해주면 된다.        

<br><br><br><br>

## Docker 설치하기
[Docker docs](https://docs.docker.com/engine/install/ubuntu/)            

<br>

### apt 저장소 사용
HTTPS를 통해 리포지토리를 사용할 수 있도록 패키지 인덱스를 업데이트 apt하고 패키지를 설치

```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
```
<br>

Docker의 공식 GPG 키를 추가
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
<br>

repository 설정
```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```


<br><br><br>

### docker engine 설치
package update
```bash
sudo apt-get update
```

<br>

Docker Engine, containerd 및 Docker Compose를 설치(최신 버전)
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

<br>

확인하기
```bash
sudo docker -v
```

<br><br><br><br>

## MySql 설치하기
[Docker hub](https://hub.docker.com/_/mysql)

<br>

MySQL 인스턴스 시작
```bash
$ docker run --name ${some-mysql} -e MYSQL_ROOT_PASSWORD=${my-secret-pw} -d mysql:tag
```
some-mysql는 컨테이너에 할당하려는 이름, my-secret-pw는 MySQL 루트 사용자에 대해 설정할 비밀번호, 

tag는 원하는 MySQL 버전을 지정하는 태그

<br>

→ ex) `docker run -p 3306:3306 --name chat -e MYSQL_ROOT_PASSWORD=1234 -d mysql:latest`

앞의 3306 port는 ec2 port이고, 뒤의 3306은 container port이다.

<br>

ex) `docker run -p 1111:2222 --name hello -e MYSQL_ROOT_PASSWORD=1234 -d mysql:latest`

- `-p 1111:2222`: 호스트의 1111 포트를 컨테이너의 2222 포트와 연결한다. 호스트와 컨테이너 간의 포트 매핑을 설정하는 옵션이다.

- `--name hello`: 실행되는 컨테이너의 이름을 "hello"로 지정한다.

- `-e MYSQL_ROOT_PASSWORD=1234`: 컨테이너 내에서 실행되는 MySQL 서버의 루트 계정의 비밀번호를 "1234"로 설정한다.         
      환경 변수를 설정하는 옵션이다.

- `-d`: 컨테이너를 백그라운드 모드로 실행한다. 컨테이너가 백그라운드에서 실행되면 콘솔에 로그가 출력되지 않는다.
   
- `mysql:latest` : 실행할 컨테이너의 이미지를 지정한다. 여기서는 "mysql" 이미지의 최신 버전을 사용한다.   

<br>

따라서 이 명령어는 호스트의 1111 포트를 MySQL 컨테이너의 2222 포트와 연결하고, "hello"라는 이름의 컨테이너를 생성하며, 

MySQL 서버의 루트 계정 비밀번호를 "1234"로 설정하여 MySQL 컨테이너를 백그라운드 모드로 실행하는 역할을 한다.

<br><br>

1111은 호스트 컴퓨터의 포트 번호를 나타내며, 2222는 컨테이너 내부의 MySQL 서비스가 실제로 실행되는 포트 번호이다. 

-p 옵션은 호스트와 컨테이너 간의 포트 매핑을 설정하는데 사용된다.

즉, 위의 예시에서 1111 포트로 호스트 컴퓨터에 접속하면, 해당 요청은 컨테이너 내부의 2222 포트로 전달되어 MySQL 서비스에 도달하게 된다. 

이를 통해 호스트 컴퓨터에서 MySQL 서버에 접근할 수 있다.

포트 포워딩을 사용하여 컨테이너 내부의 서비스를 호스트 컴퓨터나 외부로 노출시킬 수 있으며, 

이를 통해 컨테이너의 서비스에 접근하거나 외부에서 컨테이너의 서비스에 접속할 수 있게 된다.

<br><br><br>

### Port Forwarding(포트 포워딩)?
포트 포워딩(Port Forwarding)은 네트워크에서 사용되는 용어로, 

컴퓨터의 네트워크 트래픽을 한 포트에서 다른 포트로 전달하는 기술이다. 

이를 통해 외부에서 내부 네트워크의 서비스에 접근할 수 있게 된다.

일반적으로 컴퓨터는 여러 개의 포트를 가지고 있다. 

각 포트는 특정 프로토콜(예: HTTP, SSH, FTP)이나 서비스(예: 웹 서버, 데이터베이스)와 연결되어 있다. 

포트 포워딩을 사용하면 외부에서 컴퓨터의 특정 포트로 요청이 들어오면 

이를 내부 네트워크에서 다른 포트로 전달하여 서비스에 접근할 수 있게 된다.

<br>

예를 들어, 웹 서버가 80번 포트를 사용하고 있고, 로컬 컴퓨터의 8080번 포트를 외부로 열어두었다고 가정해 본다. 

이 때 포트 포워딩을 설정하면 외부에서 8080번 포트로 접속한 요청이 내부 네트워크의 웹 서버로 전달되어 웹 페이지를 볼 수 있게 된다.

포트 포워딩은 주로 컴퓨터나 네트워크 장치에 있는 방화벽 또는 라우터에서 설정되며, 

내부 네트워크의 서비스를 외부로 공개하거나 원격으로 접근할 때 유용하게 사용된다.

<br><br><br>

MySql을 설치했으니 project에 db를 연결해본다.

ec2의 Ipv4 주소를 입력한다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/f4d6c67b-d278-4a6d-ad0a-94a86e6dc73b)


user와 password는 MySql 인스턴스를 만들 때 설정한 값을 넣어주면 된다.

<br><br><br><br>

## edit Configuration

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/384931ce-42c0-49a9-9419-e955dc99644b)

edit Confguration을 눌러서 새로 Spring Boot를 만들고 기존 project에 맞게 설정 후 환경변수를 위와 같이 작성해주면 된다.

☑️ edit Configuration에서 Active profiles에 작성하는게 아니라 Environment variables에 환경변수 넣는 것!

`jdbc:mysql://{EC2공개주소}:{host port}/스키마 이름`

위에서 MySQL을 설치할 때 `docker run -p 3306:3306 --name chat -e MYSQL_ROOT_PASSWORD=1234 -d mysql:latest`로 설치했기 때문에

3306으로 설치했고 chat을 입력한 것이다.

만약에 EC2 public IP 주소가 1.2.3.4이고 host port가 1111로 매핑되어 있다면, SPRING_DATASOURCE_URL은 다음과 같이 설정될 수 있다

`jdbc:mysql://1.2.3.4:1111/스키마이름`

<br><br><br><br>

## 인바운드 규칙 설정
![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/cc5d6e6f-3df2-4c8d-ade4-44d30bfa7bb3)

MySQL을 적용했으니 인바운드 규칙에 MySQL을 넣어준다.

<br><br>

**보안**

보안적으로 모든 IP를 허용해버리면 안되기 때문에 DB는 내 IP만 접근 가능하게 설정해준다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/3a0221a1-c332-432a-94de-1d4cc3942ca5)

<br><br>


스키마(chat) 생성 후 run 을 하면 정상적으로 동작한다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/f53176ab-382f-4118-8ac9-0706ce6f5027)

<br><br>

application.properties에 `spring.jpa.hibernate.ddl-auto=create` 속성을 넣어주면 실행과 동시에 테이블이 생성되는 것을 볼 수 있다.

<br><br><br>

*reference*         
[[AWS, Docker] aws ec2 instance에 docker, mysql 설치하기](https://velog.io/@coastby/AWS-Docker-aws-ec2-instance%EC%97%90-docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0)                

