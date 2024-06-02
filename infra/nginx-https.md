# Nginx, HTTPS 설정

### 1. DNS 등록하기

---

#### 1.1 Route53에 도메인 등록

먼저 시작에 앞서 가비아에서 도메인을 구입하여 줍니다.

<br>

![alt text](<Screenshot 2024-06-02 at 11.25.51 PM.png>)

<br>

AWS 검색창에서 Route53을 검색하여 접속합니다.

<br>

![alt text](<Screenshot 2024-06-02 at 11.48.52 PM.png>)

<br>

좌측 탭에서 호스팅 영역을 선택하고 호스팅 영역 생성 버튼을 클릭합니다.

<br>

![alt text](<Screenshot 2024-06-03 at 12.20.07 AM.png>)

<br>

다음과 같이 도메인 이름에 구매한 도메인을 적어주고 유형은 퍼블릭 호스팅 영역을 선택하여 줍니다. 구성을 마쳤으면 호스팅 영역 생성 버튼을 클릭합니다. 

<br>

![alt text](<Screenshot 2024-06-03 at 12.25.44 AM.png>)

<br>

호스팅 영역 생성이 완료되면 레코드 생성 버튼을 클릭합니다.

<br>

![alt text](<Screenshot 2024-06-03 at 12.27.17 AM.png>)

<br>

다른 부분들은 모두 그대로 두고 본인 EC2 서버의 탄력적 IP 주소를 값 부분에 입력합니다. 입력을 마쳤으면 레코드 생성 버튼을 클릭합니다.

<br>

![alt text](<Screenshot 2024-06-03 at 12.36.22 AM-1.png>)

<br>

이로써 레코드 세트가 총 3개가 되었습니다.

<br>

![alt text](<Screenshot 2024-06-03 at 12.40.01 AM.png>)

<br>

이번에는 가비아에 로그인 후 접속하여 My 가비아 탭을 선택합니다. 그리고 순서대로 `도메인` -> `구매한 도메인 관리 버튼`을 클릭하여 도메인 관리 페이지로 접속합니다.

<br>

![alt text](<Screenshot 2024-06-03 at 12.42.39 AM.png>)

<br>

![alt text](<Screenshot 2024-06-03 at 12.43.34 AM.png>)

<br>

![alt text](<Screenshot 2024-06-03 at 12.44.16 AM.png>)

<br>

이제 이곳에서 네임서버를 등록해야 합니다. 다음과 같이 네임서버 설정 버튼을 클릭합니다.

<br>

![alt text](<Screenshot 2024-06-03 at 12.47.35 AM.png>)

<br>

이곳에 앞서 Route53의 호스팅 영역 NS 레코드의 값/트래픽 라우팅 대상에 해당하는 값들을 입력해 주어야 합니다. 다음과 같이 위에서부터 하나 하나 그대로 입력해 줍니다. 단, **마지막에 점(.)은 지우고 입력합니다**.

<br>

![alt text](<Screenshot 2024-06-03 at 12.49.33 AM.png>)

<br>

![alt text](<Screenshot 2024-06-03 at 12.53.07 AM.png>)

<br>

다 입력했으면 소유자 인증을 마치고 적용 버튼을 눌러줍니다.

<br>

![alt text](<Screenshot 2024-06-03 at 12.55.23 AM.png>)

<br>

### 2. Nginx

---

#### 2.1 Nginx 설치

EC2 서버 컴퓨터에 접속하여 다음 명령어를 입력합니다.

```shell
sudo yum install nginx
```

설치가 완료되었으면 다음 명령어로 Nginx를 실행합니다.

```shell
sudo systemctl start nginx.service
```

Nginx가 정상 구동 중인지 확인하고 싶다면 다음 명령어를 입력합니다. 

```shell
systemctl status nginx.service
```

#### 2.2 Proxy Server 구성

포트번호가 80인 http에서 요청이 오면, Spring Boot 프로젝트에서 8080번 포트를 바라볼 수 있도록 proxy 설정을 해주겠습니다.

먼저 요청에 대한 로그와 에러를 기록할 수 있는 디렉토리를 생성해 주겠습니다.

```shell
sudo mkdir /var/log/nginx/proxy/
```

다음으로 proxy 관련 설정을 진행합니다. 먼저 다음 명령어로 `proxy_params`라는 이름의 파일을 생성합니다.

```shell
sudo vi /etc/nginx/proxy_params
```

그리고 다음 내용을 `proxy_params`에 복사하여 추가합니다.

```shell
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header Host $http_host;
proxy_set_header X-Forwarded-Proto $scheme;
proxy_set_header X-NginX-Proxy true;

client_max_body_size 256M;
client_body_buffer_size 1m;

proxy_buffering on;
proxy_buffers 256 16k;
proxy_buffer_size 128k;
proxy_busy_buffers_size 256k;

proxy_temp_file_write_size 256k;
proxy_max_temp_file_size 1024m;

proxy_connect_timeout 300;
proxy_send_timeout 300;
proxy_read_timeout 300;
proxy_intercept_errors on;
```

다음 명령어를 입력하여 Nginx 설정 파일을 열어보겠습니다.

```shell
sudo vi /etc/nginx/nginx.conf
```

다음과 같이 아래로 내리다가 보면 `server` 블록을 찾을 수 있습니다. 해당 부분에서 다음 내용들을 수정할 것입니다.

* `server_name`에 도메인 입력
* `reverse proxy` 설정
* `CORS` 설정

<br>

![alt text](<Screenshot 2024-06-03 at 1.05.49 AM.png>)

<br>

변경 및 추가할 부분은 다음과 같이 총 세 부분입니다. **첫 번째 부분**은 등록한 도메인 명으로 요청이 들어올 때 처리하는 부분이고, **두 번째 부분**은 접근 로그와 에러 로그를 기록하는 부분입니다. 마지막 **세 번째 부분**은 Spring Boot 어플리케이션과 실제 연결되는 부분으로, 먼저 해당 location 블록에서 요청을 처리하고, 하단의 CORS 관련 헤더를 추가한 이후에, Spring Boot가 실행 중인 8080 포트로 요청을 전달합니다.

<br>

![alt text](<Screenshot 2024-06-03 at 1.34.44 AM.png>)

<br>

```python
server {
        listen       80;
        listen       [::]:80;
        server_name  {{등록한 도메인 입력}}
        root         /usr/share/nginx/html;

        access_log   /var/log/nginx/proxy/access.log;
        error_log    /var/log/nginx/proxy/error.log;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / { # location 블록
                include /etc/nginx/proxy_params;
                proxy_pass http://{{서버의 Public IP}}:8080;    # reverse proxy의 기능
                if ($http_x_forwarded_proto = 'http') {
                        return 301 https://$host$request_uri;
                }
                # CORS 설정
                if ($request_method = 'OPTIONS') {
                        add_header 'Access-Control-Allow-Origin' 'http://localhost:3000'; # 프론트엔드 주소
                        add_header 'Access-Control-Allow-Methods' 'GET, POST, PATCH, OPTIONS, PUT, DELETE';
                        add_header 'Access-Control-Allow-Headers' 'Authorization,DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range';
                        add_header 'Access-Control-Allow-Credentials' 'true';
                        add_header 'Access-Control-Max-Age' 1728000;
                        add_header 'Content-Type' 'text/plain charset=UTF-8';
                        add_header 'Content-Length' 0;
                        return 204;
                }
                add_header 'Access-Control-Allow-Origin' 'http://localhost:3000' always; # 프론트엔드 주소
                add_header 'Access-Control-Allow-Methods' 'GET, POST, PATCH, OPTIONS, PUT, DELETE' always;
                add_header 'Access-Control-Allow-Credentials' 'true' always;
                add_header 'Access-Control-Allow-Headers' 'Authorization,DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range' always;
        }

        error_page 404 /404.html;
        location = /404.html {
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
```

<br>

위에서 주의할 점은 추후 프론트엔드와 통신할 때 CORS 설정 중 `Access-Control-Allow-Origin` 부분을 와일드 카드(모두 허용)를 뜻하는 `*`이 아닌 프론트엔드 주소를 입력해야한다. 쿠키를 이용한 통신에서는 사이트 간 통신에서 와일드 카드가 허용되지 않기 때문이다. 프론트엔드 배포가 완료되면 배포 주소로 변경한다.