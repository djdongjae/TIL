# Device Driver

## 1. 개요

부트로더, 커널, 파일 시스템까지 구축하고 나면 시스템이 주변 하드웨어 장치들과 원할하게 통신하기 위해 디바이스 드라이버를 구축해야 한다. 리눅스에서는 디바이스를 기본적으로 파일로 본다. /dev에 해당하는 위치에서 디바이스 관련 파일들을 확인할 수 있다. 리눅스에서는 디바이스 드라이버를 캐릭터, 블록, 네트워크 세 종류로 나누어 관리하고 있지만 블록과 네트워크는 특정 상황이나 장치에 특화된 드라이버이기 때문에 실제적으로는 캐릭터로 관리한다고 생각하면 된다.

<br>

결국 메이저 번호랑 파일 오퍼레이션을 드라이버에게 인식시켜 주어 드라이버의 내용을 커널이 인식하게끔 만들어야 한다.

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