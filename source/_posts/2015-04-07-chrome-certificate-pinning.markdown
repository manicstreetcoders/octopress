---
layout: post
title: "Chrome Certificate Pinning"
date: 2015-04-07 15:44:42 +0900
comments: true
categories: 
---

구글 chrome 이 도입한 certificate pinning.

* Certificate chain 에 pinning 된 퍼블릭키가 존재해야 한다.
* Google 도메인의 경우, Verisign, Google Internet Authority, Equifax, GeoTrust 의 퍼블릭키가 whitelisting 되어 있다.
* 동일한 퍼블릭키로 여러개의 certificate 이 발급될 수 있기때문에, certificate hash 를 pinning 에 사용하지 않는다. 
* SubjectPublicKeyInfo 를 해슁한다. public key bit string 을 해쉬하는 것이 아니라. RSA key 인지 DSA key 인지등의 정보를 포함하는 구조체를 해슁하는 것.
