# System Call

### 1. 개요

---

#### 1.1 커널의 역할

커널의 대표적인 기능은 다음과 같다.

* 프로세스 관리
* 메모리 관리
* 네트워크 관리
* 파일 시스템 관리

커널은 위의 기능을 이용하여 유저에게 서비스를 제공하는 역할을 하는데, 이 과정에서 인터페이스를 통해 커널에 접근할 수 있도록 하는 표준화된 포맷이 존재한다. 이 포맷을 `System Call Interface`라고 한다. 한편 커널보다 더 깊이, 즉 하드웨어에 접근하고자 한다면 `Device Driver Interface`를 통해 접근해야 한다. 해당 과제의 목표는 새로운 서비스를 커널에 등록하고 시스템 콜 인터페이스를 조작하여 애플리케이션에서 이를 사용할 수 있도록 하는 것이다.

#### 1.2 인터럽트

우리가 마우스를 움직이거나 키보드에 키를 입력하면 CPU에 인터럽트가 발생한다. 마이크로 프로세서는 본인이 하던 일을 잠시 멈추고 인터럽트 핸들러에 들어가 어떤 인터럽트가 발생했는지 해석하는 과정을 거친 후 다시 본래 하던 일로 돌아간다. 이와 같이 프로세서는 인터럽트를 처리하기 위해 `User Mode`와 `Privileged Mode` 두 가지가 모드를 지닌다. `User Mode`의 경우 일반적으로 유저 프로그램이 사용되는 모드이고 `Privileged Mode`는 입출력과 같이 특수한 이벤트를 처리할 때 사용되는 모드이다.

> ✅ ARM에서 `User Mode`에서 `Privileged Mode`로 전환되기 위해 필요한 인자를 `Exception`이라고 한다. Java에서와 같은 예외가 아닌 **하드웨어적인 예외**이다. 대표적인 예시로 `irq(interupt request)`, `fiq(fast interrupt)`, `undefined(인스트럭션 해석 불가 예외)`등이 있다.

위에서 제시한 인터럽트의 특징은 **예상치 못한 인터럽트**라는 점이다. 이와 달리 **소프트웨어 인터럽트**는 유저가 의도한 바에 따라 예외를 발생시켜 인터럽트를 유도할 수 있는 인터럽트이다. 이와 같은 인터럽트는 대부분 하드웨어적인 I/O를 처리하고자 하는 목적에 의해 유도된다.

커널에서의 **System Call**은 이러한 방식으로 구현되었다. 마이크로 프로세서의 소프트웨어 인터럽트를 활용하여 커널의 서비스를 이용하고 싶을 때 우리는 **System Call Interface**를 통해 이용하게 되는 것이다. 자주 사용하는 System Call에는 파일 시스템 서비스를 활용하는 `Read`, `Write`, `Open`, `Close` 함수 등이 있다.

### 2. System Call 구현

---

#### 2.1 System Call의 동작 구조

`fread()` 함수는 기본적으로 POSIX(Portable Operating System Interface) 라이브러리에 등록되어있다. POSIX 라이브러리는 운영체제 간 호환성을 고려하기 때문에 어떤 플랫폼(윈도우, 리눅스 등)에서도 사용할 수 있다. 

이와 달리 다이렉트 리드는 직접적으로 운영체제에 접근하여 시스템 프로그래밍을 할 때 사용한다. 반면 `fread()`의 경우 스탠다드 라이브러리를 통해 시스템 콜을 호출하게 된다. `fread()` 함수가 존재하는 이유는 **범용성(Portability)** 때문이다. 다이렉트 리드를 통해 시스템 콜을 호출하기 위해서는 운영체제 별로 지원하는 시스템 콜을 따로 공부해야 한다. 반면 `fread()`의 경우 스탠다드 라이브러리가 이를 대신해 준다. 또한 버퍼링과 같은 기능을 지원해 주기도 한다.

앞서 설명했다시피 시스템 콜은 소프트웨어 인터럽트를 발생시킨다. 특정 시스템 콜에 일치하는 소프트웨어 인터럽트를 매칭하기 위해 소프트웨어 인터럽트마다 번호를 부여한다. 마이크로 프로세서는 소프트웨어 인터럽트 발생시 유저 모드에서 특권 모드로 모드를 변경하고 해당 번호에 맞는 핸들러에 따라 특정 함수를 수행한다. 예를 들어 유저가 `fread()` 함수를 호출하여 소프트웨어 인터럽트 발생시 특정 번호에 위치한 `sys_read()` 함수가 이어서 호출된다.

따라서 우리가 System Call을 구현하기 위해서는 우선 **새로운 소프트웨어 인터럽트에 해당하는 번호를 할당 받아야 하고**, 이 번호에 대한 **핸들러를 구현**해 줘야한다. 그리고 이를 애플리케이션에서 사용할 수 있도록 **함수를 구현한다**.

#### 2.2 나만의 System Call 만들기

##### 2.2.1 새로운 소프트웨어 인터럽트에 해당하는 번호를 할당

![alt text](<./image/Screenshot 2024-05-28 at 3.24.35 PM.png>)

<br>

`kernel` 디렉토리에 오리지널 커널 복사본을 만든다.

<br>

![alt text](<./image/Screenshot from 2024-05-16 11-22-10.png>)

<br>

아키텍쳐에 의존적인 파일들은 `arch` 디렉토리 밑에 존재한다. 다음 위치로 이동해 `vi unistd.h` 명령어를 입력한다.

<br>

![alt text](<./image/Screenshot from 2024-05-16 11-23-39.png>)

<br>

각각의 번호가 할당된 시스템 콜 함수를 확인할 수 있다. 0번부터 시작하여 375번까지 총 376개의 함수가 등록된 것을 확인할 수 있다. 함수 내부에는 소프트웨어 인터럽트를 유도하는 어셈블리어로 구성된다.

<br>

![alt text](<./image/Screenshot from 2024-05-16 11-24-01.png>)

<br>

375번에 해당하는 함수 라인을 `yy`명령어로 복사하여 376번에 해당하는 함수를 등록해 준다.

<br>

![alt text](<./image/Screenshot from 2024-05-16 11-26-23.png>)

##### 2.2.2 핸들러 구현

`arch/arm/kernel` 위치에서 `vi entry-common.S` 명령어를 입력한다. 해당 파일은 확장자가 S인 어셈블리어 파일이다. 번호와 함수를 매핑해주는 테이블, `sys_call_table`에 새로운 정보를 등록해야한다.

<br>

![alt text](<./image/Screenshot from 2024-05-16 11-28-40.png>)
![alt text](<./image/Screenshot from 2024-05-16 11-29-31.png>)
![alt text](<./image/Screenshot from 2024-05-16 11-29-45.png>)

<br>

밖으로 나와 `vi calls.S` 명령어를 통해 해당 파일을 열어준다. 해당 파일은 번호에 대해 어떤 함수를 호출할 것인가에 대한 테이블 정보를 담고 있는 파일이다. 따라서 **376번에 해당하는 위치에 우리의 함수를 등록해준다**.

<br>

![alt text](<./image/Screenshot from 2024-05-16 11-32-16.png>)


##### 2.2.3 함수 구현

만들고자 하는 함수가 CPU에 의존적인 함수가 아니기 때문에 `arch` 밑에가 아니라 그냥 커널 아래에 `vi mysyscall.c` 명령어를 통해 함수를 만들어 준다.

<br>

![alt text](<./image/Screenshot from 2024-05-16 11-35-38.png>)

<br>

간단한 프린트 함수를 만든다. 단 커널에서 출력할 내용이라는 점에서 `printf`가 아닌 `printk`를 활용한다.

<br>

![alt text](<./image/Screenshot from 2024-05-16 11-37-33.png>)

<br>

해당 c 파일을 컴파일하기 위해 Makefile을 수정한다.

<br>

![alt text](<./image/Screenshot from 2024-05-16 11-38-31.png>)

<br>

`-y` 옵션이 붙으면 해당 파일을 무조건 컴파일한다는 뜻이다. 다음과 같이 새롭게 라인을 추가한다. 이름은 파일 이름을 따라간다.

<br>

![alt text](<./image/Screenshot from 2024-05-16 11-39-39.png>)

추가를 완료했으면 한 디렉토리 위로 올라가서 `make clean` 명령어를 실행해준다.

<br>

![alt text](<./image/Screenshot from 2024-05-16 11-40-44.png>)

<br>

이후 `make zImage`를 통해 컴파일을 진행하고, 부트로더 모드에서 fastboot를 연결하여 커널을 플래시 해준다. 명령어는 `fastboot flash kernel zImage`.


### 3. 어플리케이션에서 System Call 함수 호출하기

---

#### 3.1 test_mysyscall.c 작성 및 컴파일

`~/class/embedded_linux/work/test` 위치에 `vi test_mysyscall.c` 명령어를 통해 실습할 c 파일을 하나 만들어 준다.

<br>

![alt text](<./image/Screenshot from 2024-05-23 09-34-01.png>)

<br>

`syscall()` 이라는 함수는 마이크로 프로세서에 소프트웨어 인터럽트를 걸어주는 함수이다. 매개변수로 시스템 콜의 번호를 전달한다. 해당 번호들은 `arch/arm/unistd.h` 밑에 정의 되어있기 때문에 의존성을 추가해준다.

<br>

![alt text](<./image/Screenshot from 2024-05-23 09-32-54.png>)

<br>

어플리케이션 컴파일이기 때문에 `arm-linux` 컴파일러를 이용해서 해당 파일을 컴파일 해준다. 하지만 `unistd.h`의 위치를 지정해 주지 않았기 때문에 호스트 우분투 컴퓨터에 위치한 `unistd.h`가 호출되기 때문에 당연하게도 `__NR_mysyscall`에 해당하는 번호가 정의되지 않았다는 에러가 발생한다.

<br>

![alt text](<./image/Screenshot from 2024-05-23 09-35-26.png>)

<br>

따라서 헤더 파일의 위치를 직접 지정해 주는 컴파일 옵션 `I`를 통해 `unistd.h`의 위치를 지정해줘서 다시 컴파일한다.

<br>

![alt text](<./image/Screenshot from 2024-05-23 09-41-26.png>)

<br>

컴파일 완료한 실행 파일을 Target에 전달하기 위해서는 `tftp`를 이용한다. 따라서 `sudo cp test_mysyscall /var/lib/tftpboot/` 위치에 `test_mysyscall` 파일을 복사한다. 

<br>

![alt text](<./image/Screenshot from 2024-05-23 09-43-10.png>)

#### 3.2 네크워크 연결 및 test_mysyscall 실행

`minicom`을 실행하여 Target의 콘솔을 확인하고 부팅해준다. 부팅이 완료되면 네트워크부터 설정해 준다.

<br>

![alt text](<./image/Screenshot from 2024-05-23 09-45-37.png>)

<br>

네트워크 설정이 완료되었으면 `tftp -g -r test_mysyscall 10.10.10.1`을 통해 `test_mysyscall` 파일을 받아온다. 또 적당한 실행 권한을 부여한다. 그리고 실행한다.

<br>

![alt text](<./image/Screenshot from 2024-05-23 09-51-22.png>)

<br>

`tail -f /var/log/messages` 명령어를 통해 커널의 메시지를 지속적으로 확인해 볼 수 있다. 우리가 등록한 함수의 커널 메시지를 확인할 수 있다.
