# 리눅스 네트워크 설정

## 1. 네트워크 설정 준비

---

### 1.1 개요

이전 시간에 부트로더를 일부 수정하여 명령어를 새롭게 추가했던 작업과 같이 커널에서도 `System Call Interface`와 `Device Driver Interface`를 일부 수정하여 새로운 작업을 시도하고자 한다.


### 1.2 Target Host Connection

현재까지 Target과 Host의 연결은 콘솔 확인하기 위한 `serial` 연결과 부트로더 모니터링 모드 진입을 위한 `usb`연결로 이루어졌다. 이와 더불어 새로운 `application`을 로드하기 위해 `ethernet network` 연결을 진행하고자 한다.

`usb` 연결에서 데이터 통신을 위해 `fastboot`라는 클라이언트-서버 프로그램을 사용했던 것과 같이 `ethernet network` 연결에서 또한 `tftp` 서버 프로그램을 설치해야 한다.


### 1.3 Windows에서 고정 IP 확인하기

타켓에는 `ethernet network`를 연결할 수 있는 인터페이스가 있다. 호스트 컴퓨터에는 `usb ethernet`을 통해 네트워크를 연결한다. Windows와 같이 리눅스 컴퓨터 또한 고정 IP를 설정할 필요가 있다.

* USB 연결
* Windows 부팅을 마쳤으면 제어판을 선택
* 네트워크 및 인터넷 -> 네트워크 및 공유센터
* 어댑터 설정 변경 -> 리얼텍 온보드
* 하이스피드 USB 이더넷 확인
* 온보드 이더넷 우측 마우스 클릭 후 속성 확인
* TCP/IP 버전 4 확인
* IP 주소, 서브넷 마스크, 게이트웨이, DNS 서버 확인 후 옮겨 적기

## 2. 리눅스에서 네트워크 설정하기

---

### 2.1 USB 이더넷 케이블 연결하기

컴퓨터 네트워크를 연결할 때는 흔히 랜선이라고 부르는 `UTP` 케이블을 사용하게 된다. 이 케이블은 다이렉트 케이블과 크로스 오버 케이블 두 종류가 있는데, 컴퓨터와 컴퓨터를 연결할 때는 크로스 오버 케이블을 사용하고 컴퓨터와 스위치를 연결할 때는 다이렉트 케이블을 사용한다. 하지만 요즘은 다이렉트 케이블이라 하더라도 컴퓨터와 컴퓨터를 연결할 때 자동으로 맞추어 인식하기 때문에 어떤 케이블을 사용해도 문제 없다.

<br>

다음 명령어들을 통해 케이블이 정상 연결되어있는지 확인한다.

<br>

```shell
 ifconfig
 lspci
 lsusb
```

<br>

![alt text](<./image/Screenshot from 2024-05-09 09-42-05.png>)

<br>

![alt text](<./image/Screenshot from 2024-05-09 09-42-17.png>)

<br>

![alt text](<./image/Screenshot from 2024-05-09 09-42-58.png>)

<br>

### 2.2 네트워크 설정 방식 소개

리눅스에서 네트워크를 설정하는 방식에는 크게 3가지가 있다. 첫 번째는 **`ifconfig`와 `route` 명령어**를 이용해서 설정하는 전통적인 방식, 두 번째는 **우분투 리눅스 배포판에서 사용 가능한 `netplan`** 을 이용해 설정하는 방식, 마지막으로 손쉽게 **GUI를 이용**해서 설정하는 방식이 있다.

<br>

먼저 GUI를 이용하는 방식의 경우 우측 상단에 네트워크 탭에서 다음과 같은 화면에서 메뉴얼 설정 방식을 통해 네트워크 정보를 조작할 수 있다.

<br>

![alt text](<./image/Screenshot from 2024-05-09 09-49-07.png>)

<br>

![alt text](<./image/Screenshot from 2024-05-09 09-49-56.png>)

<br>

네트워크를 설정하는 큰 순서 3가지는 1번 **IP 주소 설정**, 2번 **디폴트 케이트워이 설정**, 3번 **DNS 설정**으로 이루어진다.

<br>

### 2.3 IP 주소 설정하기

먼저 현재 사용하고 있는 IP 주소는 `192.168.63.XX`로 중간에 63이 들어간 DHCP 도메인이다. 따라서 IP주소를 고정 IP로 변경할 필요가 있다. 먼저 다음 명령어를 통해 네트워크 인터페이스에 대한 정보를 확인한다.

```shell
netstat -i
```

<br>

![alt text](<./image/Screenshot from 2024-05-09 09-53-30.png>)

<br>

그리고 `ifconfig` 명령어를 통해 나온 첫 번째 인터페이스 이름에 대해 다음 명령어를 입력한다. 여기서는 IP주소는 본인 컴퓨터의 IP 주소를 작성한다.

```shell
sudo ifconfig enp2s0 192.168.60.163 netmask 255.255.255.0
```

<br>

![alt text](<./image/Screenshot from 2024-05-09 09-54-25.png>)

<br>

![alt text](<./image/Screenshot from 2024-05-09 09-55-45.png>)

### 2.4 디폴트 게이트웨이 설정하기

<br>

라우팅 테이블에 방금 설정한 네트워크 정보가 등록되어있지 않다면 수동으로 라우팅 테이블에 방금 설정한 IP를 등록해 준다. 먼저 IP 주소 설정을 마쳤으면 다음 명령어로 라우팅 테이블의 정보를 확인한다.

<br>

등록 되어있지 않다면 다음 명령어로 수동으로 정보를 등록한다.

```shell
sudo route add -net 192.168.60.0 netmask 255.255.255.0 enp2s0
```
<br>

다음 명령어로 디폴트 게이트웨이를 설정하여 준다.

```shell
sudo route add default gw 192.168.60.1
```

<br>

다음과 같이 라우팅 테이블의 정보를 다시 확인해 보면 제대로 등록된 것을 확인할 수 있다.

```shell
route -n
```

<br>

![alt text](<./image/Screenshot from 2024-05-09 10-07-31.png>)

<br>

IP 주소 설정과 디폴트 게이트웨이 설정을 마치면 기본적인 네트워크 통신이 가능하다. 테스트를 위해 임의의 주소에 핑을 찍어보면 제대로 전송되는 것을 확인할 수 있다.

<br>

![alt text](<./image/Screenshot from 2024-05-09 10-08-05.png>)

<br>

### 2.5 DNS 설정하기

DNS 설정의 경우 직접적으로 설정을 진행하는 명령어가 따로 없기 때문에 설정 파일을 직접 수정해 주어야 한다. 해당 파일을 수정하면 네트워크 매니저라는 데몬이 해당 파일을 자동으로 읽어들인다. 해당 설정 파일의 위치는 다음과 같다.

```shell
/etc/netplan/01-network-manager-all.yaml
```

<br>

그리고 해당 파일을 vi 명령어로 열어준다.

<br>

![alt text](<./image/Screenshot from 2024-05-09 10-37-40.png>)

<br>

처음 들어가면 해당 파일 안에는 별 내용이 적혀 있지 않다. `enp2s0`에 해당하는 이더넷 이름을 입력해 준다. 그리고 dhcp는 사용하지 않을 것이므로 no를 입력하고 `addresses`에 현재 해당 컴퓨터의 IP 주소를 입력한다. 디폴트 게이트웨이 정보를 다음으로 입력하고 DNS 서버 정보를 마지막으로 입력하여 준다. (성공회대에서 관리하는 DNS 주소가 두 개라 둘 모두 입력한다.)

<br>

![alt text](<./image/Screenshot from 2024-05-09 10-44-01.png>)

<br>

저장하고 나오고 나서 설정 파일을 다시 읽어들이기 위해 다음 명령어를 입력한다.

```shell
sudo netplan apply
```

![alt text](<./image/Screenshot from 2024-05-09 10-45-55.png>)

<br>

비록 경고 문구가 뜨지만 적용이 된 것이다. 경고 내용은 야믈 파일에 대한 권한이 너무 오픈 되어있다는 것이다. other와 group의 권한 중 읽기 권한을 제외한다.

```shell
sudo chmod o-r 01-network-manager-all.yaml #other
sudo chmod g-r 01-network-manager-all.yaml #group
```

<br>

![alt text](<./image/Screenshot from 2024-05-09 10-52-02.png>)

<br>

## 3. Target 네트워크 연결

---

미니컴으로 콘솔을 연결하고 이더넷까지 제대로 케이블을 연결하여 준다.

<br>

이번에는 연결된 이더넷 인터페이스에 대해 IP 주소를 설정하여 준다. 방식은 외부 네트워크 인터페이스에 대해 IP 주소를 설정했던 것과 동일하다.

```shell
sudo ifconfig enx00e9e0003e8f 10.10.10.1 netmask 255.255.255.00
```

<br>

![alt text](<./image/Screenshot from 2024-05-09 11-00-27.png>)

<br>

Target으로 이동해서 `ifconfig` 명령어를 입력한다. `eth0` 인터페이스를 확인한다.

<br>

![alt text](<./image/Screenshot from 2024-05-09 11-04-15.png>)

<br>

그리고 이번에도 이에 대한 IP 주소를 등록하여 준다. 네트워크는 Host와 연결될 네트워크와 동일하게 설정한다.

```shell
ifconfig eth0 10.10.10.2 netmask 255.255.255.0
```

<br>

등록을 마쳤으면 ping으로 연결 테스트를 해본다.

<br>

![alt text](<./image/Screenshot from 2024-05-09 11-19-54.png>)

<br>

반대로 Host에서도 `ping 10.10.10.2`에 대한 요청이 제대로 전달이 된다면 다시 설정 파일을 열어서 `enx00e9e0003e8f` 인터페이스에 대한 네트워크 설정을 해준다. 디폴트 게이트웨이와 DNS는 하나의 인터페이스에서만 한 번 해주면 되기 때문에 생략한다.

```shell
sudo vi /etc/netplan/01-network-manager-all.yaml
```

<br>

![alt text](<./image/Screenshot from 2024-05-09 11-30-07.png>)

<br>

설정을 완료했으면 다시 재반영 해준다.

```shell
sudo netplan apply
```

<br>

`ifconfig`를 입력해보면 다음과 같이 잘 반영된 것을 확인할 수 있다.

<br>

![alt text](<./image/Screenshot from 2024-05-09 11-30-49.png>)

<br>
