---
layout: post
title: "What is the 'new' keyword in JavaScript?"
date: 2012-10-14 20:45
comments: true
categories: 
---

스택오버플로우에서 좋은 질문을 찾았다. 

* [What is the 'new' keyword in JavaScript?](http://stackoverflow.com/questions/1646698/what-is-the-new-keyword-in-javascript)
* [Relation between \[\[Prototype\]\] and prototype in JavaScript
](http://stackoverflow.com/questions/383201/relation-between-prototype-and-prototype-in-javascript)

간단히 번역 & 정리해본다.

### JavaScript 의 상속

"constructor 함수 가지고 만들어진 Object 는, constructor 함수의 *prototype* 프라퍼티가 가리키는 Object 로부터 상속받는다."

### JavaScript 의 new 가 하는 일

JavaScript 의 new 가 하는 일은 3 가지이다.

1. 새로운 object 를 생성한다. 생성된 object 의 type 은 심플하게 그냥 *object* 이다.
2. 생성된 object 의 접근불가한 *[[Prototype]]* 프라퍼티를 constructor 함수의 접근가능한 외부용 *prototype* 이 가리키는 오브젝트로 세팅한다.
3. constructor 함수를 실행한다. (함수에서 `this` 가 나오면 새로 생성된 오브젝트를 사용)

예제를 가지고 알아보자.

     js> function C() { this.a = "babo" }
     js> C.prototype = { b: "babo2" }
     ({b:"babo2"})
     js> obj = new C()
     ({a:"babo"})
     js> obj.a
     "babo"
     js> obj.b
     "babo2"
     js> obj.c
     js> C.prototype.c = "babo3"
     "babo3"
     js> obj.c
     "babo3"
     js>

간단한 constructor 함수를 정의한다.
그리고 이 constructor 함수의 *prototype* 을 간단히 `{ b: "babo2" }` object 로 세팅한다.

`new C()` 하는 순간 

1. 새로운 object 가 생성되고. *obj* 라고 칭하면.
2. `obj.[[Prototype]] = C.prototype` 이 실행되고.
3. `{ obj.a = 'babo' }` 가 실행된다.

풀어서 따라가보면

1. `obj.[[Prototype]]` 은 `{ b: "babo2" }` object 를 가리키고, 
constructor 가 수행되면서 `obj.a` 가 `"babo"` 로 세팅되었다.
2. 그래서 `obj.a` 와 `obj.b` 가 각각 `"babo"`, `"babo2"` 로 evaluated 된다.
3. 그 다음에 `C.prototype.c = "babo3"` 로 `{ b: "babo2" }` 에 `c`를 추가한다.
4. 그러면 `obj.[[Prototype]]` 은 이제 `{ b: "babo2", c: "babo3" }` object 를 가리키게 된다.
5. 그래서 `obj.c` 가 "babo3" 로 evaluated 된다.

정리하면

* function 만이 *prototype* 프라퍼티를 갖는다.
* 모든 object 는 숨겨진 *[[Prototype]]* 프라퍼티를 갖는다.
* Browser implementation 에 따라 *[[Prototype]]* 이 *__proto__* 인 경우도 있다.
* function 도 *[[Prototype]]* 프라퍼티를 갖는다. ("Functions are first-class objects in JavaScript.")
* 따라서 function 만 *[[Prototype]]* 과 *prototype* 을 모두 가진다.
* 그외의 object 는 *[[Prototype]]* 만 가진다.
* object 의 *[[Prototype]]* 은 new 키워드를 통해서만 세팅 가능하다. (브라우저 임플리멘테이션에 따라서 다를 수 있지만)

### Crockford 책에 나오는 beget()

이제, Douglas Crockford 책에 나오는 beget() 를 보도록 하자.

     if (typeof Object.beget !== 'function') {
       Object.beget = function (o) {
         var F = function () {};
         F.prototype = o;
         return new F();
       };
     }
     var another_stooge = Object.beget(stooge);

상속을 위한 Constructor 함수를 대신 만들어서 오브젝트를 생성해준다. 

### Eloquent JavaScript 에 나오는 예제

### Further reading:

* [Prototype.js 의 class 와 상속에 관한 아티클](http://prototypejs.org/learn/class-inheritance)
