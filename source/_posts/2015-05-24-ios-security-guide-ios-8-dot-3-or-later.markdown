---
layout: post
title: "iOS Security Guide - iOS 8.3 or later"
date: 2015-05-24 14:07:16 +0900
comments: true
categories: 
---

[iOS Security Guide - iOS 8.3 or later](https://www.apple.com/business/docs/iOS_Security_Guide.pdf)

Apple Watch 가 나오면서 저 문서가 update 되었다.

* [Apple Shoots, Scores in Security. Quietly.](http://fromsiliconvalley.com/tag/secure-enclave/)

### Secure Enclave

* 우선 ROM 에 Apple Certificate 이 박혀있어서, 애플이 서명한 컨텐츠를 검증할 수 있는데, ROM 에 박혀있기 때문에,
손대는 것이 불가능.

* 아이폰에는 Secure Enclave 라는 암호 코프로세서가 존재하는데, 이 안에 디바이스마다 유니크한 256-bit UID 가
제작 과정에서 각인됨. 애플에 따르면 자기들도 기록해두지 않아서 모르고, 생산된 디바이스에서 UID 를 recover 
하는 것은 불가능. 아무도 모르는 유니크한 256-bit 가 코프로세서 안에 존재하는 셈.

* Secure Enclave 는 암호화 엔진을 가지고 있는데, 암호키를 만들때 유저가 입력한 패스워드에 UID 를 섞기
때문에 고성능 FPGA 를 이용한 딕셔너리 공격이 어렵게 된다. Enc(사전에 나오는 단어+UID, Key) 를 돌려야하는데,
UID 는 코프로세서 밖으로 추출이 불가능하기때문에, 해당 아이폰에서만 복호화가 가능한 구조. 패스워드와 UID 를 섞는 과정은 80ms 가 걸리도록 튜닝되어있음.
