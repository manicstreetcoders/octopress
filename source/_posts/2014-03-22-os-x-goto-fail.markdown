---
layout: post
title: "OS X goto fail"
date: 2014-03-22 08:22:47 +0900
comments: true
categories: 
---

[iPhone and OS X Users Beware, All Your Data is Public (eg. When at your fav Starbucks)](http://www.msuiche.net/2014/02/22/sslverifysignedserverkeyexchange-a-k-a-the-goto-epicfail-bug/)

무조건 SSLVerifySignedServerKeyExchange() 의 도입부에 `goto fail` 이 삽입되어있었다는 이야기. 전체 코드를 보지 않아서 확실히는 모르겠는데, 저 단계에서 SHA1.update 가 실패할 일은 없기때문에 대부분의 경우 err = 0 으로 return 되는 현상이 벌어질 것같다. 즉 sign verify 가 항상 성공한다는 이야기.

    static OSStatus
    SSLVerifySignedServerKeyExchange(SSLContext *ctx, bool isRsa, SSLBuffer signedParams,
        uint8_t *signature, UInt16 signatureLen)
    {
        OSStatus    err;
        ...
        if ((err = ReadyHash(&SSLHashSHA1, &hashCtx)) != 0)
            goto fail;
        if ((err = SSLHashSHA1.update(&hashCtx, &clientRandom)) != 0)
            goto fail;
        if ((err = SSLHashSHA1.update(&hashCtx, &serverRandom)) != 0)
            goto fail;
        if ((err = SSLHashSHA1.update(&hashCtx, &signedParams)) != 0)
            goto fail;
            goto fail;
        if ((err = SSLHashSHA1.final(&hashCtx, &hashOut)) != 0)
            goto fail;
        ...

    }

