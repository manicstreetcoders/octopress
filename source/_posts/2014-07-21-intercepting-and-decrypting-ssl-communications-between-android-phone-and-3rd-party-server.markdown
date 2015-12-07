---
layout: post
title: "Intercepting and decrypting SSL communications between Android phone and 3rd party server"
date: 2014-07-21 12:39:58 +0900
comments: true
categories: 
---

안드로이드 HTTPS 트래픽 디버깅 하는 방법은 [Intercepting and decrypting SSL communications between Android phone and 3rd party server](http://www.myhowto.org/java/81-intercepting-and-decrypting-ssl-communications-between-android-phone-and-3rd-party-server/) 을 참조한다.

안드로이드에 임의의 CA certificate 을 system 으로 인스톨하는 방법은 [Installing CAcert certificates on Android as 'system' credentials without lockscreen - instructions](http://wiki.pcprobleemloos.nl/android/cacert) 을 참조한다.

DigitalOcean 에서 [How To Setup Your Own VPN With PPTP](https://www.digitalocean.com/community/tutorials/how-to-setup-your-own-vpn-with-pptp) 를 참조해 VPN 서버를 설치한다.

전체적으로는 [blukat29/PPTP VPN Server](https://github.com/blukat29/blukat-config/issues/4) 를 참조한다.

    $ echo 1 > /proc/sys/net/ipv4/ip_forward
    $ iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    $ iptables -t nat -A PREROUTING -i ppp0 -p tcp --dport 443 -j REDIRECT --to-port 4443

    # Generate CA key
    $ openssl genrsa -des3 -out ca.key 4096

    # Generate CA cert
    $ openssl req -new -x509 -days 365 -key ca.key -out ca.crt

    # Generate server key
    $ openssl genrsa -des3 -out server.key 4096

    # Generate server cert signing request
    $ openssl req -new -key server.key -out server.csr

    # Sign server's cert using CA's key
    $ openssl x509 -req -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt

    # Get the hash of the ca.crt certificate:
    $ openssl x509 -inform PEM -subject_hash_old -in ca.crt | head -1  
    5ed36f99
    $ mount -o remount,rw /system
    $ cp /data/local/tmp/5ed36f99.0 /system/etc/security/cacerts/
    $ chmod 644 /system/etc/security/cacerts/*

    $ cat go.sh
    socat -v OPENSSL-LISTEN:4443,reuseaddr,verify=0,cert=./server.crt,key=./server.key,cafile=./ca.crt,debug,fork EXEC:./babo.sh
    $ cat babo.sh
    openssl s_client -CAfile /etc/ssl/certs/ca-certificates.crt -quiet -connect api.babo.com:443
    $ go.sh

    ext% /usr/libexec/java_home
    /Library/Java/JavaVirtualMachines/jdk1.7.0_25.jdk/Contents/Home

* OpenSSL
* SoCat (http://www.dest-unreach.org/socat)
* Bouncy Castle library (http://bouncycastle.org/download/bcprov-jdk16-141.jar) - place this JAR file into your .../jre/lib/ext directory
* Charles Proxy
* [MITMProxy](http://mitmproxy.org)
* `emulator -avd [name of the avd] -http-proxy 127.0.0.1:8080`
