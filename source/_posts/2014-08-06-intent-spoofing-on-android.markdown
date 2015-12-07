---
layout: post
title: "Intent Spoofing on Android"
date: 2014-08-06 15:37:16 +0900
comments: true
categories: 
---

* [FourGoats Vulnerabilities: Intent Spoofing](http://pwntest.wordpress.com/2012/11/17/168935099/)
* [Intent Spoofing on Android](http://blog.palominolabs.com/2013/05/13/android-security/)
* [Android App Security: What (not) to do!](http://www.slideshare.net/mrtompa/android-app-security-what-not-to-do/)

#### Action

Intent spoofing 에 취약한 App 을 exploit 하는 것은 다음 코드와 같다. `startActivityForResult()` 로 인텐트를 날린다.

    protected void onCreate(Bundle paramBundle)
    {
        super.onCreate(paramBundle);
        setContentView(2130903063);
        Intent localIntent = new Intent();
        localIntent.putExtra("galleryFlowID", 100);
        localIntent.putExtra("carowner", "suzuki");
        localIntent.putExtra("Intent_new_title", "Toyota");
        localIntent.setComponent(new ComponentName("com.toyota.tconnect", "com.toyota.tconnect.RemoteStartPINActivity"));
        startActivityForResult(localIntent, 0);
    }

#### Intent spoofing

이런 취약점이 발생하는 이유는, 인텐트 receiver 를 정의할때, 퍼미션 세팅을 안했기때문.

    <receiver android:name="one.special.receiver">
      <intent-filter>
        <action android:name="one.intent.action" />
      </intent-filter>
    </receiver>

* Public components and senders with weak permissions
* Malicious app sends Intent resulting in data injection or state change

#### Intent spoofing 방지

    <receiver android:name="one.special.receiver" 
                android:exported=false>
      <intent-filter>
        <action android:name="one.intent.action" />
      </intent-filter>
    </receiver>

    <receiver android:name="one.special.receiver" 
                android:exported=false
                android:permission="one.permission">
      <intent-filter>
        <action android:name="one.intent.action" />
      </intent-filter>
    </receiver>

#### Unauthorized Intent Receipt

인텐트를 다음과 같이 보내면 안된다.

    Intent intent = new Intent();
    intent.setAction("a.special.action");
    startActivity(intent);

* Given a public Intent which doesn't require strong permission in the receiving component
* Intercepted by malicious app
* May leak sensitive data and/or change in control flow

좋은 예

    Intent fixedIntent = new Intent();
    fixedIntent.setClassName("pkg.name","pkg.name.DestinationName");

    Intent fixedIntent2 = new Intent();
    fixedIntent2.setAction("a.special.action");
    sendBroadcast(fixedIntent2, "a.special.permission");

위와 같이 보내는 것이 좋다.
