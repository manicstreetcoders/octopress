---
layout: post
title: "Hooking SSL_write"
date: 2015-04-14 17:55:07 +0900
comments: true
categories: 
---

#### LoadLibrary() + DLL path

CreateRemoteThread() 와 minhook 를 사용해서 openssl 을 사용하는 .exe 에 code 를 inject 해서, SSL_write() 을 후킹하여 plaintext 를 덤프하였다.

```
OpenProcess()
VirtualAllocEx()
GetModuleHandle(L"kernel32.dll")
GetProcAddress(... "LoadLibraryA")
CreateRemoteThread()
```

로 타겟 프로세스에 DLL 을 inject 한다.

`OpenProcess()` 로 target process 에 attach 한다. 그리고, target process 의 메모리에 메모리를 할당해서, DLL 의 path 를 카피해놓는다.
그리고, `LoadLibraryA()` 의 어드레스를 구해서, 이 어드레스가 동일할거라는 가정하에, target process 에 저 어드레스를 시작점으로 remote thread 를 생성한다.
다행히도 `LoadLibraryA()` 가 arg 를 1 개만 받으니, thread 를 생성하는 식으로 간편하게 `LoadLibraryA()` 를 API call 할 수 있다.
DLL 의 `DLL_PROCESS_ATTACH` 때 돌아가게 코드를 만들어 놓으면, 간편하게 DLL injection 을 할 수 있다. 

Injection 후에, 

```
GetModuleHandle(L"ssleay32.dll")
GetProcAddress(... "SSL_write")
MH_Initialize()
MH_CreateHook()
MH_EnableHook()
```

로 `ssleay32.dll` 의 `SSL_write()` 함수를 detour 시켰다. Detour function 에서 buffer 를 `OutputDebugString()` 으로 덤프하여 plaintext 를 떨어뜨렸다.

``` c
    OutputDebugString(L"Begin dump...");
    for (int i=0;i<num;i++) {
        j = ((unsigned char*)buf)[i];
        wsprintfW((LPWSTR)line, L"%02X", j);
        OutputDebugString((LPCWSTR)line);
    }
    OutputDebugString(L"End dump...");
```

귀찮아서 그냥 한바이트씩 출력하고, 

``` python
f = open("payload","r")
data = f.read()
f.close()
hex = data.replace('\n','')
print hex.decode('hex')
```

로 HTTPS 내용을 출력했다.

이 방식의 단점이라면, Process Explorer 등에 DLL 이 그대로 노출되는 것.

#### Full DLL

Target process 의 메모리에 DLL path 만 write 한다음에 CreateRemoteThread 로 LoadLibrary() 를 target 에 돌려서 DLL 을 끌어올리는 방식 말고, 아예 target process memory 에 DLL 사이즈만큼의 메모리를 확보해서, DLL 을 읽어서 집어넣어주고, 실행시키는 방법이 있다.

#### Reflective DLL Injection

Meterpreter 에서 사용하는 Reflective DLL Injection 도 좀 뒤져봐야겠다. 
