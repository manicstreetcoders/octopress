---
layout: post
title: "A Memory Allocator"
date: 2015-04-27 19:38:35 +0900
comments: true
categories: 
---

Doug Lea 의 dlmalloc 소스를 보다가 간단하게 하나 만들어 보기로 했다.

find-first-fit 으로 그냥 대충 만들어봤다. `unlink()` 가 들어있어서, exploit 은 가능한 버전으로.

binning 은 없다. ㅋㅋㅋ `sbrk()` 로 grow 하지도 않는다. ㅋㅋㅋ

``` c kcmalloc.c
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>

#define SBRK_UNIT (1024*10)

typedef struct _chunk {
    int in_use;
    size_t size;
    char *fd;
    char *bk;
} chunk;

#define mem2chunk(p) ((void*)(p) - sizeof(chunk))
#define chunk2mem(p) ((void*)(p) + sizeof(chunk))

static char *_start;
static char *_end;
static char *_top;
static chunk _free;

void kcinit()
{
    _start = (char*)sbrk(SBRK_UNIT);
    if (_start == (char*)-1) {
        printf("sbrk faield.\n");
        return;
    }
    _end = _start + SBRK_UNIT;
    _top = _start;
    _free.fd = _free.bk = &_free;
}
void add_list(chunk *head, chunk *item)
{
    chunk *tail;
    tail = (chunk*)head->bk;
    tail->bk = (char*)item;
    item->bk = (char*)head;
    head->bk = (char*)item;
    if (head->fd == head)
        head->fd = item;
}
#define unlink(P,BK,FD) {\
    BK = P->bk; FD = P->fd; FD->bk = BK; BK->fd = FD; }
chunk *find_first_fit(size_t alloc_size)
{
    chunk *fd;
    chunk *bk;
    chunk * thru;
    thru = (chunk*)_free.fd;
    while (thru != &_free) {
        if (alloc_size <= thru->size) {
            unlink(thru, bk, fd);
            return thru;
        }
    }
    return NULL;
}
void *kcmalloc(size_t size)
{
    char *p;
    chunk *c;
    size_t alloc_size;

    alloc_size = size + sizeof(chunk);

    if ((p=find_first_fit(alloc_size)) == NULL) {
        if (_top+alloc_size >= _end) {
            printf("kcmalloc failed.\n");
            return NULL;
        }
        p = _top;
        _top += alloc_size;

        c = (chunk*)p;
        printf("kcmalloc: c %x\n", c);
        p += sizeof(chunk);
        printf("kcmalloc: p %x\n", p);

        c->in_use = 1;
        c->size = alloc_size;
    } else {
        c = (chunk*)p;
        p += sizeof(chunk);
        c->in_use = 1;
    }
    return p;
}
void dump_list(chunk *head)
{
    chunk *thru;

    thru = (chunk*)head->fd;
    printf("dump free_list\n");
    while (thru != head) {
        printf("free_list -> %x\n", thru);
        thru = (chunk*)thru->bk;
    }
}
void kcfree(void *mem)
{
    chunk *c;
    c = mem2chunk(mem);
    c->in_use = 0;
    add_list(&_free, c);
}
void main()
{
    char *a;
    char *b;

    kcinit();

    a = kcmalloc(1024); /* <- [1] */
    b = kcmalloc(128);  /* <- [2] */

    strcpy(a,"babo");
    puts(a);
    strcpy(b,"babo2");
    puts(b);

    kcfree(a);          /* <- [3] */
    dump_list(&_free);

    a = kcmalloc(1000); /* <- [4] */
    puts(a);            

    kcfree(b);          /* <- [5] */
    dump_list(&_free);
}
```

이걸 실행시키면

```
% ./m
kcmalloc: c 7f6000
kcmalloc: p 7f6020
kcmalloc: c 7f6420
kcmalloc: p 7f6440
babo
babo2
dump free_list
free_list -> 7f6000
babo
dump free_list
dump free_list
free_list -> 7f6420
%
```

`[3]` 에서 `a` 에 alloc 되었던 chunk 가 free list 로 들어간다. 그리고 `[4]` 에서 1000 을 alloc 하면 free list 에서 가져오게 된다. chunk 가 reuse 되니까 `puts()` 에서 동일하게 `babo` 가 출력된다. free list 에서 chunk 를 떼어내면서 `unlink()` 가 들어가게 된다.

만약 프로그램 순서가 `kcfree(b)` -> `kcmalloc(100)` 으로 가고, 그 사이에서 `a` 를 이용해 `b` 의 바운더리태그의 `fd`, `bk` 를 조작할 수 있다면 `kcmalloc(100)` 할 때 `unlink()` 에서 memory write 가 가능해진다.

그럼 프로그램을 조금 바꿔보자.

``` c kcmalloc.c 
void main()
{
    char *a;
    char *b;
    char *c;

    kcinit();

    a = kcmalloc(1024); /* <- [1] */
    b = kcmalloc(128);  /* <- [2] */

    strcpy(a,"babo");
    puts(a);
    strcpy(b,"babo2");
    puts(b);

    kcfree(b);          /* <- [3] */

    memset(a,0xff,512+32);  /* <- [6] */

    a = kcmalloc(1000); /* <- [4] */
    puts(a);            

    kcfree(b);          /* <- [5] */
}
```

이제 a 뒤에 위치한 b 를 free 해서 b 를 linked list 에 집어 넣는다[3]. 그래야 unlink() 로 떼어낼 수 있을테니. 그리고 a 에 alloc 된 메모리를 쭈욱 밀어서 b 앞에 위치한 boundary tag 를 오염시킨다[6]. 그리고 kcmalloc() 에 들어간다[4]. free chunk 에서 알맞은 청크를 찾았다. 그래서 list 에서 unlink() 로 떼어날때 메모리 에러가 난다.

```
% ./m
kcmalloc: c 1d00000
kcmalloc: p 1d00020
kcmalloc: c 1d00220
kcmalloc: p 1d00240
babo
babo2
Segmentation fault (core dumped)
%
```

이런식으로 memory write 를 하는 것이 heap overflow 의 기초.

``` c unlink()
#define unlink(P,BK,FD) {\
    BK = P->bk; \
    FD = P->fd; \
    FD->bk = BK; \
    BK->fd = FD; \
    }
```

이제 본격적으로 dlmalloc 을 뒤져봐야겠다.
