---
layout: post
title: "RC4 weakness"
date: 2016-08-24 10:02:02 +0900
comments: true
categories: 
---

256 바이트의 퍼뮤테이션 테이블을 가지고 움직인다.

* key 를 장착하는 단계. 
* 그리고 p-table 을 바꿔가면서 keystream 을 만들어 XOR 하는 암복호화하는 단계.

RC4 공격 역사를 정리해보면, state 테이블에 key 를 장착하고 난 이후 내용이 어느정도 예측가능하다는 포스팅이 sci.crypt 에 올라온게 처음인 것 같다.

state 의 첫 몇 바이트는 `state[n] == sum_0_to_nth_bytes(key) + n*(n+1)/2` 일 가능성이 37% 에 달한다고 Andrew Roos 가 포스팅함.

그 다음에 Paul, Rathi, Maitra 가 논문을 publish.

``` c rc4.c
#include <sys/types.h>
#include <stdio.h>
#include <string.h>

struct rc4_state {
    u_char perm[256];
    u_char x;
    u_char y;
};

static __inline void swap_bytes(u_char *a, u_char *b)
{
    u_char temp;
    temp = *a;
    *a = *b;
    *b = temp;
}

void rc4_init(struct rc4_state *const state,
                const u_char *key,
                int keylen)
{
    int x;
    state->x = 0;
    state->y = 0;
    for (x=0;x<256;x++) state->perm[x] = (u_char)x;

    u_char k = 0; 
    u_char y = 0;

    for (x=0;x<256;
            x++, 
            k = (k+1) % keylen) {
	y += state->perm[x] + key[k];
        swap_bytes(&state->perm[x], &state->perm[y]);
    }
}

void
rc4_crypt(struct rc4_state *const state,
            const u_char *inbuf,
            u_char *outbuf,
            int buflen)
{
    int i;
    u_char j;

    for (i=0;i<buflen;i++) {
        state->x++;
        state->y += state->perm[state->x];

        swap_bytes(&state->perm[state->x], &state->perm[state->y]);
        j = state->perm[state->x] + state->perm[state->y];
        outbuf[i] = inbuf[i] ^ state->perm[j];
    }
}
int main()
{
    struct rc4_state st;
    u_char inbuf[256] = "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a";
    u_char outbuf[256];
    int len = 10;

    rc4_init(&st, (const u_char*)"\x00\x00\x00\x00\x00\x00", 6);
    rc4_crypt(&st, inbuf, outbuf, len);
    do {
        int i;
        for (i=0;i<len;i++)
            printf("%02X ", inbuf[i]);
        printf("\n");
    } while(0);
    do {
        int i;
        for (i=0;i<len;i++)
            printf("%02X ", outbuf[i]);
        printf("\n");
    } while(0);

    rc4_init(&st, (const u_char*)"\x00\x00\x00\x00\x00\x00", 6);
    memset(inbuf, 0, 256);
    rc4_crypt(&st, outbuf, inbuf, len);
    do {
        int i;
        for (i=0;i<len;i++)
            printf("%02X ", inbuf[i]);
        printf("\n");
    } while(0);
    return 0;
}
```
