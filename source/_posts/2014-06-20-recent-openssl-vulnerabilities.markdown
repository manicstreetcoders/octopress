---
layout: post
title: "Recent OpenSSL Vulnerabilities"
date: 2014-06-20 10:36:24 +0900
comments: true
categories: 
---

#### [CVE-2014-0224 a.k.a ChangeCipherSpec](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0224)

OpenSSL before 0.9.8za, 1.0.0 before 1.0.0m, and 1.0.1 before 1.0.1h does not properly restrict processing of ChangeCipherSpec messages, which allows man-in-the-middle attackers to trigger use of a zero-length master key in certain OpenSSL-to-OpenSSL communications, and consequently hijack sessions or obtain sensitive information, via a crafted TLS handshake, aka the "CCS Injection" vulnerability.

* [How I discovered CCS Injection Vulnerability (CVE-2014-0224)](http://ccsinjection.lepidum.co.jp/blog/2014-06-05/CCS-Injection-en/index.html)
* [ccsinjection.c](https://gist.github.com/rcvalle/71f4b027d61a78c42607)
* [ccsinjection_server.c](https://gist.github.com/rcvalle/585e12e4d5d3b658cd3d)


#### [CVE-2014-0221](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0221)

The dtls1_get_message_fragment function in d1_both.c in OpenSSL before 0.9.8za, 1.0.0 before 1.0.0m, and 1.0.1 before 1.0.1h allows remote attackers to cause a denial of service (recursion and client crash) via a DTLS hello message in an invalid DTLS handshake.

#### [CVE-2014-0195](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0195)

The dtls1_reassemble_fragment function in d1_both.c in OpenSSL before 0.9.8za, 1.0.0 before 1.0.0m, and 1.0.1 before 1.0.1h does not properly validate fragment lengths in DTLS ClientHello messages, which allows remote attackers to execute arbitrary code or cause a denial of service (buffer overflow and application crash) via a long non-initial fragment.

* [ZDI-14-173/CVE-2014-0195 - OpenSSL DTLS Fragment Out-of-Bounds Write: Breaking up is hard to do](http://h30499.www3.hp.com/t5/HP-Security-Research-Blog/ZDI-14-173-CVE-2014-0195-OpenSSL-DTLS-Fragment-Out-of-Bounds/ba-p/6501002#.U5ER15SSyYl)

#### [CVE-2014-0160 a.k.a HeartBleed](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-0160)

The (1) TLS and (2) DTLS implementations in OpenSSL 1.0.1 before 1.0.1g do not properly handle Heartbeat Extension packets, which allows remote attackers to obtain sensitive information from process memory via crafted packets that trigger a buffer over-read, as demonstrated by reading private keys, related to d1_both.c and t1_lib.c, aka the Heartbleed bug.

* [HeartBleed.com](http://heartbleed.com/)
