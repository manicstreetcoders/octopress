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

이제 `uint64_t* b_size_ptr = (uint64_t*)(b-8);` 을 통해서, `struct malloc_chunk` 구조체 의 `INTERNAL_SIZE_T size;` 를 가리키는 포인터를 만든다.