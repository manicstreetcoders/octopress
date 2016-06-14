---
layout: post
title: "The poisoned NUL byte"
date: 2016-06-14 10:19:02 +0900
comments: true
categories: 
---

Google Project Zero blog 에서.

* [The poisoned NUL byte, 2014 edition](http://googleprojectzero.blogspot.kr/2014/08/the-poisoned-nul-byte-2014-edition.html)

```
$ CHARSET=//ABCDE pkexec
*** Error in `pkexec`: malloc(): memory corruption: 0x00007f15bc0732d0 ***
```
