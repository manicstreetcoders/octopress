---
layout: post
title: "SoK: Eternal War in Memory"
date: 2013-09-24 17:54
comments: true
categories: 
---

Browser fingerprinting, JavaScript obfuscation, IE exploit PoC 들은 어느정도 파악되었으니,
Windows system programming 쪽으로 이동한다. 최종 목표는 DbD 와 APT 를 수행할 수 있는
툴셋 확보.

다음 단계 일들은 API Hooking, DLL Injection 분야.

그 전에 메모리 관련 exploit 의 역사를 정리해보기로 하였다.

정리가 잘 되어 있는 SoK: Eternal War in Memory 라는 논문을 시작점 삼아 진행한다.

* James P. Anderson. Computer Security Technology Planning Study, 1972
* AlephOne, Phrack article "Smashing the Stack for Fun and Profit" 
* return-into-libc
* advanced return-into-libc
* jump oriented programming
* return oriented programming
* integer overflow 
* use-after-free
* heap overflow
* vtable
* heap spraying 
* stack protection
* heap protection
* SafeSEH
* DEP
* ASLR

등을 다뤄보겠다.

메모리 공격에 관한 중요한 마일스톤을 적어놓은 [타임라인](http://zynamics.files.wordpress.com/2010/02/code_reuse_timeline1.png)을 보면 창과 방패의 공방이 이해가 빠를 것.
(1972 년에 미정부가 진행한 연구 "Computer Security Technology Planning Study" 에 buffer overflow 공격에 대해 기술되어 있다는 점이 흥미롭다.)

### Smashing the Stack

TBD

### W-xor-X a.k.a Data Execution Prevention

TBD

### return-into-libc

`W^X` 를 피해가기 위해, [return-into-libc](http://seclists.org/bugtraq/1997/Aug/63) 가 개발되었다.

`W^X` 때문에, writable 한 buffer 에 i.e. system("/bin/sh") 과 같은 코드를 집어넣고 수행시키는 것이
불가능해졌다. 메모리에 shellcode instruction 을 집어넣을 수 있다 하더라도, 
CPU 가 수행을 안해버리니 난감한 상황이 벌어진 것.

이에 대항하여 return-into-libc 라는 기법이 Solar Designer 에 의해 제시되었다.

return-into-libc 는 excutable 하다고 마킹된 기존의 코드를 재활용하는 컨셉이다.

메모리에 로드되는 libc 코드에서 system() 의 위치와 "/bin/sh" 의 위치를 찾을 수 있고,
stack frame 을 overwrite 할 수 있다면, 다음과 같이 stack frame 을 overwrite 해서,
shellcode 를 제공하지 않고도 exploit 을 성공시킬 수 있다.

    [ pointer to "/bin/sh" ]
    [ dummy_ret_address    ]
    [ pointer to system()  ] <- this int32 should overwrite return address of a vulnerable function

### The advanced return-into-libc

여러개의 함수를 호출해야 하는 경우를 위해 [고도화된 버전](http://www.phrack.com/issues.html?issue=58&id=4)이 제시되었다.

#### "esp lifting" 기법

`-fomit-frame-pointer` 로 컴파일된 바이너리를 공격하기 위해 개발된 기법이다.
이 경우 함수의 return 부분이 다음과 같이 생성되는데,

    eplg:
            addl $LOCAL_VARS_SIZE,%esp
            ret

이제 아래와 같이 stack frame 을 overwrite 할 수 있다면, 2 개의 함수를 호출하는 것이
가능하다.

    [ pointer to "/bin/sh" ]
    [ dummy_ret_address    ]
    [ pointer to system()  ]
    [ PAD                  ]
    [ 0                    ]
    [ pointer to eplg      ]
    [ pointer to setuid()  ] <- this int32 should overwrite return address of a vulnerable function

취약점을 가지고 있는 함수가 ret 하면 setuid() 로 뛰게 되고 setuid(0) 가 수행되며,
setuid() 는 eplg 로 return 하게 된다.

eplg 는 addl $LOCAL_VARS_SIZE,%esp 로 [ 0 ] 과 [ PAD ] 를 날리고, system() 으로 ret 하게 된다.
system() 은 "/bin/sh" 을 arg 로 받아 수행하게 된다.

#### pop+ret method

address space 에서 pop+ret 인스트럭션이 있는 주소를 찾아서,

    pop-ret:
            popl any_reg
            ret

다음과 같이 stack frame 을 overwrite 한다.

    [ pointer to "/bin/sh" ] <- arg1 for system()
    [ dummy_ret_address    ] <- ret address for system(). doesn't matter in this stage.
    [ pointer to system()  ] <- supply ret address for pop+ret
    [ 0                    ] <- arg1 for setuid(). will be pop'ed by pop+ret
    [ pointer to pop+ret   ] <- supply ret address for setuid()
    [ pointer to setuid()  ] <- this int32 should overwrite return address of a vulnerable function

취약점을 가지고 있는 함수가 ret 하면 setuid(0) 로 튀게 된다.
그리고 setuid(0) 가 끝나면 pop-ret 인스트럭션이 있는 주소로 튀게 된다.
pop-ret 은 [ 0 ] 을 날리고, system() 으로 튀게 된다.

만약 setuid() 가 아니라 argument 를 2 개 먹는 함수를 호출해야 한다면,

    pop-ret2:
            popl any_reg
            popl any_reg
            ret

이런 인스트럭션의 조합을 어드레스 스페이스에서 찾아서 그 주소를 사용해야 한다.

### Return Oriented Programming

Multiple functions 을 수행하는 지경까지 가니까, 굳이 function 단위로 수행할 필요가 있겠는가?
인스트럭션을 여기저기서 모아서 짜깁기해서 돌려보자... 로 발전한 것은 어찌보면 당연하다 하겠다.

ROP 란 ret 으로 끝나는 인스트럭션 셋 (gadget 이라 칭함) 들을 여기저기서 모아서 조립하는 식이다.

    684a0f4e:
        pop eax
        ret

    684a2367:
        pop ecx
        ret

    684a123a:
        mov [ecx],eax
        ret

란 instruction set 들을 모아서 스택에서 조립한다. 

    [0x684a123a] <- mov [ecx],eax
    [0x684a2367] <- pop ecx
    [0x684a0f4e] <- pop eax

로 스택을 조립하면, pop eax 를 하고 ret 해서 pop ecx 로 튀고, 다음에는 mov[ecx],eax 로 튀는 식이다.

이런 다양한 gadget 들을 exploit 하려는 process 의 어드레스 스페이스에서 찾아낼 수 있다면,
shellcode 를 공급하지 않고도, malicious computation 을 수행할 수 있게 되는 것이다.

address space 에서 ret (opcode: 0xc3) 을 찾아서 그 앞쪽으로 써치해가면 
사용할 수 있는 gadget 들이 어떤 것인지를 파악할 수 있게 된다.

(적절한 기존 코드가 있다면 ROP 를 사용해서) 기존 프로그래밍 랭귀지와 동등하게 
computational problem solving 이 가능한 code 를 제작할 수 있다는 의미에서 
ROP 가 Turing Complete 라고도 이야기한다.

Turing Complete 라는 의미에서 기존의 EIP 기반의 instruction 수행이 
ESP 기반의 ROP 에서 어떻게 매핑되는지 보겠다.

#### NOP

TBD

#### Immediate Constants

TBD

#### Control Flow

TBD

### Jump-Oriented Programming

TBD

### ASLR

TBD

### 참고 문헌

* Solar Designer. Getting around non-executable stack (and fix)
* Nergal. The Advanced Return-into-libc Exploits: PaX Case Study. Phrack Magazine, Vol. 11, Issue 0x58, 2001.
* Hovav Shacham, Return-Oriented Programming: Exploits Without Code Injection, Black Hat USA 2008
* Ryan Roemer, Return-Oriented Programming: Systems, Languages, and Applications. ACM
* SoK: Eternal War in Memory
