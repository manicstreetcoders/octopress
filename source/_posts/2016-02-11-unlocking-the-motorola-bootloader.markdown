---
layout: post
title: "Unlocking the Motorola Bootloader"
date: 2016-02-11 15:34:54 +0900
comments: true
categories: 
---

Bits, Please! 블로그에 올라온 [Unlocking the Motorola Bootloader](http://bits-please.blogspot.in/2016/02/unlocking-motorola-bootloader.html) 를 읽어보았다.

모토로라 휴대폰의 부트체인은, PBL (Primary Boot Loader) -> SBL1 (Secondary Boot Loader) -> SBL2 -> SBL3 -> aboot -> Linux Kernel 의 순서로 올라온다고.

저자는 제조사가 제공하는 img 화일을 분석해서, aboot 이미지를 뜯어내었다. 

unlock 스트링을 검색해서, 언락을 수행하는 코드를 찾았다고 한다.

unlock 을 수행하기 위해서 sub_FF4BA80() -> sub_FF4BA60() -> sub_FF4BA40() 을 차례로 호출하는데, 그 중에 하나가 성공하면 바로 리부팅으로 들어간다.

sub_FF4BA80() 을 봤더니,

```
int sub_FF4BA80()
{
	if (!sub_FF027F8(0x6D))
		sub_FF02BEC(0x6D, 1);
	return 0;
}
```

`sub_FF027f8()`, `sub_FF02BEC()` 를 봤더니, 결국은 어떤 함수를 호출하는데, 그 함수에는 `SMC #0` 명령이 담겨있었다. 저자는 unlocking 코드가 결국은 TrustZone 에 위치하고 있다고 의심한다. LR 에 담긴 TZ 어드레스는 0x3F801.

TZ 커널 시스템 콜을 분석한 결과, 다음과 같은 엔트리를 발견했다고.

```
DCD 0x3F801
DCD aMotorola_tzbsp	; "motorola_tzbsp_ns_service"
DCD 0xD
DCD motorola_tzbsp_ns_service+1
DCD 4
DCD 4
DCD 4
DCD 4
DCD 4
```

이 콜은 QFuses 에서 읽고 쓰는 함수.

정리하면,

* aboot 가 SMC 0x3F801 를 호출 (command-code #1) -> TZ kernel 이 QFuse @ 0xFC4B86E8 을 읽어 반환
* 첫번째 비트가 0 이라면, SMC 0x3F801 을 다시 호출 (command-code #2) -> TZ kernel 이 QFuse 의 LSB 를 1 로 세팅

이걸로 unlock 이 끝난다고.

그렇다면, QFuses 를 write 하려면?

다행히도, TZ kernel 이 두개의 함수를 제공한다고. `tzbsp_qfprom_read_row()` & `tzbsp_qfprom_write_row()`

그런데, QFuses 에 쓰는 함수가 몇가지 체크를 하는데, 이 체크를 bypass 해야한다.

* 메모리 DWORD 0xFE823D5C 가 0 이 아니어야하고.
* is_allowed_fuse_row() 에서 허가된 QFuse 어드레스인지 체크함.

TZ code execution exploit 에서 저 두가지를 미리 셋업해줘야한다고 함. 미리 셋업하고, `tzbsp_qfprom_write_row()` 를 불러서 unlock 수행.

관련된 코드를 저자가 [github](https://github.com/laginimaineb/Alohamora) 에 공개하였다.

`build_shellcode.sh` 를 보았다. `shellcode.S` 를 arm-eabi-gcc 를 이용해서 어셈블, 링킹하고 arm-eabi-objcopy 를 통해서 elf -> raw binary 로 변환한다.
