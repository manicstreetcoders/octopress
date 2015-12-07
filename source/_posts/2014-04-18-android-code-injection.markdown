---
layout: post
title: "Android: Code Injection"
date: 2014-04-18 18:51:50 +0900
comments: true
categories: 
---

smali 코딩할때 예제가 필요하면,

    inject_log% javac x.java
    inject_log% dx --dex --output=x.dex x.class
    inject_log% java -jar ../baksmali-2.0.3.jar -a 10 x.dex

smali 작업을 해서 묶어서 다시 인스톨해서 테스트하고... 하는 작업을 여러번하다보니 스크립트를 만들어야겠다는 생각이 들었다.

    java -jar apktool_2.0.0b7.jar b com.babo.babophone-1
    jarsigner -verbose -keystore my-release-key.keystore ./com.babo.babophone-1/dist/com.babo.babophone-1.apk babo
    echo "copy..."
    adb push com.babo.babophone-1/dist/com.babo.babophone-1.apk /sdcard/
    echo "uninstall..."
    adb shell pm uninstall com.babo.babophone
    echo "install..."
    adb shell pm install /sdcard/com.babo.babophone-1.apk

그리고,

nAction 을 logging 하는 smali 코드를 작성해봤다. smali 는 Java Jasmine 부터 보는게 좋다.

    .method public getURL(IZ)Ljava/lang/String;
        .locals 3
        .param p1, "nAction"    # I
        .param p2, "bSupportAES"    # Z

        .prologue
        .line 87

        move v0, p1
        invoke-static {v0}, Ljava/lang/String;->valueOf(I)Ljava/lang/String;
        move-result-object v1

        const-string v0, "getURL: nAction"

        invoke-static {v0, v1}, Lcom/acme/util/DebugPrint;->printDebug(Ljava/lang/String;Ljava/lang/String;)V

        const-string v0, ""

        .line 89
        .local v0, "strURL":Ljava/lang/String;
        packed-switch p1, :pswitch_data_0
