---
layout: post
title: "sendrawpdu"
date: 2014-09-19 17:51:04 +0900
comments: true
categories: 
---

#### Unlock & Rooting

HTC One M7 을 준비했다.

`./fastboot-mac oem get_identifier_token` 로 token 을 얻어 htcdev.com 에 submit 하고 언락 코드를 얻는다. 그 코드로 
`fastboot-mac flash unlocktoken Unlock_code.bin` 를 하여서 부트로더 언락을 마친다.

Power + Volume Down 으로 들어가보면 `*** UNLOCKED ***` 라고 바뀌게 된다. S-ON 은 유지된 상태.

참고로, Unlock 하면 bootloader 등의 파티션을 접근할 수 있고, S-OFF 를 하면 HBOOT 와 RADIO 까지 write 할 수 있다.

언락후에, [ClockwordMod Recovery](http://htconeroot.com/cwmtwrp-recoveries/) 를 설치하였다.

#### Xposed framework

그 다음에, [Xposed framework](http://forum.xda-developers.com/xposed/xposed-installer-versions-changelog-t2714053) 를 설치한다.

#### SMS-DELIVERY

[국내 (SKT/KTF) SMS 처리 - HTC RIL 코드를 분석하면서](http://www.androidpub.com/724734)

    PDU 0791280102198188 44 0A A1 5117180890 00040180229101826313 0A22080B811040309814F761623132B0A1B3AA

    07: SMSC Octet Size
    91: Type-of-address of the SMSC
    28 01 02 19 81 88: Service Center +821020911888
    44: TP-UDHI, TP-MMS
    0A: Length of Sender Number
    A1: 1010 0001 National Number/ISDN/Telephone Numbering Plan
    51 17 18 08 90: 

    Text message
    From: 1571818009
    Message: 0A22080B811040309814F761623132B0A1B3AA
     
    Additional information
    PDU type: SMS-DELIVER
    Time stamp: 22/08/2010 19:10:28
    SMSC: +821020911888
    Data header: 0A22080B811040309814F7
    Data coding: Binary
     
    Original Encoded PDU fields
    SMSC: 0791280102198188
    PDU header: 44
    TP-MTI: 00
    TP-MMS: 04
    TP-SRI: 00
    TP-RP: 00
    TP-UDHI: 40
    TP-OA: 0AA15117180890
    TP-PID: 00
    TP-DCS: 04
    TP-SCTS: 01802291018263
    TP-UDL: 13
    TP-UD: 0A22080B811040309814F761623132B0A1B3AA

#### SMS-SUBMIT

#### smspdu

[crmulliner/smspdu](https://github.com/crmulliner/smspdu) 를 참조.

HTC Sense firmware 의 hidden API sendRawPdu() 를 사용하는 방법은 다음과 같다.

    import android.telephony.SmsManager;
    import java.lang.reflect.Method;

    SmsManager sm = SmsManager.getDefault();
    byte[] bb = new byte[1];
    Method m2 = SmsManager.class.getDeclaredMethod(
        "sendRawPdu",
        bb.getClass(),
        bb.getClass(),
        PendingIntent.class,
        PendingIntent.class,
        boolean.class,
        boolean.class);
    m2.setAccessible(true);
    byte[] pdu = convertHex2Bin(msg.getBytes());
    m2.invoke(sm, null, pdu, null, null, false, false);

그리고, [HushSMS 의 리버스된 소스](http://pastebin.com/6uueFLCU)를 참조.

#### Payload 읽기

    public static byte[] payload() {
        FileInputStream fis;
        byte[] by = new byte[1024];
        int count;

        try {
            fis = new FileInputStream("/sdcard/payload");
            count = fis.read(by);
            fis.close();
            return by;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return by;
    }
