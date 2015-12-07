---
layout: post
title: "Linux Memory Layout"
date: 2014-11-18 09:02:27 +0900
comments: true
categories: 
---

간단한 프로그램.

    #include <stdio.h>
    #include <stdlib.h>

    static char buf[1];
    char buf2[1];
    static char buf3[] = "b";
    long rip()
    {
        long *p;
        p = (long*)&p;
        p++;
        p++;
        return *p;
    }
    int main()
    {
        char arr[128];
        char *heap;

        printf("stack %p\n", arr);
        printf("static uninitialized %p\n", buf);
        printf("(implicit) static uninitialized %p\n", buf2);
        printf("static initialized %p\n", buf3);
        printf("rip %p\n", (void*)rip());
        heap = (char*)malloc(128);
        printf("heap %p\n", heap);
    }

    ./a.out
    stack 0x7fff9caa8b50 (Stack)
    static uninitialized 0x601051 (Uninitialized data)
    (implicit) static uninitialized 0x601052 (Uninitialized data)
    static initialized 0x4006f4 (Initialized data)
    rip 0x400648 (Code)
    heap 0x2232010 (Heap)

스택은 아주 윗쪽에,
Text 는 0x400648 언저리에 놓여있고,
Initialized data 는 Text 바로 뒤에 놓여있으며, (0x4006f4)
Uninitialized data (bss) 는 Initialized data 뒷쪽에,
그리고 힙은 bss 뒷편에

    ---------------------------
    | Stack                   |
    ---------------------------
    | Heap                    |
    ---------------------------
    | Uninitialized data (bss)|
    ---------------------------
    | Initialized data        |
    ---------------------------
    | Code                    |
    ---------------------------

그러면, `/proc/<pid>/maps` 를 보고 확인해본다.
