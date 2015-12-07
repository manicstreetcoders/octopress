---
layout: post
title: "Setting up L2TP VPN"
date: 2015-08-05 16:25:16 +0900
comments: true
categories: 
---

* [IPsec L2TP VPN Auto Install Script for Ubuntu 14.04 & 12.04 and Debian 8 & 7](https://gist.github.com/hwdsl2/9030462)

* [PPTP VPN 서버](https://wordpress.update.sh/archives/16)
* [How to setup Poptop (pptpd) VPN server on Linux](http://thuannvn.blogspot.kr/2009/03/how-to-setup-poptop-pptpd-vpn-server-on.html)
* [pptpd vpn 서버접속후 특정 web 사이트가 안여리는 증상 도움 부탁 드립니다.](https://kldp.org/node/73417)
* [CentOS에서 PPTP 및 L2TP/IPSec 프로토콜을 이용한 VPN 서버 구축하기](http://guni.tistory.com/326)
* [Intercepting and decrypting SSL communications between Android phone and 3rd party server](http://www.myhowto.org/java/81-intercepting-and-decrypting-ssl-communications-between-android-phone-and-3rd-party-server/)
* [What's an easy way to perform a man-in-the-middle attack on SSL?](http://security.stackexchange.com/questions/33374/whats-an-easy-way-to-perform-a-man-in-the-middle-attack-on-ssl)

#### Forging Certificates (by blukat29)

```
$ openssl genrsa -des3 -out ca.key 4096
$ openssl req -new -x509 -days 365 -key ca.key -out ca.crt
$ openssl genrsa -des3 -out server.key 4096
$ openssl req -new -key server.key -out server.csr
$ openssl x509 -req -days 365 -in server.csr -CA ca.crt -CAkey ca.key -set_serial 01 -out server.crt
```

#### socat

socat 옵션.

```
root@zomo:~# socat -v OPENSSL-LISTEN:4443,reuseaddr,verify=0,cert=./server.crt,key=./server.kecafile=./ca.crt,debug,fork OPENSSL:api.xxx.com:443,verify=0
```

nat 쪽 iptable

```
root@zomo:~# ip -t nat -L
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination
REDIRECT   tcp  --  anywhere             anywhere             tcp dpt:https redir ports 4443

Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination

Chain POSTROUTING (policy ACCEPT)
target     prot opt source               destination
MASQUERADE  all  --  anywhere             anywhere
```

iptable

```
root@zomo:~# ip -L
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
DROP       all  --  43.0.0.0/8           anywhere

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```
