# 리눅스 네트워크 설정

### 네트워크 설정 준비

---

#### 개요

이전 시간에 부트로더를 일부 수정하여 명령어를 새롭게 추가했던 작업과 같이 커널에서도 `System Call Interface`와 `Device Driver Interface`를 일부 수정하여 새로운 작업을 시도하고자 한다.


#### Target Host Connection

현재까지 Target과 Host의 연결은 콘솔 확인하기 위한 `serial` 연결과 부트로더 모니터링 모드 진입을 위한 `usb`연결로 이루어졌다. 이와 더불어 새로운 `application`을 로드하기 위해 `ethernet network` 연결을 진행하고자 한다.

`usb` 연결에서 데이터 통신을 위해 `fastboot`라는 클라이언트-서버 프로그램을 사용했던 것과 같이 `ethernet network` 연결에서 또한 `tftp` 서버 프로그램을 설치해야 한다.


#### Windows에서 고정 IP 확인하기

타켓에는 `ethernet network`를 연결할 수 있는 인터페이스가 있다. 호스트 컴퓨터에는 `usb ethernet`을 통해 네트워크를 연결한다. Windows와 같이 리눅스 컴퓨터 또한 고정 IP를 설정할 필요가 있다.

* USB 연결
* Windows 부팅을 마쳤으면 제어판을 선택
* 네트워크 및 인터넷 -> 네트워크 및 공유센터
* 어댑터 설정 변경 -> 리얼텍 온보드
* 하이스피드 USB 이더넷 확인
* 온보드 이더넷 우측 마우스 클릭 후 속성 확인
* TCP/IP 버전 4 확인
* IP 주소, 서브넷 마스크, 게이트웨이, DNS 서버 확인 후 옮겨 적기

### 리눅스에서 네트워크 설정하기

---

#### USB 이더넷 연결 확인

이더넷 케이블에는 양쪽 색 배열이 달라 컴퓨터와 컴퓨터를 연결하는데 사용되는 크로스 오버 케이블과 컴퓨터와 스위치 등을 연결할 때 사용되는 다이렉트 케이블이 있다. 케이블을 연결하고 `ifconfig` 명령어를 입력하여 연결을 확인한다.

![alt text](<./image/Screenshot from 2024-05-09 09-42-05.png>)

`lspci` 명령어를 통해 리얼텍 온보드와 연결된 것을 확인한다.

![alt text](<./image/Screenshot from 2024-05-09 09-42-17.png>)

usb를 통한 연결을 확인하고 싶으면 다음과 같이 `lusb` 명령을 통해 확인한다.
![alt text](<./image/Screenshot from 2024-05-09 09-42-58.png>)
인터페이스 정보 확인
![alt text](<./image/Screenshot from 2024-05-09 09-53-30.png>)
인터페이스에 IP 설정하기
![alt text](<./image/Screenshot from 2024-05-09 09-55-45.png>)
라우팅 테이블 설정하기
![alt text](<./image/Screenshot from 2024-05-09 10-05-50.png>)

![alt text](<./image/Screenshot from 2024-05-09 10-07-10.png>)
![alt text](<./image/Screenshot from 2024-05-09 10-08-05.png>)

네트워크 설정 파일
![alt text](<./image/Screenshot from 2024-05-09 10-37-40.png>)
![alt text](<./image/Screenshot from 2024-05-09 10-44-50.png>)

권한 부여
![alt text](<./image/Screenshot from 2024-05-09 10-45-55.png>)

![alt text](<./image/Screenshot from 2024-05-09 10-46-35.png>)
![alt text](<./image/Screenshot from 2024-05-09 10-46-41.png>)

![alt text](<./image/Screenshot from 2024-05-09 10-46-52.png>)
![alt text](<./image/Screenshot from 2024-05-09 10-52-02.png>)

이더넷 케이블 연결 후 핑 테스트
![alt text](<./image/Screenshot from 2024-05-09 11-04-15.png>)

![alt text](<./image/Screenshot from 2024-05-09 11-30-07.png>)
![alt text](<./image/Screenshot from 2024-05-09 11-30-49.png>)
![alt text](<./image/Screenshot from 2024-05-09 11-19-54.png>)
