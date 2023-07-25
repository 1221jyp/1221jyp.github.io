---
title: 퀴즈 사이트 만들기_1 (next.js)
author: jyp
date: 2023-07-25 11:33:00 +0900
categories: [MacOs, Next.js, javascript]
tags: [Quiz_project]
math: true
mermaid: true
---

---

## 시작하며

방학이 찾아온 김에 게으르게 놀지 말고 무언가 하나라도 만들어봐야지 하며 시작하게 되었습니다.
어떤 프로젝트를 만들면 좋을까 생각하다가, 캬훗과 비슷하지만, 다른 느낌의 퀴즈 사이트를 만들어보기로 결정했다.
카훗은 혼자 퀴즈를 풀 수 없고, 그룹장이 게임을 만들어야만 퀴즈를 풀 수 있다. 나는 혼자서도 퀴즈를 풀 수 있는
사이트를 제작해보면 좋을것 같다는 생각이 들었다.

### 퀴즈 사이트에 들어갈 기능

1. 퀴즈를 직접 작성하여 올릴수 있어야 한다.
2. 퀴즈가 만들어지면 DB에 퀴즈가 저장되고, 퀴즈 풀기를 할때 랜덤으로 유저가 작성해놓은 퀴즈가 나온다.
3. 한번에 풀 수 있는 퀴즈의 수를 20개로 제한해놓고, 다 풀었을 시 순위판이 나오고, 점수가 정산된다.

## 사용할 라이브러리/DB

### next.js

![Next.js](https://github.com/1221jyp/Quiz_project/assets/98996860/9a139896-ad27-4a38-9b87-3b425b4c17df)
vercel에서 제작한 풀스택 기반 SSR렌더링 방식의 프레임워크입니다.
다룰줄 아는 프레임워크가 이거밖에 없어 사용하게 되었습니다.

### mongodb

![MongoDB](https://github.com/1221jyp/Quiz_project/assets/98996860/41ba3c03-dc05-491a-8c3f-5f391ea73435)
mongodb는 noSQL계열의 데이터베이스 시스템입니다.
마찬가지로 다룰줄 아는 DB가 mongodb밖에 없어 사용하게 되었습니다.

## 시작하기

### node.js 설치

[node.js](https://nodejs.org)를 설치해줍시다. 최신버전과 추천버전중 아무거나 설치해줍니다.
저는 이미 설치되어있으므로 설치는 건너뛰겠습니다.

### next.js 설치

먼저 vscode를 들어온 뒤에 next.js프로젝트 폴더를 생성할 폴더에 들어와 터미널을 열어줍니다. 그 이후 터미널에 해당 명령어를 입력합니다.

```
npx create-next-app@latest
```

입력 후 질문이 들어오는데 y를 입력하고 엔터를 눌러줍니다.

<img width="1129" alt="setting" src="https://github.com/1221jyp/Quiz_project/assets/98996860/49a7b1b7-8708-412a-899c-43db7c56ac5d">

그리고 프로젝트의 이름을 지어줍니다. 저는 quiz_project로 이름을 지어주었습니다. 프로젝트 작명에는 대문자가 들어갈수 없다네요.

그리고 여러가지 설정에 대한 질문이 들어오는데, 저는 딱히 특별한 기능은 사용하지 않을것이기 때문에, src디렉토리 생성과 app router기능만 활성화했습니다. 그냥 app router만 켜두셔도 무방합니다.

### github에 업로드

<img width="764" alt="Repository" src="https://github.com/1221jyp/Quiz_project/assets/98996860/26c93a91-0f58-4566-a304-8f334756ee35">

가장 먼저, github에서 새로운 repository를 만들어준 뒤에, 저희가 만든 next.js 프로젝트 폴더를 push해줍시다.  
만들어진 next.js 프로젝트 폴더에 들어와 터미널을 열어준뒤 해당 명령어를 입력합니다.

```
git init
git add .
git commit -m "아무 커밋 이름"
git remote add origin '자신의 repository 주소'
git push origin main
```

만약 [git](https://git-scm.com)이 설치되어있지 않다면, [git](https://git-scm.com)부터 설치하셔야 합니다.

![success](https://github.com/1221jyp/Quiz_project/assets/98996860/56615a59-64fb-4802-8d05-e90d3d9b8850)
정상적으로 업로드에 성공하셨다면 github에 접속시 이런 창이 뜨게 됩니다.

## next.js 실행

### 정상적으로 작동되는지 테스트하기

```
npm run dev
```

터미널에 위 명령어를 입력하여 next.js가 정상적으로 작동되는지 확인합니다.  
터미널에 명령어 입력을 마쳤다면 [localhost:3000](http://localhost:3000)에 접속시 자신의 프로젝트가 나옵니다. 가끔 [localhost:3001](http://localhost:3001)에 접속해야 하는 경우도 있습니다.
<img width="1470" alt="test" src="https://github.com/1221jyp/Quiz_project/assets/98996860/1fd4fd07-4ce9-4db5-a870-7b7bb1bdaf53">
정상적으로 작동된다면 이러한 창이 뜨게 됩니다.  
app폴더의 page.js파일을 수정하여 프로젝트를 시작하라고 하네요.

### 새로운 페이지

<img width="1470" alt="page js" src="https://github.com/1221jyp/Quiz_project/assets/98996860/b25e81b2-4340-4a60-af99-c3d38967e3a5">
app폴더에 들어와 page.js를 들어와줍니다.

```jsx
// src/app/page.js

export default function Home() {
  return (

  )
}
```

뼈대부분만 남기고 시작해줍시다. return()안에 jsx형식으로 html작성하면 됩니다.

```jsx
// src/app/page.js

export default function Home() {
  return <h2>안녕하세요</h2>;
}
```

<img width="1470" alt="hello" src="https://github.com/1221jyp/Quiz_project/assets/98996860/11aa15d4-6627-4d2e-b94d-02307bf339c2">
성공.

## 헤더 레이아웃 만들기

저는 퀴즈를 작성하는 버튼과 퀴즈를 풀때 누를 버튼, 로그인/로그아웃 버튼만 헤더부분에 놓아주면 좋을 것 같습니다.
모든 주소에서 레이아웃이 보이게 하려면 layout.js파일에서
레이아웃을 디자인해주면 됩니다.

```jsx
// src/app/layout.js

export const metadata = {
  title: "Quiz time!",
  description: "my quiz project",
};

export default function RootLayout({ children }) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <div className="header">
          <Link href="" className="title">
            Quiz time!
          </Link>
          <Link href="" className="nav_btn">
            퀴즈 만들기
          </Link>
          <Link href="" className="nav_btn">
            퀴즈 풀기
          </Link>
          <button onClick="" className="login_btn">
            로그인
          </button>
        </div>
        {children}
      </body>
    </html>
  );
}

```

위에 title부분을 수정하면 사이트의 title이 바뀝니다.  
레이아웃을 만들었으니, 이제 디자인은 우리들의 조력자 gpt에게 부탁하여 global.css에 넣어줍시다.

```css
/* global.css */

body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 0;
  background-color: #f9f9f9;
}

.header {
  background-color: #ff6b6b;
  padding: 20px;
  display: flex;
  align-items: center;
}

.title {
  color: #fff;
  font-size: 28px;
  text-decoration: none;
  margin: 0;
  margin-right: 20px;
}

.nav_btn {
  color: #fff;
  background-color: #5f27cd;
  border: 2px solid #5f27cd;
  padding: 12px 24px;
  text-decoration: none;
  font-size: 18px;
  border-radius: 30px;
  margin-left: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  transition: box-shadow 0.3s ease, background-color 0.3s ease, color 0.3s ease;
}

.nav_btn:first-child {
  margin-left: 0;
}

.nav_btn:hover {
  background-color: #341f97;
  border-color: #341f97;
}

.login_btn {
  color: #fff;
  background-color: #2ecc71;
  border: none;
  padding: 12px 24px;
  text-decoration: none;
  font-size: 18px;
  border-radius: 30px;
  margin-left: 10px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  transition: box-shadow 0.3s ease, background-color 0.3s ease;
}

.login_btn:hover {
  background-color: #27ae60;
}
```

<img width="1470" alt="layout" src="https://github.com/1221jyp/Quiz_project/assets/98996860/37d5610a-afa7-4561-96c0-37640e94b935">
색깔을 화려하게 부탁하여 이렇게 만들었습니다. 개인적으로 만족하지만, 이런 디자인이 마음에 들지 않는다면  
각자 새롭게 만들어보시거나 새로 gpt에게 부탁하는것이 좋겠습니다.

포스팅이 길어지니 여기서 마치고 다음 포스팅에서 계속하겠습니다.
읽어주셔서 감사합니다.