---
layout: post
title: "Poison NULL byte"
date: 2016-06-13 16:46:53 +0900
comments: true
categories: 
---

[Shellphish 가 쓴 힙 공격 문서중에 하나, Poison NULL byte](https://github.com/shellphish/how2heap/blob/master/poison_null_byte.c) 를 읽고 정리한다.

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
