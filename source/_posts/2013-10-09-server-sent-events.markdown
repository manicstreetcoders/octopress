---
layout: post
title: "Server Sent Events"
date: 2013-10-09 09:52
comments: true
categories: 
---

Polling, Long Polling, Push, SSE, WebSocket 의 정확한 차이점을 알 필요가 생겼다.

### Polling

AJAX 로 HTTP query 를 주기적으로 날리는 식이다. AJAX 와 다른 특별한 기술이라고 하기는 그렇다.
새로운 Data 가 없을 경우 헛 쿼리를 하게되니, bandwidth 낭비가 된다.

### Long Polling (Hanging GET/COMET)

보낼 데이타가 없으면, 서버가 HTTP request 를 잡고 있다가, data 가 available 할 때, 
HTTP response 를 보내준다. 서버가 잡고 있기 때문에 Hanging GET 이라고도 칭한다.

[Alex Russel 의 블로그를 참조](http://infrequently.org/2006/03/comet-low-latency-data-for-the-browser/) 

### Server Sent Events

[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Server-sent_events/Using_server-sent_events) 를 보면 SSE 에 대한 좋은 예제가 있다.

서버에서 MIME Type `text/event-stream` 으로, 내용은 JSON 으로 보내되, `event:`, `data:` 로 구분해서 보내고, 클라이언트에서는 `EventSource` 인터페이스를 사용한다. 보다 자세한 설명은 [HTML5Rocks](http://www.html5rocks.com/en/tutorials/eventsource/basics/)로.

sse.html

    <html>
    <head></head>
    <body> 
        <ul id="evt">
        </ul>
    <script>
        var source = new EventSource("sse.php");
        source.addEventListener("ping", function(e) {
            var elem = document.createElement("li");
            var obj = JSON.parse(e.data);
            elem.innerHTML = "ping at " + obj.time;
            document.getElementById("evt").appendChild(elem);
        }, false);
    </script>
    </body>
    </html>

sse.php

    <?php
    date_default_timezone_set("Asia/Seoul");
    header("Content-Type: text/event-stream\n\n");

    while (1) {
        echo "event: ping\n";
        $curDate = date(DATE_RFC2822);
        echo 'data: {"time": "'. $curDate . '"}';
        echo "\n\n";
        ob_flush();
        flush();
        $r = rand(1,3)
        sleep($r);
    }
    ?>

브라우저에서 접속해보면, 

{% img /images/ssedemo.png %}

[HTML5Rocks](http://www.html5rocks.com/en/tutorials/eventsource/basics/) 의 nodejs 예제.

    var http = require('http');
    var sys = require('sys');
    var fs = require('fs');

    http.createServer(function(req,res) {
        if (req.headers.accept && req.headers.accept == 'text/event-stream') {
            if (req.url == '/events') {
                sendSSE(req,res);
            } else {
                res.writeHead(404);
                res.end();
            }
        } else {
            res.writeHead(200, { 'Content-Type': 'text/html' });
            res.write(fs.readFileSync(__dirname+'/sse.html'));
            res.end();
        }
    }).listen(8000);

    function sendSSE(req,res) {
        res.writeHead(200, {
            'Content-Type': 'text/event-stream',
            'Cache-Control': 'no-cache',
            'Connection': 'keep-alive'
        });
        var id = (new Date()).toLocaleTimeString();
        setInterval(function() { 
            constructSSE(res, id, (new Date()).toLocaleTimeString() );
        }, 5000);
        constructSSE(res, id, (new Date()).toLocaleTimeString() );
    }

    function constructSSE(res, id, data) {
        res.write('event: ping' + '\n');
        //res.write('id: ' + id + '\n');
        res.write('data: ' + '{ "time": "' + data + '"}' + '\n\n');
    }

sse.html

    <!DOCTYPE html>
    <html>
    <head></head>
    <body>
        <ul id="evt">
        </ul>
        <script>
            var source = new EventSource('/events');
            source.addEventListener("ping", function(e) {
                var elem = document.createElement("li");
                var obj = JSON.parse(e.data);
                elem.innerHTML = "ping at " + obj.time;
                document.getElementById("evt").appendChild(elem);
            }, false);
        </script>
    </body>
    </html>

HTTP connection 을 유지하면서, 서버가 random 딜레이를 두고 event 를 쏴주는데,
브라우저가 이벤트를 처리하게 된다.

[WebSocket 과 SSE 간에 큰 차이점](http://stackoverflow.com/questions/5195452/websockets-vs-server-sent-events-eventsource)은 다음과 같다.

1. WebSocket 은 Full-Duplex 채널 v.s. SSE 는 Server -> Browser
2. SSE 는 HTTP 로 구현
3. Internet Explorer 는 SSE 를 지원하지 않음

WebSocket 이나 SSE 는 브라우저 호환성때문에, 현실에서는 (i.e. Facebook) long polling 을 주로 사용하는 모양이다.
