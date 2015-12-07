---
layout: post
title: "Extended Euclidean Algorithm"
date: 2014-03-22 09:37:42 +0900
comments: true
categories: 
---

gcd(x,y) 를 찾는 Euclid 알고리즘.

    exports.map = function (fn,a) { 
        for (var i=0; i < a.length; i++)
            a[i] = fn(a[i]);
    };

    exports.reduce = function (fn, a) {
        var prev = 0;
        for (var i=0;i<a.length;i++)
            prev = fn(prev,a[i]);    
        return prev;
    };

    exports.gcd = function (a,b) {
        var t;
        if (b > a) {
            t = a; a = b; b = t;
        }
        if (b == 0)
            return a;
        return exports.gcd(b, a % b)
    };

    % node
    > var x = require('./reduce.js')
    undefined
    > x.reduce(x.gcd, [487, 28, 100])
    1
    > x.reduce(x.gcd, [84, 28, 100])
    4
    > x.reduce(x.gcd, [48, 24, 36])
    12
    >

Least positive linear combination of a and b 를 찾아보자.

    % cat least_positive.py
    from fractions import gcd

    min = 999999999999999

    def search(x,y):
        global min
        g = gcd(x,y)

        for i in range(-100,101):
            i = -1 * i
            for j in range(-100,101):
                j = -1 * j
                d = i * x + j * y
                if d <= 0:
                    continue
                if d < min:
                    min = d
                    print "New minimum: %d, gcd: %d" % (min,g)

        print "Minimum: %d, gcd: %d" % (min,g)
        print "done"

    search(100,40)

    % python least_positive.py
    ...
    New minimum: 500, gcd: 20
    New minimum: 480, gcd: 20
    New minimum: 440, gcd: 20
    New minimum: 400, gcd: 20
    New minimum: 380, gcd: 20
    New minimum: 340, gcd: 20
    New minimum: 300, gcd: 20
    New minimum: 280, gcd: 20
    New minimum: 240, gcd: 20
    New minimum: 200, gcd: 20
    New minimum: 180, gcd: 20
    New minimum: 140, gcd: 20
    New minimum: 100, gcd: 20
    New minimum: 80, gcd: 20
    New minimum: 40, gcd: 20
    New minimum: 20, gcd: 20
    Minimum: 20, gcd: 20
    done

Theorem: If a and b are nonzero integers, then their gcd is a linear combination 
of a and b, that is there exist integer numbers s and t such that

sa + tb = (a,b).

Proof: Let d be the least positive integer that is a linear combination of a and b. We write

d = sa + tb, 

where s and t are integers.

We first show that d | a. By the Division Algorithm we have

a = dq + r, where 0 <= r < d.

r = a - dq = a - q(sa + tb) = a - qsa - qtb = (1 - qs)a + (-qt)b

This shows that r is a linear combination of a and b.
Since 0<= r < d, and d is the least positive linear combination of a and b,
we conclude that r = 0, and hence d | a.

이제, a*x + b*y = gcd(x,y) 를 만족하는 a,b 를 찾는 Extended Euclid 알고리즘.
