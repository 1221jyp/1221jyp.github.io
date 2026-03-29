---
title: "[Dreamhack] MisconFlag 해설"
author: jyp
date: 2026-03-29 18:20:00 +0900
categories: [Dreamhack]
tags: [dreamhack , WebHacking]
math: true
---

## 문제 정보

> APACHE 서버의 특징을 이용한 해킹

### **난이도**: Lv.1
### **카테고리**: 웹해킹

---

## 풀이

### 문제파일구성

```
/
├── Dockerfile
└── src/
    ├── check.php
    ├── flag.txt
    ├── index.php
    ├── style.css
    └── token.php
```

### dockerfile

```dockerfile
FROM php:8.2-apache
COPY src/ /var/www/html/
```

`dockerfile`을 바탕으로 생성된 도커 컨테이너에는 php + apache가 설치 및 실행되어있고, `/src`폴더의 내용물을 리눅스의 `/var/www/html/` 주소로 복사한다.

```
/
└── var/
    └── www/
        └── html/
            ├── check.php
            ├── flag.txt
            ├── index.php
            ├── style.css
            └── token.php
```

### apache의 특성

apache는 설치되자마자 `/var/www/html/` 루트 속 파일들을 정적으로 서빙하는데, 이때 문제가 발생한다.
사이트에 접속한 뒤 `/html` 루트속 파일들을 주소에 입력시 그 파일들의 내용물이 드러나게 된다.
ex) `http://사이트주소/style.css` 입력시 스타일시트 내용물이 그대로 나옴.
따라서 주소에 `http://사이트주소/flag.txt` 를 입력시 바로 플래그를 탈취할 수 있다.

## 정답

`http://사이트주소/flag.txt`











