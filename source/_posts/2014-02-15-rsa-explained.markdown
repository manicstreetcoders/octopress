---
layout: post
title: "RSA Explained"
date: 2014-02-15 19:40
comments: true
categories: 
---

prime p, q 를 정한다.

n = p * q <br>
z = (p-1) * (q-1) <br>
z 보다 작은, z 와 co-prime 인 prime e 를 하나 찾는다. <br>
n,e 가 public key 가 된다. <br>
e * d = 1 (mod z) 인 d 를 정한다. 다시 말하면, e 의 modular inverse 가 private key 가 된다. <br>
정리하면, modulo n 과, 서로 modular inversion 관계인 e,d 가 핵심이다. <br>

Encryption 은 다음과 같다.

Plain-text 를 M 라고 하자. Cipher-text 를 C 라고 한다. <br>
Public key 인 n,e 를 이용해서 암호화한다. M ^ e = C (mod n) <br>

Decryption 은 다음과 같다.

C ^ d = M (mod n)

Public key 인 n,e 를 알경우, private key d 를 recover 하기 위해서는 n 을 p,q 로 factor 해야한다.

숫자를 넣어보면,

p = 3, q = 11 <br>
n = 3 * 11 = 33 <br>
z = 2 * 10 = 20 <br>
e 는 20 과 relative prime 인 11 을 선택. <br>

public key 는 RSA 시스템에서 exponent 라고 부르는 e = 11 과 modulo 라고 부르는 33 으로 구성된다.

이제 private key 를 계산하자.

exponent e 의 modular inversion 이 private key d 가 된다. 

e 는 z 의 원소이고, z 와 relatively prime 이기때문에 modular inversion 이 존재한다.

e*d = 1 (mod z) 니까, 11*d = 1 (mod (3-1)*(11-1)) 니까 d 는 11 이 된다.

정리하면, 모듈로 n = 33 exponent e = 11 private d = 11.
    
Encryption algorithm 은 ( Plaintext ^ e ) mod n. 

Decryption algorithm 은 ( Cipher ^ d ) mod n.

Plaintext M = 3 이라고 해보자.

( 3 ^ 11 ) mod 33 = 8.

11 의 modular inversion (z = 20) 은 11.

( 8 ^ 11 ) mod 33 = 3 이 다시 나왔다.

RSA 크립토를 깬다는 이야기는 n, e 를 알고 있고, z 를 모를때, e 의 modular inversion (z = ???) 을 구하는 문제이다.

공개키 exponent  e 와 모듈로 n 을 알경우에 비밀키 d 를 어떻게 구해야하나?

d 를 알려면 z 를 알아야 한다. 그래야 modular inversion 을 할테니. modular inversion 을 하는 건 extended Euclid 알고리즘으로 빠르게 가능한데.

z 를 recover 하려면, n 을 prime 2 개로 factor 해야하는데... prime-factoring 을 하면 그 조합은 unique 할테니...

정리하면,

    1. n = p * q

    2. 오일러 함수 pi(n) = (p-1)(q-1)

    3. pi(n) 과 co-prime 인 e 를 고른다.

    4. e 의 modular inverse 인 d 를 구한다. 

    5. 즉 d*e = 1 (mod pi(n))

    6. 당연히 M < n 이다. ( n 은 굉장히 큰 prime 의 곱이다 )

    7. C = M ^ e (mod n)

    8. C ^ d = (M ^ e) ^ d  (mod n)
       = M ^ (k*pi(n)+1) (mod n)
       = (M ^ pi(n)) ^ k * M (mod n)
       = M (mod n)

    ( M 이 p 나 q 가 아니기 때문에, M ^ pi(n) (mod n) = 1 이 된다. )

공부할겸, weak 한 RSA algorithm 을 구현하고, plaintext 의 일부를 알고 있을때 (i.e. From: , To: 헤더) brute-force 로 크랙하는 글을 한번 써봐야겠다.

[The RSA Algorithm Explained Using Simple Pencil and Paper Method](http://sergematovic.tripod.com/rsa1.html)
