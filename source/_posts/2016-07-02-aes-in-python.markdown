---
layout: post
title: "AES in Python"
date: 2016-07-02 11:51:42 +0900
comments: true
categories: 
---

AES 알고리즘을 Python 으로 구현해본다.

* [Lecture Notes](https://engineering.purdue.edu/kak/compsec/NewLectures/Lecture8.pdf)

일단 block 사이즈는 128 비트인데, key 사이즈는 여러개가 될 수 있다. key 사이즈에 따라서 round 갯수가 결정된다.

key 사이즈는 128 bit 로 정하고 구현한다.

* 128 bit 키가 44 words (word 는 4 바이트) 로 확장된다. (키 스케쥴)
* 한 라운드가 키스케쥴에서 4 words (그러니까 16 바이트)의 키를 사용한다.
* 라운드는 10 라운드가 있다.

* 처음 w_0 ~ w_3 을 사용해서 "Add round key" 를 수행
* 그리고 Round 1 ~ Round 10 수행

* 초기 셋업인, "Add round Key" 는 `4x4 state 매트릭스` 와 w_0 ~ w_3 을 xor 한다.
* 각각의 라운드는 `SubBytes`, `ShiftRows`, `MixColumns`, `AddRoundKey` 로 구성된다.
* 마지막 라운드는 `MixColumns` 가 없다.
