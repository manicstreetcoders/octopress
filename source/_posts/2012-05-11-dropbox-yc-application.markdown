---
layout: post
title: "Dropbox YC Application"
date: 2012-05-11 09:44
comments: true
categories: 
---

개발자들은 자신이 불편하면 툴을 만들어 해결할 수 있는 재주를 가지고 있다.
그래서 그들은 소스 코드 관리 도구들을 발전시켜왔다. 

중앙에 저장소를 두고 집이던지 회사이던지 어느 컴퓨터에서나 최신 소스를 다운받아 코딩을 이어나갈 수 있었다. 또한 변경된 히스토리가 남아있어서, 잘못되었을 경우 언제든지 예전 버전으로 돌아갈 수 있었다.

diff 라고 불리는 변경점만을 업데이트하도록 발전되어 왔기에, 속도도 빠르고 저장 장소도 아낄 수 있었다.

개발자들은 cron 과 rsync 를 가지고 수십대의 컴퓨터 내용을 동일하게 유지시키는 스크립트를 짤 수 있는 사람들이다.

하지만 개발자가 아닌 사람들은 여전히 USB 를 들고 다니고, email 에 화일을 첨부해서 자기 메일 주소로 보내고, CD 를 굽고, 여기 저기 흩어져 있는 동일한 제목의 문서들 중에 어떤 것이 가장 최신인지 혼란스러워했다.

dhouston 은 [Dropbox YC Application](http://dl.dropbox.com/u/27532820/app.html)에서 개발자 커뮤니티에서 검증된 기술을 가지고 일반 사람들의 문제를 해결했다고 이야기한다. i.e. subversion, trac, rsync

> Dropbox solves all these needs, and doesn't need configuration or babysitting.
> Put another way, it takes concepts that are proven winners from the dev
> community (version control, changelogs/trac, rsync, etc.) and puts them
> in a package that my little sister can figure out (she uses Dropbox to
> keep track of her high school term papers, and doesn't need to burn CDs
> or carry USB sticks anymore.)

<br />
경쟁 제품은 잘못된 추상화를 사용한다고 이야기한다. "온라인 디스크 드라이브" 라는 추상화는 사용자에게 불편하다고 이야기한다. 
(Dropbox 는 개발자에게 친숙한 Repo 라는 개념을 Box 라는 metaphor 로 바꾸었다.
Repo 는 온라인/오프라인 상관없이 작업할 수 있다.)

또한 자동이 아니라 손으로 이메일을 보내거나 웹에 업로드해야 하는 것은 버전 관리를 사람이 해야 하기 때문에 말이 안된다고 말한다. 

> Competing products work at the wrong layer of abstraction and/or force
> the user to constantly think and do things. The "online disk drive" 
> abstraction sucks, because you can't work offline and the OS support is
> extremely brittle. Anything that depends on manual email/uploading
> (i.e. anything web-based) is a non-starter, because it's basically
> doing version control in your head. But virtually all competing services
> involve one or the other.
> With Dropbox, you hit "Save", as you normally would, and everything just
> works, even with large files (thanks to binary diffs)

<br />
-----

<br />
Novell 은 NetWare 라는 제품으로 화일 카피라는 문제를 해결해버렸다. 당시에 화일을 카피하려면 1 Mb 도 안되는 용량의 5.25'' 디스크로 여러번에 나누어 옮겨야 했다.

더군다나 대용량 화일은 해결책이 없었다. 압축을 해서 화일을 쪼개는 수밖에 없었다.

N: 드라이브로 매핑되는 네트워크 드라이브는 많은 사람들의 문제를 해결해주었고, 기업이라면 NetWare 는 필수적으로 사용하였다.

그리고, NetWare 의 성공으로 Novell 은 엄청난 회사가 되었다. 하지만 Novell 은 망했고, 2010 년이 되어서도 사람들은 1980년대말과 동일한 불편에 다시 시달리고 있었다. 이를 Dropbox 가 해결해주었다.

하지만 아마존의 S3 가 없었으면, Dropbox 도 없었을 것이다.
