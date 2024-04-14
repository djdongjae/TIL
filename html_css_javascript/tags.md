# HTML CSS 태그 분석


### 개요

---

해당 보고서는 `Do it! Node.js 프로그래밍 입문` 도서의 `MyContacts` 폴더 상 위치한 html 파일과 css 파일을 분석한 보고서입니다. 


### add.html

---

#### head tag

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>연락처 관리하기</title>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.2.3/dist/css/bootstrap.min.css">
  <link rel="stylesheet" href="css/style.css">
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" integrity="sha512-iecdLmaskl7CVkqkXNQ/ZH/XLlvWZOJyj7Yy7tcenmpD1ypASozpmT/E0iPtmFIB46ZmdtAc9eNBvH0H/ZpiBw==" crossorigin="anonymous" referrerpolicy="no-referrer" />
</head>
```

* `<!DOCTYPE html>`: HTML 문서의 버전 및 유형을 지정합니다.
* `<html lang="ko">`: HTML 문서의 기본 언어를 한국어로 설정합니다.
* `<head>`: HTML 문서의 메타데이터와 외부 리소스를 정의하는 부분입니다.
    * `<meta charset="UTF-8">`: 문서의 문자 인코딩을 UTF-8로 설정합니다.
    * `<meta name="viewport" content="width=device-width, initial-scale=1.0">`: 반응형 웹 페이지를 지원하기 위한 뷰포트 설정입니다.
    * `<title>연락처 관리하기</title>`: 웹 페이지의 제목입니다.
* 이하 `link` 태그 부분은 외부에서 직접 정의한 스타일 시트부터 BootStrap, 아이콘, 폰트 등의 라이브러리를 불러오는 부분입니다.


#### body tag

```html
<body>
    <!-- Header -->
    <header class="border-shadow">
        <div class="container ">
            <nav>
                <a href="/"><i class="fa-solid fa-address-book"></i> My Contacts</a>
            </nav>
        </div>
    </header>
    <!-- /Header -->

  <!-- Main -->
    <main id="site-main">
      <div class="button-box">        
        <a href="#" class="btn btn-light"><i class="fa-solid fa-list"></i>연락처 목록</a>
      </div>
      </div>
      <h2>연락처 추가</h2>          
      <p>연락처 정보를 추가합니다.</p>
      <form method="POST" id="add-user">
        <div class="user-info">
          <div class="col-12">
            <label for="name" class="col-form-label">이름(Full Name)</label>
            <div>
              <input type="text" class="form-control" name="name" id="name" placeholder="홍길동">
            </div>
          </div>
          <div class="col-12">
            <label for="email" class="col-form-label">메일 주소(E-mail)</label>
            <div>
              <input type="text" class="form-control" name="email" id="email" placeholder="hong@abc.def">
            </div>
          </div>
          <div class="col-12">
            <label for="phone" class="col-form-label">전화번호(Mobile)</label>
            <div>
              <input type="text" class="form-control" name="phone" id="phone" placeholder="123-4567-8901">
            </div>
          </div>
          <button type="submit">저장하기</button>
        </div>
      </form>   
    </main>
  <!-- /Main -->

</body>
</html>
```
* `<body>`: HTML 문서의 실제 내용을 담고 있는 부분입니다.
* `<header>`: 페이지의 헤더를 정의하는 태그로 네비게이션 바 등이 그 예시입니다.
  * `<a href="/"><i class="fa-solid fa-address-book"></i> My Contacts</a>`: 아이콘을 클릭할 시에 홈 화면으로 이동합니다. 
* `<main id="site-main">`: 웹 페이지의 주요 콘텐츠를 정의합니다.
  * `<div class="button-box">`: 버튼을 담고 있는 상자입니다.
  * `<a href="#" class="btn btn-light"><i class="fa-solid fa-list"></i>연락처 목록</a>`: 연락처 목록을 보여주는 버튼입니다. 아이콘과 텍스트를 함께 표시합니다.
  * `<h2>연락처 추가</h2>`: 연락처 추가 섹션의 제목입니다.
  * `<p>연락처 정보를 추가합니다.</p>`: 연락처 추가에 대한 설명을 제공하는 단락입니다.
  * `<form method="POST" id="add-user">`: 사용자 정보를 추가하는 폼을 정의합니다. POST 방식으로 제출됩니다. 사용자 정보를 입력받는 필드들이 포함되어 있습니다.
  * `<button type="submit">저장하기</button>`: 폼을 제출하는 버튼입니다.
  * `<div class="user-info">`: 사용자 정보를 감싸는 컨테이너입니다.
사용자의 이름, 이메일 및 전화번호를 입력받는 입력 필드가 있습니다.


### contacts.html

---

#### head tag

```html
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Contacts Manager</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" integrity="sha512-iecdLmaskl7CVkqkXNQ/ZH/XLlvWZOJyj7Yy7tcenmpD1ypASozpmT/E0iPtmFIB46ZmdtAc9eNBvH0H/ZpiBw==" crossorigin="anonymous" referrerpolicy="no-referrer" />
  <style>
    body {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      height: 100vh;
      background-color: #eee;
    }
    h1 {
      font-size: 3em;
      text-transform: capitalize;            
    }
    h1 i {
      margin-right: 10px;
    }
  </style>
</head>
```

`style` 태그 이외의 부분은 나머지 부분과 동일하므로 해당 태그만 분석해 보겠습니다.

* `body`: 웹 페이지의 본문(body)에 적용되는 스타일입니다.
  * `display: flex;`: Flexbox 레이아웃을 사용하여 요소를 배치합니다.
  * `flex-direction: column;`: Flexbox 레이아웃 방향을 세로(column)로 설정합니다.
  * `justify-content: center;`: 요소들을 수직 방향으로 가운데 정렬합니다.
  * `align-items: center;`: 요소들을 수평 방향으로 가운데 정렬합니다.
  * `height: 100vh;`: 뷰포트의 높이(100vh)만큼 본문(body)의 높이를 설정합니다.
  * `background-color: #eee;`: 본문(body)의 배경색을 회색(#eee)으로 설정합니다.
* `h1`: 웹 페이지에서 사용되는 제목(heading) 요소에 적용되는 스타일입니다.
  * `font-size: 3em;`: 폰트 크기를 3em으로 설정합니다.
  * `text-transform: capitalize;`: 텍스트를 대문자로 변환합니다.
* `h1 i`: 제목(heading) 요소 내부에 있는 아이콘에 적용되는 스타일입니다.
  * `margin-right: 10px;`: 오른쪽 마진을 10px로 설정하여 아이콘과 텍스트 사이의 간격을 조절합니다.

해당 CSS 코드는 본문을 중앙 정렬하고, 제목을 크게 표시하며, 제목 텍스트를 대문자로 변환하고, 아이콘과 텍스트 사이의 간격을 조절하는 등의 스타일을 정의합니다.


#### body tag
```html
<body>
  <h1><i class="fa-solid fa-address-book"></i> All Contacts</h1>  
</body>
```
아이콘과 텍스트를 본문에 간단히 표시합니다.



### getAll.html

---

#### head tag

```html
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Get All Contacts</title>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" integrity="sha512-iecdLmaskl7CVkqkXNQ/ZH/XLlvWZOJyj7Yy7tcenmpD1ypASozpmT/E0iPtmFIB46ZmdtAc9eNBvH0H/ZpiBw==" crossorigin="anonymous" referrerpolicy="no-referrer" />
  <style>
    * {
      margin:0;
      padding:0;
    }
    body {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }
    h1 {
      font-family: Arial, Helvetica, sans-serif;
      font-size: 3em;
      margin-bottom: 20px;
    }
    h1 i {
      margin-right: 0.8em;
    }
    h2 {
      margin-bottom: 15px;
    }
  </style>
</head>
```

`style` 태그 이외의 부분은 나머지 부분과 동일하므로 해당 태그만 분석해 보겠습니다.

* `*`: 모든 요소에 대한 스타일을 적용합니다.
  * `margin: 0; padding: 0;`: 모든 요소의 마진과 패딩을 0으로 설정하여 기본 스타일을 초기화합니다.
* `body`: 웹 페이지의 본문(body)에 적용되는 스타일입니다.
  * `display: flex;`: Flexbox 레이아웃을 사용하여 요소를 배치합니다.
  * `flex-direction: column;`: Flexbox 레이아웃 방향을 세로(column)로 설정합니다.
  * `justify-content: center;`: 요소들을 수직 방향으로 가운데 정렬합니다.
  * `align-items: center;`: 요소들을 수평 방향으로 가운데 정렬합니다.
  * `height: 100vh;`: 뷰포트의 높이(100vh)만큼 본문(body)의 높이를 설정합니다.
* `h1`: 웹 페이지에서 사용되는 제목(heading) 요소에 적용되는 스타일입니다.
  * `font-family: Arial, Helvetica, sans-serif;`: 폰트를 지정합니다.
  * `font-size: 3em;`: 폰트 크기를 3em으로 설정합니다.
  * `margin-bottom: 20px;`: 하단 마진을 20px로 설정하여 제목 아래 여백을 추가합니다.
* `h1 i`: 제목(heading) 요소 내부에 있는 아이콘에 적용되는 스타일입니다.
  * `margin-right: 0.8em;`: 오른쪽 마진을 0.8em으로 설정하여 아이콘과 텍스트 사이의 간격을 조절합니다.
* `h2`: 웹 페이지에서 사용되는 두 번째 수준의 제목(heading) 요소에 적용되는 스타일입니다.
  * `margin-bottom: 15px;`: 하단 마진을 15px로 설정하여 두 번째 수준의 제목 아래 여백을 추가합니다.

#### body tag

```html
<body>
  <h1><i class="fa-solid fa-address-card"></i>Contacts Page</h1>
</body>
```

앞선 `contacts.html`과 내용이 유사합니다.


### home.html

---

**head 태그** 부분은 상단의 html 파일들과 동일한 요소를 포함하고 있으므로 설명을 생략하겠습니다.

#### body tag

```html
<body>
  <!-- Main -->
  <main id="site-main">
    <div class="home-container">
      <h3>로그인</h3>     
      <p>로그인이 필요한 서비스입니다.</p>
      <form class="login">
        <label for="username"><b>Username</b></label>
        <input type="text" placeholder="사용자 아이디" name="username" id="username">
        <label for="password"><b>Password</b></label>
        <input type="password" placeholder="비밀번호" name="password" id="password">
        <button type="submit">로그인</button>
      </form>
    </div>
  </main>
  <!-- /Main -->
</body>
```
* `<main id="site-main">`: 웹 페이지의 주요 콘텐츠를 나타내는 요소입니다. 여기에 로그인 폼이 포함됩니다.
  * `<div class="home-container">`: 로그인 폼과 관련된 요소들을 감싸는 컨테이너입니다.
  * `<form class="login">`: 사용자의 로그인 정보를 입력하는 폼입니다. login 클래스로 스타일을 지정할 수 있습니다.
    * `<label for="username"><b>Username</b></label>`: 사용자명을 입력하는 입력 필드의 라벨입니다.
    * `<input type="text" placeholder="사용자 아이디" name="username" id="username">`: 사용자명을 입력하는 텍스트 입력 필드입니다.
    * `<label for="password"><b>Password</b></label>`: 비밀번호를 입력하는 입력 필드의 라벨입니다.
    * `<input type="password" placeholder="비밀번호" name="password" id="password">`: 비밀번호를 입력하는 비밀번호 입력 필드입니다.
    * `<button type="submit">로그인</button>`: 폼을 제출하는 버튼입니다.

해당 HTML 코드는 사용자가 로그인할 수 있는 간단한 웹 페이지를 생성합니다.


### index.html

---

**head 태그** 부분은 상단의 html 파일들과 동일한 요소를 포함하고 있으므로 설명을 생략하겠습니다.


#### body tag

```html
<!-- Header -->
<header class="border-shadow">
  <div class="container ">
    <nav>
      <a href="/"><i class="fa-solid fa-address-book"></i> My Contacts</a>
    </nav>
  </div>
</header>
<!-- /Header -->
```
* `<header>`: 웹 페이지의 헤더를 정의합니다. 헤더 내부에는 내비게이션 링크가 포함되어 있습니다.
* `<a href="/"><i class="fa-solid fa-address-book"></i> My Contacts</a>`: 홈 링크를 포함하며, 주소록 아이콘과 함께 표시됩니다.

```html
  <!-- Main -->
  <main id="site-main">
    <div class="button-box">
      <a href="#" class="btn btn-light"><i class="fa-solid fa-user-plus"></i>연락처 추가</a>
    </div>
```
* `<main>`: 웹 페이지의 주요 콘텐츠를 정의합니다.
  * 추가 버튼이 있는 상자를 정의합니다.
  * `<a href="#" class="btn btn-light"><i class="fa-solid fa-user-plus"></i>연락처 추가</a>`: 연락처 추가 버튼을 정의하며, 사용자 플러스 아이콘이 표시됩니다.

```html
    <table class="table">
      <thead>
        <tr>
          <th>이름</th>
          <th>메일 주소</th>
          <th>전화번호</th>
          <th>&nbsp;</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Username</td>
          <td>example@gmail.com</td>
          <td>123456</td>
          <td>
            <a href="#" class="btn update" title="수정">
              <i class="fas fa-pencil-alt"></i>
            </a>
            <a href="#" class="btn delete" title="삭제">
              <i class="fas fa-times"></i>
            </a>                
          </td>
        </tr>
      </tbody>
    </table>
```
* `<table class="table">`: 연락처 목록을 표시하는 테이블을 정의합니다.
* `<thead>`: 테이블의 헤더를 정의합니다.
* `<tbody>`: 테이블의 본문을 정의합니다.
* 각 행은 연락처의 정보를 나타냅니다.
* 행 내에는 이름, 이메일, 전화번호가 포함되며, 수정 및 삭제 버튼이 있습니다.


### register.html


---


#### body tag


```html
<body>
  <!-- Main -->
  <main id="site-main">
    <h3>사용자 등록</h3>
    <form class="register">
      <label for="username"><b>아이디</b></label>
      <input type="text" placeholder="아이디" name="username" id="username">
      <label for="password"><b>비밀번호</b></label>
      <input type="password" placeholder="비밀번호" name="password" id="password">
      <label for="password2"><b>비밀번호 확인</b></label>
      <input type="password" placeholder="비밀번호 확인" name="password2" id="password2">
      <input type="submit" value="등록" class="register-btn">
    </form>    
  </main>
  <!-- /Main -->
</body>
```

* `<form class="register">`: 사용자 등록을 위한 폼을 정의합니다. register 클래스로 스타일을 지정할 수 있습니다.
* `<input type="text" placeholder="아이디" name="username" id="username">`: 사용자명을 입력하는 텍스트 입력 필드입니다.
* `<input type="password" placeholder="비밀번호" name="password" id="password">`: 비밀번호를 입력하는 비밀번호 입력 필드입니다.
* `<input type="submit" value="등록" class="register-btn">`: 폼을 제출하는 버튼입니다. 버튼에 register-btn 클래스가 지정되어 있습니다.


### update.html

---

#### body tag

중복되는 부분은 제외하고 body 태그 내부의 main 태그 부분만 분석하겠습니다.

```html
<!-- Main -->
    <main id="site-main">
      <div class="button-box">        
        <a href="#" class="btn btn-light"><i class="fa-solid fa-list"></i>연락처 목록</a>
      </div>
      <h2>연락처 수정</h2>          
      <p>연락처 정보를 수정합니다.</p>
      <form method="POST" id="add-user">
        <div class="user-info">
          <div class="col-12">
            <label for="name" class="col-form-label">이름(Full Name)</label>
            <div>
              <input type="text" class="form-control" name="name" id="name" value="도레미">
            </div>
          </div>
          <div class="col-12">
            <label for="email" class="col-form-label">메일 주소(E-mail)</label>
            <div>
              <input type="text" class="form-control" name="email" id="email" value="doremi@abc.def">
            </div>
          </div>
          <div class="col-12">
            <label for="phone" class="col-form-label">전화번호(Mobile)</label>
            <div>
              <input type="text" class="form-control" name="phone" id="phone" value="456-7890-1234">
            </div>
          </div>
          <button type="submit">수정하기</button>
        </div>
      </form>   
    </main>
```
* `<main id="site-main">`: 웹 페이지의 주요 콘텐츠를 나타내는 요소입니다.
* `<div class="button-box">`: 버튼을 담는 상자를 정의합니다.
* `<a href="#" class="btn btn-light"><i class="fa-solid fa-list"></i>연락처 목록</a>`: 연락처 목록을 보기 위한 버튼을 정의하며, Font Awesome의 리스트 아이콘이 표시됩니다.
* `<h2>연락처 수정</h2>`: 폼의 제목으로 사용됩니다.
* `<p>연락처 정보를 수정합니다.</p>`: 사용자에게 연락처 정보를 수정하는 것임을 안내하는 문구입니다.
* `<form method="POST" id="add-user">`: 연락처를 수정하기 위한 폼을 정의합니다. 폼은 POST 방식으로 데이터를 제출하며, 식별자는 "add-user"입니다.
* `<div class="user-info">`: 사용자 정보를 담는 상자를 정의합니다.
* `<div class="col-12">`: 사용자 이름을 입력하는 부분입니다.
* `<label for="name" class="col-form-label">이름(Full Name)</label>`: 이름 입력 필드의 라벨입니다.
* `<input type="text" class="form-control" name="name" id="name" value="도레미">`: 사용자 이름을 입력하는 텍스트 입력 필드입니다. 기본값으로 "도레미"가 설정되어 있습니다.
* `<div class="col-12">`: 사용자 이메일 주소를 입력하는 부분입니다.
* `<label for="email" class="col-form-label">메일 주소(E-mail)</label>`: 이메일 주소 입력 필드의 라벨입니다.
* `<input type="text" class="form-control" name="email" id="email" value="doremi@abc.def">`: 사용자 이메일 주소를 입력하는 텍스트 입력 필드입니다. 기본값으로 "doremi@abc.def"가 설정되어 있습니다.
* `<div class="col-12">`: 사용자 전화번호를 입력하는 부분입니다.
* `<label for="phone" class="col-form-label">전화번호(Mobile)</label>`: 전화번호 입력 필드의 라벨입니다.
* `<input type="text" class="form-control" name="phone" id="phone" value="456-7890-1234">`: 사용자 전화번호를 입력하는 텍스트 입력 필드입니다. 기본값으로 "456-7890-1234"가 설정되어 있습니다.
* `<button type="submit">수정하기</button>`: 폼을 제출하는 버튼입니다. 이 버튼을 클릭하면 사용자가 입력한 정보가 서버로 전송되어 수정됩니다.


### style.css

---

#### 공통 부분

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

a {
  text-decoration: none;
  color: #222;
}

.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 0 15px;
  position: relative;
}

header .login-box {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
}


.border-shadow {
  border:1px solid #eee;
  box-shadow: 1px 3px 10px #ccc;
}
.text-center {
  text-align: center;
}

header {
  width: 100%;

}
header nav {
  width: 100%;
  padding: 1em 0;
}

header nav a {
  font-size: 1.5em;
  font-weight: bold;
}
```

* `*`: 모든 요소에 대해 초기화를 위한 기본 스타일을 지정합니다.
* `a`: 모든 앵커(링크) 요소의 스타일을 정의합니다.
* `.container`: 페이지 내의 요소를 감싸기 위한 컨테이너의 스타일을 정의합니다.
* `header .login-box`: 헤더 내 로그인 상자의 위치 및 스타일을 정의합니다.
* `.border-shadow`: 그림자 효과를 갖는 테두리 스타일을 정의합니다.
* `.text-center`: 텍스트를 가운데 정렬하는 스타일을 정의합니다.
* `header, header nav`: 헤더와 그 내부 네비게이션 요소의 너비를 100%로 설정합니다.
* `header nav`: 네비게이션의 패딩을 설정합니다.
* `header nav a`: 네비게이션 링크의 스타일을 정의합니다.


#### 로그인 등록 폼, 조회에 사용되는 버튼 등의 요소

```css
/* list */
#site-main {
  width: 800px;
  margin: 6em auto 0;
}
.button-box {
  display:flex;
  justify-content: flex-end;
}
.button-box > a {
  font-size: 1em;
  padding: .5em 1em;
}

.button-box i {
  margin-right: .5em;
}

#site-main .container form {
  margin: 2em 0;
}

td .btn {
  color: #ccc;
}
```

* `#site-main`: 페이지의 주요 콘텐츠를 감싸는 요소의 스타일을 정의합니다. 너비는 800px이고, 상단 여백은 6em이며, 가운데 정렬됩니다.
* `.button-box`: 버튼을 감싸는 상자의 스타일을 설정합니다. 요소들을 행 방향으로 배치하고, 오른쪽 정렬합니다.
* `.button-box > a`: 버튼 요소의 스타일을 설정합니다. 글꼴 크기는 1em이고, 패딩은 .5em 위아래, 1em 좌우로 설정됩니다.
* `.button-box i`: 버튼 내 아이콘의 스타일을 설정합니다. 아이콘과 텍스트 사이의 간격은 .5em입니다.
* `#site-main .container form`: 주요 콘텐츠 내 폼 요소의 스타일을 설정합니다. 위아래 여백은 2em으로 설정됩니다.
* `td .btn`: 테이블 내 버튼 요소의 스타일을 설정합니다. 버튼 텍스트의 색상은 회색으로 설정됩니다.

#### 반응형 웹

```css
/* responsive table */
@media only screen and (max-width: 800px) {
  table, thead, tbody, th, td, tr {
    display: block;
  }

  thead tr {
    position: absolute;
    top: -9999px;
    left: -9999px;
  }

  tr {
    border: 1px solid #ddd;
  }

  td {
    border: none;
    border-bottom: 1px solid #ddd;
    position: relative;
  }
}
```
* `@media only screen and (max-width: 800px) { ... }`: 화면 너비가 800px 이하일 때 적용되는 스타일을 정의합니다.
* `table, thead, tbody, th, td, tr { display: block; }`: 테이블과 관련된 요소들을 블록 요소로 표시합니다.
* `thead tr { position: absolute; top: -9999px; left: -9999px; }`: 테이블 헤더 행을 화면 밖으로 이동시켜 보이지 않도록 합니다.
* `tr { border: 1px solid #ddd; }`: 각 행에 회색 테두리를 추가합니다.
* `td { border: none; border-bottom: 1px solid #ddd; position: relative; }`: 각 셀의 테두리를 없애고, 각 행의 마지막 셀의 아래쪽에 회색 테두리를 추가합니다. 위치는 상대적으로 설정됩니다.

#### 등록하기 추가 부분

```css
/* add */
#site-main h2 {
  font-size: 1.6em;
  margin-bottom: 1em;
  text-align: center;
}
#site-main p {
  color: #aaa;
  font-size: 1em;  
  margin: 1em auto 2em;
  text-align: center;
}

.user-info {
  max-width: 620px;
  margin: auto;
}

.user-info input[type="text"] {
  width: 100%;
  padding: .6em 1em;
  margin: .5em 0;
  border: 1px solid #ddd;
  font-size: 1em;
  border-radius: .2em;
}

#site-main form button:not([class="login-btn"]) {
  margin-top: 1em;
  width: 100%;
  padding: .5em 0;
  background-color: #eee;
  border:1px solid #ddd;
  transition: background-color .5s ease;
}

#site-main form button:hover {
  background-color: #222;
  color: #fff;
}
```

* `#site-main h2`: 주요 콘텐츠 내 제목의 스타일을 정의합니다. 글꼴 크기는 1.6em, 아래 여백은 1em, 가운데 정렬됩니다.
* `#site-main p`: 주요 콘텐츠 내 단락의 스타일을 정의합니다. 텍스트 색상은 회색(#aaa), 글꼴 크기는 1em, 여백은 위 1em, 아래 2em, 가운데 정렬됩니다.
* `.user-info`: 사용자 정보 입력 폼을 감싸는 상자의 스타일을 정의합니다. 최대 너비는 620px이고, 가운데 정렬됩니다.
* `.user-info input[type="text"]`: 사용자 정보 입력 필드의 스타일을 정의합니다. 너비는 100%, 패딩은 .6em 위아래, 1em 좌우, 여백은 .5em 위아래, 테두리는 회색(#ddd) 1px, 글꼴 크기는 1em, 테두리의 모서리는 둥글게 처리됩니다.
* `#site-main form button:not([class="login-btn"])`: 로그인 버튼이 아닌 모든 버튼의 스타일을 정의합니다. 위쪽 여백은 1em, 너비는 100%, 패딩은 .5em 위아래, 배경색은 회색(#eee), 테두리는 회색(#ddd) 1px입니다. 배경색이 변하는 효과가 있습니다.
* `#site-main form button:hover`: 버튼에 마우스를 올렸을 때의 스타일을 정의합니다. 배경색은 검은색(#222), 텍스트 색상은 흰색(#fff)로 변경됩니다.

#### 로그인 요소 추가

```css
/* Login Form */
.login, 
.register {
  width: 80%;
  margin: 30px auto;
}
.login label,
.register label {
  visibility: hidden;
}
.login input,
.register input {
  display: block;
  width: 60%;
  padding: 10px;
  margin-top: 25px;
  font-size: 15px;
  border: none;
  outline: none;
  border-bottom: 2px solid #eee;
}
.login-btn,
.register-btn {
  border:none;
}

hr {
  border: none;
  margin-top: 30px;
}
```

* `.login, .register`: 로그인 및 등록 폼의 스타일을 정의합니다. 너비는 화면의 80%로 설정되고, 가운데 정렬됩니다. 상단 여백은 30px입니다.
* `.login label, .register label`: 로그인 및 등록 폼 내 레이블의 스타일을 정의합니다. 레이블을 화면에서 숨깁니다.
* `.login input, .register input`: 로그인 및 등록 폼 내 입력 필드의 스타일을 정의합니다. 입력 필드는 블록 요소로 표시되며, 너비는 화면의 60%로 설정되고, 상단 여백은 25px입니다. 글꼴 크기는 15px이며, 테두리는 없고, 아래쪽 테두리만 회색(#eee) 2px으로 설정됩니다.
* `.login-btn, .register-btn`: 로그인 및 등록 버튼의 스타일을 정의합니다. 테두리는 없습니다.
* `hr`: 수평 선의 스타일을 정의합니다. 테두리가 없고, 상단 여백은 30px입니다.