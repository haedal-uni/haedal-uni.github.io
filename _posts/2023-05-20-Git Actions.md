---
categories: Study Cloud
tags: [study, summary, Docker, AWS, EC2, CI/CD, Git Actions]
---

스터디 - 배포 과정 정리

# Git Actions

## Create an example workflow
많은 repository가 있지만 직접 설정한다.

<img width="1149" height="420" alt="Image" src="https://github.com/user-attachments/assets/ad71bde2-763b-46fe-a95f-f3e065e18d34" /> 

<br><br>

[GitHub Docs](https://docs.github.com/ko/actions/learn-github-actions/understanding-github-actions#create-an-example-workflow)

yml 파일 자체는 들여쓰기가 중요하다.
```yaml
name: learn-github-actions
run-name: ${{ github.actor }} is learning GitHub Actions
on: [push]
jobs:
  check-bats-version:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '14'
      - run: npm install -g bats
      - run: bats -v
```
`name`이 actions의 이름이다.

`on`은 트리거이다.(특정 이벤트가 발생하면 해당 workflow가 실행) 

<br>

`jobs`의 바로 밑에 설정된 check-bats-version는 job의 이름이다.

<br>

`runs-on` : job을 어떤 운영체제에서 실행시킬 건가? → ubuntu 최신버전

<br>

job에는 여러 `steps`가 있다. steps에는 "-"(대시)당 하나의 step이라고 보면 된다.

`steps` 섹션 내에 있는 각 항목은 작업(Job)의 단계(Step)를 나타낸다. 

`actions/checkout@v3`와 `actions/setup-node@v3`는 각각 하나의 단계로 간주된다.

여러 개의 단계를 정의할 수 있으며, 각 단계는 순차적으로 실행된다. 

즉, 첫 번째 단계가 완료된 후 두 번째 단계가 실행되는 식이다.

따라서 위의 예시에서는 하나의 작업(Job)이 있고, 해당 작업은 두 개의 단계(Step)로 구성되어 있다. 

먼저 actions/checkout@v3를 실행하고, 그 다음에 actions/setup-node@v3를 실행한다.

<br>

마켓플레이스에서 쓰는 것들을 `uses`에 넣는다. uses의 `name`은 생략될 수 있다.

`with`는 uses가 필요한 인자 같은 것들을 전달한다.

`run` : cmd 창에서 쓰는 명령어처럼 action 안에서 실행을 시킨다고 보면 된다.

<br><br><br>

### 예제 적용해보기    
```yml
name : Hello World
on: workflow_dispatch

jobs:
  hello:
    runs-on: ubuntu-latest    
    steps:
      - name : Run hello
        run : echo "Hello World"
```
<img width="293" height="135" alt="Image" src="https://github.com/user-attachments/assets/8e37b6af-fe09-4e8b-940d-f556ea26c629" /> 

name이 actions의 이름이다. 지금은 Hello World로 설정했으니 실행하면 Hello World로 설정 되어있을 것이다.

<br>

on은 트리거이다.(특정 이벤트가 발생하면 해당 workflow가 실행) 

workflow_dispatch로 설정해서 수동으로 직접 돌리는 것으로 설정했다.

<br>

jobs의 바로 밑에 설정된 hello는 job의 이름이다.

<img width="302" height="180" alt="Image" src="https://github.com/user-attachments/assets/8e885987-ff0d-43c1-84a6-b5143b4cfe02" /> 

<br>

`runs-on` : job을 어떤 운영체제에서 실행시킬 건가? → ubuntu 최신버전

echo "hello" : hello를 출력시켜라 라는 의미이다.

<br><br><br>

### 수동으로 run 하기

Actions 탭에서 Hello World를 클릭하고 아래와 같이 Run workflow를 눌러주면 실행되는 것을 볼 수 있다.

<img width="311" height="177" alt="Image" src="https://github.com/user-attachments/assets/96a70e27-9279-46d3-8845-333f50c66a02" />

<br><br>

**Hello World가 출력되고 있다.**

<img width="268" height="119" alt="Image" src="https://github.com/user-attachments/assets/620acc32-9c1f-4a30-a621-5d95f393e98c" /> 

<br><br><br><br>

## Gradle build
[github marketplace](https://github.com/marketplace/actions/gradle-build-action)

위 사이트에서 기본적으로 제공되고 있는 설정 파일에 gradle 최신 버전을 적용했다.

<img width="627" height="263" alt="Image" src="https://github.com/user-attachments/assets/224c7dc5-83d9-4ae4-944a-e1d3b385b248" /> 

<br>

```yml
name: Run Gradle
on: workflow_dispatch
jobs:
  gradle:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-java@v3
      with:
        distribution: temurin
        java-version: 11
        
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew     
        
    - name: Gradle Build Action
      uses: gradle/gradle-build-action@v2.4.2

    - name: Execute Gradle build
      run: ./gradlew build -x test
```
Run Gradle 이라는 이름으로 수동으로 run한다.

uses의 checkout은 repository를 clone한다는 의미이다.

☑️ [이 전](https://haedal-uni.github.io/posts/CICD-%EC%88%98%EB%8F%99/)에 수동으로 build할 때 java를 설치했었고 test에서 오류가 떠서 test를 제외하고 build를 했었다.

<br><br>

```yml
    - name: Gradle Build Action
      uses: gradle/gradle-build-action@v2.4.2

    - name: Execute Gradle build
      run: ./gradlew build -x test
```
- `name: Gradle Build Action`은 GitHub Marketplace에 등록된 `gradle/gradle-build-action@v2.4.2 action`을 사용하고 있다. 

  uses 키워드를 사용하여 해당 액션을 호출하고 실행한다. 이 action은 Gradle 프로젝트 빌드를 수행하는 데 사용된다.

- name: Execute Gradle build는 command 라인에서 직접 ./gradlew build -x test 명령을 실행하고 있다. 

  run 키워드를 사용하여 해당 명령을 실행할 수 있다. 이는 command를 실행하는 방식이며, 필요한 경우 스크립트 또는 명령을 자유롭게 작성할 수 있다.
  
따라서, 첫 번째 단계는 Marketplace에서 가져온 Gradle Build Action을 실행하고, 두 번째 단계는 직접적으로 Gradle 빌드를 실행하는 명령을 실행하는 것이다.


<br><br>

```
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew 
```
gradlew 파일에 대해 실행 권한을 부여했다.

나는 해당 명령어를 입력해야 Actions에서 통과가 된다. 

(운영체제마다 다른 건가? 라는 추측을 해본다. 나와 운영체제가 다른 팀원들은 모두 권한 명령어를 작성하지 않아도 통과가 되었다.)     

<br><br><br><br>

## Dockerfile 작성하기  
```dockerfile
FROM openjdk:11-slim
WORKDIR /app
COPY ./*-SNAPSHOT.jar ./app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar","app.jar"]
```
ec2 public ipv4주소와 같은 정보가 들어가지 않게 주의한다.

<br><br><br><br>

## yml 수정하기

### jar 파일
dockerfile 작성했으니 jar 파일이 있어야한다.

<br><br>

Git Actions의 각 Job은 별도의 가상 환경에서 독립적으로 실행된다. 

따라서 Job 간에는 직접적으로 파일을 공유할 수 없다. 

각 Job은 독립적인 실행 단위로 간주되며, 작업이 완료되면 해당 Job의 환경은 종료되고 제거된다.

<img width="696" height="241" alt="Image" src="https://github.com/user-attachments/assets/7b6e2bc4-0b1a-4a4e-a8e1-d0dac483c3d9" /> 

<br>

그러나 Git Actions에서는 Job 간에 데이터를 전달하고 공유할 수 있는 몇 가지 방법이 있다.

Artifacts를 사용하여 하나의 Job에서 생성된 출력물을 저장하고 다른 Job에서 사용할 수 있다.

GitHub Actions에서는 Artifact를 생성하고 업로드하기 위해 actions/upload-artifact 액션을 사용할 수 있다. 

업로드된 Artifact는 다른 작업에서 actions/download-artifact 액션을 사용하여 다운로드할 수 있다.

Artifact를 사용하면 작업의 결과를 효율적으로 관리하고, 다른 작업과의 데이터 공유와 협업을 용이하게 할 수 있다.

etc) 외부 서비스(예: 클라우드 스토리지, 데이터베이스 등)를 활용하여 데이터를 전달하는 방법

<br><br><br>

### Artifact

[Upload a Build Artifact](https://github.com/marketplace/actions/upload-a-build-artifact)
```yml
- uses: actions/upload-artifact@v3
  with:
    name: my-artifact
    path: path/to/artifact/world.txt
```
기본적으로 제공되는 코드이다. 여기서 path를 jar 파일 위치로 수정하면 된다.

<br>

[Download a Build Artifact](https://github.com/marketplace/actions/download-a-build-artifact)
```yml
steps:
- uses: actions/checkout@v3

- uses: actions/download-artifact@v3
  with:
    name: my-artifact
    path: path/to/artifact
```
기본적으로 제공되는 코드이다. 여기서 path는 root 위치에서 download 하므로 생략했다.

<br><br>

전체 yml 코드
```yml
name: Run Gradle
on: workflow_dispatch
jobs:
  gradle:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-java@v3
      with:
        distribution: temurin
        java-version: 11
        
    - name: Grant execute permission for gradlew
      run: chmod +x gradlew     
        
    - name: Gradle Build Action
      uses: gradle/gradle-build-action@v2.4.2

    - name: Execute Gradle build
      run: ./gradlew build -x test
      
    - uses: actions/upload-artifact@v3
      with:
        name: my-artifact
        path: ./build/libs/*-SNAPSHOT.jar


  docker:
    needs: gradle 
    runs-on: ubuntu-latest
    steps :
        - uses: actions/checkout@v3
        - uses: actions/download-artifact@v3
          with:
            name: my-artifact
            
        - run : ls -ll
```

<img width="547" height="170" alt="Image" src="https://github.com/user-attachments/assets/0d5c0175-acbc-42c7-af66-caa29ff9942c" /> 

job을 2개로 나눠서 gradle의 문제인지 docker의 문제인지 파악할 수 있게 작성했다.

<br><br><br>

### 우선 순위 지정
```
  docker:
    needs: gradle 
```
docker를 실행하려면 gradle의 작업이 끝나야 한다. → 작업의 순서를 정해줘야 한다.


<br><br><br><br>

### GitHub Container Registry
[GitHub Container Registry](https://docs.docker.com/build/ci/github-actions/push-multi-registries/)

<br>

Docker hub 대신 GitHub Container Registry를 사용하여 Docker 이미지를 관리할 수 있다.
   
아래는 공식 사이트에서 기본적으로 제공되는 workflow 일부다.  
```java
      -
        name: Set up QEMU
        uses: docker/setup-qemu-action@v2
      -
        name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2
      -
        name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      -
        name: Login to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.repository_owner }}
          password: ${{ secrets.GITHUB_TOKEN }}
      -
        name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: true
          tags: |
            user/app:latest
            user/app:1.0.0
            ghcr.io/user/app:latest
            ghcr.io/user/app:1.0.0
```
![image](/assets/img/posts/dfa52ff0-d23b-4468-8383-e9fbce55738f.png){: width="70%"}   

<br><br><br>

### 파일 작성하기   

```  
name: Run Gradle
on: workflow_dispatch
jobs:
  gradle:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: actions/setup-java@v3
      with:
        distribution: temurin
        java-version: 11

    - name: Grant execute permission for gradlew
      run: chmod +x gradlew     
        
    - name: Gradle Build Action
      uses: gradle/gradle-build-action@v2.4.2

    - name: Execute Gradle build
      run: ./gradlew build -x test
      
    - uses: actions/upload-artifact@v3
      with:
        name: my-artifact
        path: ./build/libs/*-SNAPSHOT.jar



  docker:
    needs: gradle
    runs-on: ubuntu-latest
      
    steps :
        - uses: actions/checkout@v3
        - uses: actions/download-artifact@v3
          with:
            name: my-artifact

        - 
          name: Set up QEMU
          uses: docker/setup-qemu-action@v2

        -
          name: Set up Docker Buildx
          uses: docker/setup-buildx-action@v2
          
          # GitHub Container Registry에 로그인
        -
          name: Login to GitHub Container Registry
          uses: docker/login-action@v2
          with:
            registry: ghcr.io
            username: `${{ github.repository_owner }}`
            password: `${{ secrets.GITHUB_TOKEN }}`

        # Build 및 GitHub Package 에 이미지 업로드 
        -
          name: Build and push
          uses: docker/build-push-action@v4
          with:
            # 현재 디렉토리 지정
            context: .
            push: true
            tags: ghcr.io/`${{ github.repository }}`:latest
```
![image](/assets/img/posts/1857970f-ecae-4908-9787-42d903ea0bd2.png){: width="70%"}   

기본으로 제공되는 변수 알아보기 → [GitHub Docs](https://docs.github.com/ko/actions/learn-github-actions/variables)

변수를 대문자로 쓸꺼면 "_"를 쓰고 소문자를 쓸꺼면 "." 을 쓴다

<br>

- `uses` : GitHub Marketplace에서 제공되는 액션(Action)을 사용하기 위한 필드

- `with` : `uses` 필드와 함께 사용되는 옵션을 지정하는 데 사용

  - 액션에 필요한 매개변수, 환경 변수, 인증 정보 등을 설정

- `registry` : Container Registry를 지정하는 field

  - Container Registry는 Docker 이미지를 저장하고 관리하는 저장소
   
   `registry: ghcr.io`는 GitHub Container Registry를 사용하겠다는 의미

   이를 통해 Docker 이미지를 GitHub Container Registry에 푸시하거나, 해당 레지스트리에서 이미지를 가져올 수 있다.

   docker/login-action을 통해 레지스트리에 로그인하여 인증 정보를 제공해야 한다.
   

- `path` : build context로 사용되는 경로를 지정
  
  -  path는 context로 대체하여 사용할 수도 있다.
   

- `tags` : 이미지에 대한 식별 가능한 값을 넣을 수 있다.(버전, release명 등)
  
  - `latest`는 이미지의 최신 버전을 가리키는 태그

<br><br><br>

### unexpected status: 403 Forbidden

실행하게 되면 아래와 같은 에러가 뜬다.

`buildx failed with: ERROR: failed to solve: failed to push ghcr.io/hihi/cicdtest:latest: unexpected status: 403 Forbidden`

<br>

아래와 같이 작성해서 권한을 허용시켜준다.
```
    permissions:
      contents: read
      packages: write
```

<br><br><br><br>

## EC2로 배포

### 이미지 확인
```
sudo docker images
```
이미지 확인을 해보면 아무것도 안띄워져 있는 것을 볼 수 있다.

<br><br><br>

### Amazon EC2 인스턴스에 접속
```
sudo ssh -i adme.pem ubuntu@{ipb4 주소}      
```

<br><br><br>

### Docker login
```
sudo docker login -u {github username} ghcr.io
```
password를 입력하라고 뜨는데 github의 password를 입력하면 실패한다.

accessToken을 발급받은 후에 해당 token값을 입력해서 password로 사용하면 Login Succeeded가 뜬다.

<br>

[프로필 settings 클릭] → [왼쪽 하단의 Developer settings 클릭] → [Personal access tokens (classic)에서 Generate new token 클릭 후 token 발급]

<br><br><br>

git actions를 run하면 package가 하나 생긴다.

package에 들어가면 pull을 받는 명령어가 띄워지는데 해당 부분을 copy하면 된다.
```
sudo docker pull {yml에서 설정한 docker}
```

<br>

```
name: Build and push
uses: docker/build-push-action@v4
with:
  context: .
  push: true
  tags: ghcr.io/${{ github.repository }}:latest
```
위와 같이 ghcr.io~로 설정했기 때문에 아래와 같은 형식으로 설정하면 된다.

ex) `sudo docker pull ghcr.io/hello-world/test:latest`           

<br><br>

#### 다른 사람이 만든 이미지를 받아올 수 있을까?
docker 로그인 후 pull 받아오려고 하니 denied가 떴다.

<img width="521" height="49" alt="Image" src="https://github.com/user-attachments/assets/42cfad61-c330-45e6-8e92-de5f373feb82" /> 

Docker image를 private으로 설정하면 다른 사람은 해당 이미지를 pull 받을 수 없다.

private image는 소유자 또는 이미지에 대한 액세스 권한을 가진 사용자만이 pull 및 사용할 수 있다.

따라서 package는 private으로 설정해준다.

<br><br><br>

### 로그인 후 이미지 확인
```
sudo docker images
```

<img width="795" height="67" alt="Image" src="https://github.com/user-attachments/assets/2ca7b777-fd81-4b9e-a652-bb3a1fed71cf" /> 

<br>

```
sudo docker run --name cicd -p {ec2 port}:{container port} ghcr.io/{github username}/{reposiory name}:latest
```
<img width="502" height="208" alt="Image" src="https://github.com/user-attachments/assets/3ce75db7-bfe5-4e52-8afe-24c977dd9e69" /> 

실행 후 `http://{ipv4 주소}:8080/{rest api 주소}`로 입력하면 제대로 띄워져 있는 것을 확인할 수 있다.     
