---
layout: post
title: "Linux VM with QEMU"
date: 2014-08-06 14:54:22 +0900
comments: true
categories: 
---

1. Mac의 경우 brew install qemu 로 qumu 가상 머신을 설치. qemu-system-i386 사용.

2. debian 기반 linux 이미지 https://people.debian.org/~aurel32/qemu/i386/debian_wheezy_i386_standard.qcow2 (root 패스워드가 "root")

3. 가상머신을 start. 

    $ qemu-system-i386 -hda debian_wheezy_i386_standard.qcow2

4. 하이버네이션을 위해서 debian 내에 power manager 패키지를 설치.

    # apt-get install pm-utils

5. Hibernate.

    # pm-hibernate
