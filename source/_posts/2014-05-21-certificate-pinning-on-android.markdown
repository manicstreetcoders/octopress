---
layout: post
title: "Certificate Pinning on Android"
date: 2014-05-21 13:03:01 +0900
comments: true
categories: 
---

[BlackHat 2012 에서 발표된 Certificate Pinning 에 관련된 발표](https://media.blackhat.com/bh-us-12/Turbo/Diquet/BH_US_12_Diqut_Osborne_Mobile_Certificate_Pinning_Slides.pdf).

    private InputStream makeRequest(Context context, URL url) {
        // Load custom Trust Store from a file
        AssetManager assetManager = context.getAssets();
        InputStream keyStoreInputStream = assetManager.open("yourapp.store");
        KeyStore trustStore = KeyStore.getInstance("BKS");
        trustStore.load(keyStoreInputStream, "somepass".toCharArray());

        TrustManagerFactory tmf = TrustManagerFactory.getInstance("X509");
        tmf.init(trustStore);

        // Init an sslContext with the custom Trust Store
        SSLContext sslContext = SSLContext.getInstance("TLS");
        sslContext.init(null, tmf.getTrustManagers(), null);

        // Use this sslContext for subsequent HTTPS connections
        HttpsURLConnection urlConnection = (HttpsURLConnection)url.openConnection();
        urlConnection.setSSLSocketFactory(sslContext.getSocketFactory());

        return urlConnection.getInputStream();
    }

서버의 certificate 을 확인.

    openssl s_client -connect www.babo.com:443 -tlsextdebug


그럼 왜 Certificate Pinning 을 하는가. PKI 의 신뢰 체인이 한계에 다달았기 때문.

간단하게

* Root CA 가 너무 많다.
* Root CA 를 믿을 수 없다.

Browser 같이 불특정 다수의 웹 사이트를 상대해야 하는 경우라면 어쩔 수 없지만,
모바일 전용 앱과 같은 경우 한두개의 서버만 상대하면 되기때문에, Certificate Pinning 을 사용하는 것이 더 안전하다.

* [Certificate pinning in Android 4.2](http://nelenkov.blogspot.kr/2012/12/certificate-pinning-in-android-42.html)
* [Public key pinning (04 May 2011)](https://www.imperialviolet.org/2011/05/04/pinning.html)
* [Certificate Pinning for Mobile Applications](http://h30499.www3.hp.com/t5/Fortify-Application-Security/Certificate-Pinning-for-Mobile-Applications/ba-p/6223897#.U6OGK5S1bOM)

Certificate Pinning 이 적용된 App 은 리버싱하기가 불편하기 때문에, disable 시키는 kill-switch patch 를 적용한다.

* [github.com/iSECPartners/ios-ssl-kill-switch](https://github.com/iSECPartners/ios-ssl-kill-switch)
