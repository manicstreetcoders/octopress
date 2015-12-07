---
layout: post
title: "Linux x86_64 Shellcode 6"
date: 2014-11-10 22:26:49 +0900
comments: true
categories: 
---

#### ret2libc

이제 바로 system() 으로 튀는 경우를 상정해본다.

#### 참고

* [Solar Designer: Getting around non-executable stack (and fix)](http://seclists.org/bugtraq/1997/Aug/63)
* [Nergal: Advanced return-into-lib(c) exploits (PaX case study)](http://phrack.org/issues/58/4.html)
