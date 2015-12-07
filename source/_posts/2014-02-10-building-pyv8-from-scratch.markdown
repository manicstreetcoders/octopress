---
layout: post
title: "Building PyV8 From Scratch"
date: 2014-02-10 21:53
comments: true
categories: 
---

Google V8 을 Ubuntu 12.04 LTS 에서 컴파일해보았다. x64 로 컴파일하는데 에러가 나고 삽질 연속. MBA 11 하즈웰에서 VMware Fusion 으로 버추얼 머신을 돌리고 거기서 컴파일하는데 컴파일이 하세월이다. 안드로이드 컴파일은 MBPr 에서 해야겠다.

PyV8 빌드가 최종 목표고, boost 빌드 -> V8 빌드 -> PyV8 빌드의 순서로 가본다.

V8 Build

* `export V8_HOME=/blah/blah/v8` 로 환경변수를 잡아주고.
* `werror=no` 로 워닝 폭탄을 제거한다.
* PyV8 과 붙일려고 하는 거라서, -fPIC (Position Independent Code) 로 만들어야한다. `export CCFLAGS=-fPIC`
* `scons arch=x64` 를 했는데 scons 에서 에러가 난다. 찾아보니 GYP 로 가면서 scons 는 deprecate 되었다고. `make x64` 로 땜빵해보자.
