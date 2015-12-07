---
layout: post
title: "How To Build CyanogenMod Android for HTC One (M8)"
date: 2014-07-22 15:55:59 +0900
comments: true
categories: 
---

[How To Build CyanogenMod Android for HTC One (M8)](http://wiki.cyanogenmod.org/w/Build_for_m8)

HTC One M8 에 올릴 CyanogenMod 를 빌드하였다. 빌드 머신은 Ubuntu 14.04.1 LTS 64 bit.

중간에 ./extract-files.sh 에서 에러가 난 것 빼고는 순조롭게 마쳤다.
중간에 2 개의 shared object 가 없다고 중단이 되었는데, www.xda-developers.com 에서 답을 찾았다.

* [Q: Missing files in ./exctract-files.sh (sic) from CM](http://forum.xda-developers.com/htc-one-m8/help/missing-files-exctract-files-sh-cm-t2814905)
* [TheMuppets/proprietary_vendor_htc](https://github.com/TheMuppets/proprietary_vendor_htc/tree/cm-11.0/m8/proprietary)
에서 다운받았다.

노트북에서 VM 으로 빌드해서 그런지 하룻밤 걸렸다.
