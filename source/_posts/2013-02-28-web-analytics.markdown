---
layout: post
title: "Web Analytics"
date: 2013-02-28 17:06
comments: true
categories: 
---

### Visits

Web Analytics 에서 쓰는 Visits 는 session 과 많은 경우에 동일하다.
Session 의 정의를 하자면 "a collection of requests from someone who is on your website" 라고 할 수 있다.

우선 세션에 대해서 알아보자.

JavaScript tag 기반의 web analytics 솔루션을 사용한다고 가정하자.

1. 웹사이트의 페이지를 처음으로 요청할때, 그 브라우저에 대해서 session 을 만든다.
2. 그 브라우저에서 오는 계속된 요청은 unique session ID 와 묶이게 된다.
3. 그 사람이 사이트를 떠나면, 그간의 요청된 page view 들을 unique session ID 로 묶어서 하나의 세션으로 처리.
4. Total Visits 는 주어진 기간동안의 모든 세션의 갯수들의 합.
5. 29 분의 inactivity 가 있으면 세션을 끝내버린다.

이 세션을 Visits 라고 흔히 말한다.

### Unique Visitors

그렇다면 Unique Visitors 란?

역시 JavaScript tag 기반의 web analytics 솔루션을 사용한다고 가정.

1. 웹사이트의 페이지를 처음으로 요청할때, analytics tool 은 브라우저에 unique 한 쿠키를 세팅한다.
2. 웹사이트를 떠나도 그 unique 한 쿠키는 살아있는다.
3. 그 브라우저를 가지고 웹 사이트를 방문할 때마다, persistent unique cookie ID 를 가지고 동일한 브라우저인지를 판별한다.
4. Unique Visitors 는 주어진 기간동안의 "the count of all the persistent unique cookie IDs" 라고 할 수 있다.

### Metric 의 유행

Hits -> Page Views -> Visitors 로 중요 메트릭이 변해가다가...

Bounce Rate, Exit Rate, Conversion Rate, Engagement 같은 메트릭이 중요하게 여겨지고 있다.

