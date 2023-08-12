---
title: 퀴즈 사이트 만들기_4 (next.js)
author: jyp
date: 2023-08-11 11:33:00 +0900
categories: [MacOs, Next.js, javascript]
tags: [Quiz_project]
math: true
mermaid: true
---

# 시작하며

지난 시간에는 퀴즈 풀기 페이지, 회원기능 , DB를 이용한 회원아이디 저장 등의 기능을 만들었다.
이번 시간에는 순위판 페이지를 완성시키고, AWS에 파일을 업로드할 것이다.

# 순위판 구성

---

순위가 높은 순서대로 자료를 출력하기 위해 mongodb에서 데이터를 가져올때, sort()함수를 이용합니다.
sort함수를 사용하면 특정 자료의 값에 따라 오름차순/내림차순의 순서대로 가져올 수 있게 해줍니다.
저는 result값 (점수)가 높은 순서대로 자료를 출력하게 했습니다.

```jsx
import { connectDB } from "/Users/jyp/Documents/GitHub/quiz_project/src/util/db.js";
import "./style.css";

export default async function Ranking() {
  const client = await connectDB;
  const db = client.db("Score");
  let result = await db.collection("userscore").find().sort({ result: -1 }).toArray();

  return (
    <div className="list">
      <h1>순위판</h1>
      {result.map((a, i) => (
        <div className="container">
          <div className="text"> {i + 1} 위</div>
          <div className="text">이름 : {result[i].user}</div>
          <div className="text">점수 : {result[i].result}</div>
        </div>
      ))}
    </div>
  );
}
```

이렇게 해서 간단히 순위판 기능을 완성했습니다. 순위판에 작성된 사람이 많아지면, 다음 페이지에서 조회하는 기능은
일단 AWS에 이 프로젝트를 업로드한 뒤에 수정해보겠습니다.

# 프로젝트 빌드하기

```terminal
npm run build
```

<img width="691" alt="quiz_project_4-1" src="https://github.com/1221jyp/Quiz_project/assets/98996860/7710a965-6acd-4ff2-8559-925b5e3dfe8f">

빌드를 마치면 터미널에 이러한 창이 뜨게 됩니다.
λ 기호가 들어가 있는 페이지는 dynamic 렌더링으로, 페이지에 접속할때마다 새로운 html을 띄워주는 라우팅 방식이고,
○ 기호가 들어가 있는 페이지는 static 렌더링으로, 계속 고정적으로 띄워주는 페이지입니다.
저희는 로그인, 로그아웃 레이아웃을 모든 페이지에 띄워 놓고 있으므로, 모두 dynamic 렌더링을 사용해주어야 합니다.

빌드된 프로젝트의 파일을 테스트하기 위해서는 밑의 명령어를 실행해보면 됩니다. 지금까지 사용했던 npm run dev는 테스트용이고,
지금 현재 이 명령어는 실제 서버에서 띄워주는 페이지랑 같습니다.

```terminal
npm run start
```

# AWS 업로드

---

확인해보고 이상이 없으면 AWS에 업로드를 시작해봅시다.
