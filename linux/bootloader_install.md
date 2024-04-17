# Boot Loader 컴파일 및 설치



### fastboot

---

임베디드 시스템 ( *Target* )과 Ubuntu Linux ( *Host* )를 연결하여 Boot Loader, Kernel, File System을 정상 작성 ( *Flashing: Downloading + Writing* ) 하기 위해서는 각 시스템에 fastboot 소프트웨어가 필요하다. 해당 소프트웨어가 매개체가 되어 둘 사이의 통신을 담당한다.

#### Target

임베디드 시스템이 부팅되기 전, 즉 Kernel이 적재되지 않아 하드웨어를 제어할 수 있는 *monitoring mode*에 진입한다. 해당 모드에서 `fastboot` 명령어를 입력하면 디스크의 *partition* 정보가 출력된다. 내용은 다음과 같다.

![Alt text](<image/Screenshot from 2024-04-11 09-37-00.png>)

* *partition 0* ~ *partition 3* 은 low level partition으로 **Boot Loader**가 적재되는 공간이다.
* *partition 4* 에는 **Kernel**이 적재된다.
* *partition 6* ~ *partition 9* 은 noramal partition으로 **File System**으로 사용되는 공간이다 
* *partition 5* 'ramdisk' 또한 file system에 해당하나 잘 사용하지 않는 영역이다.
  
#### Host

Host 쪽 fastboot 또한 확인하여 본다. 설치 되어있지 않을 경우 `sudo apt install fastboot`를 통해 설치하여 준다.

![Alt text](<image/Screenshot from 2024-04-11 09-55-42.png>)

명령어 `sudo fastboot devices` 현재 연결된 장치간 물리적 연결 상태를 확인하여 준다.

![Alt text](<image/Screenshot from 2024-04-11 10-02-59.png>)

### Flashing Kenerl and File System

---

`~/class/embedded_linux/work/` 디렉토리 밑에 `fusing` 이라는 디렉토리를 생성하여 Boot Loader, Kernel, File System의 이미지를 이곳에 적재한다.

![Alt text](<image/Screenshot from 2024-04-11 10-07-22.png>)

* **Boot Loader** : 앞서 Target의 partition에 적재되는 Boot Loader 영역은 0번부터 3번까지 총 4개의 영역이 존재함을 확인하였다. 이 4개의 영역에는 칩 제조사에서 제공하는 Boot Loader 2개와 우리가 직접 compile 하는 Boot Loader가 2개 있다. 이 중 `u-boot.bin`과 `bl2.bin`은 우리가 직접 compile 하는 Boot Loader 이미지에 해당한다.
* **Kernel** : `zImage`가 커널이다.
* **File System** : `rootfs`가 파일 시스템이다.

이 상태에서 우선 Kernel을 flash 해본다. Target과 Host 간의 usb 연결 상태를 확인하고 Host에서 `fastboot flash kernel zImage` 명령어를 통해 flash를 진행한다.

![Alt text](<image/Screenshot from 2024-04-11 10-13-36.png>)

Target의 콘솔창도 확인하여 본다.

![Alt text](<image/Screenshot from 2024-04-11 10-13-43.png>)

File System flash도 동일한 방식으로 진행하여 본다. Host 컴퓨터에서 `fastboot flash system rootfs_ext4.img` 명령어를 입력한다.

![Alt text](<./image/Screenshot from 2024-04-11 10-15-48.png>)

Target의 콘솔창도 확인하여 본다.

![Alt text](<./image/Screenshot from 2024-04-11 10-19-15.png>)

`fdisk -p 0` 명령어를 통해 디스크의 파티션 정보를 확인할 수 있다. 1번 ~ 4번 파티션으로 구분되어 있다.

![Alt text](<./image/Screenshot from 2024-04-11 10-46-00.png>)


### Boot Loader 컴파일 준비

---

작업을 위해 bootloader 디렉토리를 생성하여 준다.

![Alt text](<./image/Screenshot from 2024-04-11 10-53-10.png>)

컴파일 대상을 확인하기 위해 `~/class/embedded_linux/HSmart4412TKU_v201505/Development/Source/bootloader` 에 위치한 bootloader의 압축 파일을 확인한다. 이 중 uboot로 시작하는 압축 파일이 직접 compile 해야 하는 Boot Loader의 소스 코드를 담고 있는 파일이다.

![Alt text](<./image/Screenshot from 2024-04-11 10-54-41.png>)

해당 압축 파일이 목적 디렉토리를 생성하는 지 확인하고 압축 해제 명령어를 입력하여 `/class/embedded_linux/work/bootloader` 위치에 압축을 해제한다.

![Alt text](<./image/Screenshot from 2024-04-11 10-59-06.png>)

압축 해제를 완료하면 `uboot_4412` 이름의 디렉토리가 생성이 되는데 해당 디렉토리를 살펴보면 `arch/arm/cpu` 하위에 cpu에 의존적인 폴더와 파일들을 확인할 수 있다.

![Alt text](<./image/Screenshot from 2024-04-11 11-02-07.png>)

리눅스에서 소스코드를 컴파일할 때 `make` 명령어를 이용한다. 이는 소스코드 파일 하나하나 컴파일하는 대신에 서로 의존적인 코드들을 연쇄적으로 한 번에 컴파일해 주는 도구이다. 일반적으로 make 명령어를 이용한 소스코드의 컴파일 과정은 다음과 같다.

### Make를 이용한 Boot Loader 실제 컴파일 과정

---

* **Makefile 준비(config.mk 수정)** : 직접 Makefile을 구성하는 대신, 대부분의 경우 이를 직접 제공해준다. 단, 구체적인 세부 설정을 위해서는 Makefile을 직접 수정하지 않고 `config.mk` 파일의 내용을 수정한다.
* **make distclean** : make distclean 명령은 일반적으로 빌드 프로세스에서 생성된 모든 파일을 제거하여 프로젝트를 초기 상태로 되돌리는 데 사용된다. 이는 컴파일된 바이너리, 오브젝트 파일, 라이브러리 등을 제거하여 깨끗한 빌드 환경을 유지하고자 하는 의도의 명령이다.
* **make smdk4412_config** : 앞서 설정한 파일에 맞게 빌드를 구성하는 명령이다. "smdk4412" 라는 보드에 맞게 빌드를 구성하는 명령으로 해당 보드에 맞는 장치 드라이버, 커널 구성 및 기타 시스템 설정을 선택한다.
* **make** : 최종적으로 소스코드를 실제 컴파일하는 명령에 해당한다.

다음과 같이 `config.mk` 파일 중 크로스 컴파일러의 위치를 지정하는 변수를 실제 컴파일러가 위치한 주소로 변경하여 준다.

![Alt text](<./image/Screenshot from 2024-04-11 11-18-17.png>)

다음으로 `make clean` 또는 `make distclean`을 입력하는데 make 명령어를 찾을 수 없다는 에러 메시지가 출력되면 make 관련 패키지를 설치하여 준다.

 ![Alt text](<./image/Screenshot from 2024-04-11 11-25-29.png>)

 ![Alt text](<./image/Screenshot from 2024-04-11 11-26-08.png>)

 이후 앞서 설정한 파일에 맞게 빌드를 구성하기 위해 `make smdk4412_config` 명령어를 입력해 준다.

![Alt text](<./image/Screenshot from 2024-04-11 11-32-49.png>)

해당 명령어로 인하여 include 파일에 필요한 헤더 파일들이 새롭게 포함된 모습을 확인할 수 있다.

![Alt text](<./image/Screenshot from 2024-04-11 11-34-47.png>)

최종적으로 `make` 명령어를 실행한다.

![Alt text](<./image/Screenshot from 2024-04-11 11-38-27.png>)

해당 명령어를 실행하고 나면 Target에 적재할 Boot Loader인 `bl2.bin`과 `u-boot.bin`이 생성된 것을 확인할 수 있다.

![Alt text](<./image/Screenshot from 2024-04-11 11-43-29.png>)

![Alt text](<./image/Screenshot from 2024-04-11 11-43-35.png>)