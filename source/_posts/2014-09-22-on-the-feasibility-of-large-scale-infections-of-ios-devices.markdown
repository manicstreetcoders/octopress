---
layout: post
title: "On the Feasibility of Large-Scale Infections of iOS Devices"
date: 2014-09-22 09:00:23 +0900
comments: true
categories: 
---

23rd USENIX Security Symposium 에서 발표된 [On the Feasibility of Large-Scale Infections of iOS Devices](https://www.usenix.org/system/files/conference/usenixsecurity14/sec14-paper-wang-tielei.pdf) 을 따라가보자.

저자는 Tielei Wang, Yeongjin Jang, Yizheng Chen, Simon Chung, Billy Lau, Wenke Lee.

#### Introduction

Non-jailbroken iOS 디바이스를 대규모로 감염시키는 것이 어려운 이유.

1. Apple has powerful revocation capabilities.

2. The mandatory code signing mechanism in iOS ensures that only apps signed by Apple or certified by Apple can be installed and run on iOS devices.

3. The DRM technology in iOS prevents users from sharing apps among arbitary iOS devices, which has a side effect of limiting the distribution of malicious apps published on the App Store.

#### Delivery of Apple-Signed Malicious Apps

두 개의 취약점을 사용했다고 한다.

1. The first is a previously unknown design flaw in the iTunes syncing mechanism, which makes the iTunes syncing process vulnerable to MiTM attacks. 

2. The second security issue we discovered is that an iOS device can be stealthily provisioned for development through USB connections. This weakness allows a compromised computer to arbitrarily remove installed third-party appps from connected iOS devices and install any app signed by attackers in possession of enterprise or indivisual developer licenses issued by Apple.
