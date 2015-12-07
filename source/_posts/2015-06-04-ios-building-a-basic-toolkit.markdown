---
layout: post
title: "iOS: Building a Basic Toolkit"
date: 2015-06-04 13:23:03 +0900
comments: true
categories: 
---

#### Cydia
#### BigBoss Recommended Tools
#### otool

```
# Inspect the Objective-C segment to reveal class and method names:
$ otool -oV MAHHApp

# List the libraries used by the binary:
$ otool -L MAHHApp

# List the symbols exported by a binary
$ otool -IV MAHHApp

# Display the short-form header information:
$ otool -hV MAHHApp

# Display the binary load commands:
$ otool -l MAHHApp
```

#### nm

```
$ nm MAHHApp
```

#### lipo

```
$ lipo -info MAHHApp
$ lipo -thin <arch_type> -output MAHHApp-v7 MAHHApp
```

#### Debuggers

* gdb (Help: [here](http://www.pod2g.org/2012/02/working-gnu-debugger-on-ios-43.html)
* radare (Add cydia repo as a source, http://cydia.radare.org)

```
$ lipo -thin armv7 gdb-arm-apple-darwin -output gdb-arm7
```

#### Tools for Signing Binaries

```
# To sign or replace an existing signature, use the following command:
$ codesign -v -fs "CodeSignIdentity" MAHHApp.app/

# To display the code signature of an application:
$ codesign -v -d MAHHApp.app
```

* saurik's ldid: [here](http://www.saurik.com/id/8)

```
$ ldid -S MAHHApp
```

#### Installipa

* [here](https://github.com/autopear/ipainstaller)
* AppSync (repo: http://cydia.appaddict.org)

```
# ipainstaller Lab1.1a.ipa
Analyzing Lab1.1a.ipa...
Installing lab1.1a (v1.0)...
Installed lab1.1a (v1.0) successfully.
```

#### Exploring the Filesystem

* [Mathieu Renard](http://2013.hackitoergosum.org/presentations/Day3-04.Hacking%20apple%20accessories%20to%20pown%20iDevices%20%E2%80%93%20Wake%20up%20Neo!%20Your%20phone%20got%20pwnd%20!%20by%20Mathieu%20GoToHack%20RENARD.pdf)
* iExplorer
* iFunBox

#### Property Lists

```
# plutil com.google.Authenticator.plist
$ plutil -convert xml1 com.google.Authenticator.plist
$ plutil -convert binary1 com.google.Authenticator.plist
```
