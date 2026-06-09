---
title: "[Dreamhack] CSRF-Advanced 해설"
author: jyp
date: 2026-06-09 12:00:04 +0900
categories: [Dreamhack]
tags: [dreamhack , WebHacking]
math: true
---

# 문제 특징

> CSRF 해킹 기법을 이용하여 어드민 비밀번호 바꾸기

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
    driver = None
    try:
        options = webdriver.ChromeOptions()
        options.binary_location = CHROMIUM_BIN
        for argument in [
            "--headless",
            "--window-size=1920,1080",
            "--disable-gpu",
            "--no-sandbox",
            "--disable-dev-shm-usage",
        ]:
            options.add_argument(argument)
        service = Service(executable_path=CHROMEDRIVER_PATH)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:8000/")
        driver.add_cookie(cookie)
        driver.get(url)
    except Exception as e:
        print(str(e))
        return False
    finally:
        if driver is not None:
            driver.quit()
    return True


def check_csrf(param, cookie={"name": "name", "value": "value"}):
    url = f"http://127.0.0.1:8000/vuln?param={urllib.parse.quote(param)}"
    return read_url(url, cookie)


@app.route("/")
def index():
    session_id = request.cookies.get('sessionid', None)
    try:
        username = session_storage[session_id]
    except KeyError:
        return render_template('index.html', text='please login')

    return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not an admin"}')


@app.route("/vuln")
def vuln():
    param = request.args.get("param", "").lower()
    xss_filter = ["frame", "script", "on"]
    for _ in xss_filter:
        param = param.replace(_, "*")
    return param


@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.html")
    elif request.method == "POST":
        param = request.form.get("param", "")
        session_id = os.urandom(16).hex()
        session_storage[session_id] = 'admin'
        if not check_csrf(param, {"name":"sessionid", "value": session_id}):
            return '<script>alert("wrong??");history.go(-1);</script>'

        return '<script>alert("good");history.go(-1);</script>'


@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    elif request.method == 'POST':
        username = request.form.get('username')
        password = request.form.get('password')
        try:
            pw = users[username]
        except:
            return '<script>alert("not found user");history.go(-1);</script>'
        if pw == password:
            resp = make_response(redirect(url_for('index')) )
            session_id = os.urandom(8).hex()
            session_storage[session_id] = username
            resp.set_cookie('sessionid', session_id)
            return resp 
        return '<script>alert("wrong password");history.go(-1);</script>'


@app.route("/change_password")
def change_password():
    pw = request.args.get("pw", "")
    session_id = request.cookies.get('sessionid', None)
    try:
        username = session_storage[session_id]
    except KeyError:
        return render_template('index.html', text='please login')

    users[username] = pw
    return 'Done'


```

이 문제는 문제의 제목 그대로 `CSRF` 해킹 기법을 활용하여 푸는 문제로, 필터가 엄격하게 걸려있어 [XSS-2](https://1221jyp.com/posts/Dreamhack_Xss-2/)와 같이 `XSS` 해킹 기법으로는 뚫어내기 쉽지 않다.

# 풀이

## CSRF란? By claude code

CSRF (Cross-Site Request Forgery, 사이트 간 요청 위조) 는 피해자가 자기 의도와 상관없이 본인 권한으로 요청을 보내게 만드는 공격입니다.

공격 흐름

1. 피해자가 bank.com 에 로그인 (쿠키 보유)
2. 공격자가 만든 악성 페이지 방문
   <img src="http://bank.com/송금?to=공격자&amount=100만">
3. 피해자 브라우저가 bank.com 쿠키 자동 첨부해서 요청
4. 은행: "로그인된 사용자의 정상 요청이네" → 송금 실행

## 어드민 비밀번호 바꾸기

```python
def change_password():
    pw = request.args.get("pw", "")
    session_id = request.cookies.get('sessionid', None)
    try:
        username = session_storage[session_id]
    except KeyError:
        return render_template('index.html', text='please login')

    users[username] = pw
    return 'Done'
```

`/change_password`주소에서 비밀번호를 변경할때, GET요청으로 `pw`쿼리의 파라미터만 입력하면 되므로, `/vuln`주소를 이용해 어드민의 비밀번호를 손쉽게 변경할 수 있다.

## 페이로드

```html
<img src="/change_password?pw=1234">
```

해당 페이로드를 `/flag`의 `<input>` 박스에 넣고 전송하면 어드민의 비밀번호가 바뀌게 된다.

## 어드민 로그인

이제 로그인 창으로 이동하여
id : admin
pw : 1234
를 입력해보자.


# 정답

DH{c57d0dc12bb9ff023faf9a0e2b49e470a77271ef}
