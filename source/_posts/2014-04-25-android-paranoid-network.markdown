---
layout: post
title: "ANDROID_PARANOID_NETWORK"
date: 2014-04-25 12:35:34 +0900
comments: true
categories: 
---

    bootloader loads kernel
    -> kernel
    -> init process
    -> init spawns daemons (i.e. adbd)
    -> init launches zygote
    -> zygote launches the first DVM and proloads all core classes
    -> zygote forks itself and starts a process called system server
    -> the system server then starts all core android services i.e. activity manager

    ANDROID_PARANOID_NETWORK
    /include/linux/android_aids.h
    AID_INET
    /system/core/include/private/android_filesystem_config.h
    /system/etc/permissions/platform.xml

