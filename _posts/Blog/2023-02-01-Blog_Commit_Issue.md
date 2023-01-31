---
layout: single
title:  "블로그 커밋 이슈"
categories: blog
tag: [blog, github, git]
# search: false 검색 기능에 나오지 않게 할려면 false
---

# 커밋이 업데이트 안되는 이슈
깃허브 블로그에 커밋을 해도 잔디심기가 안되는 것을 확인할 수 있었습니다. 바로 유튜브, 구글에 검색해서 조사해봤습니다. 

<b>원인은 fork된 프로젝트의 경우 pull request를 할 때만 잔디가 심어지고 commit은 인정되지 않는다고 하네요... </b>

매번 다른 branch에 commit해서 pull request하는 것보다 어차피 나 혼자 작업하는 공간인데 바로 master branch에 commit하고 싶었어요.

조사결과 fork 해온 repository를 복사해서 깃허브에 새로운 repository로 만들면 해결된다고 합니다.

# 이슈를 해결해보자
일단 깃허브 블로그의 repository는 유저네임.github.io 라는 이름이여야 합니다. 공식문서에 따르면 저장소의 저 유저네임 부분이 사용자 이름과 일치하지 않으면 작동하지 않을 수 있다고 하네요.

따라서 setting의 rename으로 본래 repository를 잠깐만 이상하게 바꾸고 새로운 repository의 이름을 정확히 <b>유저네임.github.io</b> 로 설정했습니다.

## 기존의 repository를 새 repository로 옮기기
각자 사용하는 터미널에 가서 새 repository로 이동한 후 코드를 적습니다.
> $ git clone --bare (기존 repository주소)

![code1.png]({{site.url}}/assets/images/2023-02-01-Blog_Commit_Issue/code1.png)

<br>

이번엔 기존의 repository로 이동 후 코드를 적습니다.
> $ git push --mirror (새로운 repository주소) 

![code2.png]({{site.url}}/assets/images/2023-02-01-Blog_Commit_Issue/code2.png)

<br>

그리고 저는 깃허브 데스크탑으로 기존 레포지토리 삭제했습니다.
(혹시나 해서 local에는 살려두긴 했습니다만...)

이러니까 잔디가 잘 심어지네요!!

# 해결 후 
깃허브 블로그의 경우 repository를 private로 하면 블로그가 404뜨는 것을 확인할 수 있습니다. 꼭 깃허브 블로그 repository를 private로 하고 싶은 분들은 GitHub Enterprise account를 가져야 할 것입니다. (아무튼 돈...)
repository에서 Settings > pages 맨 밑에 확인하시면 보실 수 있어요!!