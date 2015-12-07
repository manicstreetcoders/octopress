---
layout: post
title: "JavaScript: The Good Parts"
date: 2012-05-10 19:27
comments: true
categories: 
---

> JavaScript has some extraordinarily good parts. 
> In JavaScript, there is a beautiful, elegant, highly expressive language
> that is buried under a steaming pile of good intentions and blunders.

> ...

> JavaScript's C-like syntax, including curly braces and the clunky 
> 'for' statement, makes it appear it to be an ordinary procedural language.
> This is misleading because JavaScript has more in common with functional
> language like Lisp or Scheme than with C or Java.

Doug Crockford 는 그의 책 
[Doug Crockford: JavaScript: The Good Parts](http://googlecode.blogspot.com/2009/03/doug-crockford-javascript-good-parts.html)
에서 JavaScript 을 그것이 가진 나쁜 면들때문에 싸잡아서 그저 그런 언어로 치부해서는 안된다고 주장한다.

또한 C 와 비슷한 문법과 '자바'라는 이름때문에 오해를 받고 있으며, 오히려 Lisp 의 방언(dialect) 이라고 말할 수 있다고 말한다. 

브라우저에 탑재되는 언어의 자리를 놓고 많은 경쟁자들과 싸워서 이긴 언어이며, 그 이유는 JavaScript 에는 놀라울만큼 훌륭한 점들이 많기 때문이라고 이야기한다.

<br />
-----

<br />
얼마전에 twitter bootstrap 코드의 세미콜론을 두고 개발자와 Doug Crockford 간에 논쟁이 벌어졌었다. 

개발자는 JavaScript 의 "생략가능한 세미콜론" 기능을 이용해서 코드를 짰는데, 이 코드가 JSLint 에서 걸린 것이 사건의 발단이었다.

감정적인 말이 오가고, 서로 상대방의 코드를 고치라고 뻐대는 형국이 되버렸다. 

결국 어떻게 마무리가 되었는지는 잘 모르겠지만, Crockford 가 깐깐함때문에 Cranky old-man 으로 불린다는 것. 

그럼에도 개발자들로부터 존경을 받고 있다는 것. 이 두가지는 확인할 수 있었다.

<br />
아래 코드는 C 로 된 이진수 출력 루틴의 일부인데

    b[32] = '\0';
    for (i = 32, j = (1L << 31); i > 0; i--, j >>= 1)
      *p++ = (x & j) == j ? '1' : '0';

요즘 세상에 C 는 어셈블리가 된 것 같아서, 이런 코드를 짜고 있으면,  JavaScript 의 "헐거운 타이핑" (loose typing) 이 그리워지기 마련이다.

Crockford 는 JavaScript 의 좋은 점으로 lambda 와 "헐거운 타이핑" (loose typing), prototypal inheritance, dynamic object 를 들고 있다.

