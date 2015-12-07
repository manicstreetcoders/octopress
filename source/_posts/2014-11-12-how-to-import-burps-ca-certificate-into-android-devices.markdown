---
layout: post
title: "How to import Burp's CA Certificate into Android devices"
date: 2014-11-12 16:51:43 +0900
comments: true
categories: 
---

#### Burp

Burp 를 띄우고,

Proxy -> Options -> CA Certificate -> Certificate in DER format -> Next -> Select File -> burp.der -> Save -> Next -> Close

이렇게 해서 PortSwigger CA 인증서를 export 한다.

이제

DER 인코딩을 PEM 으로 바꾼 인증서를 하나 만든다.

$ openssl x509 -inform DER -in burp.der -out burp.crt

이제 hash 를 얻는다.

$ openssl x509 -inform PEM -subject_hash_old -in burp.crt | head -1
9a5ba575

이제 안드로이드 포맷에 맞게 맞춰준다.

    $ cat burp.crt > 9a5ba575.0
    $ openssl x509 -inform PEM -text -in burp.crt -out /dev/null >> 9a5ba575.0

안드로이드로 옮긴다.

    $ adb push 9a5ba575.0 /sdcard/
    $ adb shell
    # mount -o remount,rw /system
    # cp /sdcard/9a5ba575.0 /system/etc/security/cacerts/
    # chmod 644 /system/etc/security/cacerts/9a5ba575.0

#### 참고

* [Installing CAcert certificates on Android as 'system' credentials without lockscreen](http://wiki.pcprobleemloos.nl/android/cacert)
