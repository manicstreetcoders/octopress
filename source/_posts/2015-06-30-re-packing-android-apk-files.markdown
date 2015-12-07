---
layout: post
title: "Re-packing Android APK files"
date: 2015-06-30 11:23:09 +0900
comments: true
categories: 
---

#### Packing & Unpacking

```
unzip On*.apk classes.dex
java -mx512m -jar ./baksmali-2.0.3.jar classes.dex
keytool -genkey -v -keystore my-release-key.keystore -alias babo -keyalg RSA -keysize 2048 -validity 10000
java -mx512m -jar ./smali-2.0.3.jar out -o classes.dex
zip -d On*.apk classes.dex
zip -d On*.apk "META-INF/*"
zip On*.apk classes.dex
jarsigner -storepass <password> -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore my-release-key.keystore On*.apk babo
adb shell pm uninstall com.xxx.yyy.zzz
adb install On*.apk
```

#### aapt

```
aapt dump WHAT file.apk [asset [asset ...]]
  strings
  badging
  permissions
  resources
  configurations
  xmltree
  xmlstrings
```
