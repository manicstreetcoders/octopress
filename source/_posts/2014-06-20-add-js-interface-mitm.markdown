---
layout: post
title: "add_js_interface_mitm"
date: 2014-06-20 09:13:56 +0900
comments: true
categories: 
---

Metasploit 에 들어간 add_js_interface_mitm.rb 를 훑어보았다.

일단 vulnerable 한 WebView 의 예제는 다음과 같다.

    public class MainActivity extends Activity {

        @SupressLint({ "SetJavaScriptEnabled", "JavascriptInterface" })
        @Override
            protected void onCreate(Bundle savedInstanceState) {
                ...
                WebView myWebView = (WebView) findViewById(R.id.webView1);

                // not a good idea!
                WebSettings webSettings = myWebView.getSettings();
                webSettings.setJavaScriptEnabled(true);

                // terrible idea!
                myWebView.addJavascriptInterface(new WebAppInterface(this), "Android");

                myWebView.loadUrl("http://droidsec.org/addjsif.html");
                ...

#### PoC

[Metasploit add_js_interface_mitm.rb 모듈](https://github.com/jduck/addjsif/blob/master/add_js_interface_mitm.rb)
