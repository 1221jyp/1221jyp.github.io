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
이제 마지막으로 github에 업로드만 하면 블로그 제작은 끝이 난다!

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
$ git remote add origin (원격 저장소 github URL)
$ git push origin master
```

위 과정을 통해 git push를 해주자.

### github desktop으로 파일 푸시
