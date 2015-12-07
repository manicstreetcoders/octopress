---
layout: post
title: "Paul Irish: 10 Things I Learned from the jQuery Source"
date: 2012-09-18 11:01
comments: true
categories: 
---

Paul Irish 의 [10 Things I Learned from the jQuery Source](http://vimeo.com/12529436#at=0) 에서.

<h3>Self-invoking anonymous function</h3>

     (function( window, undefined ) {
     })(window);

     (function () {
     })()

     !function () {
     }()

     +function () {
     }()

Everyone's favorite pattern:

     (function(window,document,undefined){
       // scope traversal benefit. document 를 refer 하면, 
       // local scope 만 찾아보면 되니까 약간의 성능 향상이 있을 수도.
       document.getElementById('') 
     })(this,this.document);

아규먼트의 끝에 undefined 를 붙이는 이유는, undefined 변수를 reset 하기 위함. 
"undefined = true" 와 같은 코드를 누군가 짰다면 문제가 생길수도 있음.

미니파이를 해서

     (function(A,B,C){
       // document.getElementById('whatever_id')
       B.getElementById('whatever_id')
     })(this,document)

<h3>Asynchronous Recursion</h3>

     setInterval(function(){
     },100);

     (function(){
       doStuff();
       setTimeout(arguments.callee,100)
     })()

     (function loopsiloopsiloo(){
       doStuff();
       setTimeout(loopsiloopsiloo,100)
     })()

     (function loopsiloopsiloo(){
       doStuff();
       $('#update').load('awesomething.php'',function(){
         loopsiloopsiloo();
       })
     })();

위의 예제에서처럼, setInterval 을 사용하면, 
function(){} 수행에 100 milliseconds 이상이 걸릴 경우에, 
문제가 될 수 있다.
이 경우에는 setTimeout(arguments.callee,100) 을 사용해야 한다.

