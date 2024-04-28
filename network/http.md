# HTTP 프로토콜

### 전송하는 데이터

---

**HTTP란 HyperText Transfer Protocol**의 약자로 하이퍼텍스트, 즉 **HTML 문서를 주고 받기 위해 만들어진 통신 규약**이다. 웹 초기에는 순수하게 웹 브라우저에서 웹 서버로 특정 HTML 문서를 요청하면 단순히 이를 전달하는 과정만 존재했다. 하지만 현재는 HTTP 프로토콜을 통해 HTML 문서뿐만 아니라 **이미지, 음성, 영상 등 거의 모든 형태의 데이터 전송이 가능**하다.

#### HTTP 메시지 구조

HTTP 메시지에는 클라이언트에서 서버로 보내는 **요청 메시지**, 그리고 해당 요청에 대해 서버가 클라이언트로 보내는 **응답 메시지**가 있다. 요청과 응답 메시지의 기본적인 메시지 구조는 다음과 같다.

* **start-line**: HTTP 메서드, URI, 프로토콜
* **header**: 각종 메타 정보
* **message body**: 메시지 본문
  
![alt text](<./image/Screenshot 2024-04-28 at 9.59.06 PM.png>)

#### HTTP 메서드

HTTP 메서드란 요청의 동작을 결정하는 정보이다. 주요 메서드는 다음과 같다.

* GET: 리소스 조회
* POST: 요청 데이터 처리, 주로 등록에 사용
* PUT: 리소스 전체 변경, 해당 리소스가 없으면 생성
* PATCH: 리소스 부분 변경
* DELETE: 리소스 삭제

### SSR과 CSR

---

#### SSR

앞서 말한 바와 같이 초기의 HTTP는 HTML 문서밖에 통신하지 못했다. 따라서 모든 HTML 화면을 서버에서 생성하여 클라이언트에 제공했다. 이처럼 **서버에서 화면을 생성하는 방식을 Server Side Rendering 기술, 줄여서 SSR**이라고 한다. 

![alt text](<./image/Screenshot 2024-04-28 at 2.53.45 PM.png>)

#### CSR

HTTP 프로토콜의 발전으로 JSON과 XML 형식의 데이터를 전송할 수 있게 되었다. 이로 인해 서버는 필요한 데이터만 전달하고 화면 생성은 클라이언트로 위임할 수 있게 되었다. 이처럼 **클라이언트에서 화면을 생성하는 방식을 Client Side Rendering 기술, 줄여서 CSR**이라고 한다. CSR 방식을 통해 무상태(stateless)적인 통신을 구현할 수 있다.

![alt text](<./image/Screenshot 2024-04-28 at 3.15.52 PM.png>)


### Stateful과 Stateless

---

#### Stateful

다음 대화에서 고객을 클라이언트 컴퓨터, 점원을 서버라고 가정했을 때 **서버가 클라이언트의 상태를 보존하며 통신을 진행하는 방식을 Stateful한 통신**이라고 한다. SSR로 화면을 구성하게 되면 서버에서 화면 생성을 일임하기 때문에 일부 Stateful한 방식으로 통신을 진행할 수밖에 없다.

![alt text](<./image/Screenshot 2024-04-28 at 4.32.11 PM.png>)

#### Stateless

반대로 다음 대화와 같이 **점원(서버)이 고객(클라이언트)의 상태를 보존하지 않으며 통신하는 방식을 Stateless한 통신**이라고 한다. CSR 방식으로 화면을 구성하게 되면 Stateless한 방식으로 통신을 진행할 수 있다.

![alt text](<./image/Screenshot 2024-04-28 at 4.44.18 PM.png>)

#### Stateless 통신의 장점

위와 같은 상황에서 Stateful한 방식은 점원 한 명이 고객 한 명을 처음부터 끝까지 응대해야 한다. 반면 Stateless한 방식은 중간에 점원이 바뀌어도 구매를 원활하게 진행할 수 있다. 즉, 고객이 갑자기 증가해도 점원을 대거 투입할 수 있다. **이는 곧 클라이언트의 요청이 증가했을 때 서버를 대거 증설할 수 있다는 말과 같다.**


### REST API

---

#### API?

API란 프로그램 간의 소통 창구를 의미한다. 웹 분야에서 "API를 개발한다"의 의미는 **서버에 있는 데이터를 클라이언트에게 제공하는 창구를 개발**한다는 뜻이다. 즉 서버 개발자는 데이터베이스에 있는 데이터를 클라이언트에게 효율적이고 규칙성있게 제공해야 할 필요가 있다.

#### REST?

REST API의 가장 큰 특징은 데이터를 Stateless 하게 제공한다는 것이다. 예시를 통해 확인해 보자. 우선 다음과 같은 API URI(URL) 설계는 유저의 동작을 보존하여 Stateful하게 설계되었다.

* 학생 등록 /create-student
* 학생 조회 /read-student-by-id
* 학생 전체 조회 /read-student-list
* 학생 수정 /update-student
* 학생 삭제 /delete-student

Stateless한 REST API의 가장 큰 특징은 **리소스, 즉 자원에 집중하여 설계**된다는 점이다. 자원에 집중하여 API를 설계하기 위해서는 **URI에 동사가 들어가서는 안된다.** 학생이라는 자원에 집중하여 URI를 설계하되 **등록, 조회, 수정, 삭제라는 동작은 HTTP 메서드**를 이용해 구분한다. 올바른 REST API 설계는 다음과 같다.

* 학생 등록 /students -> POST 
* 학생 조회 /students/{id} -> GET
* 학생 전체 조회 /students -> GET
* 학생 수정 /students/{id} -> PUT or PATCH
* 학생 삭제 /students/{id} -> DELETE

즉 URI는 학생이라는 리소스를 식별하는 역할만 해야 하고 행위는 HTTP 메서드를 활용해야 한다. 