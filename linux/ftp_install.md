# TFTP

## 1. 개요

### 1.1 Target - Host Connection

---

Target과 Host를 연결하는 방식에는 총 3가지 방식이 있다. 콘솔을 확인하기 위한 Serial 연결, `fastboot`를 통해 이미지 다운을 하기 위한 USB 연결, 그리고 이더넷 연결이 있다. 이더넷 연결은 커널이 모두 부팅된 이후에 어플리케이션을 주고 받는 데 사용되는 연결이다. 
> ✅ Host의 네트워크 인터페이스를 확인하면 외부 인터넷과 연결되는 온보드 연결이 있고, Target과 연결되는 `USB to Ethernet` 연결이 있다.


### 1.2 FTP

---

Host에서 제작한 애플리케이션을 Target으로 다운하기 위해서는 `FTP`라고 하는 네트워크 프로토콜이 필요하다. 하지만 현재 상황과 같이 Host와 Target을 1대1로 연결하는 과정에서 `FTP`를 사용하기는 너무 무겁고, `TFTP`를 사용한다. `TFTP`는 UDP로 동작하기 때문에 폐쇄적인 환경에서 사용하기 적합하며 에러 컨트롤, 플로우 컨트롤 등의 기능을 제공하지 않는다.


### 1.3 TFTP

---

`TFTP` 프로토콜을 사용하기 위해서는 Target에는 `TFTP Client`, Host에는 `TFTP Server`를 설치해야 한다. 다만 Target에는 이미 `TFTP Client`가 내장되어 있기 때문에 Host에서만 설치 및 설정을 진행하면 된다.

> ✅ 일반적으로 서버를 설치하는 방식에는 `standalone` 으로만 설치하는 방법과 `superdaemon`도 같이 활용하는 방식이 있다. 전자는 클라이언트로부터 요청이 빈번하게 들어올 때 사용하는 방식이고, 후자는 클라이언트로부터 요청이 들어올 때만 서버를 실행하여 시스템 자원을 효율적으로 관리하는 방식이다. `FTP`의 경우 후자의 방식을 주로 사용한다.

리눅스에서는 `xinetd`가 `superdaemon`으로 가장 많이 사용된다. 따라서 클라이언트의 요청을 정상적으로 처리하기 위해서는 `xinetd`에 `xinetd`가 관리할 서버를 등록해 주어야 한다.

## 2. 설치

### 2.1 xinetd & tftp

---

`sudo apt install xinetd` 명령어를 통해 `xinetd`를 설치하면 `/etc/`위치에 `xinetd.d`이름의 설정 디렉토리가 생성된다.<br>

![alt text](<./image/Screenshot from 2024-05-16 09-22-49.png>)

다음으로 Host에 `TFTP`를 설치하여 준다. `tftpd`는 서버이고, `tftp`는 클라이언트이다. 서버만 설치해도 되지만 둘 다 설치하여 준다.<br>
![alt text](<./image/Screenshot from 2024-05-16 09-24-27.png>)

이제 `xinetd`가 설치한 `tftp` 서버를 인식할 수 있도록 설정을 변경해 주어야 한다. 우선 `tftp`의 이름으로 `echo-udp`의 파일을 복사해 준다.<br>
![alt text](<./image/Screenshot from 2024-05-16 09-26-49.png>)

그리고 다음 내용으로 변경하여 준다. `/usr/sbin/in.tftpd`는 서버가 위치한 곳이다. `server_agrs`와 `disable`만 수정하고 나머지 설정은 default 값으로 간다고 볼 수 있다.<br>
![alt text](<./image/Screenshot from 2024-05-16 09-30-49.png>)

다음과 같이 `/var/lib/tftpboot` 디렉토리를 만들어 준다.<br>
![alt text](<./image/Screenshot 2024-05-16 at 4.00.42 PM.png>)

리눅스에서 서버를 관리할 때는 `systemctl` 명령어를 사용한다. 따라서 `xinetd`가 구동되고 있는 현재 상태를 확인하기 위해 `systemctl status xinetd.service` 명령어를 입력한다. 정상 실행 중인 것을 확인할 수 있다.<br>
![alt text](<./image/Screenshot from 2024-05-16 09-41-54.png>)

현재 사용할 수 있는 프로토콜을 확인하기 위해서는 `netstat -ap` 명령어를 입력한다. 출력 내용이 많음으로 `grep tftp`를 통해 내용을 제한한다.<br>
![alt text](<./image/Screenshot 2024-05-16 at 4.29.11 PM.png>)

결과가 나오지 않는다.<br>
![alt text](<./image/Screenshot 2024-05-16 at 4.51.18 PM.png>)

`tftp` 설정 이후 `xinetd`에 해당 설정이 반영되지 않았기 때문에 발생하는 문제이다.<br>
![alt text](<./image/Screenshot 2024-05-16 at 4.54.44 PM.png>)

다시 명령어를 입력해 보면 `tftp` 프로토콜을 사용할 수 있는 것을 확인할 수 있다.<br>
![alt text](<./image/Screenshot 2024-05-16 at 5.09.25 PM.png>)

### 2.2 hello.c 테스트

---

테스트를 위해 `hello.c`를 만들어 Target에서 실행해 보도록 하겠다.<br>
![alt text](<./image/Screenshot 2024-05-16 at 5.14.36 PM.png>)

```C
// hello.c
#include <stdio.h>
int main() {
    printf("Hello World!\n");
    return 0;
}
```

Target에서 실행할 것이기 때문에 `arm`용으로 컴파일을 진행해야 한다. 명령어는 다음과 같다. `arm-none-`과 `arm-linux-`의 차이는 전자는 바이너리 코드로 컴파일 되어 커널과 부트로더를 컴파일할 때 사용하며, 후자는 커널이 올라간 상태에서 사용될 어플리케이션을 컴파일할 때 사용된다.<br>
![alt text](<./image/Screenshot 2024-05-16 at 5.19.08 PM.png>)

다음과 같이 정상적으로 실행파일이 생성된 것을 확인할 수 있다. `file` 명령어로 정보를 확인해 보면 `arm`용 실행 파일이다.<br>
![alt text](<./image/Screenshot 2024-05-16 at 5.24.05 PM.png>)

`tftp`서버를 이용하여 Target에 파일을 전달하기 위해 `/var/lib/tftpboot` 위치로 파일을 복사한다.<br>
![alt text](<./image/Screenshot 2024-05-16 at 5.26.41 PM.png>)

Host 쪽에서의 준비는 모두 끝났다. `tftp`서버를 설치하였고, `xinetd`가 가동 중이며 클라이언트에 전달할 파일도 생성하여 대기중이다. 따라서 다음으로 `minicom`을 실행하고 Target을 준비하여 준다. Host와 정상적으로 연결되어있는지 확인하기 위해 `ping 10.10.10.1`을 입력한다.<br>

![alt text](<./image/Screenshot 2024-05-21 at 2.03.33 AM.png>)

`tftp` 클라이언트에서 파일을 받아오기 위해서는 명령어에 옵션과 호스트를 지정한다. 명령어 `tftp -g -r hello_arm 10.10.10.1`란 리모트 호스트 10.10.10.1에서 `hello_arm`이라는 파일을 가져오겠다는 의미다.<br>

![alt text](<./image/Screenshot 2024-05-21 at 2.22.39 AM.png>)

![alt text](<./image/Screenshot 2024-05-21 at 2.24.16 AM.png>)

현재 `hello_arm` 파일에는 `user`, `group`, `other` 모두에 실행 권한(x)이 없다. 따라서 `chmod a+x` 명령어를 통해 모든 그룹에 실행 권한을 부여한다.<br>
![alt text](<./image/Screenshot 2024-05-21 at 2.25.24 AM.png>)

정상 실행된다.<br>
![alt text](<./image/Screenshot 2024-05-21 at 2.28.56 AM.png>)