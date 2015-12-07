---
layout: post
title: "Structured Exception Handling"
date: 2014-09-21 05:39:25 +0900
comments: true
categories: 
---

[A Crash Course on the Depths of Win32(TM) Structured Exception Handling](http://www.microsoft.com/msj/0197/exception/exception.aspx) 를 따라가본다.

#### Exception Handler

    DWORD scratch;

    EXCEPTION_DISPOSITION
    __cdecl
    _except_handler(
    struct _EXCEPTION_RECORD *ExceptionRecord,
    void * EstablisherFrame,
    struct _CONTEXT *ContextRecord,
    void * DIspatcherContext)
    {
        unsigned i;
        printf("Hello from an except handler\n");
        ContextRecord->Eax = (DWORD)&scratch;
        return ExceptionContinueException;
    }

#### Set-up

`_EXCEPTION_REGISTERATION` 을 셋업한다. 예전 VS 에는 어셈블리 INC 에 정의되어있었는데, 최신 VS 에는 C 로 정의되어있다.

    _EXCEPTION_REGISTRATION struc
        prev dd ?
        handler dd ?
    _EXCEPTION_REGISTRATION ends

FS 가 [TIB](http://en.wikipedia.org/wiki/Win32_Thread_Information_Block) 을 가리키고 있는데, 처음 DWORD 가 thread 의 EXCEPTION_REGISTRATION 구조체를 가리킨다.

`FS:[0]` 이 현재 EXCEPTION_REGISTRATION 을 가리키고 있으니, `push handler` `push FS:[0]` 을 연달아 하면 `prev` `handler` 가 메모리에 예쁘게 세팅되고, `mov FS:[0],ESP` 로 새로운 EXCEPTION_REGISTRATION 을 `FS:[0]` 에 세팅할 수 있다.

그 다음에 `mov [eax],1` 로 액세스 바이올레이션을 일으킨다. 그러면, `printf("Hello from an except handler\n");` 가 `printf("After writing!\n");` 보다 먼저 handler 가 실행되게 된다.

익셉션 핸들러에서 `ContextRecord->Eax = (DWORD)&scratch;` 로, EAX 를 0 에서 올바른 주소로 설정하고 continue 시킨다. OS 에서, `ExceptionContinueException` 를 받으면, 문제가 해결된 것으로 보고, fault 를 만들었던 instruction 을 재실행한다.

    DWORD handler = (DWORD) _except_handler;
    __asm {
        push handler
        push FS:[0]
        mov FS:[0],ESP

        mov eax,0
        mov [eax],1
    }
    printf("After writing!\n");
    __asm {
        mov eax,[ESP]
        mov FS:[0],EAX
        add esp,8
    }
    _gettch();

