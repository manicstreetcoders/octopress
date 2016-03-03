---
layout: post
title: "The DROWN Attack"
date: 2016-03-03 09:38:55 +0900
comments: true
categories: 
---

[The DROWN Attack](https://drownattack.com)

1,000 번의 TLS 핸드쉐이크를 관찰하고, 40,000 번의 SSLv2 연결을 시도해보고, 2^50 번의 symmetric cipher 연산을 하면, 2048-bit RSA TLS 핸드쉐이크를 해독하는 것이 가능하다...고 한다.

배정된 CVE 는 다음과 같다.

* CVE-2016-0800
* CVE-2015-3197 
* CVE-2016-0703
* CVE-2016-0704

DROWN 공격은 RSA PKCS# v1.5 padding 구조를 이용함.

N = p*q, l = len(N) 이라고 할때, k 는 symmetric key 라고 할 때, 암호화를 돌릴 블럭은 다음과 같이 만든다. `m = 00 || 02 || PS || 00 || k`

`len(PS) >= 8` 이어야하고, `02` 는 block type, 처음 `00` 은 암호화를 돌릴 블럭(EB)가 N 보다 작은 정수가 되기 위해서 집어 넣는 것. PS 는 패딩. 패딩을 prepend 하는 구조. 이 EB 를 정수로 converting 한다. 256 의 i 승을 계속 곱하면서 더해나가는 식으로 변환.

SSLv2 에서는 master_key 가 k 라고 보면 된다.
