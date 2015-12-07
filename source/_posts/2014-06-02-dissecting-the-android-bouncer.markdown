---
layout: post
title: "Dissecting The Android Bouncer"
date: 2014-06-02 10:00:56 +0900
comments: true
categories: 
---

찰리 밀러의 발표 [Dissecting The Android Bouncer](https://jon.oberheide.org/files/summercon12-bouncer.pdf) 를 읽어보았다.

### Android Bouncer

마켓에 제출된 App 에 대한 자동화된 스캐닝을 수행하는 서비스. Malware 나 스파이웨어, Trojan 을 찾기 위함.

### Fingerprinting the bouncer

C&C 로 connect back 하는 간단한 앱을 안드로이드 마켓에 제출했다.

QEMU 베이스의 에뮬레이터에서 돌린다.

[A fistfull of red-pills: How to automatically generate procedures to detect CPU emulators](http://static.usenix.org/event/woot09/tech/full_papers/paleari.pdf)

이 사람들... [Remote connect-back shell](http://www.youtube.com/watch?v=ZEIED2ZLEbQ) 까지 돌려봤다.
