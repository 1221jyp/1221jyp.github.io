---
title: github 블로그 만들어보기_2 chirpy테마 github업로드오류
author: jyp
date: 2023-05-05 11:33:00 +0900
categories: [MacOs, github blog, chirpy]
tags: [github_blog]
math: true
mermaid: true
---

---

## 들어가며

1편에서 chirpy jekyll 테마를 적용하고 포스팅을 끝냈다.  
이제 마지막으로 github에 업로드만 하면 블로그 제작은 끝이 난다.

## github에 블로그 폴더 업로드

github에 파일을 푸시할때는 보통 터미널,cmd를 이용하거나 [github_desktop](https://desktop.github.com)을 사용하는데,
필자는 github desktop이 편해 github desktop을 사용중이다.

### cmd로 파일 푸시

```
$ cd '자신의 블로그 로컬 폴더 위치'
```

먼저 자신의 블로그 파일이 있는 폴더를 선택해주고,

```
$ git init
$ git add .
$ git commit -m "아무 커밋 이름"
$ git remote add origin '자신의 repository 주소'
$ git push origin main
```

위 과정을 통해 git push를 해주자.

### github desktop으로 파일 푸시

로그인을 하고

![스크린샷 2023-05-07 오전 12 52 47](https://user-images.githubusercontent.com/98996860/236634285-4e8694be-5653-4d2a-9213-102a74f855e7.png)

필자가 써놓은 '블로그 글 게시' 칸에 자신의 commit 이름을 정해 넣고,  
파란 버튼을 눌러 commit을 진행한다. 그러고 난 뒤에 origin push라는 버튼을 눌러
깃허브에  
파일을 업로드하면 된다.

## github pages 만들기

![스크린샷 2023-05-07 오전 1 26 36](https://user-images.githubusercontent.com/98996860/236635841-70a1a96e-f4ea-4499-8690-88c9e25efed4.png)

(테스트용으로 repository를 따로 하나 만들었습니다.)

자신의 블로그 repository에 들어가 setting을 들어간뒤, branch항목에서 main/root를 선택해주어
github pages 도메인을 만들어준다.

## 로컬에선 정상작동하던 블로그가 github에서 오류가??

<img width="1269" alt="github-blog_2-3" src="https://user-images.githubusercontent.com/98996860/236642073-70c984e3-7a71-43ed-b9e4-00ccdab2861d.png">

어째서인지 `chirpy`테마를 적용한뒤 github에 업로드하였을때,
페이지가 작동을 안한다! 🥲 다른 테마를 적용해봤을때는 이런 문제가 일어난적이 없었는데,
유독 chirpy테마가 이렇게 말썽을 일으키는 듯 했다.

### deploy from a branch에서 github actions으로 변경

아까 설정창에서 github page를 만들때, `branch`항목 위에 `source`라는 항목이 있었는데,(위 사진 참고)  
이것을 `deploy from a branch`에서 `github actions`로 바꾸어준다.

![스크린샷 2023-05-07 오전 1 37 32](https://user-images.githubusercontent.com/98996860/236636352-03508495-48eb-42c8-a895-2b8f8e8f6b6b.png)

바꾼 뒤에 이러한 항목이 나온다면, `github pages jekyll` 의 configure버튼을 눌러준뒤 오른쪽 위에
commit버튼을 눌러준다.

### ruby 버전 다운그레이드

ruby의 버전이 [github pages](https://pages.github.com/versions/) 버전보다 높으면 문제가 생길수도 있다.
ruby의 버전을 다운그레이드 해주면 해당 문제가 해결되었다. 마침 우리는 ruby의 버전을 관리해주는 `rbenv`를 미리 설치해놓았었다.

```
$ rbenv install 2.7.4
$ rbnev rehash
$ rbenv global 2.7.1
$ ruby --version
```

ruby를 다운그레이드하였다고 끝이 아니다. bundle update까지 시켜주어야 완전히 끝나는것이다.

```
$ cd '자신의 블로그 로컬 폴더 위치'
$ bundle update
```

## 다시 push해보기

이제 github에 다시 업로드하여 블로그가 잘 작동하는지 확인해보자. `git push`를 하기전에 먼저 `git pull`을 해주고
(github 웹사이트에서 commit을 한번 진행했기 때문이다.)
`git push`를 진행해준다. 필자는 이 두개의 해결 방안을 이용해서 chirpy 테마를 적용하는데에 성공했다.

기본적인것들은 모두 끝났고, 나중에 시간이 되면 설정 변경 등등 잡다한것들 다뤄보겠습니다.
읽어주셔서 감사합니다.
