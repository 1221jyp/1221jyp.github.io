---
title: "[Dreamhack] simple-ssti 해설"
author: jyp
date: 2026-06-09 12:00:07 +0900
categories: [Dreamhack]
tags: [dreamhack , WebHacking]
math: true
---

# 문제 특징

> SSTI 기법을 활용한 해킹

## **난이도**: Lv.1
## **카테고리**: 웹해킹

---

# 문제 구조

## 문제파일구성

```
/
└── app2.py
```

## 주요 코드 

```python

#app2.py

@app.route('/')
def index():
    return render_template('index.html')

@app.errorhandler(404)
def Error404(e):
    template = '''
    <div class="center">
        <h1>Page Not Found.</h1>
        <h3>%s</h3>
    </div>
''' % (request.path)
    return render_template_string(template), 404
```

이 문제는 Flask 웹 어플리케이션 서버의 특성/취약점을 활용한 문제이다.


# 풀이

## SSTI란?

**SSTI (Server-Side Template Injection, 서버 사이드 템플릿 인젝션) 는 사용자 입력이 서버의 템플릿 엔진에서 코드로 실행되어 버리는 취약점입니다.**

## 경로에 Jinja2 명령어 넣기

이 웹 서버는 `Flask`라는 라이브러리를 통해 만들어졌고, 이 라이브러리는 `Jinja2`라는 템플릿 엔진을 쓴다.
`Jinja2`의 특성은 `{{}}`안의 값을 계산하여 가져온다는 점이다. 따라서 존재하지 않는 주소에 접속했을때, `404`오류와 함께 해당 경로를 띄워주는 이 사이트에서 경로를 `/{{}}`로 설정하고 이 속에 페이로드를 작성하는 것이다.


## 페이로드

http://타겟주소/{{config['SECRET_KEY']}}


# 정답

DH{6c74aac721d128c637eab3f11906a44b}