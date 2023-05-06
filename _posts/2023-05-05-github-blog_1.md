---
title: github 블로그 만들어보기_1
author: jyp
date: 2023-05-05 11:33:00 +0900
categories: [MacOs, github blog]
tags: [github_blog]
math: true
mermaid: true
---

---

## 들어가며

나는 작업을 하다가 모르는게 있으면 당연히 구글링을 통해서 문제를 해결한다.
구글링을 하면 다양한 개발 블로그의 글들을 볼수 있는데, 블로그들을 보며 나도 개발 블로그나 한번 해볼까 싶은 생각이 들었다.
그래서 나도 한번 만들어보자 하는 생각에 블로그의 종류들을 살펴보았다.  
`tistory`, `velog`, `notion` 등등 다양한 종류들이 있었는데, 그중 github 블로그가 너무 멋있어 보였다.  
매우 어렵다는 특징이 나에게는 매우 매력적으로 다가왔고, 추후 광고로 수익을 낼수도 있다고 해서 마음에 들었다.  
그래서 무턱대고 github 블로그 만들기를 도전하였고, ~~시험기간엔 뭐든 해보고 싶어진다.~~ 성공하여 이 글을 남기고 있다.  
_저의 모든 글은 MacOs silicon을 기준으로 제작되었습니다._

## github repository 생성

<img width="742" alt="create_repository" src="https://user-images.githubusercontent.com/98996860/236457379-f21a6f72-880c-40ac-a872-8bf90882deda.png">

[github](http://github.com)에 접속하여
`자신의 깃허브 아이디.github.io` 로 리포지토리 이름을 정하고 만든다.
캡처된 이미지에는 readme.md를 켜두지 않았지만, 확인용으로 readme.md를 켜두자.
만약 이미 다른 이름으로 이름을 지어 놓았다면 설정에서 추후에 바꿔주어야 한다.

생성을 하고 난 후에는 `git clone`으로 작업폴더를 만들어 둔다.

## git clone 하기

깃을 설치하고 터미널이나 cmd를 통해서 clone을 해와도 좋지만,
[github_desktop](https://desktop.github.com)을 이용하여 `git clone`을 해오는것도 좋은 방법이다.
터미널이나 cmd는 하나하나 명령어를 다 쳐줘야하는데, [github_desktop](https://desktop.github.com)은
별도의 명령어 없이 버튼 몇번 딸깍딸깍하면 푸시가 되니 너무 좋았다.

### cmd로 git clone

```
$ cd '자신이 작업할 폴더 주소'
```

먼저 자신이 블로그를 작업할때 쓸 폴더를 미리 만들어두고, `cmd`에 그 위치를 정해둔다.

```
$ git clone '자신의 repository 주소'
```

그런다음 git clone으로 자신의 파일을 가져온다.
`readme.md`파일이 날아왔다면 성공.

### github desktop으로 git clone

설치와 로그인을 마치고, 자신이 만들어뒀던 repository를 선택하고, 밑에 파란 버튼으로 repository파일을
clone해준다. 이후 이 폴더에서 생기는 변경점들은 github desktop에서 모두 확인이 가능하다.

## ruby 설치하기 (MacOs)

이후 cmd에서 rbenv라는 ruby 버전 관리 툴을 설치한다.

```
$ brew update
$ brew install rbenv
```

그리고 rbenv를 통해 작성일 기준 가장 최신 버전인 `3.2.2` ruby를 설치해준다.

```
$ rbenv install 3.2.2
$ rbenv rehash
$ rbenv global 3.2.2
```

이후 ruby가 정상적으로 설치되었는지 `ruby -v`를 통해 확인해본다.

### ruby 환경설정

```
$ vi ~/.zshrc
```

터미널에 해당 명령어를 입력하면 vim 편집기로 이동하게 된다.
`i`를 눌러 insert 모드에 들어가고

```vim
export PATH={$Home}/.rbenv/bin:$PATH && \
eval "$(rbenv init -)"
```

해당 명령어를 붙여넣기 한뒤 `esc`를 눌러 편집 모드를 나가고
`:wq`를 입력하여 vim 편집기를 나가준다.

## jekyll 설치하기

여기까지 잘 따라왔다면, 마지막 작업으로는 `jekyll`설치만이 남아있다.

```
$ gem install jekyll
```

jekyll 설치 후 `jekyll -v`로 정상적으로 설치되었는지 확인한다.

## jekyll 테마 적용하고 업로드하기

[github](http://github.com)에서 `clone` 해왔던 폴더에 jekyll 테마를 적용할것이다.  
나는 우선 가장 예쁘다고 느껴졌던 [chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)테마를
설치하여 붙여넣기 하였다.
<img width="1465" alt="스크린샷 2023-05-05 오후 11 23 36" src="https://user-images.githubusercontent.com/98996860/236486070-51d39257-528e-48fa-9e81-dc04eee47efd.png">

[chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) 주소에 접속하여 zip 파일을 다운로드하고

![스크린샷 2023-05-05 오후 11 30 09](https://user-images.githubusercontent.com/98996860/236487139-39864e61-ea27-47ca-b279-03298317cbc1.png)
이처럼 clone해왔던 폴더에 싹다 붙여넣기 하였다.

마지막으로 로컬 블로그 파일을 실행해서 잘 작동하는지 테스트 해본다.

```
$ cd `자신의 블로그 로컬 폴더의 주소`
```

위 명령어를 통해서 cmd에 폴더 위치를 지정해놓고,

```
$ bundle install
$ bundle exec jekyll serve
```

명령어를 입력하여 잘 작동하는지 확인한다.

<img width="1468" alt="스크린샷 2023-05-05 오후 11 41 29" src="https://user-images.githubusercontent.com/98996860/236489992-30bc525f-45f1-451b-b42a-99504f66da6a.png">

성공적인 결과물.

포스팅이 너무 길어지는 것 같아, github 업로드는 다음 포스팅에서 진행하겠습니다.  
읽어주셔서 감사합니다!
