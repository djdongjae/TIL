# Device Driver

## 1. 개요

<br>

부트로더, 커널, 파일 시스템까지 구축하고 나면 시스템이 주변 하드웨어 장치들과 원할하게 통신하기 위해 디바이스 드라이버를 구축해야 한다. 리눅스에서는 디바이스를 기본적으로 파일로 본다. /dev에 해당하는 위치에서 디바이스 관련 파일들을 확인할 수 있다. 리눅스에서는 디바이스 드라이버를 캐릭터, 블록, 네트워크 세 종류로 나누어 관리하고 있지만 블록과 네트워크는 특정 상황이나 장치에 특화된 드라이버이기 때문에 실제적으로는 캐릭터로 관리한다고 생각하면 된다.

<br>

결국 메이저 번호랑 파일 오퍼레이션을 드라이버에게 인식시켜 주어 드라이버의 내용을 커널이 인식하게끔 만들어야 한다.

<br>

## 2. 실습

### 2.1 소스코드 다운

---

먼저 다음 위치에 `drivers` 라고 하는 디렉토리를 하나 생성한다.

```shell
mkdir ~/class/embedded_linux/work/drivers
```

<br>

그리고 다운로드 받은 스켈레톤 코드를 해당 디렉토리에 압축 해제한다.

<br>

![alt text](<./image/Screenshot from 2024-05-30 10-08-08.png>)

<br>

`skeleton.c` 코드를 살펴보면 `skeleton_` 로 시작하는 각종 함수들을 확인할 수 있다.

<br>

![alt text](<./image/Screenshot from 2024-05-30 11-03-21.png>)

<br>

### 2.2 Device Driver 컴파일 후 등록

---

우선 컴파일을 해야 하기 때문에 `Makefile` 에서 컴파일러의 위치와 종류, 그리고 커널의 위치를 재지정해 주어야 한다.

<br>

![alt text](<./image/Screenshot from 2024-05-30 11-17-44.png>)

<br>

다 수정했으면 `make` 명령어를 통해 컴파일해준다.

<br>

![alt text](<./image/Screenshot from 2024-05-30 11-18-31.png>)

<br>

NFS를 사용중이기 때문에 Target에서도 컴파일된 결과물들을 확인할 수 있다.

<br>

![alt text](<./image/Screenshot from 2024-05-30 11-21-09.png>)

<br>

현재 시스템에 올라와 있는 디바이스 정보를 보는 명령어는 다음과 같다.

```shell
cat /proc/devices
```

<br>

현재 올라와 있는 모듈을 보고 싶다면 `lsmod` 를 입력한다.

<br>

![alt text](<./image/Screenshot from 2024-05-30 11-23-01.png>)

<br>

하나 밖에 올라와 있지 않기 때문에 모듈을 올려준다. 명령어는 `insmod skeleton.ko`

<br>

![alt text](<./image/Screenshot from 2024-05-30 11-25-19.png>)

<br>

skeleton_init 함수가 호출되고 메이저 번호로 248번이 할당된 것을 확인할 수 있다. `cat /proc/devices` 명령어로 다시 확인한다.

<br>

![alt text](<./image/Screenshot from 2024-05-30 11-30-11.png>)

<br>

이제 커널은 아는 것이다. "아 248번에 스켈레톤이라는 드라이버가 올라갔구나". 이제 어플리케이션이 이 함수들을 사용하기 위해 디바이스 파일을 만들면 된다.

<br>

/dev 위치에 드라이버 파일 하나를 만들어 준다. 명령어는 다음과 같다.

```shell
mknod skeleton 248
mknod skeleton c 248 0
```

<br>

![alt text](<./image/Screenshot from 2024-05-30 11-31-43.png>)

<br>

![alt text](<./image/Screenshot from 2024-05-30 11-33-25.png>)

<br>

어플리케이션 코드를 열어본다. `vi skeleton_test.c`

<br>

![alt text](<./image/Screenshot from 2024-05-30 11-34-15.png>)

<br>

Target에서 `skeleton_test`를 실행해 보면 다음과 같이 잘 실행되는 것을 확인할 수 있다.

<br>

![alt text](<./image/Screenshot from 2024-05-30 11-36-06.png>)

<br>

## 3. Device Driver 동작 구조

<br>

리눅스에서는 기본적으로 모든 주변 장치를 파일로 다룬다. 따라서 특정 하드웨어에 접근하기 위해서는 파일 시스템 매니지먼트를 통해야만 한다. 어플리케이션에서 함수를 호출하고 소프트웨어 인터럽트를 통해 시스템 콜 인터페이스까지 연결되는 과정이 지난 시간까지 진행했던 내용이다.

<br>

다음으로 파일 시스템 매니지먼트까지 접근을 완료했으면 **디바이스 인터페이스**를 통해 디바이스에 실제적으로 접근해야 한다. 즉 커널이 아래에 위치하는 하드웨어의 존재를 알아야 한다는 것이다. 해당 과정은 커널에 등록되어있는 메이저 번호를 통해 이루어진다. 한편, 디바이스 드라이버를 모듈 방식으로 코딩하기 때문에 디바이스 드라이버를 로딩할 때는 인스 모드라는 명령어를 통해 호출한다. 물론 커널에 직접 빌트인 하여 디바이스 드라이버를 컴파일할 수도 있다.

<br>

### 3.1 skeleton.c 다시 살펴 보기

---

<br>

`skeleton.c`의 소스코드를 살펴보면 `insmod` 이후 모듈이 로드될 때 가장 먼저 호출되는 `module_init()` 함수를 코드 최하단에서 볼 수 있다. `register_character_dev()`의 경우 디바이스 드라이버를 구동하는 가장 중요한 함수인데 인자로 메이저 번호, 이름, 파일 오퍼레이션을 받는다. 해당 코드에서는 메이저 번호를 0번으로 주고 있는데 이는 지정된 숫자가 아닌 무작위로 비어있는 번호를 할당하는 다이나믹 할당임을 의미한다.

<br>

## 4. 실제 주변 장치(Dot Metrix, Seven Segements 등)과 연결하기

<br>

키트를 확인해 보면 다양한 주변 장치들이 있는 것을 확인할 수 있다. 이 주변 장치들과 CPU는 개별로 직접 연결되기 보다는 중간에 FPGA를 하나 두어 망 형태로 구축되어있다.

<br>

### 4.1 FND 동작 구조

---

<br>

먼저 제공받은 `fnd-202401.tar.gz` 파일을 적당한 위치의 폴더에 압축해제 한다.

<br>

```shell
tar zxvf /home/dongjaeoh/Downloads/fnd-202401.tar.gz
```

<br>

![alt text](<./image/Screenshot from 2024-06-13 09-58-54.png>)

<br>

제공 받은 파일 중 `fnd.c` 코드를 살펴보면 fnd의 동작 구조를 확인할 수 있다.

<br>

![alt text](<./image/Screenshot from 2024-06-13 09-59-34.png>)

<br>

우측 상단 세로 막대기부터 시계 방향으로 돌며 총 8비트의 LED 표시를 할 수 있다. 즉 `0011 1111` 이면 0을 표시하고, `1100 0000`이면 1을 표시한다. 다만 값을 저장할 때는 값을 전부 뒤집어서 저장을 해준다. 0을 저장하고자 한다면 `1100 0000` 이고 이를 16진수로 표현하면 `0xC0`이다. 1의 경우 `0xF9`가 된다.

<br>

해당 디렉토리에는 `Makefile`이 없기 때문에 기존에 `skeleton` 디렉토리에서 이를 복사해서 내용을 변경한다.

```shell
 cp ../skeleton/Makefile .
```

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-02-00.png>)

<br>

다음과 같이 `skeleton`이라고 적힌 부분들을 전부 `fnd`로 변경 해 준다. 이를 한 번에 변경하는 vi 명령어는 다음과 같다.

```vi
 :1,$s/skeleton/fnd/g
```
> ✅ 1번 행부터 $행(끝)까지 s(substitution), 즉 교체를 하겠다. `skeleton`을 `fnd`로. g(global)하게

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-06-22.png>)

<br>

`Makefile`을 수정했으니까 `make` 명령어로 컴파일 한다.

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-06-34.png>)

<br>

콘솔 및 네트워크를 연결하고 NFS를 통해서 컴파일된 파일을 로드한다. 위치는 `work/drivers/fnd`

<br>

현재는 `cat /proc/devices` 를 통해 등록된 디바이스 드라이버 모듈을 확인해 보면, 231번에 이미 커널에 빌트인된 fnd가 등록된 것을 확인할 수 있다. 따라서 현재 정적으로 번호를 할당 중인 상태에서 231번에는 번호를 할당할 수 없기 때문에 기존의 커널을 갈아 엎고 새롭게 해당 fnd를 `insmod` 해서 우리의 모듈이 231번을 사용할 수 있도록 해야 한다.

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-13-47.png>)

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-14-21.png>)

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-15-53.png>)

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-16-58.png>)

<br>

### 4.2 커널에 빌트인 된 fnd 제거하고 다시 컴파일

---

<br>

기존 커널의 내용을 수정해야 한다. 먼저 기존 커널의 내용을 수정할 수 있도록 `work/kernel/kernel4412`에 해당하는 위치에서 `make menuconfig`를 입력한다.

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-41-46.png>)

<br>

`Device Drivers`에 진입하면 `Exynos4412TKU_IEB`에 해당하는 즉 보드 관련 컴파일 옵션을 수정할 수 있는 항목이 존재한다.

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-42-54.png>)

<br>

해당 항목에 진입하면 fnd 관련 모듈을 빌트인 형식으로 컴파일 할 것인지, 모듈 형식으로 컴파일 할 것인지, 아예 컴파일 하지 않을 것인지를 선택할 수 있다.

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-42-59.png>)

<br>

해당 코드를 바로 수정하기에는 원본이 손상되므로 커널 코드를 복사해서 설정을 다시 고쳐보자. 먼저 `work/kernel/` 위치에 `kernel_4412` 디렉토리 복사본을 생성해 준다.

<br>

```shell
cp -r kernel_4412 kernel_4412-wofnd
```

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-44-58.png>)

<br>

그리고 해당 디렉토리로 진입하여 다시 한 번 `make menuconfig` 를 입력한다.

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-46-39.png>)

<br>

아까와 동일한 경로로 진입하여 `IEB fnd support` 부분의 모듈 빌트인 컴파일 항목을 해제해 준다. 뺄 때는 스페이스바로 움직인다.

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-47-20.png>)

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-48-10.png>)

<br>

설정이 변경되었으니 `make clean`을 통해 기존의 컴파일 기록을 정리하고 `make zImage`를 통해 커널을 컴파일한다.

<br>

![alt text](<./image/Screenshot from 2024-06-13 10-50-41.png>)

<br>

커널이 정상적으로 컴파일 되었으면 fastboot를 연결하고 `work/kernel/kernel_4412/arch/arm/boot` 위치에 있는 `zImage`를 Target 컴퓨터로 flash 해준다.

<br>

```shell
 fastboot flash kernel zImage
```

<br>

### 4.3 이제 진짜로!!!

---

이제 정말로 다시 모듈을 로드해 본다. 우선 현재 커널에 올라가 있는 모듈을 확인한다. `lsmod`

<br>

아무것도 없기 때문에 현재 모듈이 올라가 있지 않은 것을 확인할 수 있다. 다음 명령어를 통해 231번이 비어있는지 다시 확인한다.

<br>

```shell
cat /proc/devices
```

<br>

확인을 마쳤으면 이제 다시 NFS를 연결하고 컴파일된 파일을 로드한다. 위치는 `work/drivers/fnd`

<br>

```shel
 insmod fnd.ko
```

<br>

다시 한번 커널에 등록된 디바이스 드라이버를 확인해 보면 231번에 fnd가 정상적으로 등록된 것을 확인할 수 있다.

<br>

![alt text](<./image/Screenshot from 2024-06-13 11-08-37.png>)

<br>

이제 어플리케이션이 디바이스 드라이버를 인식할 수 있도록 드라이버 파일을 생성해야 한다. 기존에 존재하는 fnd 파일을 삭제하고 새롭게 만든다. `/dev/` 위치에서 fnd 파일을 삭제하고 새롭게 생성한다.

<br>

```shell
 rm -rf fnd
 mknod /dev/fnd c 231 0
```

<br>

![alt text](<./image/Screenshot from 2024-06-13 11-20-23.png>)

<br>

해당 위치에서 `fnd_test.c` 를 실행하고 확인한다. 

<br>

![alt text](<./image/Screenshot from 2024-06-13 11-16-25.png>)

<br>