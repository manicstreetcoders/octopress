---
layout: post
title: "Fermat's theorem"
date: 2014-03-22 20:54:02 +0900
comments: true
categories: 
---

소수인 p 가 있을때,

{ 1,2,3,...,p-1 } 의 숫자들 중에서 하나 골라서 x 라고 하면.

x^(p-1) = 1 in Z(p)

을 만족한다.

i.e. 

p = 5 라고 하면, p 의 residue class 는 { 0, 1, 2, 3, 4 } 가 된다.

그 중에서 modular inverse 가 있는 수는 { 1, 2, 3, 4 } 가 된다.

    1*1 = 1 in Z(5)

    2*3 = 1 in Z(5)

    3*2 = 1 in Z(5)

    4*4 = 1 in Z(5)

이기 때문.

페르마의 Theorem 대로, 이 modular inverse 가 있는 수들의 (p-1) 제곱을 한 수들은, 나머지가 1 이 된다.

    1^(5-1) = 1 in Z(5)

    2^(5-1) = 1 in Z(5)

    3^(5-1) = 1 in Z(5)

    4^(5-1) = 1 in Z(5)

정리하면, Z(p) 에서 p 가 소수이면, { 1,2,...,p-1 } 까지가 모두 모듈러 인버스를 가지게 되고,
그 집합의 사이즈는 p-1, x^(p-1) 의 residue class 는 1.
