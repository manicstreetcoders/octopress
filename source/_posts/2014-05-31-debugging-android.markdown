---
layout: post
title: "Debugging Android"
date: 2014-05-31 20:50:15 +0900
comments: true
categories: 
---

기본적인 툴들을 인스톨한다.

* Android shell 은 MirBSD Korn shell. 불편하니 sshd 나 dropbear 를 인스톨한다.
* Android 의 대부분의 CLI 커맨드는 toolbox 로 구현되었다. busybox 를 인스톨한다.
* Emulator 의 /system/xbin 에 있는 툴들을 pull 해서, device 로 push 한다. Android 버전이 맞아야 하고, `objdump` 로 필요한 라이브러리를 파악해서 같이 옮겨준다.

Dalvik 과 인터페이스하는 CLI 툴들.

* am: Interface to activity manager
* bmgr: Backup Manager
* content: Interface with Android Content Providers
* ime
* input
* monkey
* pm
* settings
* svc
* wm
