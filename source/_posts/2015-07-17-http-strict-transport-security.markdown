---
layout: post
title: "HTTP Strict Transport Security"
date: 2015-07-17 15:04:07 +0900
comments: true
categories: 
---

#### HTTP Strict Transport Security

HSTS 를 안쓰면, HTTP -> 301 Moved Permanently -> HTTPS 로 바꿔야한다.

HSTS 를 쓰면, HTTP -> 307 Internal Redirect -> HTTPS 로 간다.

HSTS 의 경우 Chrome 이 request 조차 보내지 않는다.

`Strict-Transport-Security: max-age=31536000; includeSubdomains; preload` 와 같은 헤더때문인데.

웹사이트가 HSTS 인지 아닌지 처음엔 모르기때문에, 첫 커넥션까지 HTTPS 로 가게 하려면,

https://hstspreload.appspot.com 에 등록해야한다.

#### HTTP Public Key Pinning

HTTP Public Key Pinning 의 경우도 중요한 기술.
