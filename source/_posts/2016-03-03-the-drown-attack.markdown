---
layout: post
title: "The DROWN Attack"
date: 2016-03-03 09:38:55 +0900
comments: true
categories: 
---

[The DROWN Attack](https://drownattack.com)

1,000 번의 TLS 핸드쉐이크를 관찰하고, 40,000 번의 SSLv2 연결을 시도해보고, 2^50 번의 symmetric cipher 연산을 하면, 2048-bit RSA TLS 핸드쉐이크를 해독하는 것이 가능하다...고 한다.

배정된 CVE 는 다음과 같다.

* CVE-2016-0800
* CVE-2015-3197 
* CVE-2016-0703
* CVE-2016-0704

#### The PKCS#1 v1.5 padding structure

DROWN 공격은 RSA PKCS# v1.5 padding 구조를 이용함.

N = p*q, l = len(N), e 는 euler_pi(N) 과 coprime 인 정수(아마도 3 이나 65537)라고 할때, k 는 symmetric key 라고 할 때, 암호화를 돌릴 블럭은 다음과 같이 만든다. `m = 00 || 02 || PS || 00 || k`

* 처음 `00` 은 암호화를 돌릴 블럭(EB)가 N 보다 작은 정수가 되게 만들기 위해 집어 넣는 것
* `02` 는 block type
* PS 는 패딩. len(PS) >= 8 이어야 한다. 패딩을 prepend 하는 구조.
* k 는 주로 symmetric key 가 되는데, SSLv2 에서는 master_key 가 k 라고 보면 된다.
* 이 EB 를 정수로 변환한다. 변환은 256 의 index 승을 계속 곱해서 더해나가며 바이트 블럭을 변환하는 식.
* 정수로 변환해서 c = m^e mod N 

#### The SSLv2 handshake protocol

* 클라이언트가 `ClientHello` 를 보냄
* 서버는 `ServerHello` 를 보냄
* c->s, `ClientMasterKey`
* s->c, `ServerVerify`
* c->s, `Finished`
* s->c, `Finished`

그럼 보내는 내용을 정리하면,


* c->s, `ClientHello`: 클라이언트 지원 cipher suites. 그리고 클라이언트 nonce (챌린지)
* s->c, `ServerHello`: 서버 지원 cipher suites. 서버 인증서. 서버 nonce (커넥션 ID)
* c->s, `ClientMasterKey`: 공통의 cipher suites. 그리고 master_key 를 위한 키 데이터.
* s->c, `ServerVerify`: 
* c->s, `Finished`
* s->c, `Finished`

Export 등급 `ClientMasterKey` 단계를 더 정리하면 (40-bit SSL_RC2_128_CBC_EXPORT40_WITH_MD5 가정).

* clear_key_data
* secret_key_data (RSA PKCS#1 v1.5 로 암호화)
* master_key = clear_key_data || secret_key_data
* 40-bit 경우, secret_key_data 는 5 bytes. (5*8 = 40)

Non-export 등급은 master_key 전체가 암호화되어있고, clear_key_data 의 길이는 zero. 즉 Export 등급은 의도적으로 약화시킨 것이라고 보면 된다.

하여간, 이 정보들로 c,s 는 각각 session key 를 계산한다.

* server_write_key = MD5( mk || 0 || challenge || 커넥션 ID )
* client_write_key = MD5( mk || 1 || challenge || 커넥션 ID )

그리고, challenge 를 `server_write_key` 로 암호화해서 `ServerVerify` 메시지로 보낸다.

#### The Drown attack

Drown 공격은 SSLv2 핸드쉐이크의 다음 몇가지를 활용한다.

* secret_key_data 는 PKCS#1 v1.5 로 암호화한다는 사실
* 그리고, 클라이언트가 보내준 secret_key_data 를 자신의 private key (d) 로 복호화해서 master_key 를 만든다음에 그걸로 클라이언트가 보내준 `challenge` 를 암호화해서 다시 클라이언트에게 보내준다는 점. 이런 복호화 서비스를 서버는 항상 해준다는 점. 그것도 바로바로 즉시즉시. 
* secret_key_data 는 수출등급의 경우 단지 5 바이트에 불과하다는 사실

이런 사실때문에 Drown 공격은 chosen-ciphertext attack.

#### OpenSSL SSLv2 cipher suite selection bug

Drown 공격이 OpenSSL 과 만나 심각해지는 포인트.

SSLv2 핸드쉐이크를 진행할때, `ClientMasterKey` 로 공통의 cipher suites 를 보내야하는데,
OpenSSL version <= 1.1.0 의 경우, 서버가 지정하지않은 것도 client 맘대로 보내서 관철시킬 수 있는 버그가 있었다. 서버가 지원하지도 않는 수출등급 cipher 를 강제시킬 수 있는 것이다. CVE-2015-3197.

#### Bleichenbacher's attack
