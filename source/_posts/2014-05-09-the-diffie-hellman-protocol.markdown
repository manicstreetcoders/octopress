---
layout: post
title: "The Diffie Hellman protocol"
date: 2014-05-09 19:54:33 +0900
comments: true
categories: 
---

큰 소수 p 를 정한다. (e.g. 600 digits -> approx. 2000 bits) 

{1,...,p} 중에서 정수 하나를 고른다. g.

1. Alice : choose random a in {1,...,p-1}
2. send A = g^a (mod p) to Bob
3. Bob: choose random b in {1,...,p-1}
4. send B = g^b (mod p) to Alice
5. Shared key = g^(ab) (mod p)

Eavesdropper sees: p,g, A = g^a (mod p), and B = g^b (mod p)

저 정보를 가지고 g^(ab) (mod p) 를 계산할 수 있을까?

Physical 한 예를 들어본다. 

e.g. 열쇠를 공유하지 않고 상자의 내용물을 안전하게 지키면서 deliver 하고 싶다.

1. Alice 는 상자에 자신의 열쇠로 잠궈서 Bob 에게 보낸다.
2. 상자를 받은 Bob 역시 자신의 열쇠로 잠궈서 Alice 에서 다시 보낸다.
3. Alice 는 상자를 잠근 두 개의 Lock 중에서 자신이 잠근 Lock 을 자신의 열쇠로 풀어서, 다시 Bob 에게 보낸다.
4. Bob 은 상자를 잠근 자신의 Lock 을 푼다.

중간에 상자를 가로채더라도 Eavesdropper 는 상자의 Lock 을 해제할 수 없다.

수학적으로 열쇠를 채우는 연산은 exp + mod prime 연산인데, 이게 왜 계산이 어려운지 설명해보겠다.

예를 들어 32478543^743921429837645 의 remainder 를 찾는 문제를 생각해보자.
