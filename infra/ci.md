# AWS CodeDeploy 설정하기

## 1. AWS IAM

<br>

일반적으로 AWS 서비스에 외부 서비스가 접근하기 위해서는 접근 가능한 권한을 가진 Key를 생성해서 사용해야 합니다. AWS에서는 이러한 인증과 관련된 기능을 제공하는 서비스로 **IAM(Identity and Access Management)** 이 있습니다. 저희는 IAM을 통해 GitHub Actions가 S3와 CodeDeploy에 접근할 수 있도록 할 것입니다.

<br>

먼저 AWS 검색창에 IAM을 입력하고 접속합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 3.45.26 PM.png>)

<br>

좌측에 사용자 항목을 선택합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 3.50.08 PM.png>)

<br>

사용자 생성 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 3.51.19 PM.png>)

<br>

적당한 이름을 부여하고 다음으로 넘어갑니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 4.11.52 PM.png>)

<br>

먼저 우측에 직접 정책 연결 항목을 선택합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 4.14.13 PM.png>)

<br>

그리고 권한 정책 검색창에 `s3full`을 입력하고 선택합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 4.17.38 PM.png>)

<br>

다음으로 `codedeployfull`을 입력하고 선택합니다. 선택을 마쳤으면 다음으로 넘어갑니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 4.21.27 PM.png>)

<br>

검토 사항을 확인하고 추후 식별의 용이를 위해 `Name`에 해당하는 키 값에 적당한 이름을 부여합니다. 설정을 마쳤으면 사용자 생성 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 4.30.39 PM.png>)

<br>

생성이 완료되면 대시보드에서 방금 생성한 사용자를 선택합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 4.33.50 PM.png>)

<br>

우측 상단에 엑세스 키 만들기를 선택합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 4.36.30 PM.png>)

<br>

기타를 선택하고 다음으로 넘어갑니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 4.38.17 PM.png>)

<br>

적당한 설명을 부여하고 액세스 키 만들기를 선택합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 4.40.23 PM.png>)

<br>

액세스 키 아이디와 비밀 액세스 키를 적당한 위치에 메모해 두고 완료 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 4.41.58 PM.png>)

<br>


## 2. AWS S3

<br>

먼저 AWS에 접속하여 S3를 검색합니다.

<br>

![alt text](<./image/Screenshot 2024-07-05 at 4.38.30 PM.png>)

<br>

버킷 만들기를 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-05 at 4.43.30 PM.png>)

<br>

버킷에 적당한 이름을 부여합니다.

<br>

![alt text](<./image/Screenshot 2024-07-05 at 4.46.01 PM.png>)

<br>

별다른 설정없이 넘어가면 되는데 이미지 업로드할 때와는 다르게 퍼블릭 엑세스 차단 항목에서 "모든 퍼블릭 엑세스 차단"에 반드시 체크 표시를 해야 합니다. 버킷에는 Jar 파일이 들어갈 것이기 때문에 코드나 설정값, 주요 키값들이 탈취될 수 있기 때문입니다.

<br>

![alt text](<./image/Screenshot 2024-07-05 at 4.48.14 PM.png>)

<br>

버킷을 생성합니다.

<br>

![alt text](<./image/Screenshot 2024-07-05 at 4.48.33 PM.png>)

<br>

## 3. CodeDeploy

### 3.1 EC2에 CodeDeploy 연동 권한 추가

---

<br>

AWS의 배포 시스템인 Codedeploy를 이용하기 전에 배포 대상인 EC2가 CodeDeploy를 연동 받을 수 있게 IAM 역할을 하나 생성하겠습니다. 앞서 만들었던 IAM 사용자와 역할은 다음과 같은 차이점이 있습니다.

* 사용자
  * AWS 서비스 외에 사용할 수 있는 권한
  * 로컬 PC, GitHub Actions, Spring Boot, IDC 서버 등
* 역할
  * AWS 서비스에만 할당할 수 있는 권한
  * EC2, CodeDeploy, SQS 등

<br>

앞에서와 동일하게 검색창에 IAM을 검색합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 4.57.12 PM.png>)

<br>

이번에는 좌측 배너에서 역할을 선택하고 역할 생성 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 4.57.51 PM.png>)

<br>

![alt text](<./image/Screenshot 2024-07-06 at 5.00.42 PM.png>)

<br>

다음과 같이 신뢰할 수 있는 엔티티 유형은 AWS 서비스를, 사용 사례의 경우 EC2를 선택합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 7.00.59 PM.png>)

<br>

권한 정책 검색창에 `EC2roleforaws`까지 입력하고 `AmazonEC2RoleforAWSCodeDeploy`를 선택하고 다음으로 넘어갑니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 7.04.02 PM.png>)

<br>

다음으로 역할에 적당한 이름을 부여합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 7.14.26 PM.png>)

<br>

아래로 내려가서 적당한 이름의 태그를 부여하고 역할 생성을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-06 at 7.16.24 PM.png>)

<br>

이렇게 만든 역할을 EC2 서비스에 등록하겠습니다. 다시 검색창에 EC2를 입력하고 인스턴스 목록으로 이동합니다.

<br>

![alt text](<./image/Screenshot 2024-07-07 at 8.47.02 AM.png>)

<br>

![alt text](<./image/Screenshot 2024-07-07 at 8.47.44 AM.png>)

<br>

다음과 같이 현재 실행 중인 EC2 인스턴스 아이디 위에서 오른쪽 마우스를 클릭하고 보안 -> IAM 역할 수정을 차례로 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-07 at 8.49.50 AM.png>)

<br>

방금 생성한 역할을 선택하고 업데이트 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-07 at 8.52.52 AM.png>)

<br>

![alt text](<./image/Screenshot 2024-07-07 at 8.53.27 AM.png>)

<br>

역할 선택 및 업데이트를 완료하면 재부팅을 해야지만 역할이 정상적으로 반영되기 때문에 재부팅을 합니다.

<br>

![alt text](<./image/Screenshot 2024-07-07 at 8.58.10 AM.png>)

<br>

### 3.2 EC2에 CodeDeploy Agent 설치

---

<br>

EC2 서버 컴퓨터에 접속해서 EC2에서 CodeDeploy를 사용할 수 있도록 `CodeDeploy 에이전트`를 설치해 보겠습니다. 접속을 마치고 다음 명령어를 입력합니다.

```shell
 aws s3 cp s3://aws-codedeploy-ap-northeast-2/latest/install . --region ap-northeast-2
```

<br>

다운로드가 성공했다면 `download: s3://aws-codedeploy-ap-northeast-2/latest/install to ./install`과 같은 메시지가 콘솔에 출력됩니다.

<br>

다운받은 install 파일에 실행 권한이 없기 때문에 실행 권한을 추가합니다.


```shell
 chmod +x ./install
```

<br>

install 파일로 설치를 진행합니다.

```shell
 sudo ./install auto
```

<br>

중간에 다음과 같은 메시지가 뜬다면 루비라는 언어가 설치 안 된 상태라서 그런 것이기 때문에 루비를 설치합니다.
```shell
 sudo yum install ruby
```

<br>

![alt text](<./image/Screenshot 2024-07-07 at 9.14.18 AM.png>)

<br>

ruby를 설치하고 나면 다시 install 파일로 설치를 진행합니다.

<br>

![alt text](<./image/Screenshot 2024-07-07 at 9.17.34 AM.png>)

<br>

설치가 끝났으면 `Agent`가 정상적으로 실행되고 있는지 상태 검사를 합니다.

```shell
 sudo service codedeploy-agent status
```

<br>

![alt text](<./image/Screenshot 2024-07-07 at 9.19.29 AM.png>)

<br>

### 3.3 CodeDeploy를 위한 권한 생성

---

<br>

다시 AWS로 돌아와서 이번에는 CodeDeploy에서 EC2에 접근하기 위한 권한 즉 IAM 역할을 생성하겠습니다. 먼저 검색창에 이전과 같이 IAM을 입력하고 역할 생성 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-07 at 9.24.15 AM.png>)

<br>

이번에는 사용 사례 선택에서 CodeDeploy를 검색하고 선택합니다. 다음으로 넘어갑니다.

<br>

![alt text](<./image/Screenshot 2024-07-07 at 9.29.59 AM.png>)

<br>

CodeDeploy는 권한이 하나뿐이라 바로 다음으로 넘어갑니다.

<br>

![alt text](<./image/Screenshot 2024-07-07 at 9.31.31 AM.png>)

<br>

이전과 동일하게 적당한 이름과 태그를 입력하고 역할 생성을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-07 at 9.32.56 AM.png>)

<br>

![alt text](<./image/Screenshot 2024-07-07 at 9.33.30 AM.png>)

<br>

### 3.4 CodeDeploy 생성

---

<br>

검색창에 CodeDeploy를 입력하고 선택합니다.

<br>

![alt text](<./image/Screenshot 2024-07-05 at 5.00.14 PM.png>)

<br>

화면 중앙에 있는 애플리케이션 생성 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-05 at 5.01.29 PM.png>)

<br>

적당한 이름을 부여하고 컴퓨팅 플랫폼은 `EC2/온프레미스`를 선택합니다. 그리고 애플리케이션 생성 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-05 at 5.18.24 PM.png>)

<br>

생성이 완료되면 배포 그룹 생성 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-07 at 9.40.20 AM.png>)

<br>

적당한 배포 그룹 이름을 입력해주고 서비스 역할은 방금 생성한 역할을 선택하여줍니다.

<br>

![alt text](<./image/Screenshot 2024-07-07 at 9.47.59 AM.png>)

<br>

배포 구성 항목은 현재 위치로 설정해주고 환경 구성에서는 Amazon EC2 인스턴스를 선택합니다. 태그 그룹은 아래 사진과 같이 본인의 EC2를 선택합니다.

<br>

![alt text](<./image/Screenshot 2024-07-07 at 9.50.30 AM.png>)

<br>

나머지 항목은 넘어가고 마지막에 로드밸런서 부분에서 활성화 체크를 해제하고 배포그룹 생성 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-07-07 at 9.55.54 AM.png>)

<br>