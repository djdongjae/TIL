# Boot Loader 설치

### 디스크 포맷

---

임베디드 시스템에서 보조기억장치 역할을 하는 sd card를 카드 리더기에 장착하여 이를 Host 컴퓨터와 연결하여 본다. `fdisk -l` 명령어를 통해 확인한 구조는 다음과 같다.

![Alt text](<./image/Screenshot from 2024-04-18 09-24-59.png>)

위와 같이 총 4개의 파티션으로 분리되어있는 모습을 확인할 수 있다. `sdc2` start 주소 136620 번지부터 시작해 `sdc1`의 end 위치인 15377339 번지까지 공간을 차지하고 있는 것을 확인할 수 있다.

디스크를 안전하게 포맷하기 위해서는 usb의 마운트를 해제하고서 포맷을 진행해야 한다. `df` 명령어를 통해 마운트 되어있는 파일시스템을 다음과 같이 확인하고, `sdc`와 관련하여 마운트 되어있는 파일시스템을 전부 마운트 해제 시켜준다. 명령어는 `umount` 이다.

![Alt text](<./image/Screenshot from 2024-04-18 09-35-48.png>)

마운트 해제를 완료한 이후 다음과 같이 포맷 명령어를 입력한다. 명령어는 `sudo dd if=/dev/zero of=/dev/sdc bs=512 count=136619`  이다.

![Alt text](<./image/Screenshot from 2024-04-18 09-50-52.png>)

해당 명령어는 디스크의 내용을 모두 제거하고 모든 데이터를 0으로 덮어쓰는 명령어로 /dev/zero 디바이스에서 읽어온 0 값을 /dev/sdc 디바이스에 512바이트 블록 단위로 총 136619번 쓰기를 수행한다는 의미이다. 수행을 완료하면 `/dev/sdc` 위치의 값이 이전과 다르게 파티션이 분배되지 않은 모습을 확인할 수 있다.

![Alt text](<./image/Screenshot from 2024-04-18 09-51-50.png>)


### 디스크 굽기

---

포맷을 완료했으면 이전 시간에 직접 컴파일한 두 개의 부트로더 binary file과 칩 제조회사에서 제공해주는 부트로더 binary file을 담고 있는 압축 파일, 그리고 해당 데이터를 일괄적으로 구워주는 shell script 파일인 `sd_fusing_4412.sh`를 작업 폴더 위치로 복사하여 준다.

![Alt text](<./image/Screenshot from 2024-04-18 10-02-37.png>)

그리고 칩 제조사에서 제공해 주는 압축 파일을 압축 해제 한다.

![Alt text](<./image/Screenshot from 2024-04-18 10-03-17.png>)

`vi sd_fusing_4412.sh` 명령어를 통해 현재 파일 위치에 맞게 binary file들의 경로를 수정한다.

모든 설정이 끝났으면 다음과 같이 쉘 파일을 실행하여 4개의 부트로더 binary file을 디스크에 구워준다.

![Alt text](<./image/Screenshot from 2024-04-18 10-15-59.png>)

### 디스크 장착 후 테스트

---

#### 순서

1. sd card를 다시 target 기기에 장착한다.
2. 전원을 연결하고 먼저 serial을 연결해 준다.
3. 새로운 터미널 창을 열고 `sudo minicom hssamrt4412tku`를 입력하여 콘솔창에 들어간다.
4. target의 전원을 켜주고 monitoring mode에 진입한다.
5. `fastboot`를 입력한다.
6. Host에서도 `fastboot manage devices`를 입력한다.
7. Target에서 `fdisk -c 0` 명령어를 통해 디스크 시스템을 새롭게 구성해 준다.
8. `~/class/embedded_linux/work/fusing` 디렉토리 위치에서 커널과 파일 시스템을 flash 한다.
   1. `sudo fastboot flash kernel zImage`
   2. `sudo fastboot flash system rootfs_ext4.image`

해당 상태에서 Boot Loader가 정상적으로 compile 되고 구워졌는지 확인하기 위해 `hello.c` 파일을 작성하여 hello 명령어를 생성하여 테스트해 본다.

`arch/arm/cpu/armv7` 위치의 `start.S`에서부터 부팅이 시작되어 `board`를 지나 최종적으로 `common`의 `main.c`에 도달하게 된다. `common` 폴더에는 또한 다양한 종류의 명령어를 지시하는 소스코드가 담겨있다. 이 중에서 가장 간단한 version 명령어 소스코드를 복사하여 hello 명령어를 생성해본다.

`cp cmd_version.c cmd_hello.c`를 입력한다. 그리고 알맞게 내용을 수정하여 본다.

cmd_hello.c
```C
#include <common.h> 
#include <command.h>

int do_hello(void)
{
    printf("\nHello World\n");
    return 0;
}

U_BOOT_CMD( 
    hello, 1, 1, do_hello,
    "Test command for hello",
    ""
);
```

다시 컴파일할 때는 `make clean`을 통해 obj 파일을 전부 정리해 준다. 원래 최상위 디렉토리 안에 Makefile이 존재하지만 이와 같이 서브 디렉토리가 많은 경우에는 해당 디렉토리 안에 또 다른 Makefile이 있다.

`common/Makefile`를 vi 명령을 통해 열어둔 후, `#others` 위치에 `COBJS-y += cmd hello.o`를 추가한다. 이는 방금 작성한 `cmd_hello.c` 파일을 컴파일러에게 컴파일하라고 지시하는 Makefile의 코드이다.

![Alt text](<./image/Screenshot from 2024-04-18 11-13-26.png>)

최상위 디렉토리로 돌아와서 `make clean` 그리고 `make` 명령어를 통해 다시 컴파일 해준다.

![Alt text](<./image/Screenshot from 2024-04-18 11-14-47.png>)

정상적으로 컴파일 되었으면 다음과 같이 `u-boot.bin`과 `bl2.bin`이 생성된 것을 확인할 수 있다.

![Alt text](<./image/Screenshot from 2024-04-18 11-15-05.png>)

![Alt text](<./image/Screenshot from 2024-04-18 11-15-10.png>)

이전과 달리 이미 target에는 부트로더가 설치되어있기 때문에 sd card로 다시 구워줄 필요 없이 바로 flash하여 자기 자신 또한 fusing 할 수 있다. 다음 명령어를 통해 새롭게 flash 해본다.

1. `sudo fastboot flash bl2 bl2.bin`
2. `sudo fastboot flash bootloader u-boot.bin`

![Alt text](<./image/Screenshot from 2024-04-18 11-26-58.png>)

이제 target의 콘솔에서 hello 명령어가 정상 등록되었는지 확인하여 본다.

![Alt text](<./image/Screenshot from 2024-04-18 11-27-50.png>)

![Alt text](<./image/Screenshot from 2024-04-18 11-28-15.png>)
