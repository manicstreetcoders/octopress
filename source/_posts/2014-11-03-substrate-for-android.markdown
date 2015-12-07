---
layout: post
title: "Substrate for Android"
date: 2014-11-03 10:40:38 +0900
comments: true
categories: 
---

#### Tools

* [Xposed Framework](http://repo.xposed.info)
* [Cydia Substrate](http://www.cydiasubstrate.com)
* [Cycript](http://www.cycript.org)
* [ddi/adbi](https://github.com/crmulliner/)
* [Frida](http://www.frida.re)

#### References

* [JustTrustMe: SSL Pinning kill switch](https://github.com/Fuzion24/JustTrustMe)
* [XPrivacy](https://github.com/M66B/XPrivacy)
* [iSECPartners Introspy-iOS ](https://github.com/iSECPartners/Introspy-iOS)

#### Cydia Substrate

우선 Violet example. Substrate 를 깔고, violet.apk 를 설치하면 일부 시스템 메시지들이 색이 violet 으로 바뀐다. Build 가 잘 안되면, SDK 폴더 밑에 있는 `saurikit/cydia_substrate/substrate-api.jar` 화일을 프로젝트 폴더로 카피.

#### AndroidManifest.xml

    <?xml version="1.0" encoding="utf-8" standalone="no"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.saurik.android.violetexample">
        <application android:label="@string/app_name">
            <meta-data android:name="com.saurik.substrate.main" android:value=".Main"/>
        </application>
        <uses-permission android:name="cydia.permission.SUBSTRATE"/>
    </manifest>


#### Main.java

    package com.saurik.android.violetexample;
    import android.util.Log;
    import java.lang.reflect.Method;
    import android.content.res.Resources;
    import com.saurik.substrate.MS;
    import com.saurik.substrate.MS.ClassLoadHook;
    import com.saurik.substrate.MS.MethodAlteration;

    public class Main {
        public static void initialize() {
            Log.d("HOOK", "hook initialized");

            MS.hookClassLoad("android.content.res.Resources", new ClassLoadHook() {
                public void classLoaded(Class Resources) {
                    try {
                        MS.hookMethod(Resources, Resources.getDeclaredMethod("getColor", new Class[]{Integer.TYPE}),
                            new MethodAlteration<Resources, Integer>() {
                                public Integer invoked(Resources resources, Object... args) throws Throwable {
                                    return Integer.valueOf((((Integer) invoke(resources, args)).intValue() & 
                                        ~0x0000ff00) | 0x00ff0000);
                                }
                            });
                    } catch (NoSuchMethodException e) {
                    }
                }
            });
        }
    }

#### com.android.internal.telephony.gsm.SmsMessage

이제 pdu 를 캡쳐해보자.

    public class Main {
        public static void initialize() {
            MS.hookClassLoad("com.android.internal.telephony.gsm.SmsMessage", new MS.ClassLoadHook() {
                public void classLoaded(Class<?> smsMessage) {
                    Method getSubmitPdu;
                    try {
                        getSubmitPdu = smsMessage.getMethod("getSubmitPdu",
                            String.class,
                            String.class,
                            String.class,
                            Boolean.TYPE,
                            byte[].class,
                            Integer.class,
                            Integer.class,
                            Integer.class,
                            Integer.class
                            );
                    } catch (NoSuchMethodException e) { getSubmitPdu = null; }
                    if (getSubmitPdu != null) {
                        final MS.MethodPointer old = new MS.MethodPointer();
                        MS.hookMethod(smsMessage, getSubmitPdu, new MS.MethodHook() {
                            public Object invoked(Object smsMessage, Object... args) throws Throwable {
                                Object submitPdu;
                                submitPdu = old.invoke(smsMessage, args);
                                try {
                                    Field field;
                                    field = submitPdu.getClass().getSuperclass().getDeclaredField("encodedMessage");
                                    field.setAccessible(true);
                                    if (field.getType() == byte[].class) {
                                        Object val = field.get(submitPdu);
                                        Log.d("Substrate", "pdu: " + this.bytesToHex((byte []) val));
                                    }
                                } catch (Exception e) {
                                    Log.e("Substrate", "Error in Hook_getSubmitPdu. " + e.toString());
                                }
                                return submitPdu;
                            }
                        }, old);
                    }
                } 
            });
        }
    }

추가로, pdu 를 교체하는 것도 가능하다.

##### SELinux

* [Saurik's post](http://forum.xda-developers.com/showthread.php?t=2466101&page=3)
* [SELinuxModeChanger](http://forum.xda-developers.com/showthread.php?t=2524485)
