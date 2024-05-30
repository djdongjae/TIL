# File System

### 1. 개요

---

기존에 제공된 파일 시스템 이미지는 `Development/Image/` 위치에 `rootfs_ext4.img`의 이름으로 저장되어있었다. 해당 이미지를 커널에 마운트 시키면 파일 시스템이 형성된다. 

<br>

![alt text](<./image/Screenshot from 2024-05-23 10-27-45.png>)

<br>

하지만 이와 같은 파일 시스템만으로는 개발한 응용 프로그램을 실행시킬 수 없다. 프로그램이 의존하는 라이브러리가 파일 시스템 상에 존재하지 않기 때문이다. 따라서 파일 시스템은 지속적으로 업데이트가 되어야 한다.

<br>

![alt text](<./image/Screenshot from 2024-05-23 10-33-36.png>)

<br>

기존에는 `ramdisk`라고 하는 휘발성 메모리 기반의 파일 시스템을 사용했다. 그러다 파일 시스템의 규모가 점점 커지다보니 플래시를 이용한 파일 시스템이 등장하게 되었다. 이후 플래시 파일 시스템을 거쳐 현재는 하드디스크와 SSD를 이용한 파일 시스템을 주로 활용한다.
> ✅ 파일 시스템을 설계할 때 개발자가 처음부터 끝까지 모든 것을 설계하는 것이 아니라 `BusyBox` 라는 소프트웨어를 이용하여 임베디드 시스템 구축에 필요한 다양한 패키지를 로드한다.

### 2. File System 디렉토리 살펴보기 및 이미지 굽기

---

![alt text](<./image/Screenshot from 2024-05-23 10-45-58.png>)

<br>

해당 위치에 루트 권한으로 `rootfs_h4412.tgz` 파일을 압축 해제하여 준다. 

<br>

![alt text](<./image/Screenshot from 2024-05-23 10-49-14.png>)

이와 같은 파일 시스템은 시스템을 업데이트할 때 기반이 되는 틀이다. 예를 들어 아이폰에서 "오늘밤 시스템 업데이트를 진행합니다" 라는 말의 의미는 해당 파일 시스템에 응용 프로그램과 라이브러리를 추가하여 새롭게 생성된 이미지를 다시 구워주는 과정을 거친다는 뜻이다. 물론 커널까지 업데이트하는 대규모 업데이트가 진행되기도 한다.

<br>

해당 파일시스템의 이미지를 생성하는 스크립트 쉘이 따로 존재한다. 해당 쉘 코드를 복사하여 현재 디렉토리로 옮겨 준다.

<br>

![alt text](<./image/Screenshot from 2024-05-23 10-53-27.png>)

<br>

해당 파일을 실행해보고자 한다면 권한이 없어 오류가 발생하기 때문에 `chmod a+rx` 명령어로 읽기와 실행 권한을 부여한다. 필요하다면 유저에게 쓰기 권한도 부여한다. `chmod u+w`

<br>

![alt text](<./image/Screenshot from 2024-05-23 10-59-23.png>)

<br>

`sudo ./mkfs.sh /rootfs_H4412/` 명령어를 통해 이미지를 구워보면 다음과 같은 에러가 발생한다.

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-02-01.png>)

<br>

루프 디바이스에 관련된 문제이기 때문에 현재 우리 시스템의 루프 디바이스가 어떻게 설정되어있는지 확인한다. (명령어: `losetup -a`)

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-03-11.png>)

<br>

0번부터 12번까지 루프 디바이스가 사용중인 것을 확인할 수 있다. (명령어: `l /dev/loop*`)

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-03-23.png>)

<br>

사용되고 있지 않은 루프 디바이스를 확인하기 위해 다음 명령어를 입력한다. 13번이 비어있는 것을 확인할 수 있다. (명령어: `sudo losetup -f`)

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-04-02.png>)

<br>

따라서 현재 사용 중인 0번 루프 디바이스가 아닌 비어있는 13번 루프 디바이스를 사용하기 위해 쉘 스크립트를 다시 열어서 `loop0`에 해당하는 부분들을 전부 `loop13`으로 변경해준다. (명령어: `vi mkfs.sh`)

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-05-39.png>)

<br>

변경을 완료하였으면 저장하고 나와서 다시 실행하면 정상적으로 이미지가 생성된 것을 확인할 수 있다. (명령어: `sudo ./mkfs.sh rootfs_H4412/`)

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-07-20.png>)

<br>

이미지 생성이 완료되었으면 fastboot를 이용해서 커널에 이미지를 플래시 해준다. (명령어: `sudo fastboot flash system root_ext4.img`)

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-09-16.png>)

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-09-22.png>)

<br>

이미지 플래시를 완료한 이후 다시 부팅을 해보면 부팅이 되지 않는다. 이 부분은 사실 해결할 수 없는 부분이다. 64비트용 우분투에서 32비트용 파일 시스템을 만들 수 없어 생기는 문제라 그냥 넘어가자.

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-13-51.png>)

<br>

과정만 이해하고 넘어간 후 칩 제조사에서 제공해주는 정상 작동하는 이미지로 다시 플래싱해준다. `Hsmar~/Development/Image` 상에 위치한 이미지를 다시 구워준다. (명령어: `sudo fastboot flash system root_ext4.img`)

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-21-50.png>)

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-25-23.png>)
