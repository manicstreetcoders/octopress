---
layout: post
title: "Windows API Hooking"
date: 2013-08-20 09:37
comments: true
categories: 
---

몇 가지 Windows API Hooking 라이브러리를 테스트 해보기로 했다.

Microsoft 의 Detours. x86 버전만 free 이고, x64 버전은 돈을 받는다. 탈락.

[MinHook - The Minimalistic x86/x64 API Hooking Library](http://www.codeproject.com/Articles/44326/MinHook-The-Minimalistic-x86-x64-API-Hooking-Libra) 을 발견.

읽고 정리해본다.

Inline hooking 을 하는 듯 하다.

Target funciton 의 진입 부분을 JMP 로 overwrite 한다.

x86 모드에서는 Target function 이 0x4000 0000 부근에 있다고 가정하면, 
5 bytes 를 차지하는 32bit relative JMP 로 커버 가능하다.

    0x4000 0000: E9 FB FF FF BF           JMP 0x0
    0x4000 0000: E9 FA FF FF BF           JMP 0xFFFFFFFF 

Target funciton (i.e. USER32.dll!MessageBoxW) 의 진입부분을 JMP 로 overwrite 하고,
Detour 로 뛰었다가, 다시 원래 코드로 돌아가야 한다.

이 부분을 Trampoline Function 이라고 부르는데, "원래 함수의 앞부분을 클론한 것" + 
"JMP to 원래 함수" 로 이루어져 있다.

    ; Original
    0x770E 11E4: 48 83 EC 38              SUB RSP, 0x38
    0x770E 11E8: 45 33 DB                 XOR R11D, R11D
    ; Trampoline
    0x7706 4BD0: 48 83 EC 38              SUB RSP, 0x38
    0x7706 4BD4: 45 33 DB                 XOR R11D, R11D
    0x7706 4BD7: FF 25 5B E8 FE FF        JMP QWORD NEAR [0x77053438]
    0x7705 3438: EB 11 0E 77 00 00 00 00

Visual Studio 2013 으로 빌드해서 테스트해보니 잘 동작한다.
