---
layout: post
title: "Logjam: Diffie-Hellman Falls Down"
date: 2015-06-05 18:30:24 +0900
comments: true
categories: 
---

* [Imperfect Forward Secrecy: How Diffie-Hellman Fails in Practice](https://weakdh.org/imperfect-forward-secrecy.pdf)
* [Attack of the week: Logjam](http://blog.cryptographyengineering.com/2015/05/attack-of-week-logjam.html)

### Matthew Green Blog 요약

* 알려진 가장 좋은 방법은 g^a 를 캡쳐해서, log(g^a) = a 인 이산로그 문제를 푸는 것.
* p 가 2048 bits 정도이면 수학적으로 풀기 어렵다.
* p 가 512 bits 라면?

* 서버가 DHE-EXPORT 를 지원한다면, 공격자가 negotiation message 를 edit 해서, client 가 지원하는 cipher list 를 DHE-EXPORT 만 지원한다고 바꿔놓는다.
* 이제 서버는 client 가 DHE-EXPORT 만 지원한다고 믿게 된다.
* 이제 서버는 512-bit export-grade Diffie-Hellman tuple 을 보낸다.
* 클라이언트는 512-bit DH tuple 을 믿어버린다. (클라이언트는 DHE-EXPORT 컨텍스트에서 negotiation 되고 있다는 것을 모른다)
* 허잡한 파라메터가 오는데도 클라이언트는 모른다.
* Negotiation 마지막에 Finished 메시지에 지금까지 교환되었던 negotiation message 의 MAC 이 교환되는데, 이것때문에 뽀록난다.
* DH key 를 깨야한다. Finished 메시지가 교환되기전에.
* 깰 수 있다면, forged Finished 메시지를 만들 수 있다.

### Can you really solve a discrete logarithm before a TLS handshake times out?

Number Field Sieve 를 사용해야하는데, 이걸 optimize 할 수 있다.

* Pre-computation (for a given prime p): Polynomial selection, sieving, linear algebra 로 구성된 작업. p 에만 dependant.
* Solving to find a (for a given g^a mod p): precomputation 에서 만들어진 table 을 사용한다. g, g^a 를 가지고 table 을 사용하여 a 를 찾는 알고리즘.

Pre-computation 은 p 에 dependant 한데. 그렇다면 실제 TLS 에서 p 는 어떻게 generate 되는 걸까?

Large-scale 로 진행된 internet scan 에서 Apache/mod_ssl 의 92% 는 DH-EXPORT 의 경우, 2 개의 소수만 사용했다고 한다. 이 경우 2 개의 p 에 대해서만 테이블을 만들면 된다.
