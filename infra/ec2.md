# AWS EC2

### 1. EC2 세팅

---

#### 1.1 AWS Region 설정하기

우측 상단에 AWS의 리전이 서울로 설정 되어있는지 확인합니다. 서울이 아니라면 **아시아 태평양 서울**로 변경하여 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 6.07.56 PM.png>)


#### 1.2 EC2 인스턴스 생성하기

검색창에 'EC2'를 입력하고 EC2 대시보드를 선택하고 인스턴스 시작하기 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 6.40.17 PM.png>)

<br>

![alt text](<./image/Screenshot 2024-06-01 at 6.56.05 PM.png>)

<br>

인스턴스에 적당한 이름을 설정하여 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 7.10.50 PM.png>)

<br>

다음으로 AMI를 설정하여 줍니다. 여기서는 **Amazon Linux**의 **Amazon Linux 2023 AMI**를 선택합니다. 아키텍처는 **64비트(x86)** 을 선택하여 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 7.17.05 PM.png>)

<br>

다음은 인스턴스 유형을 선택하는 단계입니다. 인스턴스 유형에서는 프리티어로 표기된 **t2.micro**를 선택합니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 7.23.41 PM.png>)

<br>

인스턴스로 접근하기 위해서는 비밀 키가 필요합니다. 인스턴스는 지정된 pem 키(비밀키)와 매칭되는 공개키를 가지고 있어서 해당 pem 키 외에는 접근을 허용하지 않습니다. `새 키 페어 생성`을 클릭하여 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 7.31.35 PM.png>)

<br>

다음과 같이 키페어에 적당한 이름을 설정하고, RSA 유형의 .pem 키 파일을 생성하고 다운로드 받아줍니다. 한 번 다운받은 키 파일은 두 번 다시 다운받을 수 없기 때문에 보관 위치를 잘 기억합니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 7.47.42 PM.png>)

<br>

다음 네트워크 설정에서는 추후에 다시 설정할 것이기 때문에 지금은 별다른 설정 없이 넘어갑니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 10.39.02 PM.png>)

<br>

다음 단계는 스토리지 구성입니다. 스토리지는 여러분이 흔히 하드디스크라고 부르는 서버의 디스크를 이야기하며 서버의 용량을 설정하는 단계입니다. 기본값 설정은 8GB이지만 30GB까지 프리티어로 가능하므로 변경하여 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 10.45.14 PM.png>)

<br>

요약이 아래 화면과 같다면 인스턴스 시작 버튼을 눌러줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 10.48.43 PM.png>)

<br>

인스턴스 탭으로 이동해 보면 정상적으로 인스턴스가 생성된 것을 확인할 수 있습니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 10.57.12 PM.png>)

#### 1.3 보안 그룹 설정하기

보안 그룹은 방화벽을 이야기합니다. **서버로 8080포트 이외의 요청은 허용하지 않는다** 는 역할을 하는 방화벽이 AWS에서는 보안 그룹으로 사용됩니다.

> ✅ **인바운드 규칙(inbound)** : 외부에서 EC2나 RDS 등의 내부로 접근할때 사용되는 방화벽 규칙<br>
> ✅ **아웃바운드 규칙(outbound)** : EC2나 RDS 등의 내부에서 외부로 접근할때 사용되는 방화벽 규칙

저희는 EC2에 접속해서 서버를 띄우는 것이 목적이기 때문에 인바운드 규칙만 설정해 보겠습니다. 좌측 **네트워크 및 보안 그룹 탭에서 보안 그룹**을 선택해 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 11.10.57 PM.png>)

<br>

상단의 **보안 그룹 생성** 버튼을 클릭하여 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 11.14.26 PM.png>)

<br>

보안 그룹에 적당한 이름과 설명을 적고 하단의 인바운드 규칙 추가 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 12.37.28 AM.png>)

<br>

아래 화면과 같이 총 4개의 규칙을 추가합니다. 규칙에 대한 설명은 아래와 같습니다.

* **8080**: 스프링 부트 기반의 서버를 열어줄 것이기 때문에 사용자 지정으로 8080 포트를 개방하고 URL을 아는 누구나 접속할 수 있도록 Anywhere-IPV4로 설정합니다.
* **22(SSH)**: 현재 사용 중인 컴퓨터에서 서버로 접속하고자 할 때 사용합니다. Anywhere-IPV4로 SSH 트래픽을 허용해야 집에 아니라 카페, 강의실과 같은 곳에서도 접속이 가능합니다.
* **80(HTTP)**: HTTP 연결 시 사용됩니다.
* **433(HTTPS)**: HTTPS 연결 시 사용합니다.

<br>

![alt text](<./image/Screenshot 2024-06-01 at 11.27.54 PM.png>)

<br>

작성을 완료하였으면 보안 그룹 생성 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 12.32.13 AM.png>)

<br>

다시 EC2 인스턴스 탭으로 돌아와서 인스턴스 ID 위에서 우측 마우스를 클릭하고 **보안 -> 보안그룹변경**을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 12.39.59 AM.png>)

<br>

보안 그룹 선택에서 앞서 생성한 보안그룹을 선택하여 추가하고 기존의 보안 그룹은 제거해 줍니다. 마지막으로 저장 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 12.42.40 AM.png>)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 12.44.01 AM.png>)

#### 1.4 탄력적 IP 할당하기

**AWS의 고정 IP를 Elastic IP(EIP, 탄력적 IP)** 라고 합니다. 인스턴스 생성 시에 항상 새 IP를 할당하는데, 인스턴스를 중지했다가 다시 시작할 때도 새 IP가 할당됩니다. 이렇게 되면 CPU 사용률이 99%가 되어 인스턴스를 잠시 중지할 때도, 요금 절약을 위해 잠시 중지할 때도 다시 시작하면 IP가 변경됩니다. 다시 접속할 때마다 IP를 다시 확인해야 하는 번거로운 작업을 거치게 되기 때문에 인스턴스를 중지하여도 IP가 변경되지 않는 고정IP를 할당합니다.

<br>

좌측의 네트워크 및 보안 탭에 탄력적 IP를 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 12.55.17 AM.png>)

<br>

탄력적 IP 주소 할당 버튼을 클릭하여 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.00.44 AM.png>)

<br>

별다른 설정 없이 할당 버튼을 클릭합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.03.16 AM.png>)

<br>

방금 생성한 탄력적 IP를 선택하고 우측 상단의 **작업 -> 탄력적 IP 주소 연결**을 눌러줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.08.04 AM.png>)

<br>

주소 연결을 위해 생성한 인스턴스와 프라이빗 IP 주소를 선택하고 연결해줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.12.21 AM.png>)

<br>

인스턴스 상세 페이지에서 탄력적 IP가 제대로 할당되었는지 확인하여 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.15.22 AM.png>)

<br>

**주의할 점이 있습니다!!!** 방금 생성한 탄력적 IP는 생성하고 EC2 서버에 연결하지 않으면 비용이 발생합니다. 즉, 생성한 탄력적 IP는 무조건 EC2에 바로 연결해야 하며 더는 사용할 인스턴스가 없을 때도 아래와 같이 탄력적 IP를 삭제(릴리스)해야 합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.22.55 AM.png>)

<br>


### 2. EC2 서버에 접속하기

---

#### 2.1 SSH 연결

터미널 창을 열고 홈 디렉토리에 `.ssh` 디렉토리가 있는지 확인해 줍니다. 참고로 리눅스에서 점(.)으로 시작하는 파일이나 디렉토리는 숨김 처리됩니다. (명령어: `ls -al | grep .ssh`)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.27.45 AM.png>)

<br>

없다면 만들어 줍니다. (명령어: `mkdir .ssh`)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.26.14 AM.png>)

<br>

해당 디렉토리로 이동합니다. (명령어: `cd .ssh`) 이동을 완료했으면 현재 디렉토리로 이전에 다운로드 받은 `.pem` 파일을 이동시켜 줍니다. (명령어: `mv .pem파일위치 .`) 정확한 경로를 외울 필요 없이 GUI에서 .pem 파일을 복사하고 터미널 창에 붙혀넣기하면 됩니다. 

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.41.53 AM.png>)

<br>

해당 디렉토리로 제대로 들어왔는지 확인합니다. (명령어: `ls -al`)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.48.04 AM.png>)

<br>

다시 EC2 인스턴스 대시보드로 돌아와서 생성한 인스턴스를 선택하고 연결 버튼을 눌러줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.49.37 AM.png>)

<br>

SSH 클라이언트 탭에서 하단에 위치한 연결 명령어 예시를 복사합니다. (왼쪽에 네모 클릭하면 자동 복사)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.51.28 AM.png>)

<br>

터미널 창으로 돌아와서 해당 명령어를 복사하여 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.54.03 AM.png>)

<br>

엔터를 누르고 질문에 yes를 입력합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.55.55 AM.png>)

<br>

다음과 같은 오류가 발생하면, other의 권한을 변경하여 줍니다. other와 group에게 읽기, 쓰기, 실행 그 어떤 권한도 부여하지 않습니다. (명령어: `chmod 600 LikeLionServer.pem`)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 2.01.29 AM.png>)

<br>

다시 접속을 시도해 보면 정상적으로 접속되는 것을 확인할 수 있습니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 2.03.06 AM.png>)

#### 2.2 타임존 변경하기

EC2 서버의 기본 타임존은 UTC입니다. 이는 세계 표준 시간으로 한국의 시간대가 아닙니다. 즉 한국의 시간과는 9시간이 차이가 발생합니다. 서버의 타임존을 **한국 시간(KST)** 으로 변경합니다.

우선 현재 타임존이 어떻게 설정되어 있는지 확인합니다. (명령어: `date`)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 2.05.43 AM.png>)

<br>

다음 명령어를 차례로 수행합니다.

* `sudo rm /etc/localtime`
* `sudo ln -s /usr/share/zoneinfo/Asia/Seoul /etc/localtime`

<br>

다시 한국 시간으로 타임존이 잘 변경되었는지 확인합니다. (명령어: `date`)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 2.15.28 AM.png>)

<br>

#### 2.3 스왑 메모리 설정

운영체제 메모리의 데이터들 중에서 어느 순간 활발하게 읽거나 쓰여지는 부분은 많지 않고 대부분은 아주 가끔 읽거나 쓰여집니다. 따라서 메모리에서 활발하게 읽거나 쓰여지는 부분만 RAM 에 보관하고, 아주 가끔 읽거나 쓰여지는 부분은 디스크(HDD 나 SDD)에 보관하게 된다면 메모리를 더 효율적으로 사용할 수 있습니다. 디스크의 이러한 공간을 **스왑 영역**이라고 합니다.

<br>

현재 메모리의 용량과 사용량을 확인합니다. 약 1GB 정도의 용량을 가지고 있다. 이 정도로는 Nginx와 Tomcat을 구동하기 어림도 없습니다. (명령어: `free -m`)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 2.25.54 AM.png>)

<br>

스왑 영역으로 사용할 파일을 만들어 줍니다. 다음 명령어를 순서대로 입력합니다.

* **스왑 영역으로 사용할 파일 만들기**: `sudo dd if=/dev/zero of=/swapfile bs=1M count=1024`
* **소유자만 읽고 쓸 수 있게 권한을 설정**: `sudo chmod 600 /swapfile`
* **swapfile 파일을 스왑 영역으로 만들기1**: `sudo mkswap /swapfile`
* **swapfile 파일을 스왑 영역으로 만들기2**: `sudo swapon /swapfile`
  
<br>

![alt text](<./image/Screenshot 2024-06-02 at 2.36.37 AM.png>)

<br>

다음으로 `/etc/fstab` 파일에 swapfile 등록합니다. vi 명령어를 통해 `/etc/fstab`를 열어서 다음 코드를 복사하여 마지막 줄에 추가합니다. (명령어: `sudo vi /etc/fstab`)

```shell
/swapfile swap swap defaults 0 0
```

<br>

![alt text](<./image/Screenshot 2024-06-02 at 2.48.09 AM.png>)

<br>

`vi editor`에서 편집과 저장을 위한 명령어는 다음과 같습니다.

* `i` : 편집모드로 전환
* `esc`키 입력: 명령모드로 전환
* `:wq!`: 변경하사항 저장하고 나오기
* `:q!`: 변경사항 저장 안하고 나오기 

따라서 현재는 `i`를 누르고 위 코드를 복사 붙혀넣기 한 후, `esc` 키를 누르고 `:wq!`를 명령어를 입력하여 변경사항을 저장하고 종료합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 2.53.15 AM.png>)

<br>

스왑 메모리가 정상적으로 반영되었는지 확인합니다. 스왑 메모리가 1GB 생성된 것을 확인할 수 있습니다. (명령어: `free -m`)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.00.19 AM.png>)

#### 2.4 java 설치

다음 명령어로 java 17을 설치하여 줍니다. 중간에 질문이 나오면 `y`를 입력합니다.

```console
sudo yum install java-17-amazon-corretto-headless
```

<br>

![alt text](<./image/Screenshot 2024-06-02 at 2.56.36 PM.png>)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 2.56.58 PM.png>)

<br>

제대로 설치되었는지 확인하고 싶다면 `java --version`을 입력합니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 2.57.09 PM.png>)

#### 2.5 git clone 하기

우선 git 설치를 위해 다음 명령어를 입력해 줍니다. 중간에 질문이 나오면 `y`를 입력합니다.

```shell
sudo yum install git
```
  
<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.21.06 AM.png>)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 3.21.26 AM.png>)

<br>

잘 설치되었는지 확인 하고 싶다면 `git --version`을 입력합니다.

<br>

그리고 배포하고 싶은 깃허브 레포지토리의 URL을 복사하고 홈 디렉토리에서 클론해 줍니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 1.57.25 PM.png>)

<br>

현재 배포 서버에는 `application-prod.yml` 파일이 업로드 되어있지 않기 때문에 로컬 컴퓨터에서 해당 파일을 업로드해 줍니다. 업로드 하는 방식에는 다음과 같이 두 가지가 있습니다.

1. 로컬 컴퓨터에서 서버 컴퓨터로 파일 직접 업로드
2. vi 편집기를 이용해 파일을 만들고 내용만 복사

1번 방식이 정석이지만 사람마다 파일 위치가 다를 것을 고려하여 2번 방식으로 대체하겠습니다. 1번 방식의 명령어만 소개하자면 다음과 같습니다.

```shell
scp -i 키파일 application-prod.yml ec2-user@IP 주소
# 위 명령에 의해서, 내 PC 의 ~/application-prod.yml 파일이, EC2 서버의 홈디렉토리(~)로 업로드 됩니다.
```

<br>

우선 서버 컴퓨터에서 `application-prod.yml`이 있어야 할 위치로 이동합니다. (명령어: `cd /home/ec2-user/LikeLionServer/src/main/resources`)

<br>

그리고 이곳에서 `application-prod.yml` 파일을 생성하여 줍니다. (명령어: `vi application-prod.yml`)

<br>

![alt text](<./image/Screenshot 2024-06-02 at 2.23.39 PM.png>)

<br>

해당 파일에 기존의 `application-prod.yml` 내용을 그대로 복사합니다. 그리고 저장하고 나옵니다.

<br>

![alt text](<./image/Screenshot 2024-06-02 at 2.28.35 PM.png>)

