# 무중단 배포

## 1. Profile

### 1.1 profile API

---

무중단 배포를 구축하기 위해서는 두 개의 포트에서 각각 스프링 부트 애플리케이션이 구동되고 있어야 합니다. 현재 서비스 중인 포트를 확인하기 위해 API를 하나 추가하도록 하겠습니다.

<br>

먼저 현재 프로젝트 코드로 돌아와서 controller 패키지에 `ProfileController.java`를 생성합니다.

<br>

```java
package LESSERAFIM.lesserafim.controller;

import org.springframework.core.env.Environment;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Arrays;
import java.util.List;

@RestController
public class ProfileController {
    
    private final Environment env;

    public ProfileController(Environment env) {
        this.env = env;
    }

    @GetMapping("/profile")
    public String profile() {
        List<String> profiles = Arrays.asList(env.getActiveProfiles());
        List<String> realProfiles = Arrays.asList("real1", "real2");
        String defaultProfile = profiles.isEmpty() ? "default" : profiles.get(0);

        return profiles.stream()
                .filter(realProfiles::contains)
                .findAny()
                .orElse(defaultProfile);
    }
}

```

만약 Spring Security를 사용중이라면 `/profile`이 인증 없이 호출될 수 있도록 제외 코드를 추가합니다.

<br>

다음으로 resources 폴더 안에 `application-real1.yml`과 `application-real2.yml` 파일을 생성하고 다음과 같이 작성합니다.

```yml
# application-real1.yml
server:
    port: 8081
```
```yml
# application-real2.yml
server:
    port: 8082
```

<br>

### 1.2 엔진엑스 설정 수정

---

무중단 배포의 핵심은 엔진엑스 설정입니다. 배포 때마다 엔진엑스의 프록시 설정이 순식간에 교체됩니다. 다음 과정은 프록시 설정을 교체하는 방법입니다. 먼저 EC2 서버 컴퓨터로 접속합니다.

<br>

엔진엑스 설정이 모여있는 `/etc/nginx/conf.d`에 `service-url.inc`라는 파일을 하나 생성합니다.

```shell
sudo vi /etc/nginx/conf.d/service-url.inc
```

<br>

그리고 다음 코드를 입력합니다.

```shell
set $service_url http://127.0.0.1:8080;
```
<br>

![alt text](<./image/Screenshot 2024-07-08 at 10.14.54 AM.png>)

<br>

해당 파일을 Nginx가 사용할 수 있도록 설정 파일을 수정합니다.

```shell
sudo vi /etc/nginx/nginx.conf
```

<br>

`location / {}` 코드 블록을 찾아 다음과 같이 추가 및 변경합니다.

```shell
include /etc/nginx/default.d/*.conf;
        include /etc/nginx/conf.d/service-url.inc;

        location / { # location 블록
                include /etc/nginx/proxy_params;
                proxy_pass $service_url;    # reverse proxy의 기능
```

<br>

![alt text](<./image/Screenshot 2024-07-08 at 10.26.53 AM.png>)

<br>

저장하고 종료한 뒤 nginx를 재시작합니다.

```shell
sudo systemctl restart nginx.service
```

<br>

### 1.3 설정 파일 이동

---

이번 배포에서는 전체 코드가 아닌 실행에 필요한 빌드된 jar 파일만 전달할 것입니다. 따라서 실행에 필요한 application.yml 등을 미리 서버 컴퓨터에 세팅해 두어야 합니다.

<br>

먼저 서버 컴퓨터에서 jar 파일과 설정 파일이 위치할 디렉토리를 생성합니다.
```shell
# EC2
mkdir ~/app && mkdir ~/app/config
```

<br>

![alt text](<./image/Screenshot 2024-07-08 at 11.48.15 AM.png>)

<br>

그리고 로컬 컴퓨터에서 필요한 설정 파일들을 모아 압축하겠습니다. 프로젝트 폴더에서 bash shell을 열고 설정 파일이 있는 위치로 이동합니다.

```shell
# Local
cd src/main/resources
```

<br>

`.yml`로 끝나는 파일들을 모아 `config.tgz`라는 파일로 압축합니다.
```shell
# Local
tar -zcvf config.tgz *.yml
```

<br>

그리고 `pwd` 명령어를 입력해서 config.tgz가 위치한 경로를 메모해 둡니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 11.55.50 AM.png>)

<br>

키 파일이 있는 `.ssh` 디렉토리로 이동합니다.
```shell
# Local
cd ~/.ssh
```

<br>

먼저 키파일에 쓰기 권한을 부여합니다.
```shell
# Local
chmod 600 LikeLionServer.pem
```

<br>

해당 위치에서 scp 명령어를 통해 config.tgz 파일을 EC2 서버로 복사하겠습니다. 앞서 메모해 둔 config.tgz의 위치, 본인의 IP 주소 등을 정확하게 작성합니다.
```shell
# scp -i 키파일 본인경로/config.tgz ec2-user@IP 주소:/home/ec2-user/app/config/
scp -i "LikeLionServer.pem" /Users/dongjae/Desktop/LikeLionServer/src/main/resources/config.tgz ec2-user@13.209.167.101:/home/ec2-user/app/config/
```

<br>

![alt text](<./image/Screenshot 2024-07-08 at 12.26.54 PM.png>)

<br>

서버 컴퓨터로 돌아와서 제대로 전송되었는지 확인합니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 12.29.00 PM.png>)

<br>

`~/app/config/` 위치에서 `config.tgz`을 압축해제 합니다.

<br>

```shell
# EC2
tar -xvf config.tgz
```

<br>

![alt text](<./image/Screenshot 2024-07-08 at 12.36.20 PM.png>)

<br>

여기서 `application-prod.yml`에 해당하는 정보들, 예를 들면 데이터베이스 url 등이 원격 서버에 맞게 설정되어있지 않다면 변경해야 합니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 1.34.41 PM.png>)

<br>

## 2. 배포 스크립트 작성

* `appspec.yml`
* `stop.sh`
* `start.sh`
* `health.sh`
* `switch.sh`
* `profile.sh`

### 2.1 appspec.yml

---

AWS CodeDeploy를 통해 EC2로 코드가 배포되면 배포 생명주기에 맞는 특정 동작을 수행하도록 지정할 수 있는데 이 역할을 하는 것이 appspec.yml입니다. 해당 파일을 프로젝트 최상단에 위치하도록 합니다.

```yml
version: 0.0 #CodeDeploy 버전을 이야기합니다. 무조건 0.0으로 고정합니다.
os: linux
files:
  - source: / #destination으로 이동시킬 파일. 여기서는 전체 파일을 의미.
    destination: /home/ec2-user/app/zip/ #source에서 지정된 파일을 받는 위치
    overwrite: yes #기존 파일들을 덮어쓸지 여부

permissions: #CodeDeploy에서 EC2 서버로 넘겨준 파일들을 모두 ec2-user 권한을 갖도록 합니다.
  - object: /
    pattern: "**"
    owner: ec2-user
    group: ec2-user

hooks:
  AfterInstall:
    - location: stop.sh #엔진엑스와 연결되어 있지 않은 스프링 부트를 종료합니다.
      timeout: 60 #스크립트 60초 이상 수행되면 실패합니다.
      runas: ec2-user #stop.sh를 ec2-user 권한으로 실행하게 합니다.
  ApplicationStart:
    - location: start.sh #엔진엑스와 연결되어 있지 않은 Port로 새 버전의 스프링 부트를 시작합니다.
      timeout: 60
      runas: ec2-user
  ValidateService:
    - location: health.sh #새 스프링 부트가 정상적으로 실행됐는지 확인합니다.
      timeout: 60
      runas: ec2-user
```

<br>

![alt text](<./image/Screenshot 2024-07-08 at 9.28.26 AM.png>)

<br>

### 2.2 `profile.sh`

---

다른 배포 스크립트에서 공용으로 사용할 포트 체크 로직을 구현합니다. 먼저 프로젝트 최상단 위치에 `scripts` 디렉토리를 만들고 이곳에 `profile.sh` 파일을 만들어 줍니다. 작성할 때 유의할 점은 본인의 배포 도메인을 적는 부분을 신경써서 적어야 한다는 점입니다.

```shell
#!/usr/bin/env bash

# 쉬고 있는 profile 찾기: real1이 사용 중이면 real2가 쉬고 있고 반대면 real1이 쉬고 있음

function find_idle_profile() {
    RESPONSE_CODE=$(curl -s -o /dev/null -w "%{http_code}" https://cookie-house.store/profile) # 현재 Nginx가 바라보고 있는 스프링 부트가 정상적으로 수행 중인지 확인하고 응답값으로 상태코드를 전달받음

    if [ ${RESPONSE_CODE} -ge 400 ] # 400번대 이상의 오류일 경우 real2를 사용
    then
      CURRENT_PROFILE=real2
    else
      CURRENT_PROFILE=$(curl -s https://cookie-house.store/profile)
    fi

    if [ ${CURRENT_PROFILE} == real1 ]
    then
      IDLE_PROFILE=real2
    else
      IDLE_PROFILE=real1
    fi

    echo "${IDLE_PROFILE}" # bash 스크립트는 반환 기능이 없기 때문에 echo로 값을 출력하고 클라이언트에서 그 값을 잡아서 사용
}

# 쉬고 있는 profile의 port 찾기
function find_idle_port() {
  IDLE_PROFILE=$(find_idle_profile)

  if [ ${IDLE_PROFILE} == real1 ]
  then
    echo "8081"
  else
    echo "8082"
  fi
}
```

<br>

![alt text](<./image/Screenshot 2024-07-08 at 10.57.08 AM.png>)

<br>

### 2.3 `stop.sh`

---

<br>

기존에 Nginx에 연결되어 있지는 않지만, 실행 중이던 스프링 부트를 종료하는 역할을 합니다.

<br>

```shell
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0) # 현재 stop.sh가 속해 있는 경로를 찾습니다.

ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh # 일종의 import 구문으로 stop.sh에서도 profile.sh의 function을 사용할 수 있게 합니다.

IDLE_PORT=$(find_idle_port)

echo "> $IDLE_PORT 에서 구동 중인 애플리케이션 pid 확인"
IDLE_PID=$(lsof -ti tcp:${IDLE_PORT})

if [ -z ${IDLE_PID} ]
then
  echo "> 현재 구동 중인 애플리케이션이 없으므로 종료하지 않습니다."
else
  echo "> kill -9 $IDLE_PID"
  kill -9 ${IDLE_PID}
  sleep 5
fi
```

<br>

### 2.4 `start.sh`

---

배포할 신규 버전의 스프링 부트 프로젝트를 stop.sh에서 종료한 profile로 실행합니다.

```shell
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh

REPOSITORY=/home/ec2-user/app

echo "> Build 파일을 복사합니다."

cp $REPOSITORY/zip/*.jar $REPOSITORY/

echo "> 새 애플리케이션 배포"
JAR_NAME=$(ls -tr $REPOSITORY/*.jar | tail -n 1)

echo "> JAR NAME: $JAR_NAME"

echo "> $JAR_NAME 에 실행 권한을 부여합니다."

chmod +x $JAR_NAME

IDLE_PROFILE=$(find_idle_profile)

echo "> 새 애플리케이션을 $IDLE_PROFILE 로 실행합니다."

# 설정 파일의 위치를 지정하고 active profile을 통해 구동될 포트를 지정합니다. 
nohup java -jar \
-Dspring.config.location=$REPOSITORY/config/application.yml,\
$REPOSITORY/config/application-prod.yml,\
$REPOSITORY/config/application-$IDLE_PROFILE.yml \
-Dspring.profiles.active=$IDLE_PROFILE,prod \
$JAR_NAME > $REPOSITORY/nohup.out 2>&1 &
```

<br>

### 2.5 `health.sh`

---

`start.sh`로 실행한 프로젝트가 정상적으로 실행되고 있는지 체크합니다.

```shell
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source ${ABSDIR}/profile.sh
source ${ABSDIR}/switch.sh

IDLE_PORT=$(find_idle_port)

echo "> Health Check Start!"
echo "> IDLE_PORT: $IDLE_PORT"
echo "> curl -s http://127.0.0.1:$IDLE_PORT/profile"
sleep 10

for RETRY_COUNT in {1..10}
do
  RESPONSE=$(curl -s http://127.0.0.1:${IDLE_PORT}/profile) 
  UP_COUNT=$(echo ${RESPONSE} | grep 'real' | wc -l)

  if [ ${UP_COUNT} -ge 1 ] # Nginx와 연결되지 않은 포트로 스프링 부트가 잘 실행되었는지 체크합니다
  then # $up_count >= 1 ("real" 문자열이 있는지 검증)
    echo "> Health Check 성공"
    switch_proxy # 잘 실행되어 있다면 프록시 설정을 변경합니다
    break
  else
    echo "> Health Check의 응답을 알 수 없거나 혹은 실행 상태가 아닙니다."
    echo "> Health Check: ${RESPONSE}"
  fi

  if [ ${RETRY_COUNT} -eq 10 ]
  then
    echo "> Health check 실패. "
    echo "> 엔진엑스에 연결하지 않고 배포를 종료합니다."
    exit 1
  fi

  echo "> Health check 연결 실패. 재시도..."
  sleep 10
done
```

<br>

### 2.6 `switch.sh`

---

Nginx가 바라보는 스프링 부트 프로젝트를 최신 버전으로 변경합니다.
```shell
#!/usr/bin/env bash

ABSPATH=$(readlink -f $0)
ABSDIR=$(dirname $ABSPATH)
source $ABSDIR/profile.sh

function switch_proxy() {
    IDLE_PORT=$(find_idle_port)

    echo "> 전환할 port: $IDLE_PORT"
    echo "> Port 전환"
    echo "set \$service_url http://127.0.0.1:${IDLE_PORT};" | sudo tee /etc/nginx/conf.d/service-url.inc # 엔진엑스가 변경할 프록시 주소를 생성하여 service-url.inc로 덮어 씁니다.

    echo "> 엔진엑스 Reload"
    sudo systemctl reload nginx
}
```

<br>

추가로 GitHub Actions에서 `gradlew`를 실행할 수 있도록 권한을 변경합니다. 터미널 창을 열고 다음 명령어를 입력합니다.

```shell
chmod a+x gradlew
```

<br>

또한 `build.gradle`에서 다음 코드를 추가하고 코끼리를 눌러 단독 실행 불가한 `plain.jar`가 생성되지 않도록합니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 4.15.39 PM.png>)

<br>

최종 디렉토리 구조는 다음과 같습니다. 변경 사항을 커밋하고 푸쉬합니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 1.30.20 PM.png>)

<br>

## 3. GitHub Actions

### 3.1 지금까지 정리

---

24시간 365일 운영되는 서비스에서 끊김 없는 배포 환경을 구축하기 위해 지금까지 많은 작업을 했습니다. 코드 버전 관리 시스템, 주로 Git에 코드를 Push 하면 자동으로 테스트와 빌드를 수행하여 **안정적인 배포 파일을 만드는 과정을 CI(Continuous Integration)** 이라고 합니다. 그리고 이 빌드된 결과를 자동으로 운영 **서버에 무중단 배포되는 과정을 CD(Continuous Deployment)** 이라고 합니다.

<br>

단, 여기서 주의할 점은 CI/CD 도구를 도입했다고 해서 CI/CD를 하고 있는 것은 아닙니다. 마틴 파울러에 따르면 CI란 **테스팅을 자동화해서 언제든지 건전한 테스트를 수행할 수 있고, 누구나 현재 실행 파일이 가장 완전한 실행 파일임을 확신할 수 있어야 하는 과정** 이라고 했습니다. 따라서 테스트 코드가 없는 프로젝트는 CI를 할 수 없습니다.

<br>

저희가 지금까지 진행한 과정은 CI/CD 도구를 도입했을 뿐 CI/CD를 한 것이라고 볼 수 없습니다. 그럼에도 불구하고 해당 툴을 도입하는 것은 전체 개발 과정에 있어 매우 편리하기 때문에 익혀두어서 나쁠 것은 없습니다. 전체 아키텍처는 다음과 같습니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 2.23.49 PM.png>)

<br>

특히 EC2 인스턴스 내부의 구조는 다음과 같습니다. 먼저 하나의 AWS EC2 안에 Nginx 한 대와 2 대의 Jar을 8081 포트와 8082 포트에 나누어 구동합니다.


1. 사용자가 서비스 URL로 접속합니다.
2. Nginx가 요청을 받고 연결된 8081 포트의 Spring Boot Jar로 요청을 전달합니다.

<br>

![alt text](<./image/nginx1.png>)

<br>

3. 코드 수정 후 다시 빌드된 Jar를 CodeDeploy를 통해 EC2로 전달합니다. CodeDeploy의 appspec.yml 파일은 생명 주기에 맞는 Script 파일을 순서대로 실행합니다.

<br>

![alt text](<./image/codedeploy1.png>)

<br>

4. 가장 먼저 어떤 포트에서 새로운 Jar를 구동할 수 있을 지 찾습니다. 여기서는 8082 포트입니다. (`profile.sh`)
5. 다음으로 8082 포트에서 실행중인 Spring Boot Jar를 종료합니다. (`stop.sh`)
6. 이후 새로운 Jar을 8082포트에서 실행합니다. (`start.sh`)
7. health.sh를 통해 8082에서 Jar가 제대로 구동되는지 확인하고 Nginx의 reverse proxy 포트를 8082로 변경합니다.(`switch.sh`)

<br>

![alt text](<./image/nginx2.png>)

<br>

코드가 수정되고 새로운 Jar 파일이 배포될 때마다 위 과정이 8081 포트와 8082 포트가 번갈아 가며 교체됩니다.

<br>

### 3.2 Workflow 작성

---
가장 먼저 CI에 필요한 AWS 키를 등록하겠습니다. 상단 탭에서 settings를 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 2.41.02 PM.png>)

<br>

좌측에 `Secrets and Variables`에서 `Actions` 항목을 선택합니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 2.42.00 PM.png>)

<br>

`New Repository Secret` 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 2.45.27 PM.png>)

<br>

`AWS_ACCESS_KEY`에 해당하는 이름으로 일전에 메모해 두었던 `s3-codedeploy-user` IAM 사용자 ID를 입력합니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 2.47.36 PM.png>)

<br>

추가하고 동일한 방식으로 `AWS_SECRET_KEY`에 해당하는 이름으로 `s3-codedeploy-user` IAM 사용자 SECRET을 입력합니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 2.52.09 PM.png>)

<br>

이제 스크립트를 작성하겠습니다. 프로젝트 레포지토리에서 상단에 Actions 탭을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 2.34.34 PM.png>)

<br>

`Java with Gradle`에 해당하는 항목을 선택합니다. 없다면 검색합니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 2.36.02 PM.png>)

<br>

기존에 있던 코드를 전부 지우고 다음 코드를 입력합니다. 단 모든 부분을 동일하게 작성하면 안되고 본인의 CodeDeploy 애플리케이션 이름, CodeDeploy 배포 그룹 이름, S3 버킷 이름에 맞게 내용을 일부 고쳐가며 작성합니다.

<br>

```yml
name: Java CI/CD with Gradle and AWS CodeDeploy

on:
  push:
    branches:
      - main

permissions:
  contents: read

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Set up JDK 17
      uses: actions/setup-java@v2
      with:
        distribution: 'adopt'
        java-version: '17'

    - name: Build with Gradle
      run: ./gradlew clean build -x test

    - name: Prepare artifacts for deployment
      run: |
        mkdir -p before-deploy
        cp scripts/*.sh before-deploy/
        cp appspec.yml before-deploy/
        cp build/libs/*.jar before-deploy/
        cd before-deploy && zip -r before-deploy *
        cd ../ && mkdir -p deploy
        mv before-deploy/before-deploy.zip deploy/likelion.zip

    - name: Deploy to S3 (GitHub Artifacts)
      uses: actions/upload-artifact@v2
      with:
        name: deploy
        path: deploy

    - name: AWS 자격증명 설정
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_KEY }}
        aws-region: ap-northeast-2

    - name: S3에 배포
      run: aws s3 cp deploy/likelion.zip s3://likelion-build/likelion.zip

    - name: AWS CodeDeploy를 사용한 배포
      run: |
        aws deploy create-deployment \
          --application-name likelion-deploy \
          --deployment-group-name likelion-deploy-group \
          --s3-location bucket=likelion-build,key=likelion.zip,bundleType=zip \
          --region ap-northeast-2
```

<br>

![alt text](<./image/Screenshot 2024-07-08 at 3.07.44 PM.png>)

<br>

변경을 완료했으면 `Commit changes`를 입력하고 커밋합니다. 이제 모든 과정이 끝났습니다. 테스트를 위해 코드를 변경하고 푸쉬합니다.

<br>

GitHub Actions가 구동되는 플로우는 Actions 탭에서 다음과 같이 확인할 수 있습니다.

<br>

![alt text](<./image/Screenshot 2024-07-08 at 4.19.54 PM.png>)

<br>

또한 EC2 서버에서 전달받은 코드의 스크립트가 실행되는 로그를 다음 명령어를 통해 확인할 수 있습니다.

```shell
tail -f /opt/codedeploy-agent/deployment-root/deployment-logs/codedeploy-agent-deployments.log
```
![alt text](<./image/Screenshot 2024-07-08 at 4.22.52 PM.png>)

<br>

![alt text](<./image/Screenshot 2024-07-08 at 4.45.50 PM.png>)
