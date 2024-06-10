# Kernel Compile

### Linux Kernel

---

#### 정리

이전 시간까지 부트로더를 직접 컴파일하고 이를 디스크에 굽는 과정을 거쳤다면 이번에는 kernel을 직접 컴파일하고 system call을 일정 부분 수정하여 flashing 하는 과정을 거치고자 한다.

#### Vanila Kernel

`https://www.kernel.org/` 은 리눅스 커널을 배포하는 메인 사이트이다. 리눅스 커널 아카이브라고도 하며 현재 6.8.8 버전까지 배포되어 있다. 이곳에서 내려받는 커널을 `vanila kernel`이라고도 한다. 즉, 아무것도 없는 깨끗한 커널을 의미한다. 임베디드 리눅스의 경우 이러한 바닐라 커널을 임베디드 시스템에 맞게 `cpu fetch`,`board fetch`등의 과정을 거치고 사용하는 커널이다. 우분투 리눅스에서 사용하는 커널을 확인하고 싶은 경우 `uname -a` 명령어를 입력하면 현재 사용 중인 커널의 버전 정보를 확인할 수 있다. 우분투 리눅스에서 커널을 `vmlinuz`로 지칭한다. 이를 지속적으로 업데이트하고 여러 종류의 커널을 관리하며 새로운 기능을 활용해 보는 것도 좋은 리눅스 공부 방법이다.

### Kernel Source

---

우선 중간고사 시험에서 삭제하였던 `work` 디렉토리를 다시 만들어 준다. 그리고 이후 그 안에 `kernel` 디렉토리를 생성해준다.

![alt text](<./image/Screenshot 2024-05-08 at 11.36.46 PM.png>)

해당 위치에 `4412tku_linux_kernel.tgz`에 해당하는 커널 소스의 압축 파일을 해제하여 준다.

![alt text](<./image/Screenshot 2024-05-08 at 11.40.57 PM.png>)

커널 소스의 버전을 확인하기 위해 `kernel_4412` 디렉토리 위치 내에서 `vi Makefile` 명령어를 입력한다.

![alt text](<./image/Screenshot 2024-05-08 at 11.53.23 PM.png>)

`arch/arm`라는 디렉토리에 들어가면 cpu 제조사 별로 fetch된 코드를 확인할 수 있다. `kernel` 디렉토리에 들어가면 커널의 메인 코드들을 확인할 수 있다. 이 외에도 커널의 메인 기능 4가지에 해당하는 `process management`, `memory management`, `network`, `file system` 와 관련된 코드들이 존재하는 것을 확인할 수 있다. `process mangement`와 `memory management`는 cpu에 종속적인 코드와 독립적인 코드로 나뉘어져 있다.

### Kernel Compiler 변경

---

우선 `vi Makefile`을 통해 makefile의 크로스 컴파일러 위치를 재지정 해준다. vi 에디터에서 찾기 명령어는 `/`이다. 따라서 크로스 컴파일러의 내용을 찾기 위해 명령 모드에서 `/CROSS`를 입력해 준다. 다음 것을 찾고 싶다면 `n`을 입력하여 이동한다.

![alt text](<./image/Screenshot from 2024-05-02 10-31-46.png>)

변경하고자 하는 위치를 발견하면 해당 내용의 디렉토리 위치를 `/opt/class/embedded_linux/toolchain/CodeSourcery/(이하동일)`으로 변경하여 준다. 줄 마지막에 공백이 포함되지 않도록 조심한다.

![alt text](<./image/Screenshot from 2024-05-02 10-34-03.png>)


### Kernel 초기화

---

다음과 같이 `ls -al` 명령어를 입력해보면 컴파일 되었던 흔적(`.obj` 등)을 확인할 수 있다. 따라서 정상적인 진행을 위해 kernel을 초기화 해준다.

![alt text](<./image/Screenshot from 2024-05-02 10-35-19.png>)

kernel 초기화는 컴파일 흔적과 더불어 스크립트 환경까지 초기화 해 주는 `make distclean`를 통해 진행해야 한다. `make clean`이 컴파일한 흔적을 정리해주는 명령어라면 `make mrproper`는 스크립트 환경을 정리해주는 명령어이다. 그리고 둘 모두 정리해 주는 명령어는 `make distclean`이다.

![alt text](<./image/Screenshot 2024-05-09 at 1.15.59 AM.png>)

![alt text](<./image/Screenshot 2024-05-09 at 1.18.31 AM.png>)


### make menuconfig와 .config 생성

---

kernel compile에 들어가기 위해서는 많은 소스코드 중 컴파일할 코드와 하지 않을 코드를 지정해줘야 한다. 이 설정을 kernel에서는 전통적으로 `make menuconfig` 명령어를 통해 설정한다.

![alt text](<./image/Screenshot 2024-05-09 at 1.25.13 AM.png>)

명령어를 실행해 보면 다음과 같이 `ncurses libraries`가 필요하다는 에러 메시지가 출력되고 이에 따라 `sudo apt install libncurses5-dev`를 입력하여 필요한 라이브러리를 다운 받아준다.

이후 다시 `make menuconfig` 명령어를 실행해 보면 다음과 같은 kernel configuration 창을 확인할 수 있다.

![alt text](<./image/Screenshot from 2024-05-02 10-44-10.png>)

필요한 설정을 마쳤다 가정하고 exit을 진행하면 다음과 같이 `.config` 내용을 저장하겠냐는 문구와 함께 종료 이후 `.config` 파일이 생성된 것을 확인할 수 있다.

![alt text](<./image/Screenshot from 2024-05-02 10-46-59.png>)

![alt text](<./image/Screenshot from 2024-05-02 10-47-14.png>)

다만 해당 `.config` 파일은 현재 시스템과 관련된 설정이 포함되어있지 않기 때문에 해당 파일로 컴파일을 진행할 수는 없다. 따라서 보통은 칩 제조사에서 소스코드와 함께 설정 파일을 함께 제공하며 이 파일이 여기서는 `h4412_linux_config`이다. 원래는 해당 파일을 `.config` 파일로 변경해주어야 하지만 `arch/arm/configs/` 디렉토리에 해당 파일을 복사해 주면 플랫폼에 맞게 `.config`가 자동으로 변경된다.

![alt text](<./image/Screenshot from 2024-05-02 10-51-20.png>)

![alt text](<./image/Screenshot 2024-05-09 at 1.52.34 AM.png>)

`hybus_smdk4412_linux_defconfig`가 생성된 것을 확인할 수 있다.

![alt text](<./image/Screenshot from 2024-05-02 10-54-57.png>)

해당 설정 파일을 읽어와서 `.config` 파일을 생성하는 방법은 `make 설정파일 이름`이 된다.

![alt text](<./image/Screenshot from 2024-05-02 10-57-10.png>)

![alt text](<./image/Screenshot from 2024-05-02 11-00-05.png>)

`diff` 명령어를 통해 두 파일이 같은 파일임을 확인할 수 있다.

![alt text](<./image/Screenshot from 2024-05-02 11-00-42.png>)


### 컴파일 진행

---

make menuconfig를 통해 `.config`파일 생성과 수정을 마쳤다면 `make zImage` 명령어를 통해 컴파일을 진행한다.

진행 도중 `kernel/timeconst.pl` 파일의 373번 라인 위치에서 에러가 발생했다는 메시지를 확인할 수 있다.

![alt text](<./image/Screenshot 2024-05-09 at 2.23.11 AM.png>)

`kernel` 디렉토리의 위치에서 vi 에디터를 통해 `timeconst.pl` 파일을 수정한다. 에러 메시지에 따라 373라인으로 이동한다. `:373` 명령어를 입력하면 373라인으로 바로 이동한다. 참고로 `:1`이면 첫 줄로 이동하고 `:$`이면 마지막 줄로 이동한다. 

![alt text](<./image/Screenshot from 2024-05-02 11-22-38.png>)

에러 메시지에 따라 defined를 사용하지 않고 코드를 수정해 준다. 원래 거는 코멘트로 바꿔준다.

컴파일 중간에 문제가 발생했기 때문에 `make clean`을 통해 컴파일한 오브젝트 파일들만 정리해준다. 그리고 다시 `make zImage` 명령어로 컴파일을 진행한다.

![alt text](<./image/Screenshot from 2024-05-02 11-24-20.png>)

다음과 같이 `arch/arm/boot` 위치에 `zImage`가 생성이 되었으면 정상적으로 컴파일이 된 것이다.

![alt text](<./image/Screenshot from 2024-05-02 11-31-01.png>)


### fastboot를 통한 compile 테스트

---

target과 host 모두에서 fastboot를 정상 연결하고 `fastboot flash kernel zImage`를 통해서 새롭게 컴파일한 kernel을 flashing한다.

![alt text](<./image/Screenshot from 2024-05-02 11-32-54.png>)

다음과 같이 프롬프트에 정상적으로 flash된 것을 확인한다.

![alt text](<./image/Screenshot from 2024-05-02 11-33-02.png>)

