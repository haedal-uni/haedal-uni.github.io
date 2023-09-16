---
categories: Project
tags: [Java, error]
---

실습용으로 연습용 project를 하나 만들다가 build 과정에서 오류가 떴다.

Spring Boot 3.0부터는 Java 17 이상만 지원하는데 11로 사용하려고 해서 오류가 뜬 것이었다.

여기서는 java 17로 적용하는 과정을 작성했다.

<br>

## Java version 변경하기
### Download
[여기](https://www.oracle.com/java/technologies/downloads/#jdk17-windows)에서 java 17을 다운받는다.

<br><br>

### Setting
![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/791ceef9-118b-4812-bc09-a9a9b02dbaf3)

<br><br>

시스템 변수에 있는 `JAVA_HOME`을 클릭한 후 편집 버튼을 누르고 설치한 jdk 17의 경로를 넣어주면 된다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/48dd1efa-e416-466d-b05c-7a3312a5b917)

<br><br>


#### [기존에 다른 버전의 JDK가 설치되어 있지 않을 경우에만 해당]
Path 환경 변수에 JDK의 bin directory를 등록해주려면 Path의 편집을 누른다.

`JAVA_HOME` 이라는 환경변수의 값을 사용하고자 할 때 `%JAVA_HOME%`과 같이 사용 한다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/30abfd90-7772-4537-abbf-69570dec866c)

<br><br>

### version 확인하기
윈도우 + R 버튼을 눌러 cmd에서 `java -version`를 확인해 제대로 변경되었는지 확인한다.

![image](https://github.com/haedal-uni/haedal-uni.github.io/assets/74857364/91700990-0e54-45df-a297-aa67a48ca7eb)

<br><br><br>

## 환경변수
환경변수는 운영 체제에서 사용되는 변수로, 시스템의 동작에 영향을 미치는 값을 저장하는 데 사용된다.

사용자 변수 : OS내의 사용자 별로 다르게 설정가능한 환경변수

시스템 변수 : 시스템 전체에 모두 적용되는 환경변수

<br><br>

### Path
"PATH"는 시스템 변수 중 하나로, 실행 가능한 프로그램의 경로를 저장하는 데 사용된다. 
 
운영 체제는 PATH 변수에 지정된 디렉토리를 검색하여 프로그램 실행 시 해당 디렉토리에서 프로그램을 찾을 수 있도록 도와준다.         

Java를 실행하기 위해서는 Java 실행 파일의 경로가 PATH에 포함되어야 한다.      


<br><br><br>

*reference*     
[환경변수 설정을 하는 이유](https://s2junn.tistory.com/37)                      
[[Java] 자바 버전 변경하는 방법 ( JDK 8 -> JDK 17)](https://coding-factory.tistory.com/823)                      
[%JAVA_HOME% %가 의미하는것이 몬가요?](https://www.codeit.kr/community/questions/UXVlc3Rpb246NWU5ZDc0NzQ2Y2E4YTU3OGY0NTRmOGNi)                       
