---
layout: post
title: "Type Casting Verification: Stopping an Emerging Attack Vector"
date: 2015-08-20 17:23:58 +0900
comments: true
categories: 
---

USENIX 2015 에 발표된, Type Casting Verification: Stopping an Emerging Attack Vector.

#### C++ Bad-casting Demystified

논문에 나온 코드. Downcasting 을 잘못해서 memory 에러가 나오는 경우를 예로 든다.

```
class SVGElement: public Element { ... };

Element *pDom = new Element();
SVGElement *pCanvas = new SVGElement();

// valid upcast
Element *pEle = static_cast<Element*>(pCanvas);

// valid downcast
SVGElement *pCanvasAgain = static_cast<SVGElement*>(pEle);

// invlaid downcast
SVGElement *p = static_cast<SVGElement*>(pDom);

// leads to memory corruption
p->m_className = "my-canvas";

// invalid downcast with dynamic_cast, but no corruption
SVGElement *p = dynamic_cast<SVGElement*>(pDom);
if (p) {
    p->m_className = "my-canvas";
}
```

#### Bad-casting: CVE-2013-0912

논문에서는 Pwn2Own 2013 에서 Chrome 을 깨는데 사용되었던 CVE-2013-0912 를 예로.

* HTML5 에서, SVG 이미지는 svg 태그를 사용해 HTML 에 Embed 될 수 있다.
* svg 는 SVGElement class 로 구현된다.
* SVGElement 는 Element 의 child class 이다.
* Unknown tag 는 HTMLUnknownElement 로 구현.
* 만약, HTMLUnknownElement 가 Element 로 upcasting 된다음에, SVGElement 로 downcasting 된다면? (SVG rendering 을 위해서)
* SVGElement 오브젝 사이즈는 160 bytes > HTMLUnknownElement 오브젝트의 사이즈는 96 bytes.
* HTMLUnknownElement 로 alloc 받아서, SVGElement 로 casting 해서 쓰면, 97~160 bytes 를 더 addressing 할 수 있다.
* 이걸 활용해서, 가까이 있는 오브젝의 vtable 포인터를 오염시켜서 control-flow 하이재킹을 수행.
* HTMLUnknownElement 는 sibling 이 56 개 이상. Element class 는 parent 가 10 개 이상.
