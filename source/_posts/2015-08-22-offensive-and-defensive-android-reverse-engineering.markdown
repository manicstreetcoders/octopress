---
layout: post
title: "Offensive and Defensive Android Reverse Engineering"
date: 2015-08-22 22:03:57 +0900
comments: true
categories: 
---

DEFCON 2015, Offensive and Defensive Android Reverse Engineering 에서.

Github.com 의 rednaga/training 에 가보면 있음.

#### Alcatel One Touch Pop Icon A564C

우선 전략을 짜면,

* Factory img 는 없고
* OTA zip 은 완전하지 않고
* JTAG 은 노력이 너무 들고
* Chip off 로 가면 기계가 망가지고



우선 sharedUserId

```
sharedUserId="android.uid.system"
```

그리고 서비스 퍼미션.

```
<service android:name="com.qualcomm.agent.PhoneProcessAgent"
```

엔트리 포인트.

```
public int onStartCommand(Intent intent, int flags, int startId) {
    ...
    else if (Values.ACTION_AGENT.equals(intent.getAction())) {
        this.doSystemActions(intent.getStringExtra("para")); // ACTION_AGENT = "android.system.agent"
    }
    ...
}
```

doSystemAction() 을 따라가보면.

#### HTC Desire 310

```
$ adb shell ps | grep root | grep system
/system/bin/vold
/system/bin/debuggered
/system/bin/netd
/system/bin/eapd
$
```

`vold`, `debuggered`, `netd` 까지는 괜찮은데, `eapd` 는 뭐지?

