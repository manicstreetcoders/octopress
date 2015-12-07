---
layout: post
title: "Blackphone goes to Def Con and gets hacked"
date: 2014-08-13 15:29:48 +0900
comments: true
categories: 
---

Justin Case 의 작업 [Blackphone goes to Def Con and gets hacked --sort of](http://arstechnica.com/security/2014/08/blackphone-goes-to-def-con-and-gets-hacked-sort-of/) 을 따라가보자.

#### Step 1: ADB 활성화

블랙폰은 ADB 접속이 disable 된 채로 판매되는데, apk 를 인스톨하고 실행해서 enable 시킨다. `com.android.settings.Settings$DevelopmentSettingsActivity` 을 인텐트로 보낸다.

    public class MainActivity extends Activity
    {
      protected void onCreate(Bundle paramBundle)
      {
        super.onCreate(paramBundle);
        ComponentName localComponentName = new ComponentName("com.android.settings", "com.android.settings.Settings$DevelopmentSettingsActivity");
        Intent localIntent = new Intent("android.intent.action.MAIN");
        localIntent.setComponent(localComponentName);
        startActivity(localIntent);
        finish();
      }
    }

#### Step 2: system 권한 획득

이제 ADB 액세스를 얻었다. `remotewipe` app 이 system 으로 돌고 있고, debuggable 하다. jdb 를 붙여서 `system` 권한의 쉘을 떨어뜨렸다고 한다. How-to 는 saurik 의 글을 참조 (http://www.saurik.com/id/17)

    $ axml AndroidManifest.xml
    <?xml version="1.0" encoding="utf-8"?>
    <manifest
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:shareUserId="android.uid.system"
        android:versionCode="6"
        android:versionName="0.8.2"
        package="com.karumi.blackphone.wipe"
        >

    <application
        ...
        android:debuggable="true"
        android:allowBackup="false"
        >

#### Step 3: Privilege Escalation

(Justin Case 는) 아직 공개하지 않은 버그를 이용해 `system` -> `root` 으로 올렸다고 한다.
