---
layout: post
title: "Modular Inversion"
date: 2014-03-22 13:48:16 +0900
comments: true
categories: 
---

모든 x 가 modular inversion 를 가지고 있는 것은 아니다.
그렇다면 어떤 x 만 modular inversion 을 가지게 되나?

Lemma: x in Z(N) has an inverse if and only if gcd(x,N) = 1

Proof:

1) gcd(x,N) = 1 일 경우

There exist a,b such that a*x+b*N = 1 

등식은 Z(N) 으로 reduce 해도 성립한다. 

b*N = 0 in Z(N) 이니까,

a*x = 1 in Z(N)

따라서 x 의 modular inversion 은 a.

2) gcd(x,N) > 1 일 경우

x 의 modular inversion 이 어떤 a 라고 가정하면,

gcd(a*x,N) > 1 이게 된다. ( gcd(x,N) > 1 이니까 )

예를 들어, gcd(x,N) = 2 라고 하면, 

x 와 a*x 모두 2 의 배수, 짝수가 된다.

N 도 짝수이기때문에, a*x = b*N + 1 일 수가 없다. 짝수 = 짝수 + 1 ???

a*x = 1 in z(N) 이 아니게 된다. 그러므로 저런 a 는 있을 수 없다.

예를 들어, gcd(x,N) = 3 이라고 하면,

x 와 a*x 모두 3 의 배수.

N 도 3 의 배수. a*x = b*N + 1 일수가 있을까? 

3 의 배수에 1 을 더해서 3 의 배수가 만들어질 수 있을까?
