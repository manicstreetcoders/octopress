---
layout: post
title: "Atomic XCHG"
date: 2014-12-13 08:08:28 +0900
comments: true
categories: 
---

`lock xchg` 인스트럭션을 이용해서, 유저스페이스에서 mutex 를 만들어보았다.

#### Atomic XCHG

``` c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <unistd.h>

pthread_t t1,t2;

int babo;
int tmp;
int lock = 0;

int xchg(int *m,int inval)
{
    register int val = inval;
    __asm__ __volatile__ ( "lock xchg %1,%0" : "=m" (*m), "=r" (val) : "1" (inval) );
    return val;
}

static void *thread_p(void *arg)
{
    int loops = 100000;
    int i;
    struct timespec time1,time2;
    time1.tv_sec = 0;
    time1.tv_nsec = 1000000;

    for (i=0;i<loops;i++) {
        while (xchg(&lock,1) == 1) 
            nanosleep(&time1,&time2);
        
        tmp = babo;
        tmp++;
        babo = tmp;

        lock = 0;
    }
    return NULL;
}

int main(int ac,char **av)
{
    pthread_create(&t1,NULL,thread_p,NULL);
    pthread_create(&t2,NULL,thread_p,NULL);
    pthread_join(t1,NULL);
    pthread_join(t2,NULL);
    printf("babo = %d\n", babo);
}
}
```

컴파일해서 실행해보았다.

``` sh
$ gcc atomic.c -o atomic -lpthread
$ ./atomic
babo = 200000
$
```

`while (xchg(&lock,1) == 1) nanosleep(&time1,&time2);` 으로 lock 을 acquire 하고, `lock = 0;` 으로 lock 을 free 한다. `futex_lock_pi()` 도 비슷하게, user space 에서 lock 을 해제할 수 있었던 버그가 이제는 패치되었다.

mutex 가 걸린 것처럼 동작했다. 100000 + 100000 = 200000

#### 참고

* [Atomic operations on the x86 processors](http://www.mohawksoft.org/?q=node/78)
