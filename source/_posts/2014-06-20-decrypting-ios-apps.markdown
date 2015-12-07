---
layout: post
title: "Decrypting iOS Apps"
date: 2014-06-20 10:22:20 +0900
comments: true
categories: 
---

[Decrypting iOS Apps](http://www.infointox.net/?p=114)

evasi0n 으로 iPod 를 Jailbreak 했다.

openssh 패키지를 설치하고, `ssh root@XXX.XXX.XXX.XXX` 로 접속했다.
그리고, `root` 와 `mobile` 의 password 를 세팅하였다.

#### Extract the application

`/private/var/mobile/Applications/` 에서 타겟을 담고 있는 디렉토리를 찾았다.

`/private/var/mobile/Application/D411A4CE-B33E-4EFE-9041-E356718123EE/OM1X.app/OM1X` 가 찾는 바이너리.

scp 로 OM1X 를 가져온다.

    # otool -arch all -Vh OM1X
    OM1X:
    Mach header
          magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
       MH_MAGIC     ARM          9  0x00     EXECUTE    33       4056   NOUNDEFS DYLDLINK TWOLEVEL BINDS_TO_WEAK PIE

#### Find encrypted area

이제 암호화된 부분을 찾아야 한다.

    # otool -arch all -Vl TM1G | grep -A5 LC_ENCRYP
              cmd LC_ENCRYPTION_INFO
          cmdsize 20
     cryptoff  16384
     cryptsize 819200
     cryptid   1
    Load command 12

