---
layout: post
title: "SSLv3 POODLE Attack"
date: 2014-10-18 09:20:32 +0900
comments: true
categories: 
---

* Adam Langley 의 [POODLE Attacks on SSLv3](https://www.imperialviolet.org/2014/10/14/poodle.html)
* Matthew Green 의 [Attack of the week: POODLE](http://blog.cryptographyengineering.com/2014/10/attack-of-week-poodle.html)
* Bodo Möller, Thai Duong, Krzysztof Kotowicz 의 [This POODLE Bites: Exploiting The SSL 3.0 Fallback](https://www.openssl.org/~bodo/ssl-poodle.pdf)

을 읽고 정리해본다.

#### Objective

* 공격의 목표는 HTTP header 의 cookie 를 읽어서, facebook.com 에 로그인할 수 있는 세션아이디를 알아내는 것이라고 하자.

#### Assumptions

* Man-in-the-middle 포지션을 가지고 있어야 한다.
* Victim 브라우저에 JavaScript 을 돌릴 수 있어야 한다.
* Fallback 을 시키든 어떻게든 SSLv3 로 다운그레이딩시킬 수 있다고 가정한다. 그리고 block cipher 는 CBC 모드를 쓴다고 가정.
* facebook.com 은 그냥 예일뿐이다. 이 버그를 가지고 있는지 없는지 모른다.

#### Steps

1. 유저가 이상한 링크를 클릭하거나, 아니면 Man-in-the-middle 포지션을 활용해서 강제로 HTTP redirect 를 보내서

2. www.hacker.com 으로 redirect 시킨다.

3. JavaScript 를 내려보내서 브라우저를 뺑뺑이 돌리는데, facebook.com 을 향해 수천 번의 https open request 를 감행하게 된다.

4. `GET /a HTTP/1.1 Cookie: ABCDE This_is_content MAC PADDING` 이 plaintext 가 될텐데, 이런 request 를 평균 256번 돌려서 cookie 1 글자를 recover 한다.

5. SSLv3 의 Block cipher 로 암호화 블럭을 만들었을때, 마지막이 padding 만으로 이루어지도록 plaintext 의 length 를 조절한다. i.e. 128 bit block cipher 를 쓸 경우, 16 byte 로 떨어지면 null padding 이 붙게 된다. 즉 padding block 이 '00 00 00 00 00 00 ... 00 15' 되게 plaintext 의 length 를 조절한다. 이로서 ciphertext 의 마지막 바이트를 15 로 통제할 수 있다.

6. Man-in-the-middle 포지션을 활용해서, 통과하는 암호 스트림을 관찰하다가 마지막 padding 블럭을 조작한다. 어떻게 조작하냐면, 암호화된 cookie block 으로 overwrite 한다. 자기 아이디가지고 facebook.com 에 로그인해서, proxy 를 걸고, ciphertext 와 plaintext 를 비교해보면, cookie block 의 offset 정도는 알 수 있을 것이다.

7. 서버가 decrypt 에 성공하는지 관찰한다.

8. SSLv3 가 제대로 구현되었다면, 무조건 decrypt 에러를 내는게 맞다. 하지만 (대부분의) SSLv3 구현은 padding length 만 사용하지, padding 값이 무엇인지는 상관안한다고 한다. 여기서 문제가 시작되는 것이다.

9. 서버가 SSL 에러를 안내면, 조작한 마지막 블럭의 마지막 바이트가 15 로 decrypt 되어 SSL error 가 나지 않은 경우이니, 마지막 바이트를 decrypt (and xor) 한 것이 15 라는 것을 알게되고.

10. (Decrypted) Padding size byte > 15 면 SSL_PADDING_LENGTH_ERROR (or whatever) 가 날 것이고, padding size byte < 15 면 padding 앞의 MAC 이 뒤로 밀리면서 SSL_MAC_ERROR (or whatever) 가 날 것이다.

11. 만약 SSL 에러가 나면, JavaScript 를 이용해서, 다시 https request 를 돌리고, SSL symmetric cipher 의 키는 random 으로 만들어지니, 언젠가는 15 와 맞아떨어지겠지 하면서, 서버가 decrypt 에 성공할때까지 돌린다.

12. 만약 서버가 decrypt 에 성공하면, CBC 모드의 특성상, decrypt(cookie 의 마지막 바이트) xor (끝에서 두번째 블럭의 ciphertext) == 15 란 이야기고, `끝에서 두번째 블럭의 ciphertext`는 네트워크에서 관찰하여 알 수 있으니, cookie 의 마지막 바이트는 recover 가능하다.

13. `GET /a HTTP/1.1 Cookie: ABCDE This_is_content MAC PADDING` -> `GET /ab HTTP/1.1 Cookie: ABCDE This_is_conten MAC PADDING` 와 같이 URI 는 `/ab` 로 1 글자 늘이고, HTTP content 부분은 1 글자 줄여서 Cookie: ABCDE 를 1 byte 오른쪽으로 shift 한다. 이제 `E` 는 recover 했고, block boundary 에 `...ABCD` 가 걸려서 D 가 끝이라고 가정하자. 전체 length 는 변하지 않았기 때문에, 원래가 마지막이 padding only 였다면, 이렇게 shift 한 것도 padding only 가 된다.

14. 이제 4. 로 돌아가서 다시 한글자를 브포한다.

1 글자 recover 하는데 256 번 브포를 한다고 치면, cookie length * 256 번의 https negotiation 이 필요한 작업.

#### Moral of the story

SSLv3 가 padding length 만 체크하지, padding 의 내용을 확인하지 않아서 생기는 문제.

Router 등을 장악해서 Man-in-the-middle 포지션을 확보했으면, 다른 더 좋은 공격 방법도 많을 것이라서, 굳이... 이렇게까지... 라는 생각은 든다.
