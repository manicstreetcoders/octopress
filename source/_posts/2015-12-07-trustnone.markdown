---
layout: post
title: "TRUSTNONE"
date: 2015-12-07 17:14:52 +0900
comments: true
categories: 
---

[TRUSTNONE](http://theroot.ninja/disclosures/TRUSTNONE_1.0-11282015.pdf) 을 읽고 정리해본다.

Sean Beaupre 씨가 발견한 버그. 스냅드래곤 805 트러스트존 커널의 메모리/레지스터를 읽고 쓸 수 있다.

#### 백그라운드

`tzbsp_smmu_fault_regs_dump` 라는 시스템콜에 버그가 있어서. 

1. 4 개의 인자를 기대하는데.
2. `R4-R8` 과 링크 레지스터를 스택에 저장하고
3. `R0`,`R1`,`R2`,`R3` 에 담긴 arg0, arg1, arg2, arg3 를 `R5-R8` 에 옮기고. 이때 `adds` 로 `R2` 가 0 인지 아닌지를 체크하고
4. `BEQ` 로 `R2 != 0` 이면 `is_debugging_disabled` 체크하는 루틴으로 뛰고
5. `CBZ` 로, `is_debugging_disabled` 가 1 이면 `loc_FE85881E` 로 뛰고
6. 
