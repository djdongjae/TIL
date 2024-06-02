# AWS RDS 구축 및 Spring Boot 실행하기

### 1. RDS 설정

---

#### 1.1 RDS 인스턴스 생성하기

다음과 같이 검색창에 rds를 입력하고, RDS 대시보드에서 데이터베이스 생성 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.13.13 PM.png>)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.13.45 PM.png>)

<br>

다양한 선택지가 있지만 저희는 익숙한 MySQL을 선택하도록 하겠습니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.17.20 PM.png>)

<br>

기타 항목은 기본값으로 두고, 템플릿 부분에서 프리티어를 선택하여 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.17.58 PM.png>)

<br>

다음으로 설정 부분은 아래 화면과 같이 동일하게 작성합니다. 여기서 마스터 사용자 이름은 `root`로, 마스터 암호는 `root1234`로 지정하겠습니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.20.36 PM.png>)

<br>

인스턴스 구성과 스토리지는 기본값으로 넘어갑니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.23.29 PM.png>)

<br>

다음으로 연결 항목에서 다른 부분은 모두 기본값으로 두고 퍼블릭 엑세스 부분만 '예'로 변경합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.27.34 PM.png>)

<br>

모든 설정을 완료하였으면 데이터베이스 설정 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.30.52 PM.png>)

<br>

생성되는 동안 시간이 걸릴 수 있습니다. 천천히 기다려 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.38.36 PM.png>)

#### 1.2 RDS 운영환경에 맞는 파라미터 설정하기

RDS를 처음 생성하면 몇가지 설정을 필수로 해야 합니다. 우선 다음 3개의 설정을 차례로 진행해 보겠습니다.

* 타임존 설정
* Character Set
* Max Connection

좌측 카테고리에서 파라미터 그룹 탭을 클릭해서 이동합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.41.29 PM.png>)

<br>

파라미터 그룹 생성 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.56.13 PM.png>)

<br>

세부 정보 구성은 다음과 동일하게 입력합니다. DB 엔진은 반드시 이전에 선택한 MySQL 버전과 동일하게 설정해야 합니다. 입력을 완료하면 생성 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 4.06.51 PM.png>)

<br>

방금 생성한 파라미터 그룹을 클릭하고 편집 버튼을 눌러줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 4.09.55 PM.png>)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 4.10.21 PM.png>)

<br>

이제 편집 모드에서 하나씩 설정을 변경해 보겠습니다. 우선 검색 창에 `time_zone`을 입력합니다. 그리고 아래 항목을 `Asia/Seoul`로 입력합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 4.15.52 PM.png>)

<br>

그대로 검색창으로 돌아와서 `char`까지만 입력하고 다음에 해당하는 항목들을 전부 `utf8mb4`로 입력해 줍니다. 이모지 저장을 위함입니다.

* character_set_client
* character_set_connection
* character_set_database
* character_set_filesystem
* character_set_results
* character_set_results
* character_set_server

![alt text](<./image/Screenshot 2024-06-02 at 4.22.02 PM.png>)

<br>

그리고 다시 검색창으로 돌아와서 `colla`까지만 입력하고 다음에 해당하는 항목들을 전부 `utf8mb4_general_ci`로 입력합니다.

* collation_connection
* collation_server

<br>

![alt text](<./image/Screenshot 2024-06-02 at 4.26.48 PM.png>)

<br>

마지막으로 `Max Connection`을 수정합니다. RDS의 `Max Connection`은 프리티어의 사양에 따라 자동으로 설정되기 때문에 더 넉넉하게 150개로 설정해 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 4.29.58 PM.png>)

<br>

설정을 완료하였으면 우측 상단에 변경 사항 저장 버튼을 클릭하여 최종 저장합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 4.31.51 PM.png>)

<br>

이렇게 생성된 파라미터 그룹을 데이터베이스에 적용하겠습니다. 좌측 탭에서 데이터베이스를 클릭하고 생성한 데이터베이스를 선택 후 수정 버튼을 눌러줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 4.34.07 PM.png>)

<br>

그리고 가장 하단의 추가 구성 항목에서 다음과 같이 방금 생성한 파라미터 그룹으로 변경해 줍니다. 이후 계속을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 4.36.18 PM.png>)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 4.37.38 PM.png>)

<br>

즉시 적용을 반영하고 수정사항을 저장합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 4.38.02 PM.png>)

<br>

다음과 같이 수정중 상태가 나타납니다. 조금 기다려 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 4.42.17 PM.png>)

<br>

간혹 변경사항이 제대로 반영되지 않는 경우가 있기 때문에 수정이 완료되면 작업 탭에서 재부팅 버튼을 다시 눌러줍니다. 그리고 다시 기다려줍니다.

<br>

### 2. 데이터베이스 연결

---

#### 2.1 IntelliJ에서 연결하기

다시 생성한 데이터베이스 상세 페이지로 돌아와서 아래 화면의 엔드포인트 부분을 복사하여 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 4.47.25 PM.png>)

<br>

그리고 진행 중인 프로젝트를 인텔리제이로 열어주고 우측 탭에서 Database를 클릭하고 상단의 플러스 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 6.11.27 PM.png>)

<br>

그리고 다음과 같이 `Data Source` -> `MySQL`을 순서대로 선택하여 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 6.13.00 PM.png>)

<br>

`Host` 부분은 방금 복사한 엔드포인트로, `User`와 `Password`는 `root`와 `root1234`를 입력하여 줍니다. 혹시라도 하단에 드라이버가 설치되지 않았다는 메세지가 뜨면 설명대로 설치하여 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 6.16.47 PM.png>)

<br>

연결 테스트를 해보고 성공하면 그대로 `Apply` 버튼과 `Ok` 버튼을 순서대로 눌러줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 6.20.01 PM.png>)

<br>

로딩이 끝나면 다음과 같은 `console` 탭이 하나 생깁니다. 이 콘솔 탭에서 데이터베이스 스키마를 생성해 보겠습니다. 다음 SQL 명령어를 입력하고 순서대로 실행합니다.

```SQL
create schema likelionserver;
use likelionserver;
```

<br>

![alt text](<./image/Screenshot 2024-06-02 at 6.38.26 PM.png>)

<br>

이제 해당 콘솔에서는 각종 데이터베이스 관련 작업을 수행할 수 있습니다.

#### 2.2 EC2 서버에 데이터베이스 정보 입력하고 Spring Boot 실행하기

이제 EC2 서버에 있는 `application-prod.yml` 파일에 데이터베이스 정보를 추가해 보겠습니다. 서버 컴퓨터로 이동해서 `application-prod.yml` 폴더를 열어줍니다.

```shell
vi /home/ec2-user/LikeLionServer/src/main/resources/application-prod.yml
```

이제 URL, username, password를 차례대로 입력하여 줍니다. 형식은 다음과 같습니다. 물론 Host 부분은 본인의 엔드포인트로 변경하여 줍니다.

```yaml
spring:
  config:
    activate:
      on-profile: prod

  datasource:
    url: jdbc:mysql://likelionserverdb.c9ebrdortuwk.ap-northeast-2.rds.amazonaws.com:3306/likelionserver
    username: root
    password: root1234
    driver-class-name: com.mysql.cj.jdbc.Driver
```
<br>

![alt text](<./image/Screenshot 2024-06-02 at 6.53.33 PM.png>)

<br>

작성을 완료하였으면 `:wq!`로 저장하고 나옵니다.

<br>

이제 모든 설정이 끝났으니 Spring Boot를 실행해 보겠습니다. 우선 다음 위치로 이동하여 줍니다.

```shell
cd /home/ec2-user/LikeLionServer
```

그리고 다음 명령어를 통해 프로젝트를 빌드하여 줍니다. 현재는 테스트 코드를 작성하지 않았기 때문에 테스트 없이 빌드 합니다. 다음 명령어를 순서대로 실행하여 줍니다.

```shell
chmod a+x gradlew
./gradlew clean build -x test
```

<br>

![alt text](<./image/Screenshot 2024-06-02 at 7.20.24 PM.png>)

<br>

다음 위치로 이동하면 2개의 jar 파일이 빌드된 것을 확인할 수 있습니다.

```shell
cd /home/ec2-user/LikeLionServer/build/libs
```

![alt text](<./image/Screenshot 2024-06-02 at 7.27.05 PM.png>)

<br>

이제 다음 명령어로 jar 파일을 백그라운드에서 실행합니다. 즉 터미널 창을 닫아도 애플리케이션이 계속 실행됩니다.

```shell
nohup java -jar lesserafim-0.0.1-SNAPSHOT.jar &
```

엔터를 누르면 다음과 같은 메시지가 출력됩니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 7.29.20 PM.png>)

<br>

다음 명령어로 로그를 확인합니다. 둘 중에 하나만 입력하면 됩니다.

```shell
cat nohup.out # 단순 로그 출력
tail -f nohup.out # 로그를 이어서 출력
```

<br>

정상적으로 실행이 된 것을 확인할 수 있습니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 7.32.52 PM.png>)

<br>

애플리케이션의 종료를 원할 때는 Spring이 돌아가고 있는 PID를 다음 명령어를 통해 확인한 다음 프로세스를 종료합니다.

```shell
ps -ef | grep jar
```
![alt text](<./image/Screenshot 2024-06-02 at 8.11.44 PM.png>)

<br>

위 화면에서 PID는 58110입니다. 따라서 애플리케이션을 종료하고 싶다면 다음 명령어를 입력합니다.

```shell
kill -9 58110
```

<br>

#### 3. 배포한 사이트에 접속

배포한 사이트에 접속하는 방법은 두 가지가 있습니다. 우선 EC2 인스턴스 상세페이지로 이동합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 7.49.18 PM.png>)

<br>

위에 `퍼블릭 IPv4 DNS`나 `탄력적 IP 주소` 둘 중 하나를 복사하여 브라우저 검색창에 다음과 같이 입력합니다.

```
http://ec2-3-37-100-167.ap-northeast-2.compute.amazonaws.com:8080
```

또는

```
http://3.37.100.167:8080
```
