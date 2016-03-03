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
* PS 는 패딩. len(PS) >= 8 이어야 한다. 패딩을 prepend 하는 구조. EB 와 modulo N 이 같은 byte length 를 갖도록 맞춰주는 녀석이라고 보면 된다.
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

Bleichenbacher 의 공격은 padding oracle attack. RSA ciphertext 는 PKCS#1 v1.5 padding format 을 준수하는 plaintext 로 복호화되어야한다는 사실을 활용한다.

그럼 PKCS#1 v1.5 padding format 을 준수한다는 것의 의미는,

* `00||02||PS||00||k` 형태
* |PS| >= 8
* m[3]~m[10] 에는 0x00 이 없어야 함 (적어도 8 바이트)

그럼, Bleichenbacher 알고리즘은 

* 공격자는 valid 한 PKCS#1 v1.5 ciphertext 를 가지고 있음 (이걸 시작 포인트로 삼음)
* 공격자는 private key 는 접근 못함
* 그러나 oracle 을 가지고 있다고 가정한다.
* oracle 은 1 or 0 을 리턴하는데,
* m = c^d mod N 의 값이 0x00 02 로 시작하면 1
* 아니면 0 을 돌려준다.

이런 oracle 을 가지고 있다고 가정할 경우.

* oracle 이 1 을 돌려주면, plaintext m 이 어떤 특정 레인지 안쪽에 있다고 생각할 수 있다.
* 그 레인지는 0x00020000.... 보다는 크거나 같고, 0x0002FFFF.... 보다는 작거나 같고.
* 식으로 표시하면 2B <= m <= 3B -1, where B = 2^(8*(len-2))

그 다음 스텝은 RSA 연산의 변형성을 활용하는 것.

* d 는 몰라도 e 는 알고 있으니, 
* `c_0 = m_0 ^ e mod N` 이라고 하면 양쪽에 s^e 를 곱하면
* `c = (c_0 * s^e) mod N = (m_0 * s)^e mod N` 이 되니까, m_0 에 s 를 곱한 값을 encrypt 시킨 셈이 된다.
* 이 encrypted 된 값을 가지고 oracle 에 물어본다.

* oracle 이 0 이라고 대답하면, s 를 증가시켜서 다시 물어본다. 
* 이런 식으로 특정 규칙을 가지고 만들어진 가라 ciphertext 를 가지고 oracle 에 계속 확인을하다가 1 이 걸리면
* `c_0 * s^e` 가 decrypt 되었는데, 0x0002... 보다 크거나 같고, 0x0003... 보다 작은 정수로 decrypt 되었다는 것을 알게된다.
* 그 plaintext 는 m_0 * s 와 어떤 연관성을 가질 것이다.
* 그 연관성은 어떻게 될까?
* modulo 연산이니까  m_0 * s 값과 N 의 배수만큼의 차이가 있을 것
* 즉 range 가 2B ~ 3B 사이가 아니라 2B + rN ~ 3B + rN 만큼 보정이 들어간다는 것.
* s,r 을 try 해서 레인지를 좁혀나가면 m_0 를 구할 수 있다고 한다. (Bleichenbacher 에 따르면)

#### The SSLv2 oracle

그렇다면, 이 연구자들이 OpenSSL SSLv2 code 에서 찾아낸 oracle 은 무었일까?

일단 general 한 oracle 이 있고 special 한 oracle 이 있다. 

* 일단 공격 target 은 TLS 1.0 ~ 1.2 그리고, TLS_RSA ciphersuites 를 사용한다고 가정.
* 그리고, 같은 RSA public key 가 TLS / SSLv2 양쪽에서 관찰되고 있다고 가정.
* passive monitoring 만하고, active 공격은 안한다고 가정. 즉, MitM 상황이 아님.
* 서버에 최대한 가까운 네트웍에서 1,000 개 정도의 incoming 커넥션을 캡쳐할 수 있다고 가정.

그리고, SSLv2 oracle 에 대한 가정은 다음과 같다. 

* PKCS#1 v1.5 padding format 을 준수하는지 알려주는 oracle 이 노출하는 정보량은 다음과 같다.
* 처음이 00 || 02 로 시작하고
* non-zero padding 이 있고
* 마지막이 00 || k 라는 것
* SSL_RSA_EXPORT_WITH_RC2_CBC_40_MD5 의 경우 secret message k 는 5 bytes.
* SSL_DES_192_EDE3_CBC_WITH_MD5 의 경우 k 는 24 bytes.
* 각각의 경우 +3 을 하면 8, 27 bytes 에 대한 정보(!!!)가 노출됨.

이제 SSLv2 oracle 을 가지고 TLS 를 해독하는 것이 이 연구자들의 시나리오다.