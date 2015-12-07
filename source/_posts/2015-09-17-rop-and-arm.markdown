---
layout: post
title: "ROP and ARM"
date: 2015-09-17 17:38:41 +0900
comments: true
categories: 
---

ARM 에서 메모리에다가 인스트럭션을 쓰고, 바로 control flow 를 그리로 돌리면, cache 때문에 공격자의 payload 가 실행되지 않는 케이스가 몇가지 있다. ARM 에서 첫단계는 ROP 로 구성해야하는 이유. Instruction cache flushing 을 하는 system call 이 있기는 한데, i.e. Linux 는 `cacheflush()`, 저걸 부르기 위해서는 먼저 code 수행을 만들어야하니 닭과 달걀의 문제가 되버린다고. 다행히 `mprotect()` 를 부르면 cache 가 flushing 되니, ROP 에서 mprotect() 로 payload 를 executable 하게 마크해주면 일거양득.

#### Basics of ROP on ARM

ARM ABI 에 따르면, subroutine 의 return address 는 link register 인 `lr` 에 저장된다. `bl` 이나 `blx` 는 다음에 실행할 인스트럭션의 주소를 `lr` 에 넣고는 subroutine 으로 뛴다. subroutine 에서는 `bx lr` 로 돌아온다. 

Return address 를 stack 에 쌓는 것은 별도로 코딩해줘야 하는데,

`stmia sp!, {r4, lr} # store link register and callee-saved r4 on stack`

`bl subroutine # call subroutine, trashing link register`

`ldmia sp!, {r4, lr} # load original link register and r4 from stack`

`bx lr # return to calling code`

가 되고, thumb 모드에서는

`push {lr} # store link register on stack`

`bl subroutine # call subroutine, trashing link register`

`pop {pc} # load original link register and return to calling code`

가 된다.

#### Combining Gadgets into a Chain

* ARM gadget 은 `ldmia sp!, {..., lr}; bx lr` 로 끝나는 명령어들
* Thumb gadget 은 `pop {..., pc}` 로 끝나는 명령어들

Stack 을 overwrite 할 수 있다고 가정하면, `pop {r0-r4, pc}` gadget 을 찾아서, 그 어드레스를 스택에 넣고, argument `r0-r4` 도 넣어준다. 함수가 return 할 때, `pop {pc}` 로 원래 돌아가야할 곳이 아닌, `pop {r0-r4, pc}` gadget 으로 튀고, 여기서 다시 새로운 gadget 주소로 튀고, 튀고 튀고...

이제 `lr` 조정을 해주는 chain 을 보면... 스택에 `lr` 값까지 overwrite 하고, `pop {r0, lr}; bx lr` 로 `lr` 세팅을하고 `lr` 이 가리키는 주소로 튀고, 거기서 `pop {pc}` gadget 으로 다시 튀고. 이제 `lr` 은 `pop {pc}` 를 가리키고 있으니, `bx lr` 을 만나도 ROP 체인이 끊기지 않는다.

#### Case Study: Android 4.0.1 Linker

Android 4.0 대에서는 rild 의 dynamic linker 부분이 항상 0xb0001000 에 위치한다는 것. 4.1 부터는 이것마저도 ASLR 이 적용되었지만.

ROP chain 이 하는 일은 1 page 를 alloc 해서 (4096 byts), executable 하게 만들고, 공격자의 payload 를 카피해서 글루 튀는 것.

Stack pivot 부터 시작해야하는데, stack pivot 은 sp 를 내가 내용을 컨트롤하는 메모리를 가리키게해놓는것.

`mmap()` 으로  메모리를 할당하는데, `0xb1008000` 에 할당하도록 한다고.

다음은 mmap() 으로 executable 한 메모리를 `0xb1008000` 에 할당하는 ROP 체인.

Stack pivot 할 때, `lr = 0xb006545` 로 맞춰줘야한다. 0xb0006544 에 있는 gadget 은 `pop {r4,r5,pc}`. 이 gadget 이 mmap() 의 5,6 번째 파라메터를 팝팝하고 다시 다음 체인으로 튀게 해준다.

```
0xb00038ca  # pop {r0-r4,pc} gadget 이 있는 주소
0xb0018000  # r0: static allocation target address
0x00001000  # r1: size to allocate = one page, 1 page 만큼 메모리를 할당
0x00000007  # r2: protection = read, write, execute
0x00000032  # r3: flags = MAP_ANON | MAP_PRIVATE | MAP_FIXED
0xdeadbeef  # r4: don't care
0xb0001678  # pc: __dl_mmap, returning to lr = 0xb006545 (mmap 이 subroutine 을 호출해서, lr 이 mmap() 함수 안에 어딘가를 가리키게 된다)
0xffffffff  # 5 번째 파라메터는 stack 에. fd = -1
0x00000000  # 6 번째 파라메터도 stack 에. offset = 0
0xdeadc0de  # next gadget 의 주소. (mmap 이 pop {pc} 를 해준다면)
```

