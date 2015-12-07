---
layout: post
title: "GCC -fstack-protector"
date: 2014-03-21 22:33:38 +0900
comments: true
categories: 
---

GCC -fstack-protector 를 지정했을때 에 들어가는 stack guard 코드.

%fs 는 x86-64 에서 TLS selector. %fs:0x28 에서 canary 값을 가져온다.

    push    %rbp
    mov     %rsp,%rbp
    sub     $0x30,%rsp
    mov     %fs:0x28,%rax
    mov     %rax,-0x8(%rbp)
    ...
    callq   <memcpy@plt>
    mov     -0x8(%rbp),%rax
    xor     %fs:0x28,%rax
    je      0x4005fe
    callq   <__stack_chk_fail@plt>
    leaveq
    retq

Canary 값이 깨졌으면, GNU C Library 의 debug/stack_chk_fail.c 의 __stack_chk_fail() 을 호출한다.

Stack protection OFF, ASLR OFF, stack execution ON

    gcc overflow.c -o overflow -fno-stack-protector -z execstack
    sudo echo 0 > /proc/sys/kernel/randomize_va_space

