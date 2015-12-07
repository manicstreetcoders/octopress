---
layout: post
title: "JavaScript Method Invocation Patterns"
date: 2013-09-11 12:14
comments: true
categories: 
---

JavaScript: The Good Parts 에서 funciton 에 관한 내용을 발췌/정리한다.

자바스크립에는 function 호출에 4 가지 패턴이 있다.

### The Method Invocation Pattern

function 이 Object 의 property 로 저장되어있을때.

    var myObject = { 
        value: 0, 
        increment: function(inc) { 
            this.value += typeof inc === 'number' ? inc : 1; 
        }
    }

JavaScript 에서 `this` 의 binding 은 invocation time 에 이루어진다. late binding.

이 경우 `this` 는`myObject` 에 바인딩된다.

### The Function Invocation Pattern

Object 의 property 가 아니면 method 가 아니라 function.

    var sum = add(3, 4)

this 는 global object 에 binding 됨. Web Browser 에서 돌고 있다면,

DOM object 인 `window` object 에 binding 될 것.

    myObject.double = function() {
        var add = function(a,b) { return a+b; };
        var helper = function() { 
            this.value = add(this.value, this.value);
        };
        helper();
    };

란 코드를 보자. inner function 의 this 는 어디에 binding 이 될까?

테스트를 해보자.

    <html>
    <head></head>
    <body>
    <script>
    var myObject = {
        value: 0,
        increment: function(inc) {
            this.value += typeof inc === 'number' ? inc : 1;
        }
    };

    myObject.double = function() {
        var add = function(a,b) { return a+b; };
        var helper = function() {
            if (this === window) console.log('this === window');
            this.value = add(this.value, this.value);
        };
        helper();
    };

    console.log(myObject.value);
    myObject.increment();
    console.log(myObject.value);
    myObject.double();
    console.log(myObject.value);
    </script>
    </body>
    </html>

    % python -m SimpleHTTPServer 8888

이제 브라우저로 들어가 localhost:8888 을 로드하고, Console 을 띄운다.

콘솔에 나와있는 메시지들.

    0
    1
    this === window
    1

inner function 의 `this` 가 `myObject` 가 아닌, DOM object 인 `window` 에 바인딩되었음을
알 수 있다.


### The Constructor Invocation Pattern

constructor 를 정의하고, accessor 를 정의하였다.

JavaScript 에서 constructor 는 new 키워드와 함께 사용된다.

`this` 는 메시지를 받는 object 에 binding 된다.

    var Document = function ( name ) {
        if (typeof name === "string")
            this.title = name;
        else
            this.title = "<empty>";
    }

    Document.prototype.getTitle = function () {
        return this.title;
    }

    var myDoc = new Document("confused");
    console.log( myDoc.getTitle() );

    % node constructor.js
    confused
    %

### The Apply Invocation Pattern

JavaScript 은 functional OO language 이기 때문에, function 이 method 를 가질 수 있다. 

function object 에 `apply` method 를 날려서, 함수를 실행시킬 수 있다.

이 경우, arg0 에 binding 할 `this` 를 지정해줄 수 있다.

    var add = function (a,b) {
        return a+b;
    };

    var a = [ 1, 2 ];

    var sum = add.apply(this, a);

    console.log(sum);

    % node apply.js
    3
    %

이상 4 개의 호출 패턴 정리 끝.
