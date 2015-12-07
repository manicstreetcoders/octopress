---
layout: post
title: "The Vigenere Cipher"
date: 2015-04-14 08:31:54 +0900
comments: true
categories: 
---

책을 보고 정리해본다.

Introduction 의 Historical Ciphers and Their Cryptanalysis 부분.

#### The shift cipher and the sufficient key-space principle

* 가능한 키는 26 개. Plaintext candidates 도 26 개.
* 적어도 key space 가 2^70 정도는 되어야함.

#### The mono-alphabetic substitution cipher

* 이제 Key space 는 모든 bijections 이 된다. 이제 Key space 는 26!
* Key space 는 꽤 커졌지만, 알파벳 출현 빈도를 가지고 분석을 하면, 가장 자주 나타나는 글자가 'e' 에 대응할 것이라 추측할 수 있다.

#### An improved attack on the shift cipher

* p_i 를 영어에서 i 번째 글자의 출현 빈도라고 하자. (i 는 0 부터 25 까지)
* 알파벳 출현 빈도를 제곱해서 summation 하면 약 0.065 가 나온다고 한다.
* 이제 p_i 는 원판, q_j 는 cipher 에서 알파벳의 출현 빈도라고 하고, k 를 shift 라고 하면
* p_i 의 출현 빈도와 q_(j=i+k) 의 출현 빈도는 비슷할 것이다.
* 이제 k 를 1 ~ 25 까지 바꿔가면서, p_i * q_j 의 summation 을 구해본다.
* 0.065 에 근접하게 나오는 k 가 encryption key 일 확률이 높다.

#### The Vigenere cipher

``` c vigenere.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void main() {
    char cipher[256];
    char * key = "cafe";
    char * plain = "tellhimaboutme";
    int len;
    int key_len;
    int i,j;
    char c;

    memset(cipher,256,0);

    len = strlen(plain);
    key_len = strlen(key);

    for (i = 0; i < len; i++) {
        c = plain[i] - 'a';
        c += key [ i % key_len ] - 'a';
        c = c % 26;
        cipher[i] = 'a' + c;
    }

    printf("Cipher: [%s]\n", cipher);
}
```

블럭으로 변환해서 영어 글자의 출현 빈도를 뭉갰다. Key length 를 n 이라고 할 때, C_i 와 C_i+n 이 같은 alphabet 으로 변환되었다는 것에 주목하여, C_i, C_i+n, C_i+2n, C_i+3n, ... 등 같은 녀석들끼리 모아서 분석한다.
