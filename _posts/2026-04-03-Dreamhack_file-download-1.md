---
title: "[Dreamhack] file-download-1 해설"
author: jyp
date: 2026-04-03 12:00:00 +0900
categories: [Dreamhack]
tags: [dreamhack , WebHacking]
math: true
---

# 문제 정보

> Querystring과 request라이브러리를 이용한 해킹

**[Path Traversal](/posts/Dreamhack_pathtraversal/)문제와 광장히 유사하여 먼저 [Path Traversal](/posts/Dreamhack_pathtraversal/)문제를 보고 오시는것을 권장드립니다.**

## **문제 이름**: file-download-1
## **난이도**: Lv.Beginner
## **카테고리**: 웹해킹

---

# 문제 구조

## 문제파일구성

```
.
├── __pycache__
│   └── flag.cpython-314.pyc
├── app.py
├── requirements.txt
├── static
│   ├── (생략)
├── templates
│   ├── base.html
│   ├── index.html
│   ├── read.html
│   ├── upload_result.html
│   └── upload.html
└── uploads

```

`static` 폴더 속에는 폰트, css파일 등 중요한 내용이 없어 생략했다.

## 주요 코드

```python
//app.py (일부)

from flag import FLAG

APP = Flask(__name__)

UPLOAD_DIR = 'uploads'


@APP.route('/')
def index():
    files = os.listdir(UPLOAD_DIR)
    return render_template('index.html', files=files)


@APP.route('/upload', methods=['GET', 'POST'])
def upload_memo():
    if request.method == 'POST':
        filename = request.form.get('filename')
        content = request.form.get('content').encode('utf-8')

        if filename.find('..') != -1:
            return render_template('upload_result.html', data='bad characters,,')

        with open(f'{UPLOAD_DIR}/{filename}', 'wb') as f:
            f.write(content)

        return redirect('/')

    return render_template('upload.html')


@APP.route('/read')
def read_memo():
    error = False
    data = b''

    filename = request.args.get('name', '')

    try:
        with open(f'{UPLOAD_DIR}/{filename}', 'rb') as f:
            data = f.read()
    except (IsADirectoryError, FileNotFoundError):
        error = True


    return render_template('read.html',
                           filename=filename,
                           content=data.decode('utf-8'),
                           error=error)


if __name__ == '__main__':
    if os.path.exists(UPLOAD_DIR):
        shutil.rmtree(UPLOAD_DIR)

    os.mkdir(UPLOAD_DIR)

    APP.run(host='0.0.0.0', port=8000)

```
`python`의 `import`구문을 통해 `FLAG`를 가져오므로, `FLAG`는 `/flag.py`에 위치함을 알 수 있다.

# 풀이

## 쿼리 스트링과 requests.get() 함수 활용하기

[Path Traversal](/posts/Dreamhack_pathtraversal/)문제와 마찬가지로, Querystring과 request라이브러리 함수를 이용해서 풀 수 있는 문제다.

```python
@APP.route('/upload', methods=['GET', 'POST'])
def upload_memo():
    if request.method == 'POST':
        filename = request.form.get('filename')
        content = request.form.get('content').encode('utf-8')

        if filename.find('..') != -1:
            return render_template('upload_result.html', data='bad characters,,')

        with open(f'{UPLOAD_DIR}/{filename}', 'wb') as f:
            f.write(content)

        return redirect('/')

    return render_template('upload.html')

@APP.route('/read')
def read_memo():
    error = False
    data = b''

    filename = request.args.get('name', '')

    try:
        with open(f'{UPLOAD_DIR}/{filename}', 'rb') as f:
            data = f.read()
    except (IsADirectoryError, FileNotFoundError):
        error = True


    return render_template('read.html',
                           filename=filename,
                           content=data.decode('utf-8'),
                           error=error)
```

허나 이전 [Path Traversal](/posts/Dreamhack_pathtraversal/)문제와 달리, Querystring에 주소를 입력하여 값을 가져올 수 있는 path가 두개 존재하지만, `/upload`path에서는 Querystring에 ..이 있을 경우 다른 행동을 하게 하는 조건문이 존재하여 원하는 장소인 `/flag.py`로 접속할 수 없다.

하지만 `/read`path에는 그러한 예외처리가 없으므로, Querystring에 path를 입력하여 문제를 해결할 수 있다.

```
http://<서버주소>/read?name=../path.py
```

브라우저에 해당 요청을 전송하면 손쉽게 문제를 해결할 수 있다.

# 정답

```
http://<서버주소>/read?name=../path.py
```


