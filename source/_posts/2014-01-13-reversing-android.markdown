---
layout: post
title: "Reversing Android"
date: 2014-01-13 18:20
comments: true
categories: 
---

Android reversing tools:

* [jd-gui](http://jd.benow.ca/)
* [Procyon](https://bitbucket.org/mstrobel/procyon/wiki/Java%20Decompiler)
* [JAD](http://varaneckas.com/jad/)
* jadx
* CFR
* JEB Decompiler (Native Dalvik decompiler)
* IDA 6.1
* [dex2jar](https://code.google.com/p/dex2jar/)
* [smali/baksmali](https://code.google.com/p/smali/)
* [apktool](https://code.google.com/p/android-apktool/)
* [signapk](https://code.google.com/p/signapk/)
* [ded](http://siis.cse.psu.edu/ded/installation.html)
* [Android ICS JB EXT4 imagefile unpacker](http://sourceforge.net/projects/androidicsjbext/)
* [ext2read/ext2explorer](http://sourceforge.net/projects/ext2read/)
* [droidbox ](https://code.google.com/p/droidbox/)
* [androguard](http://code.google.com/p/androguard)
* [Android Emulator](http://developer.android.com/tools/help/emulator.html)
* [VTS](http://virtuous-ten-studio.com)
* [TLC UpdatezipCreator](http://forum.xda-developers.com/showthread.php?t=1248486)
* [APK Multi-Tool](http://apkmultitool.com/node/7)

안드로이드 framework 을 수정해서 폰에 올려서 커스텀 폰을 만드는 것도 재미있었고, apk 를 해제해서 smali 코드에 Log 를 code injection 하고 다시 패키징해서 폰에 인스톨해서 동적 분석하는 것도 꽤 스릴넘쳤다.

몇 가지 Tip.

### [jar 를 .java 로 풀어내는 법](http://stackoverflow.com/questions/647116/how-to-decompile-a-whole-jar-file).

일단 smali/baksmali/dex2jar 를 가지고 optimized dex -> dex -> jar (*.class) 로 바꿔서 classes-dex2jar.jar 를 만든 다음에...

    % cat jad.sh
    JAR=./classes-dex2jar.jar
    unzip -d $JAR.tmp $JAR
    pushd $JAR.tmp
    for f in `find . -name '*.class'`; do
        jad -d $(dirname $f) -s java -lnc $f
    done
    popd

위와같은 shell script 로 java 로 바꾼다. java 로 바꿔서 IntelliJ 에서 import 해서 보면 JD-GUI 로 보는 것보다 편하다.

### apk 를 서명한 private key 와 pair 인 cert 를 추출하는 법

    $ jarsigner -verify -certs -verbose testing.apk
    $ unzip testing.apk
    $ cd META-INF
    $ openssl pkcs7 -in CERT.RSA -print_certs -inform DER -out out.cer
    $ cat out.cer

### DroidBox

    wget http://droidbox.googlecode.com/files/DroidBox411RC.tar.gz
    android
    ./startmenu.sh <AVD name>
    ./droidbox.sh babo.apk

Resources:

* [Why do I get access denied to data folder when using adb?](http://stackoverflow.com/questions/1043322/why-do-i-get-access-denied-to-data-folder-when-using-adb)
* Learning Pentesting for Android Devices
* Android Hacker's Handbook
