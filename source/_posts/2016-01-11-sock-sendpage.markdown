---
layout: post
title: "sock_sendpage"
date: 2016-01-11 14:42:27 +0900
comments: true
categories: 
---

CVE-2009-2692, sock_sendpage 에서 벌어지는 NULL pointer deref 버그를 정리한다. 발견자가 Tarvis Ormandy 인지 Brad Spengler 인지는 이런저런 문서를 읽어봐도 잘 모르겠다.

Exploit 자체는 꽤 클래식해서 정리해놓을 가치가 있다고 생각한다.

[exploit.c](http://http://downloads.securityfocus.com/vulnerabilities/exploits/36038-5.c)

