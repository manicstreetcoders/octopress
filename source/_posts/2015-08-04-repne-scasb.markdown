---
layout: post
title: "REPNE SCASB"
date: 2015-08-04 17:59:16 +0900
comments: true
categories: 
---

[REPNZ SCAS Assembly Instruction Specifics](http://stackoverflow.com/questions/26783797/repnz-scas-assembly-instruction-specifics)

스택오버플로우에서 `strlen(s)` 의 idiom 을 보게되었다.

#### REPNE SCASB

```
while (ecx != 0) {
    ZF = (al == *(BYTE *)edi);
    if (DF == 0)
        edi++;
    else
        edi--;
    ecx--;
    if (ZF) break;
}
```

```
sub ecx,ecx
sub al,al
not ecx
cld
repne scasb
not ecx
dec ecx
```

```
ecx = (unsigned) -1;
while (ecx) {
    ZF = (0 == *(BYTE *)edi);
    edi++;
    ecx--;
    if (ZF) break;
}
ecx = ~ecx;
ecx--;
```

* in two's complement notation, flipping the bits of `ecx` is equivalent to `-ecx-1`
* in the loop, `ecx` is decremented before the loop breaks, so it decrements by `length(edi) + 1` in total.
* `ecx` can never be zero in the loop, since the string would have to occupy the entire address space.
* 루프가 끝나면, `ecx` 는 `-1 - (length(edi)+1) == -(length(edi)+2)`
* bit flipping 하면, `length(edi)+1`
* `dec` 하면 `length(edi)`

#### Bruce Dang

이번에는 Bruce Dang 의 책의 [연습문제 풀이](https://johannesbader.ch/2014/05/practical-reverse-engineering-exercises-page-11/).

```
SECTION .data
my_str:
    db  'The pool on the roof must have a leak.',0
SECTION .text
GLOBAL _start
_start:
    nop
    push byte 'x'
    push dword my_str
    call blackout
    add esp,8
    mov ebx,0
    mov eax,1
    int 080h
blackout:
    push ebp
    mov ebp,esp

    mov edi,[ebp+8]
    mov edx,edi
    xor eax,eax
    or ecx,0ffffffffh
    repne scasb
    add ecx,2
    neg ecx
    mov al,[ebp+0ch]
    mov edi,edx
    rep stosb
    mov eax,edx

    mov esp,ebp
    pop ebp
    ret
```

C 로 따지면,

```
char *black_out(char *str, char ch)
{
    int len = strlen(str);
    memset(str,ch,len);
    return str;
}
```
