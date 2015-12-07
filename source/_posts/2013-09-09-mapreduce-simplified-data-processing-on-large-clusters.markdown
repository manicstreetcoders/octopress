---
layout: post
title: "MapReduce: Simplified Data Processing on Large Clusters"
date: 2013-09-09 17:19
comments: true
categories: 
---

Jeff Dean and Sanjay Ghemawat 의 논문 "MapReduce: Simplified Data Processing on Large Clusters" 를 읽고 있다.

LISP-like 언어들의 Map(), Reduce() 함수가 MapReduce 의 시작이었다.

### Problems

Map/Reduce 로 풀 수 있는 문제들을 제시한다. Scalable 하게 풀 수 있는 문제들.

* Distributed Grep - 패턴이 매치하면 그 라인을 emit. 
reduce 함수가 중간 데이터를 아웃풋에 출력.
* Count of URL Access Frequency - HTTP request log 를 프로세스해서 <URL,1> 을 emit.
reduce 함수가 같은 URL 에 대해서 값들을 summation. <URL, total count> 를 emit.
* Reverse Web-Link Graph
* Term-Vector per Host
* Inverted Index
* Distributed Sort

### Map() / Reduce()

JavaScript, Ruby, Python 으로 map() / reduce() 예제를 만들어본다.

JavaScript
    
    // map.js
    function map(fn, a) {
        for (var i = 0; i < a.length; i++) 
            a[i] = fn(a[i]);
    }

    var x = [1,2,3,4,5];

    map( function(n) { return n*2; }, x);
    map( function(n) { process.stdout.write(n.toString()+'\n'); return n; }, x);

    code% node map.js
    2
    4
    6
    8
    10

    // mapreduce.js
    function map(fn, a) {
        for (var i = 0; i < a.length; i++) 
            a[i] = fn(a[i]);
    }

    function reduce(apply, array, initial_value) {
        var prev = initial_value;
        for (var i = 0; i < array.length; i++)
            prev = apply(prev, array[i]);
        return prev;        
    }

    var x = [1,2,3,4,5];

    map( function(n) { return n*2; }, x);
    process.stdout.write( 
        reduce( function(x,y) { return x+y; }, x, 0 ).toString() + '\n' );

    code% node map.js
    30

Ruby 의 경우 map(), reduce() 가 있는데,

    1.9.3-p448 :001 > (1..10).map { |n| n*2 }
     => [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
    1.9.3-p448 :002 > (1..10).reduce(0) { |prev,n| prev + n }
     => 55
    1.9.3-p448 :003 > (1..10).reduce(0) { |prev,n| prev = prev + n }
     => 55
    1.9.3-p448 :004 >

reduce() 의 의미는, list 의 아이템을 하나의 value 로 축약해내는 것.

축약 과정은 ( 전의 값, list 의 아이템 ) 에 apply function 을 적용하여 새로운 값을 
계속해서 만들어나가는 것. 

요약하면 Apply funciton 으로 어레이를 단계별로 처리해나가서 하나의 값으로 증류해내는 것.

이제 ruby 의 map(), reduce() 를 구현해서 monkey patching 으로 붙여보자.

Python 의 경우에도 map(), reduce() 함수가 있는데,

    Python 2.7.2 (default, Oct 11 2012, 20:14:37)
    [GCC 4.2.1 Compatible Apple Clang 4.0 (tags/Apple/clang-418.0.60)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>> a = [ 1,2,3,4,5 ]
    >>> map ( lambda x: x* 2, a )
    [2, 4, 6, 8, 10]
    >>> reduce( lambda x,y: x+y, a)
    15

### Word Count Example

Map-Reduce v.s. shared-memory, low-level message passing

Web 에서 crawling 한 document 를 가지고 word count 을 map-reduce 로 기술해본다.

Input: 도큐먼트와 content (d_k, 'w_1 ... w_n')
Output: (w_k, word-count-toal)

- mapper: (d_k, 'w1 ... w_n') -> emits -> [ (w_i,c_i) ]
- reducer:  (w_i, [c_i]) -> (w_i, sum of all c_i)

문서는 여러개의 mapper 에 분배된다.
mapper 는 문서를 받아서 (key,value) 를 뱉어낸다.
그 문서에 있는 단어와 단어의 카운트. combine 을 할수도 안할수도 있다.

    ('abc', 4)
    ('cat', 2)
    ('abc', 8)
    ('dog', 1)
    ('gun', 7)
    ('dog', 2)
    ...

word count 예제에서는
reducer 는 mapper 들이 생성한 (key, value) 에 sum function 을 가지고 reduce 한다.
('abc', ...) 는 1st reducer 로 가고, ('cat', ...) 는 2nd reducer 로 가고...

octo.py 를 가지고 map-reduce 를 간단하게 실습해본다.

#### wc.py

    import glob

    files = glob.glob('*.txt')

    def fbuf(fname):
        f = open(fname)
        try:
            return f.read()
        finally:
            f.close()

    source = dict( (fname, fbuf(fname)) for fname in files)

    f = open('outfile','w')

    def final(k,v):
        print k,v
        f.write( str( (k,v) ) )

    def mapfn(k,v):
        for line in v.splitlines():
            for word in line.split():
                yield word.lower(), 1

    def reducefn(k,v):
        return k,len(v)


octo.py server 가 file contents 를 mapper 에게 분배하고,
mapper 는 (word, 1) 을 emit 한다.
reducer 는 ( 'alice' , [1,1,1,1,1,1,1,1] ) 를 reduce 한다.
