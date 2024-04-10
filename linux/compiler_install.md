# Cross compiler


### 임베디드 리눅스 개발을 위한 Compiler

---

#### gcc

gcc는 native compiler이기도 하지만 동시에 다양한 CPU architecure에 대한 cross compile을 지원하기도 한다. 하지만 해당 architecture 규격에 맞는 프로그램을 만들기 위해서는 gnueabi와 armeabi 등의 규격을 따르는 코드가 필요하다.


#### gnueabi 와 armnoneabi

현재 가동하고자 하는 임베디드 시스템을 위한 소프트웨어(boot loader, kernel, file system)을 크로스 컴파일하기 위해서는 gcc 외에도 **인터페이스 규격에 대한 정보를 제공하는 eabi**가 필요하다. 지난 시간까지 gcc와 gnueabi를 설치하였고 이번 시간에는 armnoneabi를 설치해 보고자 한다.


### armnoneabi 설치 과정

---

#### arm-2010q1-188-arm-none-eabi.bin.gz 압축 해제

파일을 원하는 위치로 이동시킨 후 `gzip -d arm-2010q1-188-arm-none-eabi.bin.gz` 명령어를 통해 해당 파일의 압축을 해제한다. 압축 해제를 완료했으면 `.profile` 파일을 열어서 명령어를 사용할 수 있도록 환경변수를 설정해 준다.

#### hello.c 컴파일 과정상 발생하는 문제
설치와 환경변수 세팅을 모두 마쳤으면 `hello.c` 파일을 `arm-none-eabi-gcc -o hello_arm hello.c` 명령어를 통해 cross compile 시도한다. 하지만 해당 명령어를 실행해 보면 정상적으로 컴파일되지 않고 절차상 몇 가지 문제를 마주하게 된다.

##### 1. Bash Shell 문제

`ls l /bin/sh` 명령어를 통해 현재 실제 bash shell이 가리키는 프로그램을 확인할 수 있다. `/bin/sh -> dash*` 과 같은 결과를 보인다면 dash가 아닌 bash 프로그램을 가리키도록 전환해야 한다.

##### 2. /lib/libc.so.6 파일이 없다는 오류

`ln -s /lib64/libc.so.6 /lib/libc.so.6` 입력을 통해 링크 파일을 생성하여 원본 파일과 연결된 가상의 파일을 생성해 준다.

##### 3. libxext.so.6 파일이 없다는 오류

다시 hello.c의 컴파일을 시도해 보면 libxext.so.6의 파일이 없다는 오류가 발생한다. `apt-file search libxext.so.6` 명령어를 통해 32bit 용 `libxext6:i386` 을 다운 받는다. 

> ✅ **apt-file search < 파일명 >**
특정 파일이 없다는 오류가 발생했을 때 해당 명령어를 사용하면 찾고자 하는 파일이 포함된 패키지 목록을 출력해 준다. 이를 통해 원하는 패키지를 다운 받을 수 있다. apt-file 패키지의 설치 명령은 `sudo apt-get -y install apt-file` 이다.


### gcc cross compiler를 2개 설치한 이유

---

#### Naming Convention: arch-vendor-(os)-abi

gcc cross compiler의 이름에는 명명 규칙이 존재한다. `아키텍쳐-벤더(최종 공급자)-운영체제-abi`형식으로 이름을 짓는다. 앞으로 사용할 컴파일러 또한 `arm-none-eabi`와 `arm-linux-gnueabi` 두 가지 종류의 컴파일러를 사용하게 되는데 이는 최종 개발할 BSP의 동작 방식과 관련이 있다.

#### arm-none-eabi vs arm-linux-gnueabi

Boot Loader와 Kernel은 운영체제와 상관없이 동작하는 raw binary code인 반면에 File System은 리눅스라는 운영체제 위에서 동작하는 프로그램이다. 따라서 Boot Loader와 Kernel은 `arm-none-eabi`를 이용하여 컴파일하고 File System은 `arm-linux-gnueabi`를 사용하여 컴파일한다.


### Ubuntu Linux와 임베디드 키트 연결

---

#### Connection 종류

앞으로 Host(Ubuntu Linux)와 Target(Embedded Kit)를 연결하기 위해 총 3가지 종류의 Connection을 맺게 된다. 첫 번째는 **Serial 연결**, 두 번째는 **USB 연결**, 마지막으로 **Ethernet 연결**이다.


#### Serial 연결

해당 연결을 진행하면 임베디드 키트의 상태를 Host에서 console 화면을 통해 확인할 수 있다. 이를 위해서 minicom 패키지를 다운 받아야 한다. 필요한 설정을 마치고 나면 이를 활용하여 Kernel이 적재되기 직전의 상황인 부트로더에 진입할 수 있다. 단 부팅될 때 3초 안에 엔터를 눌러야 하고 종료할 때도 `ctrl-AX`를 입력해야 종료할 수 있다.
