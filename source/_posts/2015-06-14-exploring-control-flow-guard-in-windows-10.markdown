---
layout: post
title: "Exploring Control Flow Guard in Windows 10"
date: 2015-06-14 22:50:29 +0900
comments: true
categories: 
---

* [트렌드마이크로의 CFG in Windows 10 보고서](http://sjc1-te-ftp.trendmicro.com/assets/wp/exploring-control-flow-guard-in-windows10.pdf)
* [Exploiting CVE-2015-0311, Part II: Bypassing Control Flow Guard on Windows 8.1 Update 3](https://blog.coresecurity.com/2015/03/25/exploiting-cve-2015-0311-part-ii-bypassing-control-flow-guard-on-windows-8-1-update-3/)
* [Control-Flow Integrity - Principles, Implementations, and Applications](http://research.microsoft.com/pubs/69217/ccs05-cfi.pdf)
* [Practical Control Flow Integrity & Randomization for Binary Executables](http://www.cs.berkeley.edu/~dawnsong/papers/Oakland2013-CCFIR-CR.pdf)
* [Native Client: A Sandbox for Portable, Untrusted x86 Native Code](http://static.googleusercontent.com/media/research.google.com/en/us/pubs/archive/34913.pdf)
* [Windows 10 Control Flow Guard Internals](http://www.powerofcommunity.net/poc2014/mj0011.pdf)

```
typedef int(*fun_t)(int);
int foo(int a) { printf("hello jack: %d\n", a); return a; }
class CTargetObject
{
public:
    fun_t _fun;
} 
int _tmain(int argc, _TCHAR* argv[])
{
    int i = 0;
    CTargetObject* o_array = new CTargetObject[5];
    for (i=0;i<1000;i++)
        o_array[i]._fun = foo;
    o_array[0]._fun(1); // <- indirect call
    return 0;
}
```

Assembly 를 보면,

```
    mov esi,[esi]
    push 1
    call esi
```

CFG 가 들어간 코드는,

```
    mov esi,[esi]
    mov ecx,esi
    push 1
    call @_guard_check_icall@4
    call esi
```

`_guard_check_icall` 은 `ntdll!LdrpValidateUserCallTarget` 을 point.

하는 일은 Compiler 가 미리 만들어둔 function call address table (실제로는 bitmap) 을 보고, valid 한 펑션 start address 인지 아닌지 판단하는 것.
Compiler instrumentation 이 매우 중요. 플러스 user space / kernel space 에서의 support routine 들이 있음.
