---
layout: post
title: "Own your Android! Yet Another Universal Root"
date: 2015-09-02 09:26:22 +0900
comments: true
categories: 
---

BlackHat 2015 에 발표된 Own your Android! Yet Another Universal Root, Wen Xu, Yubin Fu. 를 정리해본다.

[BlackHat 발표 영상](https://www.youtube.com/watch?v=HVP1c7Ct1nM)

#### Bug analysis

* CVE-2015-3636: Wen Yu 와 wushi (of Keen Team) 이 [Trinity fuzzer](https://github.com/kernelslacker/trinity) 로 찾음.

```
Paging fault # virtual address 00200200.
PC is at ping_unhash+0x50/0xd4
```

ICMP Ping 을 할 때 지나가는 코드인 것 같은데.

`socket(AF_INET, SOCK_DGRAM, IP_PROTO_ICMP)` 를 할 경우에 다음과 같은 코드를 지나간다고 한다.

``` c net/ipv4/af_inet.c
int inet_dgram_connect(struct socket *sock, struct sockaddr *uaddr, int addr_len, int flags)
{
    struct sock *sk = sock->sk;

    if (addr_len < sizeof(uaddr->sa_family))
        return -EINVAL;
    if (uaddr->sa_family == AF_UNSPEC)
        return sk->sk_prot->disconnect(sk, flags);
    ...
}
```

Socket 의 어드레스 패밀리가 unspecified 일 경우, `disconnect()` 를 호출하는데, PING (ICMP) 의 경우 불리는 함수는 `udp_disconnect()`.

``` c udp_disconnect()
int udp_disconnect(struct sock *sk, int flags)
{
    struct inet_sock *inet = inet_sk(sk);

    sk->sk_state = TCP_CLOSE;
    ...
    if (!(sk->sk_userlocks & SOCK_BINDPORT_LOCK)) {
        sk->sk_proto->unhash(sk);
        inet->inet_sport = 0;
    }
    sk_dst_reset(sk);
    return 0;
}
EXPORT_SYMBOL(udp_disconnect)
```

여기서 `sk_prot->unhash(sk)` 를 호출하는데, 그러면 `ping_unhash()` 가 호출된다.

`sk_hashed(sk)` 로 socket 이 hashed 되어있는지 확인하고, TRUE 라면 sk_nulls_node 를 지운다.

``` c ping_unhash()
void ping_unhash(struct sock *sk)
{
    struct inet_sock *isk = inet_sk(sk);
    pr_debug("ping_unhash(isk=%p,isk->num=%u)\n", isk, isk->inet_num);
    if (sk_hashed(sk)) {
        write_lock_bh(&ping_table.lock);
        hlist_nulls_del(&sk->sk_nulls_node);
        sock_put(sk);
        isk->inet_num = 0;
        isk->inet_sport = 0;
        sock_prot_inuse_add(sock_net(sk), sk->sk_prot, -1);
        write_unlock_bh(&ping_table.lock);
    }
}
```

`hlist_nulls_del(&sk->sk_nulls_node)` 를 하면, `__hlist_nulls_del(n)` 으로 지우고, `n->pprev = LIST_POISON2` 가 된다.

``` c hlist_nulls_del(struct hlist_nulls_node *n)
static inline void __hlist_nulls_del(struct hlist_nulls_node *n)
{
    struct hlist_nulls_node *next = n->next;
    struct hlist_nulls_ndoe **pprev = n->pprev;
    *pprev = next;
    if (!is_a_nulls(next))
        next->pprev = pprev;
}

static inline void hlist_nulls_del(struct hlist_nulls_node *n)
{
    __hlist_nulls_del(n);
    n->pprev = LIST_POISON2;
}
```

`n->pprev = LIST_POISON2` 라는 코드가 있는데, `LIST_POISON2` 의 값은 Android 32/64 bit kernel 에서 모두 `0x200200` 이다. 

저자에 따르면,

* connect 를 두번째 부를때, `ping_unhash()` 가 또 불리고, 리스트에서 지워지기는 했지만, `sk_hashed(sk)` 체크는 통과해서 다시 `hlist_nulls_del(&sk->sk_nulls_node)` 가 또 불린다.
* `n->pprev` 는 전번의 콜 `n->pprev = LIST_POISON2` 에 의해서 LIST_POISON2 인 상황인데, `**pprev = n->pprev;` -> `*pprev = next;` 가 돌아가면 `0x200200` 에 write 가 들어간다.
* 0x200200 이 page mapping 이 안되어 있으면, kernel 에서 fault 가 난다.

여기서 local DoS 가 발생하는데, 저자에 따르면 local DoS 는 전체 그림의 일부분이라는 것.

`ping_unhash()` 에서, `sk_hashed()` 체크 이후에, 리스트에서 지우고, `sock_put()` 을 부르는데, 저자들은 이게 수상쩍다고 한다.
`sock_put()` 를 보면, `sk_refcnt` 를 줄이고, 0 이면 sk_free() 를 통해서 free 가 들어간다. 

``` c sock_put()
static inline void sock_put(struct sock *sk)
{
    if (atomic_dec_and_test(&sk->sk_refcnt))
        sk_free(sk);
}
```

* 일단 0x200200 을 mapping 해놔서 paging fault 를 막아낸 다음에, 
* `sock_put()` 을 두번 불러서 `sk_free()` 가 불려지도록 하고,
* 그 다음에 거기를 어떻게든 re-fill 해놓고
* close() 콜로 close 관련 function ptr 을 호출하는 
* Use-after-refill-after-free 상황을 만들어내는게 저자의 의도같다.

Use-after-free 를 설명하면,

* 일단 free 를 시켜야한다. allocator 가 잡고 있는 메모리에 뭘 쓰는건 어렵다.
* 근데 비정상적으로 free 시키는 것이 좋다. 정상적으로 free 시키면 진짜 없는 녀석이 되기 때문에, 나중에 사용이 불가능하다.
* 비정상적으로 free 시키고, 그 자리에 어떻게든 다른 object 를 allocate 시켜야한다.
* 다른 object 를 allocate 시킨 다음에, 그 내용을 어느정도 조작할 수 있어야한다.
* 원래 object 의 function ptr 가 위치했던 자리를 조작해낼 수 있으면 대박이다.
* 조작해낸 다음에 원래 object 의 close() 를 부르면 close function ptr 가 호출되고... 이러면 대박이다.

저자들은 여기서 refilling 이 핵심이라고 이야기한다.

#### PoC

CVE-2015-3636 의 PoC. 

``` c PoC of CVE-2015-3636
int sockfd = socket(AF_INET, SOCK_DGRAM, IPPROTO_ICMP);
struct sockaddr addr = { .sa_family = AF_INET };
int ret = connect(sockfd, &addr, sizeof(addr));
struct sockaddr _addr = { .sa_family = AF_UNSPEC };
ret = connect(sockfd, &_addr, sizeof(_addr));
ret = connect(sockfd, &_addr, sizeof(_addr));
```

이렇게 하면 crash 가 날것 같은데.

* 처음에는 AF_INET 으로 해야한다고 한다. 
* Android 디바이스에서만 동작한다고 한다. `/proc/sys/net/ipv4/ping_group_range` 에 PING socket 을 만들 수 있는 그룹이 정의되어 있다고. PC 리눅스와 Android 와는 이 부분이 다르다고.

QEMU 에 android kernel 을 올려서, 저 코드를 실행을 해봐야겠다.

#### Exploitation

Kernel 에서 SLAB/SLUB allocator 를 사용한다고 하는데.

고려해야할 점은.

* Isolated Heap. PartitionAlloc. 이런 메커니즘들.
* Privileged Access Never.
* Linux kernel 의 multi-threading support.

둘간의 차이는 여기에 [SLAB vs SLUB](http://events.linuxfoundation.org/images/stories/pdf/klf2012_kim.pdf)

이 쯤에서 저자들이 이 버그를 어떻게 찾아냈는지 추측해본다.

* Trinity 로 android kernel call fuzzing 을 심하게 걸었는데
* 0x200200 에 write 가 들어가면서 paging fault 가 나는 것을 보고 조사를 해보니,
* PING socket 에 대한 connect 가 두번 불려서 일어난 것이라는 걸 알아내고
* 아니, 시스템 콜 이중호출로 paging fault 가 일어나는 걸 보니, 수상한걸...
* 아니, 이중호출로 object 가 free() 가 되네.
* 이거 use-after-free 아녀?
* free() 한 다음에 refill 이 가능할까?

뭐 이렇게 흘러간것 같다.

* socket structure 를 삐꾸로 free() 하고, 
* memory allocator 를 어찌어찌 이용해서, 그 자리에 뭔가 object 를 alloc 시켜서 refill 을 한 다음에, 
* (socket 이 정상이라고 착각하고 있는 상황에서) socket close() 를 불러서, close_func_ptr 이 호출되게 만드는데, 
* 그 포인터는 exploit 을 향하고 있다.

이 정도가 아닐까?

#### Refill

저자들은 refill 이 가장 어려웠다고 한다. 

* `sendmmsg()` 를 가지고 refill 을 한 모양. 
* `sendmmsg()` 수행중에 `kmalloc` 이 들어가서 파라메터로 넘겨준 데이터 패킷을 저장하기 위한 버퍼를 alloc 한다고. 
* 데이터패킷의 사이즈를 지정할 수 있는데, 512 로 해서, PING socket 과 같은 cache 에 alloc 되도록 한다.

즉 흐름을 정리하면 `(512 bytes) PING SOCK` -> bug free -> `(512 bytes) FREE chunk` -> sendmmsg -> `(512 bytes) TRANSFER BUFFER` 가 되는 셈.
