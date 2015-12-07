---
layout: post
title: "Heap Overflows"
date: 2014-11-14 16:36:14 +0900
comments: true
categories: 
---

[Solar Designer](http://en.wikipedia.org/wiki/Solar_Designer) 가 2000년 7월 발표한 "JPEG COM Marker Processing Vulnerability" 는 heap overflow 에 관해 seminal 한 문서이다.

우선,

[The Art of Software Security Assessment: Identifying and Preventing Software Vulnerabilities](http://www.amazon.com/Art-Software-Security-Assessment-Vulnerabilities/dp/0321444426/ref=sr_1_1?ie=UTF8&qid=1415936171&sr=8-1&keywords=the+art+of+software+security+assessment) 에서 heap overflow 부분을 정리해본다.

### unlink operation

Doubly-linked list 에서 특정 element 를 뜯어내는 함수가 다음과 같이 있다고 하자.

    int unlink(ListElement *element)
    {
        ListElement *next = element->next;
        ListElement *prev = element->prev;

        next->prev = prev;
        prev->next = next;
        return 0;
    }

편의상 ListElement 는 다음과 같다고 하자.

    struct _ListElement {
        ListElement *prev;
        ListElement *next;
    };
    typedef struct _ListElement ListElement;

unlink() 에 골때린 input 을 줄 수 있다면?

    void main()
    {
        char buf[512];
        ListElement a;
        a->prev = (ListElement*)0x30303030;
        a->next = buf;
        unlink(&a);
    }

를 하면, `buf[0]` ~ `buf[7]` 에 `0x0000000030303030` 이 세팅된다. (그리고 죽는다. 두번째 assign 에서 죽는데, 일단 무시)

즉 linked list 의 unlink() 함수에는 memory write operation 이 들어있으니, linked list 의 element 내용을 corrupt 시키고, unlink() 를 trigger 시킬 수 있으면, memory write 를 할 수 있는 것이다.

링크드 리스트 관련 함수를 memory_write() 함수로 쓸 수가 있다!

    (gdb) break main
    Breakpoint 1 at 0x40058c: file list.c, line 17.
    (gdb) run
    Starting program: /home/z/shell/a.out
    warning: no loadable sections found in added symbol-file system-supplied DSO at 0x7ffff7ffa000

    Breakpoint 1, main () at list.c:17
    17      {
    (gdb) list
    12          next->prev = prev;
    13          prev->next = next;
    14          return 0;
    15      }
    16      main()
    17      {
    18          char buf[512];
    19          char buf2[512];
    20          ListElement a;
    21          a.prev = (ListElement *)0x30303030;
    (gdb) step
    21          a.prev = (ListElement *)0x30303030;
    (gdb) step
    22          a.next = buf;
    (gdb) step
    23          unlink(&a);
    (gdb) print buf
    $1 = "\300\224\377\367\377\177\000\000<\247\335\367\377\177\000\000\370\251\242\367\377\177\000\000\020\244\335\367\377\177\000\000\000\000\000\000\001\000\000\000\202\b\000\000\001\000\000\000@+\335\367\377\177\000\000 \346\377\367\377\177\000\000\340\335\377\377\377\177\000\000\207\360\226|\000\000\000\000\330\234\377\367\377\177\000\000\260\336\377\377\377\177\000\000\330\331\377\367\377\177\000\000#E\336\367\377\177\000\000\000\000\000\000\000\000\000\000\330\234\377\367\377\177\000\000\001\000\000\000\377\177\000\000\000\000\000\000\000\000\000\000\001\000\000\000\377\377\377\377\330\331\377\367\377\177\000\000\000\000\000\000\000\000\000\000H\003@", '\000' <repeats 15 times>"\340, \270\377\377\377\377\000\000\266\"\275\357\377\377", '\000' <repeats 16 times>, " \346\377\367\377\177\000\000@\250\335\367\001\000\000\000\000\000\000\000\001", '\000' <repeats 11 times>, "<\247\335\367\377\177\000\000\211qy\005", '\000' <repeats 29 times>"\240, \241\367\377\177\000\000؎\243\367\377\177\000\000\300\216\243\367\377\177\000\000 0\335\367\377\177\000\000ب\242\367\377\177\000\000\300\224\377\367\377\177\000\000\000\000\000\000\377\177", '\000' <repeats 14 times>...
    (gdb) print &buf
    $2 = (char (*)[512]) 0x7fffffffdc30
    (gdb) p a.prev
    $3 = (struct _ListElement *) 0x30303030
    (gdb) p a.next
    $4 = (struct _ListElement *) 0x7fffffffdc30
    (gdb) step
    unlink (a=0x7fffffffdc20) at list.c:10
    10          ListElement *next = a->next;
    (gdb) step
    11          ListElement *prev = a->prev;
    (gdb) step
    12          next->prev = prev;
    (gdb) print next
    $5 = (ListElement *) 0x7fffffffdc30
    (gdb) step
    13          prev->next = next;
    (gdb) x 0x7fffffffdc30 
    0x7fffffffdc30: 0x30303030 <- buf 에 \x30\x30\x30\x30
    (gdb)

정말 간략하게 말하면, malloc() 된 데이터의 metadata 를 고쳐서, free() 로 `*(next) = prev;` 같은 memory write 를 하는 것이다.

### Back to the 90's

예전 dlmalloc 에서 chunk 를 뜯어내던 unlink 매크로. 

``` c
/* Take a chunk off a bin list */
#define unlink(P,BK,FD) {\
    FD = P->fd;         \
    BK = P->bk;         \
    FD->bk = BK;        \
    BK-fd = FD;         \
}
``` 


최신 glibc 2.20 에 들어있는 매크로

``` c
#define unlink(P, BK, FD) {
    FD = P->fd; \
    BK = P-bk; \
    if (__builtin_expect (FD->BK != P || BK->fd != P, 0)) \
        malloc_printerr (check_action, "corrupted double-linked list", P); \
```

2.20 에는 Sanity check 가 들어간다. 뜯어내기 전 상태의 link pointer 들이 올바르게 세팅되어 있는지.

### dlmalloc

dlmalloc 에는 4 개의 chunk 가 있다. `designated victim`, `top`, `smallbins`, `treebins` 가 그것.

``` c
strcut malloc_state {
    binmap_t    smallmap;
    binmap_t    treemap;
    size_t      dvsize;
    size_t      topsize;
    ...
    mchunkptr   dv;
    mchunkptr   top;
    ...
    mchunkptr   smallbins[(NSMALLBINS+1)*2];
    tbinptr     treebins[NTREEBINS];
    ...
}
```

#### Binning

free chunk 들은 bin 으로 관리된다.
512 bytes 미만의 alloc request 를 위한 bin 들은 8 byte 단위로 존재한다.

16,24,32,... 이런 셈.

#### Algorithm for malloc in dlmalloc

For small requests < 256 bytes
(Using the dv chunk provides some locality as unserved requests get memory next to each other)

* Check if there is a free chunk in the corresponding exact-size bin
* If not, look into the next larger exact-size bin and check there
* If that bin had no chunk too, check the designated victim chunk
* If the dv chunk was not sufficiently large
- search the smallest available small-size chunk
- split off a chunk of needed size
- make the rest the designated victim chunk
* If no suitable small-size chunk was found
- split off a piece of a large-size chunk
- make the remainder the ew dv chunk
* Else, get memory from the system

Large requests >= 256 bytes

* Non-exact bins organize the chunks as binary search trees
* Two equally spaced bins for each power of two
* Every tree node holds a list of chunks of the same size
* Tree is traversed by inspecting the bits in size (from more significant to less significant)
* Everything above 12M goes into the last bin (usually very rare)

[Analysis on Dynamic Memory Allocation, Wenbin Fang](http://wenbin.org/doc/writing/malloc.pdf) 에서 가져온 dlmalloc algorithm.

```
void *malloc(size_t size) {
    if (size <= 252) {
        // 1. small bins
        free_chunk = UnlinkFromSmallBins(size);
        if (free_chunk != NULL) goto EXIT;
        // 2. designated victim
        free_chunk = GetFromDesignatedVictim(size);
        if (free_chunk != NULL) {
            dv = Split(free_chunk);
            goto EXIT;
        }
    }
    // 3. tree bins
    free_chunk = UnlinkFromTreeBins(size);
    if (free_chunk != NULL) {
        remainder = Split(free_chunk);
        if (dv != NULL) Insert(dv, SmallBins OR TreeBins);
        dv = remainder;
        goto EXIT;
    }
    // 4. top chunk
    free_chunk = SplitFromTopChunk(size);
EXIT:
    if (free_chunk != NULL) SetPinuse(NextAdjacentChunk(free_chunk));
    return free_chunk;
}
```

#### 참고

* [A Memory Allocator, Doug Lea](http://g.oswego.edu/dl/html/malloc.html) 
* [JPEG COM Marker Processing Vulnerability](http://www.openwall.com/articles/JPEG-COM-Marker-Vulnerability)
* [Vudo - An object superstitiously believed to embody magical powers](http://www.phrack.org/issues.html?issue=57&id=8#article)
* [Once upon a free()...](http://www.phrack.org/issues.html?issue=57&id=9#article)
* [Advanced Doug lea's malloc exploits](http://www.phrack.org/issues.html?issue=61&id=6#article)
* [MallocMaleficarum.txt](http://www.packetstormsecurity.org/papers/attack/MallocMaleficarum.txt)
* [Linux Heap Exploiting Revisted](https://prezi.com/wcnbbokuousb/linux-heap-exploiting-revisited-en/)
* [w00w00 on Heap Overflows](http://www.cgsecurity.org/exploit/heaptut.txt)
* [Run-time Detection of Heap-based Overflows, USENIX LISUSENIX LISUSENIX LISA](https://www.usenix.org/legacy/event/lisa03/tech/full_papers/robertson/robertson_html/)
* [Use a heap overflow to write arbitrary data](http://stackoverflow.com/questions/9646608/use-a-heap-overflow-to-write-arbitrary-data)
* [Understanding the Heap & Exploiting Heap Overflows, Mathy Vanhoef](http://www.mathyvanhoef.com/2013/02/understanding-heap-exploiting-heap.html)
* [Analysis on Dynamic Memory Allocation, Wenbin Fang](http://wenbin.org/doc/writings/malloc.pdf)
