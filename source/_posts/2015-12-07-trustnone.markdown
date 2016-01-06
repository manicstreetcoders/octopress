---
layout: post
title: "TRUSTNONE"
date: 2015-12-07 17:14:52 +0900
comments: true
categories: 
---

Sean Beaupre 씨가 쓴 글, [TRUSTNONE](http://theroot.ninja/disclosures/TRUSTNONE_1.0-11282015.pdf) 을 읽고 정리해본다.

스냅드래곤 805 트러스트존 커널의 메모리/레지스터를 읽고 쓸 수 있다.

#### 백그라운드

`tzbsp_smmu_fault_regs_dump` 라는 시스템콜에 버그가 있어서. 

1. 4 개의 인자를 기대하는데.
2. `R4-R8` 과 링크 레지스터를 스택에 저장하고
3. `R0`,`R1`,`R2`,`R3` 에 담긴 arg0, arg1, arg2, arg3 를 `R5-R8` 에 옮기고. 이때 `adds` 로 `R2` 가 0 인지 아닌지를 체크하고
4. `BEQ` 로 `R2 != 0` 이면 `is_debugging_disabled` 체크하는 루틴으로 뛰고
5. `CBZ` 로, `is_debugging_disabled` 가 1 이면 `loc_FE85881E` 로 뛰고
6. `do_regs_dump` 를 호출하는데.
7. `do_regs_dump` 에서 `validate_input` 을 부른다
8. `validate_input` 에서 `do_sloppy_signed_comparison` 을 부르는데
9. `do_sloppy_signed_comparison` 은 `CMP R0, #7` `BGE loc_FE863164` 로 흘러간다
10. BGE 일 경우, #0 을 돌려주고, 아니면 1 을 돌려준다
11. 그런데 THUMB BGE 명령어는 signed comparison 이다
12. R0 가 0~6 이면 당연히 #1 을 돌려주지만, minus 일경우에도 #1 을 돌려준다
13. minus 값으로 체크를 통과하는 듯
14. validate_something_else 가 불리는데

```
	R0 = arg0;
	R1 = arg1;
	R0 = R0 * 0x11;
	uint *R2 = (uint *)0xFE8282CC; /* this is in a data segment in TrustZone kernel */
	R2 = *R2; /* in the TrustZone image I'm testing with, this value = 0xFE827B58 */
	R0 = 4 + (R0 * 0x10);
	R0 = *(R0 + R2);
	if (R0 < R1) return 0; /* fail - we need to return 1, thus R0 needs to be > R1 */
```

`do_sloppy_signed_comparison` 에서 `R0` 가 0~6 사이의 값일 것을 검사했지만, 사실 0x80000000 ~ 0xFFFFFFFF 사이의 음수라도 검사를 통과할 수 있었다. R0 가 음수일 경우를 가정해보자. 예를 들어, 0x88888888 이라면?

```
	R0 = arg0 = 0x88888888;
	R1 = arg1;
	R0 = R0 * 0x11; /* 0x11111108 */
	R2 = 0xFE8282CC;
	R2 = *(uint *)R2; /* 0xFE827B58 */
	R0 = 4 + (R0 * 0x10); /* 0x11111084 */
	R0 = *(R0 + R2); /* R0 now is loaded with the data contained at physical address 0x11111084 + 0xFE827B58 = 0x0F938BDC /* << This memory location is NOT secure, we can control it! */ 
	if (R0 < R1) return 0;
```

validate_input 에서 돌아오면, 비슷한 계산을 또 하는데,,,
