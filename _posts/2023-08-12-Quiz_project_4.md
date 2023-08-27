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

우선 [AWS](https://aws.amazon.com/)에 가입을 해줍니다. 카드 정보까지 입력해주어야 1년 프리티어 무료체험이 가능합니다.

작업했던 폴더에서 node_modules를 제외한 모든 파일을 묶어 압축해줍니다.

<img width="1470" alt="quiz_project_4-2" src="https://github.com/1221jyp/Quiz_project/assets/98996860/40d314bb-347b-40d7-8a32-36a873705491">

우선 AWS로그인을 마치고, 콘솔 홈에 들어와줍니다. 이런 다음에 검색 창에서 IAM을 검색하고 들어가서 대시보드의 역할 버튼을 눌러줍니다.

<img width="1470" alt="quiz_project_4-3" src="https://github.com/1221jyp/Quiz_project/assets/98996860/c4f44a59-33cf-4561-a103-85de7fac2186">
<img width="1469" alt="quiz_project_4-4" src="https://github.com/1221jyp/Quiz_project/assets/98996860/21aae789-5cb2-4408-8ac4-b68967487bbd">
<img width="1470" alt="quiz_project_4-5" src="https://github.com/1221jyp/Quiz_project/assets/98996860/2428d97c-f7d7-498f-b1a4-2d0f4f6a5321">

검색창에 `beanstalk`을 검색하여 제일 위에 있는 세개를 선택해 줍니다.

<img width="1469" alt="quiz_project_4-6" src="https://github.com/1221jyp/Quiz_project/assets/98996860/8e48ba10-898e-445c-8a04-aff5e0f66792">

IAM 역할설정이 완료되었다면 이제 진짜로 AWS에 파일을 업로드해줘야 합니다.

<img width="1469" alt="quiz_project_4-7" src="https://github.com/1221jyp/Quiz_project/assets/98996860/da89784e-6059-4937-a3e1-35be0a04f966">

다시 최상단의 검색창에 `elastic beanstalk`을 검색해줍니다.
<img width="1469" alt="quiz_project_4-8" src="https://github.com/1221jyp/Quiz_project/assets/98996860/e9aeaf89-fd6e-4dde-914f-6682949dd8a6">

애플리케이션 생성을 누르시고 난 다음 아래 사진에 나오는 부분만 수정해 줍니다.(이름짓기 제외)
<img width="685" alt="quiz_project_4-9" src="https://github.com/1221jyp/Quiz_project/assets/98996860/a80d35ae-3a5e-4f12-8eb4-c14997fd0416">
<img width="798" alt="quiz_project_4-10" src="https://github.com/1221jyp/Quiz_project/assets/98996860/1e658791-a866-41a2-b588-654a50e4a2bc">
<img width="803" alt="quiz_project_4-11" src="https://github.com/1221jyp/Quiz_project/assets/98996860/7ee2e6be-2cf7-4a76-9099-c3693bae3eeb">
<img width="802" alt="quiz_project_4-12" src="https://github.com/1221jyp/Quiz_project/assets/98996860/da4ce31d-640c-4095-ae41-95d79ce5c57e">
<img width="792" alt="quiz_project_4-13" src="https://github.com/1221jyp/Quiz_project/assets/98996860/886f426b-0d38-4bd1-867f-f3369700703a">
<img width="1164" alt="quiz_project_4-14" src="https://github.com/1221jyp/Quiz_project/assets/98996860/bbfd0ca6-0284-486e-a5ba-c0c07bbd7530">

여기까지 마쳤다면 사이트 업로드가 성공된 것입니다. 아마존에서 알려주는 도메인에 들어가서 확인해 봅시다.

<img width="1470" alt="quiz_project_4-15" src="https://github.com/1221jyp/Quiz_project/assets/98996860/6c85d9b7-57a2-4b8a-9554-059fcbe04432">

정상적으로 작동되는 모습

하지만 회원가입기능이 정상적으로 작동되지 않는것을 볼 수 있습니다. 이는 우리가 깃허브, 구글에 주소를 `http://localhost:3000`으로 해놓았었는데,
이것의 주소가 안바뀌었기 때문입니다.

하지만 먼저 이것을 수정하기 전에, 도메인부터 바꾸겠습니다.

![quiz_project_4-16](https://github.com/1221jyp/Quiz_project/assets/98996860/c3e0e68d-7107-4de8-a2da-14ec45b211d9)

가장 먼저 [호스팅케이알](https://www.hosting.kr)에서 마음에 드는 도메인을 구입하고 해당 주소를 aws에 등록하고
https 리디렉션까지 끝마쳤습니다. [완성된 사이트의 주소](www.playquizus.com)  
해당 과정을 기록하려고는 했으나, 굉장이 복잡하고, 중간중간 기록을 남기지 못하여
글에 넣지 못했습니다. 다음부터는 모든 과정을 남기겠습니다.

# 마무리하며

도메인 등록하고 SLL인증서 발급받고 로드벨런서로 HTTPS리디렉션 등등 다양한 과정을 거쳐서 지금의 사이트가 완성되었는데,
기록으로 남기지 못하여 아쉬웠습니다. 다음시간에는 웹페이지의 전체 디자인과, 모바일 환경에서도 잘 작동되게끔 디자인을 수정하는
작업을 하겠습니다. 그리고 여태까지 깃으로 한개의 branch에만 commit을 하고, commit마저도 엉망진창으로 해놓아서 git 환경도 새로 만들어볼 예정입니다.  
읽어주셔서 감사합니다.
