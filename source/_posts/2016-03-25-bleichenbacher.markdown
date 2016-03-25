---
layout: post
title: "Bleichenbacher"
date: 2016-03-25 09:52:51 +0900
comments: true
categories: 
---

RSA PKCS#1.5 format 을 공격하는 한가지 방법.

찾아보니 이 아저씨는 현재 구글 근무중.

The Drown 공격의 핵심은 두가지라고 보는데, 그 중의 하나. 나머지 하나는, 상위 버전의 문제를 취약한 SSL 하위 버전에 존재하는 oracle 에 태우기 위해 RSA 연산 문제를 reduce 하는 방법.

Bleichenbacher 아저씨가 발표한 논문을 정리해본다.
