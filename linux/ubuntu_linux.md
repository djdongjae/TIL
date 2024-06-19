# Ubuntu Linux

## 컴퓨터 보조기억장치 파티션

- C(SSD): /dev/sda → 1번 하드(windows 설치)
- D: /dev/sdb → 2번 하드 설치 되어있음
    - 보통 1번과 2번에 설치가 유의미한 기본 설정 데이터가 저장되어 있으므로 3, 4, 5번 파티션을 생성하여 리눅스 관련 설정을 진행한다.

## 임베디드 리눅스 수업의 목표

- ARM 기반의 CPU에서 동작할 수 있는 Embedded Linux System(Target)을 개발하기 위해서는 Intel 기반의 리눅스 운영체제(Host)에서 해당 시스템을 개발해야 한다
- 해당 시스템이 동작하기 위해 필요한 것에는 Boot Loader, Kernal, File System이 있다
- 이를 위해서는 native compiler가 필요하다
- 즉, Compiler는 Intel에서 동작하지만 Compiler가 만든 Binary Code는 ARM에서 돌아가야 함
- 해당 과정을 크로스 컴파일이라고 함

## Ubuntu Linux 설치 방법

- 기본적으로 install ubuntu desktop을 참고하여 usb에 .iso 파일을 구워와야 함
- F11키를 부팅 시작할 때 겁나게 누른다.
- UEFI를 선택 →continue → something else
- SATAS3_2  → D drive
- SATAS3_1 → C drive
- sdb에 리눅스 설치를 진행해야 함
- 파티션에 따라서 3, 4, 5번에 설정을 진행해야 할 수도 있고 2, 3, 4번에 설정을 진행해야 할 수도 있음
- 3 → boot, 4 → swap, 5 → 나머지

![alt text](<./image/Screenshot 2024-06-19 at 7.01.59 PM.png>)

- 그 이후에 install 버튼 클릭

## Windows Boot Manager 바로 실행되게끔 바꾸기

- 가만히 냅두면 boot loader는 sda에 설치되어 windows boot manager 대신에 리눅스 부트 로더가 바로 실행이 되어 자동으로 하여금 windows boot manager가 실행될 수 있도록 변경해야 함
- GRUB2란 리눅스 부트 로더를 의미한다
- ls -al 명령어를 입력했을 때 d로 시작하는 것은 디렉토리, l로 시작하는 것은 shortcut을 의미한다
- 리눅스에서 super user의 이름은 root이다
- /etc 디렉토리에는 리눅스 관련 설정을 담당하는 중요한 파일들이 포함되어 있다
- /boot/grub/grub.cfg 파일에는 GRUB2 관련 설정이 들어있다
- vmlinuz가 리눅스 커널이고 grub가 부트 로더이다
- 위 파일에 직접 접근을 하는 것은 불가능하기 때문에 /etc/default/grub에서 설정을 변경하면 grub.cfg 파일에 변경사항이 반영된다
- 우선 sudo cat /boot/grub/grub.cfg | grep Windows 명령어를 통해 Windows Boot Manager를 찾아 준다
- 터미널 창에서 복사는 단순 텍스트 드래그를 통해, 붙혀넣기는 마우스 휠 클릭을 통해 진행한다
- sudo gedit /etc/default/grub를 입력하고
    - GRUB_DEFAULT = ‘Windows Boot Manager(on /dev/sd2)’를 추가해 준다
    - 기존의 GRUB_DEFAULT = 0은 주석처리
- 이후 설정을 반영했으면 이를 적용하기 위해 sudo update-grub

## 사용자 추가하기

- sudo adduser cse
- usermod man → 설명서 출력
- sudo usermod -aG adm, …(추가하고 싶은 그룹) cse
- id 명령어를 통해 현재 로그인 된 사용자 정보 출력

## 네트워크 관련 설정

- sudo apt install net-tools을 통해 네트워크 설정 관련 패키지를 다운로드한다
- ifconfig를 통해 네트워크 설정을 출력한다