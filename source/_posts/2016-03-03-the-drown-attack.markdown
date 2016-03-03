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

그럴려면 TLS 핸드쉐이크도 알아야하는데, 이 공격에서 중요한 점은 두가지 차이점.

1. 일단 premaster secret 이란 걸 도입했고, 언제나 항상 48 bytes. 
2. 서버가 `ClientKeyExchange` 를 받으면 `ClientFinished` 를 기다린다는 것

#### DROWN attack template

* TLS 에서 쓰이는 premaster secret 을 공격하려고 하는데.
* premaster secret 는 decrypt 되면 48 byte 가 되는데.
* SSLv2 의 경우, master key 가 PKCS#1 v1.5 포맷으로 decrypt 된는데.
* premaster secret 를 SSLv2 에 태워서 신탁 문의(oracle query)를 해보려면 난점이 있다.
* 48 bytes 라는 길이 차이.
* SSLv2 cipher suites 중에 48 bytes key 를 갖는 녀석이 있으면 좋았을텐데.

Bleichenbacher 공격을 수행하려면, TLS ciphertext 를 SSLv2 key exchange 메시지로 변형시킬 방법이 필요하다. 우리가 가진 oracle 은 SSLv2 에서 쓰이는 symmetric key 를 RSA 로 암호/복호하는 프로세스에서 생기는 녀석이기때문에.

이 한계를 극복하기 위해 다음과 같은 작업을 수행했다고 한다. 흠. 놀랐다. 이런 짓을 하다니. 흠좀무.

0. TLS RSA key exchange 메시지 다수를 캡쳐해야한다.
1. 이제 캡쳐한 암호화된 premaster secret 를 RSA PKCS#1 v1.5 로 인코딩된 ciphertext 로 바꿔야한다.
2. 이 작업을 Bardou et al. 이 발표한 방법으로 수행한다.
3. Bardou et al. 의 도움으로 SSLv2 RSA ciphertext 를 손에 쥐었다고 가정한다.
4. 이제 Bleichenbacher 변종공격으로 해독에 성공한다.
5. 이제 plaintext 를 premaster secret 로 다시 변환한다.

인터넷에는 SSLv2 가 enable 되어있는 서버들이 널려있다.
그리고, SSLv2 가 취약하다는건 누구나 알고 있는 사실이다.
하지만, Modern 클라이언트는 SSLv2 를 사용하지 않는다. 
그러니 인터넷은 안전한 것 아닌가?

아니다. 인터넷은 SSLv2 가 enable 되어있는 서버들이 많기도 하지만,
OpenSSL 버그때문에, 서버 설정에 상관없이, 강제로 SSLv2 로 네고할 수 있다.
생각보다 훨씬많은 SSLv2 oracle 이 인터넷에 널려있는 셈.

천재적인 발상은, TLS 와 SSLv2 가 private key 를 공유하고 있다는 사실에 생각이 미친 것!!!

즉 SSLv2 를 통해서 TLS 를 공격할 수 있는 것이다.

크로스 프로토콜 공격!!!

천재다!

제네럴한 버전과 스페셜한 공격, 두가지가 있다고 한다.

#### General DROWN

SSLv2 + 수출등급 cipher suites 가 먹는 서버를 oracle 로 만드는 방법.

SSLv2 에 oracle 이 생긴 이유는 다음과 같다고.

1. ClientMasterKey 를 ServerVerify 로 바로 응답해주는데, ClientMasterKey 에는 enc_server 의 pk 로 암호화(secret portion of master key) 가 들어있고, ServerVerify 는 RSA decrypt 를 해서, 그 master key 로 암호화된 ciphertext 를 보내준다는 점.
2. 40-bit 수출등급 RC2 or RC4 를 선택할경우, master key 의 11 bytes 는 clear text 로, 그리고 5 bytes 만이 암호화되서 보내진다는 점.
3. anti-Bleichenbacher 카운터메져가 구현된 서버의 경우, decrypt 했는데 invalid padding 이 나오면, 내색을 하지 않고 진행을 하는데.


이를 이용해 valid 한 RSA ciphertext 를 oracle 에게 보냈는지를 추측한다.

1. ClientMasterKey 에, 11 bytes clear key 랑 ciphertext c0 를 보낸다.
2. 서버는 c0 를 RSA decrypt 해서 완벽한 master key 를 만들려고 할 것이다.
3. 올바른 padding format 으로 decrypt 가 잘 될 것인가?
4. 서버는 `server_write_key = MD5( mk || 0 || challenge || 커넥션 ID )` 로 만들어서, challenge 를 암호화해서 ServerVerify 로 돌려줄 것이다.
5. 우리는 master key 의 5 bytes 를 모르는 것이다.
6. master key 에 모르는 5 bytes 만큼, 2^40 만큼 노가다를 해서, server_write_key 를 exhaustive search 로 암호화된 challenge 를 복호화해본다.
7. 어느 한 녀석은 복호화가 성공할 것이다.
8. 이제 5 bytes 을 알게되었다.
9. 이 5 bytes 가 PKCS#1 v1.5 EB 의 맨 뒤의 k 가 되는 셈.
10. 두가지 가능성이, 하나는 ciphertext c0 가 PKCS#1 v1.5 EB 로 제대로 떨어졌을 경우와, 그렇지 않아서 anti-Bleichenbacher 카운터메져로 임의의 key 로 진행되었을 경우.

이제 그 두 경우를 판별해내는 것이 필요하다.

이제, 다시 서버에 연결한다.

1. ClientMasterKey 에 아까와 동일한 c0 를 실어 보낸다.
2. 돌아온 ServerVerify 의 암호화된 challenge 를 `MD5( 아까 찾아낸 master key || 0 || 새로운 challenge || 새로운 커넥션 ID)` 로 decrypt 해본다.
3. 만약 decrypt 가 되어서 challenge 값이 내가 보낸 값이랑 같으면 master key 가 두 세션에 걸쳐서 안정적으로 유지되고 있다는 이야기.
4. 만약 challenge 값이 틀리면, 저번 세션과 이번 세션에 master key 가 다르다는 이야기.
5. 즉 anti-Bleichenbacher 카운터메저가 동작해서 random 으로 master key 가 만들어져서, 프로토콜이 이상없는 듯 가짜로 진행되었다는 것.

이 oracle 이 워킹하는 이유는, 수출등급으로 cipher 를 고정하고, 40 bit 에 대해서 노가다를 하는 것이 가능했기때문.

그럼 이제 SSLv2 를 이용한 RSA oracle 은 만들었으니, TLS 에서 쓰이는 48 bytes premaster secret 을 SSLv2 oracle 이 받아들이는 형태로 변환하는 것이 필요하다.

