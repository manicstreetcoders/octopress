---
layout: post
title: "Using Wireshark with Android"
date: 2015-06-04 13:49:09 +0900
comments: true
categories: 
---

```
$ adb shell "tcpdump -s 0 -w - | nc -l -p 4444"
$ adb forward tcp:4444 tcp:4444
$ nc localhost 4444 | sudo wireshark -k -S -i -
```
