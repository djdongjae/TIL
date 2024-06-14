# File System

## 1. 개요

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

## 2. File System 디렉토리 살펴보기 및 이미지 굽기

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

<br>

## 3. NFS: Network File System

### 3.1 개요

네트워크를 이용해 파일을 공유하는 시스템으로 멀티 유저 시스템인 리눅스에서 사용할 수 있는 프로그램이다. 현 시점에서는 TFTP를 대체하기 위한 용도라고 볼 수 있다. 서버의 디렉토리를 클라이언트에 마운트하는 개념이라고 볼 수 있다.

<br>

NFS도 파일시스템이기 때문에 결국 커널이 지원해야 사용할 수 있다. 따라서 현재 커널에서 NFS를 지원하고 있는지 확인하기 위해 `~/class/embedded_linux/work/kernel/kernel_4412` 위치에서 `make menuconfig` 명령어를 입력한다.

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-31-36.png>)

<br>

이제 다음 순서대로 `File systems` -> `The Extended 4 (ext4) filesystem` -> `Network File Systems` -> `NFS client support`로 진입하면 NFS를 지원하는 것을 확인 할 수 있다.

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-31-52.png>)

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-31-56.png>)

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-32-52.png>)

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-33-04.png>)

<br>

### 3.2 RPC

RPC란 Remote Procedure Call의 약자로 클라이언트에서 서버로부터 디렉토리를 받거나 마운트 시켰으면 서버의 프로시저를 호출해서 실행할 수 있게 하는 통신 기술이다. 

<br>

따라서 NFS를 이용하기 위해서는 서버에 `RPC Bind`가 설치 되어있어야 한다. 먼저 서버에 nfs 관련 패키지가 설치 되어있는 지 확인하기 위해 다음 명령어를 입력한다.

```shell
apt list --installed | grep nfs
```

다음과 같은 패키지가 설치된 것을 확인할 수 있다.

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-36-11.png>)

<br>

다음으로 `RPC Bind` 관련 프로그램이 설치 되어있는 지 확인한다. 만약 설치가 되어있다면 프로세스가 실행 중이어야 하기 때문에 다음 명령어를 입력한다.

```shell
ps -ef | grep rpcbind
```

설치가 되어있지 않기 때문에 다음 명령어로 설치를 진행한다.

```shell
sudo apt install rpcbind
```

설치를 완료하면 자동으로 프로세스가 실행된 모습을 확인할 수 있다.

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-37-28.png>)

<br>

다음으로 커널용 nfs 서버도 설치를 해주어야 한다. 다음 명령어를 입력한다.

<br>

```shell
sudo apt install nfs-kernel-server
```

<br>

![alt text](<./image/Screenshot from 2024-05-23 11-37-56.png>)

<br>

### 3.3 NFS 서버 디렉토리 설정

NFS는 기본적으로 서버의 디렉토리를 클라이언트가 자유롭게 사용할 수 있게 하는 구조이기 때문에 엄격한 절차를 거쳐 어떤 디렉토리를 사용할 수 있게 할 것인지 결정해야 한다. 다음 위치의 파일을 열어본다.

```shell
sudo vi /etc/exports
```
<br>

서버의 `/home/dongjaeoh/class/embedded_linux` 디렉토리를 `10.10.10.2`에 해당하는 컴퓨터가 사용할 수 있게 등록할 것이다. 또한 옵션으로 read, write가 가능하고 sync도 맞출 것이며 `no_root_squash` 설정을 통해 클라이언트가 가지고 있는 루트 권한을 뺏지 않게 한다. 즉, 임베디드 환경이기 때문에 클라이언트 또한 자유롭게 서버의 디렉토리에 대해 루트 권한을 행사할 수 있게 설정해 준다.

```shell
/home/dongjaeoh/class/embedded_linux 10.10.10.2(rw,sync,no_root_squash)
```

<br>

![alt text](<./image/Screenshot from 2024-05-30 09-23-51.png>)

<br>

먼저 `rpcbind` 데몬과 NFS 서버가 잘 동작하고 있는지 확인한다.

```shell
sudo systemctl status rpcbind.service #rpcbind
sudo systemctl status nfs-kernel-server.service #nfs
```

<br>

![alt text](<./image/Screenshot from 2024-05-30 09-24-57.png>)

<br>

그리고 설정 수정했으니 변경사항을 재반영하여 준다. 

```shell
sudo systemctl restart nfs-kernel-server.service
```

<br>

![alt text](<./image/Screenshot from 2024-05-30 09-26-01.png>)

<br>

RPC가 잘 동작하는지 확인하기 위해 다음 명령어를 입력한다.

```shell
rpcinfo -p
```

<br>

![alt text](<./image/Screenshot from 2024-05-30 09-26-26.png>)

<br>

설정한 exports가 제대로 반영되었는지 확인하기 위해 다음 명령어를 입력한다.

```shell
sudo exportfs -v
```

<br>

![alt text](<./image/Screenshot from 2024-05-30 09-26-38.png>)

<br>

다음 명령어로도 마운트된 파일 시스템 정보를 확인할 수 있다.

```shell
showmount -e localhost
```

<br>

![alt text](<./image/Screenshot from 2024-05-30 09-27-45.png>)

<br>

Target으로 돌아가서 케이블을 모두 연결하고 부팅을 진행하여 준다. 수동으로 마운트 되는 파일 시스템을 관리하기 위해서는 `/mnt` 경로에 진입한다. 자동으로 마운트되는 파일 시스템은 `/media`에서 관리한다.

<br>

![alt text](<./image/Screenshot from 2024-05-30 09-38-06.png>)

<br>

실습을 위해 `/mnt` 밑에 `embedded_linux` 디렉토리를 하나 생성한다.

<br>

![alt text](<./image/Screenshot from 2024-05-30 09-39-11.png>)

<br>

다음으로 Host와 통신하기 위해 이더넷 IP주소를 등록한다.

```shell
ifconfig eth0 10.10.10.2 netmask 255.255.255.0
```

<br>

NFS 서버와 통신하기 위해 디렉토리를 마운트 시켜주는 다음 명령어를 입력한다.

```shell
mount -t nfs -o nolock 10.10.10.1:/home/dongjaeoh/class/embedded_linux /mnt/embedded_linux
```

<br>

일전에 만들어 두었던 `hello_arm`이나 `hello_mysyscall` 같은 파일을 실행하여 보면 정상적으로 실행되는 것을 확인할 수 있다.

<br>

