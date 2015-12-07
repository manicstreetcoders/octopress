---
layout: post
title: "HMAC, SHA1, HTTP Authorization and, OAuth"
date: 2012-12-18 10:05
comments: true
categories: 
---

* [Hash-based message authentication code](http://en.wikipedia.org/wiki/Hash-based_message_authentication_code)
* [Dr. Dobb's Journal: The HMAC Algorithm](http://www.drdobbs.com/security/the-hmac-algorithm/184410908)
* [SHA-1](http://en.wikipedia.org/wiki/SHA-1)

HMAC: Hash-based message authentication code

* Data integrity 
* message sender 에 대한 authenticity 를 동시에 검증할 수 있다.
* 핵심은 shared secret key 가 있다는 것

MD5 나 SHA1 등의 cryptographic hash function 을 사용할 수 있다.
HMAC-MD5 또는 HMAC-SHA1 등의 함수를 조합해낼 수 있다.

HMAC-SHA1 알고리즘의 콤포넌트들을 설명하면,

key 가 64 bytes (== 512 bits) 보다 크면 SHA1 을 돌려서 digest 해서 줄인다.

SHA1 digest size 는 160 bit 니까, 이제 key size 는 무조건 64 bytes 보다 작다.

그 다음에 뒤를 0 으로 패딩한다.

그 다음에 이 (패딩된) key 를 가지고 inner pad, outer pad 를 만든다.

* inner pad = 64 bytes 각각의 byte 와 0x36 간에 XOR.
* outer pad = 64 bytes 각각의 byte 와 0x5c 간에 XOR.

이제 inner pad + message 를 붙여서, SHA1 을 돌리고 나온 digest 를 가지고.

outer pad + digest 를 붙여서 또 SHA1 을 돌린다.

그럼 나온 최종 digest 160 bits. 이게 HMAC-SHA1 의 결과값이다.

Dr. Dobb's Journal 에 나온 hmac_md5()

    /* Function: hmac_md5 */
    void
    hmac_md5(text, text_len, key, key_len, digest)
    unsigned char* text; /* pointer to data stream */
    int text_len; /* length of data stream */
    unsigned char* key; /* pointer to authentication key */
    int key_len; /* length of authentication key */
    caddr_t digest; /* caller digest to be filled in */
    {
         MD5_CTX context;
         unsigned char k_ipad[65]; /* inner padding - key XORd with ipad */
         unsigned char k_opad[65]; /* outer padding - key XORd with opad */
         unsigned char tk[16];
         int i;
         /* if key is longer than 64 bytes reset it to key=MD5(key) */
         if (key_len > 64) {
             MD5_CTX      tctx;
             MD5Init(&tctx);
             MD5Update(&tctx, key, key_len);
             MD5Final(tk, &tctx);
        
             key = tk;
             key_len = 16;
         }
         /* the HMAC_MD5 transform looks like:
          * MD5(K XOR opad, MD5(K XOR ipad, text))
          * where K is an n byte key
          * ipad is the byte 0x36 repeated 64 times
          * opad is the byte 0x5c repeated 64 times
          * and text is the data being protected
          */
         /* start out by storing key in pads */
         bzero( k_ipad, sizeof k_ipad);
         bzero( k_opad, sizeof k_opad);
         bcopy( key, k_ipad, key_len);
         bcopy( key, k_opad, key_len);
   
         /* XOR key with ipad and opad values */
         for (i=0; i<64; i++) {
                 k_ipad[i] ^= 0x36;
                 k_opad[i] ^= 0x5c;
         }
         /* perform inner MD5 */
         MD5Init(&context);               /* init context for 1st pass */
         MD5Update(&context, k_ipad, 64)      /* start with inner pad */
         MD5Update(&context, text, text_len); /* then text of datagram */
         MD5Final(digest, &context);          /* finish up 1st pass */
         /* perform outer MD5 */
         MD5Init(&context);               /* init context for 2nd pass */
         MD5Update(&context, k_opad, 64); /* start with outer pad */
         MD5Update(&context, digest, 16); /* then results of 1st hash */
         MD5Final(digest, &context);      /* finish up 2nd pass */
    }
    
    key =         0x0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b0b
    key_len =     16 bytes
    data =        "Hi There"
    data_len =    8  bytes
    digest =      0x9294727a3638bb1c13f48ef8158bfc9d
    
    key =         "Jefe"
    data =        "what do ya want for nothing?"
    data_len =    28 bytes
    digest =      0x750c783e6ab0b503eaa86e310a5db738
    key =         0xAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    key_len       16 bytes
    data =        0xDDDDDDDDDDDDDDDDDDDD...
                  ..DDDDDDDDDDDDDDDDDDDD...
                  ..DDDDDDDDDDDDDDDDDDDD...
                  ..DDDDDDDDDDDDDDDDDDDD...
                  ..DDDDDDDDDDDDDDDDDDDD
    data_len =    50 bytes
    digest =      0x56be34521d144c88dbb8c733f0e8b3f6
    
