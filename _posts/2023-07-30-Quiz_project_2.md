---
title: 퀴즈 사이트 만들기_2 (next.js)
author: jyp
date: 2023-07-25 11:33:00 +0900
categories: [MacOs, Next.js, javascript]
tags: [Quiz_project]
math: true
mermaid: true
---

---

# 시작하며

---

지난 포스팅에서는 next.js를 설치하고, 레이아웃까지 제작하였다. 이번 포스팅에서는
나머지 페이지들의 내용을 구성해볼것이다.

> 저의 모든 글은 MacOs silicon을 기준으로 작성됩니다.

# 메인 페이지 구성

---

메인 페이지에는 이 웹사이트에 있는 기능들과 사용법을 소개할 것입니다.
아직 저의 프로젝트는 완성되어있지 않고, 계속 수정될 수 있기 때문에 비워놓고 다른 페이지부터
제작하겠습니다.

# 로딩 페이지 구성

---

로딩중일때 띄워질 페이지를 구성해 봅시다. app폴더 안에 `loading.js`파일을 생성하면,
로딩중일때 `loading.js`파일의 내용을 띄워 줍니다. chatgpt를 사용해서 레이아웃을 로딩 애니메이션을
만들어달라 부탁해보았습니다.

```jsx
//src/app/loading.js
import "./globals.css";
export default function LoadingPage() {
  return (
    <div className="loading">
      <div className="loading-container">
        <div className="loading-spinner"></div>
        <p className="loading-text">Loading...</p>
      </div>
    </div>
  );
}
```

```css
/*src/app/globals.css*/

.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
  background-color: #f0f0f0;
}

.loading-container {
  text-align: center;
  font-family: Arial, sans-serif;
}

.loading-spinner {
  border: 4px solid rgba(0, 0, 0, 0.3);
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin: 0 auto 10px;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.loading-text {
  color: #333;
  font-size: 18px;
}
```

배보다 배꼽이 큰 기능을 만든 것 같지만, next.js의 기능들을 사용해보고 싶어 추가해보았습니다.

# 퀴즈 만들기 페이지 구성

---

## 만들기

```jsx
//src/app/newquiz/page.js

export default function Newquiz() {
  //map함수 사용을 위한 array 자료형
  let multiple_choice = [1, 2, 3, 4, 5];
  //체크박스중 하나만 선택되게 하는 함수
  const onlyone = (checkThis) => {
    const checkboxes = document.getElementsByName("a");
    for (let i = 0; i < checkboxes.length; i++) {
      if (checkboxes[i] !== checkThis) {
        checkboxes[i].checked = false;
      }
    }
  };
  return (
    <div>
      {multiple_choice.map((a, i) => (
        <>
          <input
            name={"q" + multiple_choice[i]}
            placeholder={multiple_choice[i] + "번 문제"}
          ></input>
          <input
            type="checkbox"
            name="a"
            value={"a" + multiple_choice[i]}
            onChange={(e) => onlyone(e.target)}
          ></input>
          <br></br>
        </>
      ))}
    </div>
  );
}
```

> Error: document is not defined

## 오류 발생?

newquiz란 폴더를 app폴더 안에 새로 생성한 뒤, 퀴즈를 만들때 사용할 페이지의 내용을 담았다.
![구상도](https://github.com/1221jyp/Quiz_project/assets/98996860/eb9d933b-a6b7-4a42-80db-38c199403c8a)

위 사진처럼 질문 입력박스와 그 왼편에 체크박스를 두어 5개의 선다중 정답인 것을 체크할 수 있게 하는 기능을 넣었는데,
정답은 1개이므로, 1개만 체크할수 있게 js로 html제어 기능을 사용하였더니 위의 오류가 출력되었다.

## 오류 해결

이러한 오류가 발생하는 이유는 next.js는 SSR (Server Side Rendering)방식으로 구동되기 때문입니다.
서버에서 렌더링하여 완성된 파일을 클라이언트에 전송하는방식이 SSR인데, 이러한 방식으로 작동되기 때문에
client에서 실행되는 html제어가 사용되지 않는 것이다.  
문제를 해결하기 위해선 SSR이 아닌 CSR을 사용하면 됩니다. html제어가 사용되는 부분을 새로운 파일로 떨어뜨려놓고
그 파일을 CSR로 구동되게 선언하면 해결됩니다.

```jsx
//src/app/newquiz/page.js

import Checkbox from "./Checkbox";
export default function Newquiz() {
  return (
    <>
      <div className="container">
        <Checkbox></Checkbox> //next.js의 컴포넌트 문법을 통해 CSR과 SSR을 분리합니다.
      </div>
      <div className="howto">
        <h2>퀴즈 작성하는법</h2>
        <h4>1. 문제의 내용을 입력해줍니다.</h4>
        <h4>2. 5문항 모두 문제를 작성해줍니다.</h4>
        <h4>3. 정답인 문항의 왼쪽에 체크표시를 해줍니다.</h4>
      </div>
    </>
  );
}
```

```jsx
//src/app/newquiz/Checkbox.js

"use client";

import "./Checkbox.css"; // Import the CSS file for styling

export default function Checkbox() {
  let multiple_choice = [1, 2, 3, 4, 5];

  const onlyone = (checkThis) => {
    const checkboxes = document.getElementsByName("a");
    for (let i = 0; i < checkboxes.length; i++) {
      if (checkboxes[i] !== checkThis) {
        checkboxes[i].checked = false;
      }
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div className="checkbox-container">
        <h1>퀴즈 만들기</h1>
        <input
          name="q"
          type="text"
          placeholder="문제의 내용을 입력하세요."
          autoComplete="off"
        ></input>
        {multiple_choice.map((a, i) => (
          <div className="checkbox-row" key={i}>
            <div className="num">{multiple_choice[i] + ")"}</div>
            <input
              type="text"
              name={"c" + multiple_choice[i]}
              placeholder={multiple_choice[i] + "번 문제"}
            />
            <input
              type="checkbox"
              name="a"
              value={"a" + multiple_choice[i]}
              onChange={(e) => onlyone(e.target)}
            />
          </div>
        ))}
        <button type="submit" className="submit-button">
          제출하기
        </button>
      </div>
    </form>
  );
}
```

제일 중요한 문제를 적는 박스를 넣는것을 까먹어 수정하고, 다른부분도 이것저것 수정했습니다.
이렇게 고치고 난 뒤, css는 또 다시 우리의 조수 chatgpt를 통해 해결해봅시다.

```css
/* src/app/newquiz/Checkbox.css */

/* 기본 컨테이너 스타일 */
.checkbox-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 20px;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #f5f5f5;
}

/* 제목 스타일 */
h1 {
  font-size: 24px;
  margin-bottom: 20px;
}

/* 입력 필드 스타일 */
input {
  margin: 5px 0;
  padding: 8px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

/* 체크박스 스타일 */
input[type="checkbox"] {
  margin-right: 10px;
  scale: 1.5;
}
input[type="text"] {
  margin-right: 10px;
  width: 500px;
}

/* 제출 버튼 스타일 */
.submit-button {
  margin-top: 20px;
  padding: 10px 20px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.submit-button:hover {
  background-color: #0056b3;
}

.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  margin: 0;
  background-color: #f0f0f0;
}

.loading-container {
  text-align: center;
  font-family: Arial, sans-serif;
}

.loading-spinner {
  border: 4px solid rgba(0, 0, 0, 0.3);
  border-top: 4px solid #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
  margin: 0 auto 10px;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.loading-text {
  color: #333;
  font-size: 18px;
}
```

# 퀴즈 풀기 페이지 구성

---

## mongodb 가입하기 & DB만들기

퀴즈 풀기 페이지는 mongodb연동부터 시작하겠습니다.
가장 먼저 [mongodb](https://www.mongodb.com/ko-kr)에 가입해줍니다.
저는 이미 mongodb에 가입해놓았기 때문에 새로운 부계정을 만들어보겠습니다.
![스크린샷 2023-08-01 오후 6 35 08](https://github.com/1221jyp/Quiz_project/assets/98996860/44fe1214-d7bc-4ed0-a168-23f98213b185)
가입이 어찌저찌 완료되면 이러한 화면이 뜹니다. 가장 오른쪽에 있는 무료 요금제를 선택해준 뒤에,
밑에 provider는 `aws`를 선택해주고, 지역은 서울 아니면 도쿄를 선택해줍니다.
![스크린샷 2023-08-01 오후 6 37 53](https://github.com/1221jyp/Quiz_project/assets/98996860/735789f1-4d39-4e4a-95ce-f00ab36b0cee)
그런 뒤에 생성을 해주고, 다음 창에서 이름과 비밀번호를 대충 만들어줍시다.
![스크린샷 2023-08-01 오후 6 38 36](https://github.com/1221jyp/Quiz_project/assets/98996860/99ca7519-f55e-4e60-ad8a-247060bc9cd2)
그런 뒤에 밑에 창에서는 ` 0.0.0.0`` 의 IP주소를 추가해줍니다. 이렇게 되면 모든 주소에서 DB를 사용할수 있게 됩니다.
원하지 않는다면, 본인 컴퓨터의 IP주소만 추가해주셔도 됩니다.
![스크린샷 2023-08-01 오후 6 40 23](https://github.com/1221jyp/Quiz_project/assets/98996860/f053c1ec-c74f-4ff8-8b5a-d3caea84f6f5)
정상적으로 완성되었다면, 이러한 창이 나오게 됩니다. 이때  `Browse Collections`를 눌러 들어가줍시다.
그런 뒤에 `Create Database`버튼을 눌러주고 아무렇게나 작명한뒤에 추가해줍니다.
![스크린샷 2023-08-01 오후 7 01 44](https://github.com/1221jyp/Quiz_project/assets/98996860/e7a1ffec-5ba7-408f-a891-af15f751f04c)
DB가 정상적으로 만들어졌다면, 이러한 화면이 뜨는데, 여기서 `insert document`를 새로 만들어줍니다.
![스크린샷 2023-08-01 오후 11 07 54](https://github.com/1221jyp/Quiz_project/assets/98996860/b06d5c57-4b31-489b-9f59-2240cff252e7)

저는 문제의 질문 한개, 객관식 5개, 정답 1개이므로 총 7개의 테이블을 추가로 만들었습니다.
저희는 문제 20개 풀이를 할것이므로, mongodb연결을 하고 퀴즈 만들기 창에서 퀴즈 20개를 제작해봅시다.

## mongodb 연결하기

```terminal
npm install mongodb
```

가장 먼저 터미널에서 mongodb를 불러오기 위해 라이브러리를 설치해줍니다.

```js
//src/util/db.js
import { MongoClient } from "mongodb";

require("dotenv").config();
const url = process.env.mongodb;
const options = { useNewUrlParser: true };
let connectDB;

if (process.env.NODE_ENV === "development") {
  if (!global._mongo) {
    global._mongo = new MongoClient(url, options).connect();
  }
  connectDB = global._mongo;
} else {
  connectDB = new MongoClient(url, options).connect();
}
export { connectDB };
```

`src` 폴더안에 `util`폴더를 만들고 그 안에 `db.js`파일을 만들고 그곳에 위 코드를 입력해주었습니다.
url코드는 dotenv를 설치하여 안보이게 해놓았지만, 위`process.env.mongodb`부분을 바꿔주어야 합니다.

다시 이곳으로 돌아와서 `Connect`버튼을 누르고

제일 상단의 Driver버튼을 누른 뒤에

그곳에 있는 자신의 mongodb주소를 `process.env.mongodb`와 바꿔주어야 합니다.
이때, `<password>`는 자신이 계정을 가입할때 만들어둔 비밀번호를 입력하면 됩니다.(<>기호 떼고 넣으셔야 합니다.)
이렇게 되면 mongodb와 연결이 완료되었습니다. 테스트를 위해 post.js에 간단한 코드를 작성해보겠습니다.

```jsx
export default async function Newquiz() {
  const client = await connectDB;
  const db = client.db("Quiz_Data");
  let result = await db.collection("Quiz").find().toArray();
  console.log(result);
  return(
    //~~~
  )
}
```

`return()`위에 해당 명령어를 입력하여 터미널에 정상적으로 데이터들이 출력되는지 확인해봅니다.  
<img width="1470" alt="스크린샷 2023-08-01 오후 9 40 42" src="https://github.com/1221jyp/Quiz_project/assets/98996860/82db8486-b9eb-44b7-99fd-1fbe26b67ffc">
터미널에서 정상적으로 작동되는 모습

# 퀴즈 만들기 api 개발

---

이제 api를 통하여 퀴즈 만들기 페이지에서 퀴즈를 작성하고 제출을 누르면, DB에 해당 문제가 저장되게 만들어야합니다.
가장먼저 `src`폴더 안에 `pages`폴더를 만들고 그 안에 `api`폴더를 만들어주었습니다.
이제 이곳에 js파일을 생성하여 api코드를 작성해주면 됩니다.

그 전에 퀴즈 만들기 페이지에서 제출하기 버튼을 눌렀을때,
문제를 넣는 칸이 비어있다면 주의 문자를 주는 기능부터 작성해봅시다.

```jsx
//src/app/newquiz/Checkbox.js

//위부분 생략
const handleSubmit = (event) => {
  event.preventDefault(); // 기본 제출 동작 방지

  const QuestionInput = event.target.elements.q;
  const inputElements = event.target.elements;
  const checkboxes = document.getElementsByName("a");

  // "c"가 포함된 "name"을 가진 input 중 하나라도 비어있는지 확인
  let isAnyInputEmpty = false;
  for (let i = 0; i < inputElements.length; i++) {
    const inputName = inputElements[i].name;
    if (inputName && inputName.includes("c") && !inputElements[i].value) {
      isAnyInputEmpty = true;
      break;
    }
  }
  // 체크박스중 하나라도 체크가 되어있는지 확인
  let isAnyCheckboxChecked = false;
  for (let i = 0; i < checkboxes.length; i++) {
    if (checkboxes[i].checked) {
      isAnyCheckboxChecked = true;
      break;
    }
  }

  if (!QuestionInput.value) {
    // 문제 내용이 비어있는경우
    alert("문제 내용을 입력해주세요!");
  } else if (isAnyInputEmpty) {
    // 문제가 비어있는경우
    alert("문제가 비었습니다!");
  } else if (!isAnyCheckboxChecked) {
    //체크박스가 비어있는경우
    alert("체크박스를 체크하여 정답을 선택하세요!");
  } else {
    event.target.submit(); // 제출
  }
};

return (
  <form action="/api/write" method="POST" onSubmit={handleSubmit}>
    <div className="checkbox-container">
      <h1>퀴즈 만들기</h1>
      <input
        name="q"
        type="text"
        placeholder="문제의 내용을 입력하세요."
        autoComplete="off"
      ></input>
      {multiple_choice.map((a, i) => (
        <div className="checkbox-row" key={i}>
          <div className="num">{multiple_choice[i] + ")"}</div>
          <input
            type="text"
            name={"c" + multiple_choice[i]}
            placeholder={multiple_choice[i] + "번 문제"}
          />
          <input
            type="checkbox"
            name="a"
            value={"a" + multiple_choice[i]}
            onChange={(e) => onlyone(e.target)}
          />
        </div>
      ))}
      <button type="submit" className="submit-button">
        제출하기
      </button>
    </div>
  </form>
);
```

form태그에 api요청을 보낼 주소도 적어주면, 이제 글 작성페이지는 완성입니다.
`pages/api/`주소에 만들어둔 js파일을 통해 api개발을 시작해봅시다.

```js
//src/pages/api/write.js

import { connectDB } from "@/util/DB";
export default async function handler(req, res) {
  try {
    console.log(req.body);
    const db = (await connectDB).db("Quiz_Data");
    db.collection("Quiz").insertOne(req.body);
    //작업이 끝나면 메인 페이지로 이동
    res.redirect(302, "/");
    //오류발생시 실행될 함수
  } catch (error) {
    return res.status(500).json("서버 오류");
  }
}
```

모든 퀴즈 작성기능이 완성되었습니다. 이제 퀴즈 만들기에 들어가서 퀴즈를 작성하면 DB에 퀴즈가 등록되는것을 확인할 수 있습니다.
이번 포스팅은 여기서 마치고, 다음 포스팅에서는 퀴즈 풀기 기능을 구현해보겠습니다. 읽어주셔서 감사합니다.
