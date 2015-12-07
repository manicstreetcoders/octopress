---
layout: post
title: "Some Android Books"
date: 2015-05-21 00:43:52 +0900
comments: true
categories: 
---

#### Bootstrapping

```
export PATH=$PATH:/path/to/sdk/tools/:/path/to/sdk/platform-tools/

# cd /usr/local/bin
# ln -s /path/to/binary

$ sudo apt-get install openjdk-6-jdk

A 64-bit system requires an additional installation of 32-bit packages needed by the SDK tools.
You can install these on Ubuntu 13.04 upward by using

$ sudo dpkg -add-architecture i386
$ sudo apt-get update
$ sudo apt-get install libncurses5:i386 libstdc++6:i386 zlib1g:i386

Prior to that version of Ubuntu, you use the following command:

$ sudo apt-get 

$ android sdk

$ android avd

$ emulator -avd kitkat

$ adb devices

$ adb -s device_id shell

$ nc localhost 5554
Android Console: type 'help' for a list of commands
OK
help

* Genymotion (http://www.genymotion.com/)
* Virtualbox running Android x86 (http://www.android-x86.org)
* Youwave (https://youwave.com)
* WindowsAndroid (http://windowsandroid.en.softonic.com/)

shell@android:/ $ ps
USER        PID     PPID        VSIZE       RSS     WCHAN       PC          NAME

shell@android:/ # ls -l /data/data
...
drwxr-x--x u0_a46   u0_a46      2014-04-10 10:41 com.amazingutils.batterysaver

class Test
{
    public static void main(String[] args)
    {
        System.out.println("It works! :D");
    }
}

$ javac Test.java
$ dx -dex -output=test.jar Test.class

$ adb push test.jar /data/local/tmp
$ adb shell dalvikvm -cp /data/local/tmp/test.jar Test
It works :D
```

#### Android Packages

Android applications are distributed in the form of a zipped archive with the file
extension of .apk, which stands for Android Package. The official MIME-type of an
Android Package is application/vnd.android.package-archive. These packages
are nothing more than zip files containing the relevant compiled application code,
resources, and application metadata required to define a complete application.
According to Google's documentation at http://developer.android.com/tools/building/index.html,
an APK is packaged by performing the following taks:

* An SDK tool named aapt (Android Asset Packaging Tool) converts all the XML resource files included
in the application to a binary form. R.java is also produced by aapt to allow referencing of 
resources from code.
* A tool named aidl is used to convert any .aidl files to .java files containing a converted
representation of it using a standard Java interface.
* All source code and converted output from aapt and aidl are compiled into .class files by 
the Java 1.6 compiler. This requires the android.jar file for your desired API version to be
in the CLASSPATH environment variable.
* The dx utility is used to convert the produced .class files and any third-party libraries into a single
classes.dex file.
* All compiled resources, non-compiled resouces (such as images or additional executables), and
the application DEX file are used by the apkbuilder tool to package an APK file. More recent version of the
SDK have deprecated the standalone apkbuilder tool and included it as a class inside sdklib.jar.
The APK file is signed with a key using the jarsigner utility. It can either be signed by a default debug
key or if it is going to production, it can be signed with your generated release key.
* If it is signed with a release key, the APK must be zip-aligned using the zipalign tool, which
ensures that the application resources are aligned optimally for the way that they will be loaded into memory.
The benefit of this is that the amount of RAM consumed when running the application is reduced.

#### Tools

```
$ adb install /path/to/yourapplication.apk
$ adb shell pm
$ python -m  SimpleHTTPServer

$ adb devices
$ adb shell
$ adb shell <command>
$ adb push /path/to/local/file /path/on/android/device
$ adb pull /path/on/android/device /path/to/local/file
$ adb forward tcp:<local_port> tcp:<device_port>
$ adb logcat

http://www.busybox.net/downloads/binaries/

$ adb push busybox-armv71 /data/local/tmp
shell@android:/ $ chmod 755 /data/local/tmp/busybox-armv71
sheel@android:/ $ export PATH=$PATH:/data/loal/tmp

shell@android:/ $ pm list packages
shell@android:/ $ pm path <package_name>
shell@android:/ $ pm install /path/to/apk
shell@android:/ $ pm uninstall <package_name>
shell@android:/ $ pm disable <package_name>
shell@android:/ $ logcat
shell@android:/ $ logcat -s tag

shell@android:/ $ getprop
shell@android:/ $ dumpsys
shell@android:/ $ service list
```

#### Drozer

[Drozer](https://www.mwrinfosecurity.com/products/drozer/community-edition/)

```
$ adb install agent.apk
$ adb forward tcp:31415 tcp:31415
$ drozer console connect

dz> list package
dz> module search -d
```

[Drozer modules](https://github.com/mwrlabs/drozer-modules/)

```
app.activity
app.broadcast
app.package
app.provider
app.service
auxiliary
exploit.pilfer
exploit.root
information
scanner
shell
tools.file
tools.setup
```

```
dz> shell
u0_a59@android:/data/data/com.mwr.dz $ id

dz> run app.package.info -a com.mwr.dz -h

dz> run app.package.info -a com.mwr.dz > /path/to/output.txt
```

#### Retrieving APK Files

```
$ adb shell pm list packages | grep twitter
package:com.twitter.android
```

```
dz> run app.package.list -f "Terminal Emulator"
jackpal.androidterm (Terminal Emulator)
```

```
$ adb shell pm path jackpal.androidterm
package:/data/app/jackpal.androidterm-2.apk
```

```
dz> run app.package.info -a jackpal.androidterm
Package: jackpal.androidterm
  Application Label: Terminal Emulator
  Process Name: jackpal.androidterm
  Version: 1.0.59
  Data Directory: /data/data/jackpal.androidterm
  APK Path: /data/app/jackpal.androidterm-2.apk
  UID: 10215
  GID: [3003, 1015, 1023, 1028]
  Shared Libraries: null
  Shared User ID: null
  Uses Permissions:
  - android.permission.INTERNET
  ...
```

* http://apkleecher.com
* http://apps.evozi.com/apk-downloader

#### Viewing Manifests

```
$ aapt dump xmltree /path/to/agent.apk AndroidManifest.xml

$ aapt l -a /path/to/agent.apk

$ unzip agent.apk

$ java -jar AXMLPrinter2.jar AndroidManifest.xml
<?xml version="1.0" encoding="utf-8"?>
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:versionCode="5"
    android:versionName="2.3.4"
    package="com.mwr.dz">
    <uses-sdk android:minSdkVersion="7" android:targetSdkVersion="18">
    </uses-sdk>
...

dz> run app.package.manifest com.mwr.dz
...
```

#### Activity & Contenet Provider

```
dz> run app.activity.start --action android.intent.action.VIEW --data-uri http://www.google.com --component com.android.browser com.android.browser.BrowserActivity

dz> run app.provider.query content://settings/system
```

#### Permissions

```
dz> run app.package.info -a com.android.browser
Package: com.android.browser
Application Label: Browser
Process Name: com.android.browser
Version: 4.4.2-938007
Data Directory: /data/data/com.android.browser
APK Path: /system/app/Browser.apk
UID: 10014
GID: [3003, 1028, 1015]
Shared Libraries: null
Shared User ID: null
Users Permissions:
- android.permission.ACCESS_COARSE_LOCATION
...

dz> run app.package.list -p android.permission.READ_SMS
com.android.phone (Phone)
com.android.mms (Messaging)
com.android.gallery (Camera)
com.android.camera (Camera)
...

dz> run app.provider.info -p android.permission.READ_SMS
Package: com.android.mms
...
```

#### Protection Levels

```
$ drozer agent build --permission android.permission.INSTALL_PACKAGES
Done: /tmp/tmp2RdLTd/agent.apk

$ adb install /tmp/tmp/2RdLTd/agent.apk
2312 KB/s (653054 bytes in 0.275s)
     pkg: /data/local/tmp/agent.apk
Success

$ adb logcat
...
W/PackageManager(  373): Not granting permission
android.permission.INSTALL_PACKAGES to package com.mwr.dz (protectionLevel=18 flags=0x83e46)
...

dz> permissions
...
```

#### Code Signing

```
$ keytool -genkey -v -keystore mykey.keystore -alias alias_name -keyalg RSA -keysize 2048 -validity 10000

$ jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore mykey.keystore application.apk alias_name

$ openssl pkcs7 -inform DER -in CERT.RSA -text -print_certs

$ keytool -printcert -file CERT.RSA
```

#### Dex Tools

* smali/baksmali

```
$ java -jar baksmali.jar app.apk
$ jav -jar smali.jar -o classes.dex out/

$ java -jar baksmali.jar -a 19 -x Settings.odex -d framework
$ java -jar smali.jar -a 19 -o Settings.dex out/
```

* IDA
* dex2jar

```
$ d2j-dex2jar.sh agent.apk -o agent.jar
```

* JEB
* apktool
* jadx
* jad
