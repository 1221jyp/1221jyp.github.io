---
title: 퀴즈 사이트 만들기_3 (next.js)
author: jyp
date: 2023-08-02 11:33:00 +0900
categories: [MacOs, Next.js, javascript]
tags: [Quiz_project]
math: true
mermaid: true
---

# 시작하며

---

지난 포스팅에서는 퀴즈 만들기 페이지와, API까지 완성시켰다. 이번 포스팅에서는
퀴즈 풀기 기능을 완성시켜볼 예정이다.

# 퀴즈 풀기 페이지 구성하기

---

## 페이지 구성

가장 먼저 퀴즈 풀기 버튼을 누르면, 문제의 카테고리를 선택할 수 있는 창을 만들어주겠습니다.
아직 한국사 문제밖에 없지만, 새로운 카테고리가 생길수도 있기때문에 만들어놓겠습니다.

```jsx
//src/app/quiz/page.js

import "./style.css";

export default function Quiz() {
  return (
    <div className="container">
      <a href={"/quiz_h"} className="button">
        한국사 문제풀기
      </a>
    </div>
  );
}
```

```css
/* src/app/quiz/style.css */

.container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.button {
  display: inline-block;
  padding: 10px 20px;
  border: 1px solid #ccc;
  border-radius: 4px;
  text-decoration: none;
  color: #333;
  font-size: 16px;
}
```

그러고 나서 이제 이 프로젝트의 핵심 페이지인 퀴즈 풀기 기능을 만들어 보겠습니다.
<br>
<br>
<br>

```jsx
//src/app/quiz_h/page.js

export default async function Quiz_h() {
  const client = await connectDB;
  const db = client.db("Quiz_Data"); //mongodb 불러오기
  let NumberOfQuestion = await db.collection("Quiz").count(); //Quiz collection 안에 있는 데이터의 개수 불러오기
  let result = await db.collection("Quiz").find().toArray(); //Quiz collection 안에 있는 모든 데이터 불러오기

  //_id를 문자열로 변환
  result = result.map((a) => {
    a._id = a._id.toString();
    return a;
  });

  //DB에 저장되어있는 문제 개수만큼 숫자 생성
  const numbers = [];
  for (let i = 0; i < NumberOfQuestion; i++) {
    numbers.push(i);
  }

  //숫자 중 랜덤한 20개의 숫자 추출
  const answer = [];
  for (let n = 0; n < 20; n++) {
    const index = Math.floor(Math.random() * numbers.length);
    answer.push(numbers[index]);
    numbers.splice(index, 1);
  }
  return (
    {/*20개의 랜덤 숫자와 DB자료 client 컴포넌트에 전송*/}
    <>
      <Util answer={answer} result={result}></Util>
    </>
  );
}
```

```jsx
//src/app/quiz_h/util.js

export default async function Util({ result, answer }) {
  //체크박스 하나만 체크되게 하기
  const onlyone = (checkThis) => {
    const checkboxes = document.getElementsByClassName("q");
    for (let i = 0; i < checkboxes.length; i++) {
      if (checkboxes[i] !== checkThis) {
        checkboxes[i].checked = false;
      }
    }
  };

  useEffect(() => {
    // 변수/상수 선언
    const outer = document.querySelector(".outer");
    const innerList = document.querySelector(".inner-list");
    const inners = document.querySelectorAll(".inner");
    const loading = document.getElementById("load");
    const buttonRight = document.querySelector(".button-right");
    const showScore = document.querySelector("#score");
    const showResult = document.querySelector(".black-bg");
    const submitButton = document.querySelector(".submitButton");

    var count = 0;
    var score = 0;
    var IsitCorrect = new Object();

    //다음문제 버튼을 클릭하였을시 실행되는 함수
    buttonRight.addEventListener("click", () => {
      const parentComponent = document.getElementById(count);
      const countCheckbox = parentComponent.querySelectorAll('input[type="checkbox"]');

      let isChecked = false;

      // 모든 체크박스를 반복하며 체크 상태를 확인합니다.
      for (let i = 0; i < countCheckbox.length; i++) {
        if (countCheckbox[i].checked) {
          isChecked = true;
          break; // 하나라도 체크되어 있으면 루프를 종료합니다.
        }
      }

      //하나도 체크되어 있지 않다면
      if (!isChecked) {
        alert("체크박스를 체크하여 정답을 선택하세요!");
      }
      //그렇지 않으면
      else {
        //다음 문제로 넘기기
        currentIndex++;
        currentIndex = currentIndex >= inners.length ? inners.length - 1 : currentIndex;
        innerList.style.marginLeft = `-${outer.clientWidth * currentIndex}px`;

        //체크된 체크박스값 보기
        const checkboxes = document.querySelectorAll('input[type="checkbox"]:checked');
        const checkedValues = Array.from(checkboxes).map((checkbox) => checkbox.value);

        //19번째 문제에 도달했을때, 20번째 문제의 버튼은 '제출'로 보이게 하기
        if (count >= 18) {
          buttonRight.innerText = "제출";
        }
        //마지막 문제까지 제출하면, 결과 보여주기
        if (count == 19) {
          showResult.classList.add("show-modal");
        }

        //체크박스에 입력한 문제가 정답이라면 answer에 1을 넣고, 스코어 추가
        else if (checkedValues == result[answer[count]].a) {
          IsitCorrect["answer" + count] = 1;
          score++;
          showScore.innerText = `${score * 5}`;
        }
        //오답시 answer에 0을 넣기
        else {
          IsitCorrect["answer" + count] = 0;
        }
        //카운트 추가
        count++;
      }
    });
    //마지막 제출버튼 눌렀을때 api/result로 POST 요청 (fetchAPI)
    submitButton.addEventListener("click", async () => {
      console.log(IsitCorrect);
      IsitCorrect["result"] = score * 5;
      await fetch("/api/result", {
        method: "POST",
        body: JSON.stringify(IsitCorrect),
      }).then(() => {
        console.log("success");
        alert("제출완료");
      });
    });

    //문제 개수만큼 div박스 늘리기
    let currentIndex = 0;

    if (outer && innerList && inners) {
      inners.forEach((inner) => {
        inner.style.width = `${outer.clientWidth}px`;
      });

      innerList.style.width = `${outer.clientWidth * inners.length}px`;
    }

    //로딩이 완료되면 로딩창 숨기기
    const onPageLoad = () => {
      loading.style.display = "none";
    };
    document.onreadystatechange = function () {
      if (document.readyState === "complete") {
        onPageLoad();
      } else {
        window.addEventListener("load", onPageLoad, false);
        // Remove the event listener when component unmounts
        return () => window.removeEventListener("load", onPageLoad);
      }
    };
  }, []);
  return (
    <>
      <div id="load">
        <LoadingPage></LoadingPage>
      </div>
      <div className="black-bg">
        <div className="white-bg">
          <div className="my-score">
            <h1>당신의 점수는</h1>
            <h1 id="score"></h1>
            <h1>점 입니다.</h1>
            <button className="submitButton btn ">결과 제출하기</button>
            <a href="/" className="submitButton btn ">
              돌아가기
            </a>
          </div>
        </div>
      </div>
      <div className="outer">
        <div className="inner-list">
          {answer.map((a, i) => (
            <div key={a} id={i}>
              <div className="inner">
                <h1 className="question">
                  {i + 1}번문제) {result[a].q}
                </h1>
                <div className="contentbox">
                  <div className="content">{result[a].c1}</div>
                  <input
                    type="checkbox"
                    value={"1"}
                    className="q"
                    name={i}
                    onChange={(e) => onlyone(e.target)}
                  ></input>
                </div>
                <div className="contentbox">
                  <div className="content">{result[a].c2}</div>
                  <input
                    type="checkbox"
                    value={"2"}
                    className="q"
                    name={i}
                    onChange={(e) => onlyone(e.target)}
                  ></input>
                </div>
                <div className="contentbox">
                  <div className="content">{result[a].c3}</div>
                  <input
                    type="checkbox"
                    value={"3"}
                    className="q"
                    name={i}
                    onChange={(e) => onlyone(e.target)}
                  ></input>
                </div>
                <div className="contentbox">
                  <div className="content">{result[a].c4}</div>
                  <input
                    type="checkbox"
                    value={"4"}
                    className="q"
                    name={i}
                    onChange={(e) => onlyone(e.target)}
                  ></input>
                </div>
                <div className="contentbox">
                  <div className="content">{result[a].c5}</div>
                  <input
                    type="checkbox"
                    value={"5"}
                    className="q"
                    name={i}
                    onChange={(e) => onlyone(e.target)}
                  ></input>
                </div>
              </div>
            </div>
          ))}
        </div>
      </div>

      <div className="button-list">
        <button className="button-right btn">다음문제</button>
      </div>
    </>
  );
}
```

```css
/* src/app/quiz_h/style.css */

input[type="checkbox"] {
  width: 1rem;
  height: 1rem;
  border-radius: 50%;
  border: 1px solid #999;
  appearance: none;
  cursor: pointer;
  transition: background 0.2s;
  display: flex;
  align-items: center;
  scale: 1.5;
}

input[type="checkbox"]:checked {
  background: #32e732;
  border: none;
}

button {
  border: none;
  background-color: transparent;
  padding: 0;
  margin: 0;
  display: inline-block;
  cursor: pointer;
  text-decoration: none;
  font-size: inherit;
  font-family: inherit;
  color: inherit;
  outline: none;
  box-shadow: none;
}

.btn {
  background-color: #3498db;
  color: #fff;
  border-radius: 5px;
  padding: 10px 20px;
  transition: background-color 0.3s, color 0.3s;
  display: block;
  align-items: center;
}

.btn:hover {
  background-color: #2980b9;
}

#load {
  width: 100%;
  height: 100%;
  position: fixed;
  display: block;
  opacity: 1;
  background: white;
  z-index: 99;
  text-align: center;
}

.black-bg {
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  position: fixed;
  background: rgba(0, 0, 0, 0.5);
  z-index: 5;
  padding: 30px;
  display: none;
}

.show-modal {
  display: block;
}

.white-bg {
  width: 800px;
  height: 500px;
  background: white;
  border-radius: 10px;
  padding: 30px;
  margin: auto;
}

.my-score {
  display: flex;
  margin: auto;
  justify-content: center;
  flex-wrap: wrap;
}

.submitButton {
  margin: 50px;
  margin-left: 300px;
  margin-right: 300px;
}

.outer {
  border: 1px solid black;
  width: 1400px;
  height: 600px;
  margin: 0 auto;
  overflow-x: hidden;
  display: block;
}

.inner-list {
  display: flex;
  height: 100%;
}

.inner {
  width: 100%;
  display: flex;
  flex-direction: column;
}

.question {
  margin-left: 20px;
}
.contentbox {
  display: flex;
  margin-left: 20px;
  margin-bottom: 30px;
}
.content {
  font-size: 20px;
  width: 400px;
}
.button-list {
  text-align: center;
  display: flex;
  justify-content: center;
}
```

제가 이런걸 처음 만들어봐서 코드가 조잡할 수 있습니다. 언제든지 수정할만한 사항이 있으면 댓글 남겨주세요.

제작하는데 생각보다 오래 걸려 과정은 생략하게 되었습니다. 서버 컴포넌트에서 랜덤한 숫자 20개와 DB의 데이터를  
 가져오고, 이를 바탕으로 클라이언트 컴포넌트에서 주로 제작하였습니다. 20개의 퀴즈를 다 풀면 결과와 함께 제출버튼이  
 나오게 됩니다.

## API 만들기

제출 버튼을 누르면 점수가 DB에 등록되게 API를 만들어 봅시다.

```js
//src/pages/api/result.js

import { connectDB } from "/Users/jyp/Documents/GitHub/quiz_project/src/util/db.js";
export default async function handler(req, res) {
  try {
    const result = JSON.parse(req.body);
    const db = (await connectDB).db("Score");
    db.collection("userscore").insertOne(result);
    //작업이 끝나면 메인 페이지로 이동
    res.redirect(302, "/");
    //오류발생시 실행될 함수
  } catch (error) {
    return res.status(500).json("서버 오류");
  }
}
```

퀴즈를 푸는 페이지를 완성하고 API 까지 제작해보았습니다.
마지막 제출버튼을 누르면 바로 메인페이지로 이동시키고싶었는데, fetch api로는 불가능하다고 하네요.

# 로그인 기능 만들기

일단은 이렇게 끝마치고 마지막 기능인 로그인기능과 순위판을 제작해보도록 하겠습니다. 순위판에는 이름이 적혀있어야 하니
이름을 넣기 위해 먼저 로그인 기능부터 구현하겠습니다.

## nextauth 설치

로그인기능은 next.js의 nextauth라는 라이브러리를 사용하여
간단하게 소셜로그인을 구현해보겠습니다.

저는 github로그인과 구글로그인을 할수 있게 만들어보겠습니다. 우선 터미널에서 nextauth부터 설치하겠습니다.

```terminal
npm install next-auth
```

nextauth를 설치한 뒤에 github와 구글에 들어가서 소셜로그인 세팅을 해보겠습니다.

## github 소셜로그인 세팅

우선 먼저 [github](https://github.com)부터 해보겠습니다.
![quiz_project_3-1](https://github.com/1221jyp/Quiz_project/assets/98996860/f58d79e1-b023-4a50-9fee-463995d8af6e)
![quiz_project_3-2](https://github.com/1221jyp/Quiz_project/assets/98996860/93dd8872-7caf-4454-8fb8-0f109a2054a3)
![quiz_project_3-3](https://github.com/1221jyp/Quiz_project/assets/98996860/39d19f0b-db4d-4b2d-864a-f52a2546b946)

저는 이미 하나 있지만 새로 만들겠습니다.

![quiz_project_3-4](https://github.com/1221jyp/Quiz_project/assets/98996860/23783f0e-2a34-400c-9cba-8bb0f378fa59)

지금은 localhost로 해놓았지만, aws같은곳에 올릴거면 주소를 바꿔줘야 합니다.

![quiz_project_3-5](https://github.com/1221jyp/Quiz_project/assets/98996860/a9f796ea-c98b-4257-bce2-f981fe059f59)

사진의 과정을 따라오면, 끝이 납니다. Key를 생성해주고, client id와 key를 .env같은 소중한곳에 잘 숨겨줍시다.

다시 돌아와서 새로운 폴더를 만들어줍니다. src/pages/api/ 주소에 auth 폴더를 생성하고 그 안에 [...nextauth].js 파일을 새로 생성해줍시다.

```js
//scr/pages/api/auth/[...nextauth].js

import NextAuth from "next-auth";
import GithubProvider from "next-auth/providers/github";
import GoogleProvider from "next-auth/providers/google";
require("dotenv").config();

export const authOptions = {
  providers: [
    GithubProvider({
      clientId: process.env.githubOAuthID,
      clientSecret: process.env.githubOAuthKey,
    }),
    GoogleProvider({
      clientId: process.env.googleOAuthID,
      clientSecret: process.env.googleOAuthKey,
    }),
  ],
  secret: process.env.JWTpassword,
};
export default NextAuth(authOptions);
```

github 로그인 구현은 완료되었습니다.

## 구글 소셜로그인 세팅

이제 구글의 provider를 가져와보겠습니다.
가장 먼저 [구글 클라우드 플랫폼](https://console.cloud.google.com/projectselector2/home/dashboard?hl=ko&supportedpurview=project)에 들어가줍니다.
![quiz_project_3-6](https://github.com/1221jyp/Quiz_project/assets/98996860/2d3e7636-1281-43b6-b024-0c0b528754fb)
![quiz_project_3-7](https://github.com/1221jyp/Quiz_project/assets/98996860/0ef6a2e9-f7ce-479a-8272-21272709dea6)
![quiz_project_3-8](https://github.com/1221jyp/Quiz_project/assets/98996860/c9c8fab7-8853-432c-9812-54c280a4d4bf)
![quiz_project_3-9](https://github.com/1221jyp/Quiz_project/assets/98996860/9f1912d9-6141-4da1-a6cf-f15de49de424)
![quiz_project_3-10](https://github.com/1221jyp/Quiz_project/assets/98996860/c9f31ae8-161b-46c5-9d78-4f0285c8b3a1)
![quiz_project_3-11](https://github.com/1221jyp/Quiz_project/assets/98996860/58295522-9d42-42dd-94ef-dd20b53651fb)

여기까지 하고 뒤에 있는 두개의 챕터는 그냥 다음 버튼 누르면서 건너뛰기 해줍니다.

![quiz_project_3-12](https://github.com/1221jyp/Quiz_project/assets/98996860/38c6b451-96bc-4e5b-88cc-a22b689ca543)
![quiz_project_3-13](https://github.com/1221jyp/Quiz_project/assets/98996860/3f578b06-6ec7-45fb-86ef-9a1df0204126)
![quiz_project_3-14](https://github.com/1221jyp/Quiz_project/assets/98996860/55bdecfe-ba15-47dc-b184-54cfab21659f)

구글 OAuth ID와 키도 받아내었습니다. 이제 홈페이지에서 정상적으로 작동되는지 테스트해보겠습니다.

![quiz_project_3-15](https://github.com/1221jyp/Quiz_project/assets/98996860/80c6acbf-c966-4efd-947c-b3f0d35abc52)

github와 google로그인 모두 오류를 뱉었습니다. 사이트의 url을 적을때 `http://localhost:3000` 이라고 적어야하는데 `https://localhost:3000`이라고
입력해버려 오류가 났습니다.

<img width="1582" alt="quiz_project_3-16" src="https://github.com/1221jyp/Quiz_project/assets/98996860/ea237ae2-56ce-41a9-9244-a96fa1d4c167">

구글 로그인의 경우에는 한가지 문제가 더 있었는데, redirect uri를 잘못 적었습니다. `http://localhost:3000`이라고 적는것이 아니라,
`http://localhost:3000/api/auth/callback/google`이라고 적어야 합니다.

이 두개의 문제를 해결하고 나니 두 로그인 방식 모두 성공적으로 끝마쳤습니다.

# 로그인기능 이용하기

## 레이아웃 변경

이제 로그아웃 버튼도 추가하고, 로그인 기능을 이용해서 글 작성과 퀴즈 풀기를 로그인을 마쳐야 풀 수 있게 해보겠습니다.

```jsx
//src/app/logoutbtn.js

"use client";
import "./globals.css";
import { signOut } from "next-auth/react";
export default function LogoutBtn() {
  return (
    <button
      className="login_btn"
      onClick={() => {
        signOut();
      }}
    >
      로그아웃
    </button>
  );
}
```

로그아웃 버튼 컴포넌트입니다.

```jsx
//src/app/layout.js

import Link from "next/link";
import "./globals.css";
import { Inter } from "next/font/google";
import LoginBtn from "./loginbtn";
import LogoutBtn from "./logoutbtn";
import { getServerSession } from "next-auth";
import { authOptions } from "@/pages/api/auth/[...nextauth]";

const inter = Inter({ subsets: ["latin"] });

export const metadata = {
  title: "Quiz time!",
  description: "my quiz project",
};

export default async function RootLayout({ children }) {
  let session = await getServerSession(authOptions);
  return (
    <html lang="en">
      <body className={inter.className}>
        <div className="header">
          <Link href="/" className="title">
            Quiz time!
          </Link>
          <Link href="/newquiz" className="nav_btn">
            퀴즈 만들기
          </Link>
          <Link href="/quiz" className="nav_btn">
            퀴즈 풀기
          </Link>
          {session ? (
            <>
              <LogoutBtn></LogoutBtn>
              <h2 className="UserName">{session.user.name}님, 환영합니다!</h2>
            </>
          ) : (
            <LoginBtn></LoginBtn>
          )}
        </div>
        {children}
      </body>
    </html>
  );
}
```

수정된 레이아웃입니다. 로그인 상태면 로그아웃 버튼이 보이고, 로그아웃 상태면 로그인 버튼이 보입니다.

## DBadapter 설치

그러고 나서 DBadapter를 설치해줍니다. 유저의 로그인 정보를 DB에 저장하기 위해 사용합니다.

```terminal
npm install @next-auth/mongodb-adapter
```

```js
import NextAuth from "next-auth";
import GithubProvider from "next-auth/providers/github";
import GoogleProvider from "next-auth/providers/google";
import { MongoDBAdapter } from "@next-auth/mongodb-adapter";
import { connectDB } from "@/util/db";
require("dotenv").config();

export const authOptions = {
  providers: [
    GithubProvider({
      clientId: process.env.githubOAuthID,
      clientSecret: process.env.githubOAuthKey,
    }),
    GoogleProvider({
      clientId: process.env.googleOAuthID,
      clientSecret: process.env.googleOAuthKey,
    }),
  ],
  secret: process.env.JWTpassword,
  adapter: MongoDBAdapter(connectDB),
};
export default NextAuth(authOptions);
```

이렇게 코드를 짜놓으면 로그인시 DB에 회원정보와 로그인 데이터가 남습니다.

## 로그아웃 상태일때, 로그인시키기

```jsx
//src/app/quiz_h/page.js
import Util from "./util";
import { connectDB } from "/Users/jyp/Documents/GitHub/quiz_project/src/util/db.js";
import { getServerSession } from "next-auth";
import { authOptions } from "@/pages/api/auth/[...nextauth]";

export default async function Quiz_h() {
  const client = await connectDB;
  const db = client.db("Quiz_Data"); //mongodb 불러오기
  let NumberOfQuestion = await db.collection("Quiz").count(); //Quiz collection 안에 있는 데이터의 개수 불러오기
  let result = await db.collection("Quiz").find().toArray(); //Quiz collection 안에 있는 모든 데이터 불러오기

  //_id를 문자열로 변환
  result = result.map((a) => {
    a._id = a._id.toString();
    return a;
  });

  //문제가 랜덤하게 나올수 있도록 Quiz collection 안에 있는 데이터의 개수의 범위 안에서 20개의 랜덤숫자 생성하기

  const numbers = [];

  for (let i = 0; i < NumberOfQuestion; i++) {
    numbers.push(i);
  }
  //

  const answer = [];

  for (let n = 0; n < 20; n++) {
    const index = Math.floor(Math.random() * numbers.length);
    answer.push(numbers[index]);
    numbers.splice(index, 1);
  }
  //로그인정보 가져오기
  let session = await getServerSession(authOptions);
  return (
    <>
      <Util answer={answer} result={result} session={session}></Util>
    </>
  );
}
```

```jsx
//src/app/quiz_h/util.js

if (!session) {
  signIn();
}
```

session에 관한 정보를 util.js 로 넘겨서 로그인이 안되어있을시 바로 로그인하게 설정해줍니다.
이 코드를 퀴즈 만들기 페이지에도 똑같이 만들어줍시다.

모두 끝났다면 퀴즈 작성, 결과 전송을 할때, 유저의 정보도 같이 DB에 올려봅시다.

```jsx
//src/app/newquiz/page.js

import Checkbox from "./Checkbox";
import { getServerSession } from "next-auth";
import { authOptions } from "@/pages/api/auth/[...nextauth]";

export default async function Newquiz() {
  let session = await getServerSession(authOptions);
  return (
    <>
      <div className="container">
        <Checkbox session={session}></Checkbox>
      </div>
    </>
  );
}
```

```jsx
//src/app/newquiz/page.js

"use client";

import "./Checkbox.css";
import { signIn } from "next-auth/react";

export default function Checkbox({ session }) {
  if (!session) {
    signIn();
  }
  let multiple_choice = [1, 2, 3, 4, 5];

  const onlyone = (checkThis) => {
    const checkboxes = document.getElementsByName("a");
    for (let i = 0; i < checkboxes.length; i++) {
      if (checkboxes[i] !== checkThis) {
        checkboxes[i].checked = false;
      }
    }
  };

  const handleSubmit = (event) => {
    const QuestionInput = event.target.elements.q;
    const inputElements = event.target.elements;
    const checkboxes = document.getElementsByName("a");
    const userName = document.querySelector("#userName");

    userName.value = session.user.name;

    // "c"가 포함된 "name"을 가진 input 중 하나라도 비어있는지 확인
    let isAnyInputEmpty = false;
    for (let i = 0; i < inputElements.length; i++) {
      const inputName = inputElements[i].name;
      if (inputName && inputName.includes("c") && !inputElements[i].value) {
        isAnyInputEmpty = true;
        break;
      }
    }
    let isAnyCheckboxChecked = false;
    for (let i = 0; i < checkboxes.length; i++) {
      if (checkboxes[i].checked) {
        isAnyCheckboxChecked = true;
        break;
      }
    }

    event.preventDefault(); // 기본 제출 동작 방지

    if (!QuestionInput.value) {
      // 제목 또는 내용이 비어있는 경우
      alert("문제 내용을 입력해주세요!");
    } else if (isAnyInputEmpty) {
      alert("문제가 비었습니다!");
    } else if (!isAnyCheckboxChecked) {
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
              value={multiple_choice[i]}
              onChange={(e) => onlyone(e.target)}
            />
          </div>
        ))}
        <input style={{ display: "none" }} type="text" name="userid" id="userName"></input>
        <button type="submit" className="submit-button">
          제출하기
        </button>
      </div>
    </form>
  );
}
```

이제 퀴즈 풀기 창에서도 유저의 이름과 함께 전송하도록 수정해줍니다.

```jsx
//src/app/quiz_h/util.js

export default async function Util({ result, answer, session }) {
  const onlyone = (checkThis) => {
    const checkboxes = document.getElementsByClassName("q");
    for (let i = 0; i < checkboxes.length; i++) {
      if (checkboxes[i] !== checkThis) {
        checkboxes[i].checked = false;
      }
    }
  };

  useEffect(() => {
    //로딩이 완료되었다면 로딩페이지 제거
    // 변수/상수 선언
    const outer = document.querySelector(".outer");
    const innerList = document.querySelector(".inner-list");
    const inners = document.querySelectorAll(".inner");
    const loading = document.getElementById("load");
    const buttonRight = document.querySelector(".button-right");
    const showScore = document.querySelector("#score");
    const showResult = document.querySelector(".black-bg");
    const submitButton = document.querySelector(".submitButton");

    var count = 0;
    var score = 0;
    var IsitCorrect = new Object();

    //다음문제 버튼을 클릭하였을시 실행되는 함수
    buttonRight.addEventListener("click", () => {
      const parentComponent = document.getElementById(count);
      const countCheckbox = parentComponent.querySelectorAll('input[type="checkbox"]');

      let isChecked = false;

      // 모든 체크박스를 반복하며 체크 상태를 확인합니다.
      for (let i = 0; i < countCheckbox.length; i++) {
        if (countCheckbox[i].checked) {
          isChecked = true;
          break; // 하나라도 체크되어 있으면 루프를 종료합니다.
        }
      }

      //하나도 체크되어 있지 않다면
      if (!isChecked) {
        alert("체크박스를 체크하여 정답을 선택하세요!");
      }
      //그렇지 않으면
      else {
        //다음 문제로 넘기기
        currentIndex++;
        currentIndex = currentIndex >= inners.length ? inners.length - 1 : currentIndex;
        innerList.style.marginLeft = `-${outer.clientWidth * currentIndex}px`;

        //체크된 체크박스값 보기
        const checkboxes = document.querySelectorAll('input[type="checkbox"]:checked');
        const checkedValues = Array.from(checkboxes).map((checkbox) => checkbox.value);

        //체크박스에 입력한 문제가 정답이라면 answer에 1을 넣고, 스코어 추가
        if (checkedValues == result[answer[count]].a) {
          //19번째 문제에 도달했을때, 20번째 문제의 버튼은 '제출'로 보이게 하기
          if (count >= 18) {
            buttonRight.innerText = "제출";
          }
          //마지막 문제까지 제출하면, 결과 보여주기
          if (count == 19) {
            showResult.classList.add("show-modal");
          }
          IsitCorrect["answer" + count] = 1;
          score++;
          showScore.innerText = `${score * 5}`;
        }
        //오답시 answer에 0을 넣기
        else {
          IsitCorrect["answer" + count] = 0;
        }
        //카운트 추가
        count++;
      }
    });

    submitButton.addEventListener("click", async () => {
      IsitCorrect["result"] = score * 5;
      IsitCorrect["user"] = session.user.name;
      await fetch("/api/result", {
        method: "POST",
        body: JSON.stringify(IsitCorrect),
      }).then(() => {
        console.log("success");
        alert("제출완료");
      });
    });

    //로그인 안되어있으면 로그인시키기
    if (!session) {
      signIn();
    }

    //문제 개수만큼 div박스 늘리기
    let currentIndex = 0;

    if (outer && innerList && inners) {
      inners.forEach((inner) => {
        inner.style.width = `${outer.clientWidth}px`;
      });

      innerList.style.width = `${outer.clientWidth * inners.length}px`;
    }
    const onPageLoad = () => {
      loading.style.display = "none";
    };
    document.onreadystatechange = function () {
      if (document.readyState === "complete") {
        onPageLoad();
      } else {
        window.addEventListener("load", onPageLoad, false);
        // Remove the event listener when component unmounts
        return () => window.removeEventListener("load", onPageLoad);
      }
    };
  }, []);
  return(

  //생략

  )}
```

코드 수정중, 문제를 다 맞혀도 95점으로 뜨게 되는 문제도 함께 수정하였습니다.

<img width="679" alt="quiz_project_3-18" src="https://github.com/1221jyp/Quiz_project/assets/98996860/4cf69a0f-c9e6-4aea-b93d-f474b16c5361">

정상적으로 DB에 아이디가 출력되는 모습

# 마치며

---

이번 시간에는 퀴즈를 푸는 페이지를 만들고, 회원기능, DB에 유저의 이름과 함께 결과를 전송하는 기능까지
제작하였습니다. 다음 시간엔 순위판을 제작하고, aws에 웹사이트를 등록해보겠습니다.

긴 글 읽어주셔서 감사합니다.
