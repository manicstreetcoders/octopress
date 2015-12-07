---
layout: post
title: "Naive Bayesian Classifier"
date: 2013-09-27 16:29
comments: true
categories: 
---

#### Conditional Probability

일반적인 조건부 확률 예제를 들어본다.

1. 주머니에 두개의 동전이 들어있다.
2. 여기서 동전을 하나 꺼내 동전 던지기를 한다.
3. 단 한개는 평범한 동전. 한개는 조작되어서 던지면 무조건 Head 가 나오는 동전.
4. 주머니에서 동전을 하나 꺼내어 던졌더니 Head 가 나왔을때,
5. 평범한 동전을 꺼냈을 확률은?

결과 트리를 그려보면, 

1. 일단계 outcome 은 평범동전 / 조작동전
2. 이단계 outcome 은 H / T / H / H 

P(평범|H) = one H / three H = 1/3 = P(평범 and H) / P(H) = (1/4) / (3/4)

#### 검색 키워드에 따라 결과 페이지에 광고를 집어 넣기

검색 엔진에 들어오는 쿼리를

* informational query
* transactional query

2 가지로 구분할 수 있다고 하자.

`transactional query` 라고 판단되면 광고를 옆에 표시해주기로 하자.

쉬운 방법은 매 query 마다 과거 기록을 검색해보면 된다.

`cheap flowers` 라고 유저가 치면 `cheap flowers` 를 과거 DB 에서 검색해서,

1. 구매관련 링크로 튀었는지 (유저가 클릭)
2. 정보성 링크로 튀었는지

검색한 후에, 구매관련 링크로 튄 경우가 많았다면 꽃가게 광고를 옆에 표시해준다.
그런데 이 방법은 response 시간도 오래 걸리고 부하가 심해서 feasible 하지 않다.

`cheap flowers`, `red flowers`, `gift flowers` 만 놓고 봐도, 

`cheap flowers` 는 transactional query 일 것이고 `red flowers` 는 informational query 일 듯.

조건부 확률을 써서 문제를 정의해보면, P(구매링크를 클릭| `cheap flowers` 를 검색) 의 확률이다.

1. 우리 DB 에 10 여년간의 수많은 검색 -> 클릭 데이터가 있다.
2. 정보성 블로그 링크인지 스폰서 광고 링크인지 link property 에 담겨져 있다. (링크 등록시)

#### Naive Bayesian Classifier

keywords: flower, red, gift, cheap

* should ads be shown or not?
* are you a surfer or a shopper?

machine learning is all about learning from past data.

* past behavior of many many searchers using these keywords

전체 n 개의 query instance 가 있었다고 하자. sample set.

red 라는 단어를 query 한 경우가 r 케이스.

구매로 연결된 query 는 k 케이스.

p(R and B) = i 케이스.

라고 하면.

red 라는 단어를 query 했을 경우 구매 이벤트가 일어날 조건부 확률 p(B|R) = i/r 

Red / Cheap 이라는 2 개의 단어의 경우로 넘어가자.

"naive" Bayesian classifier:

Assumption: All the futures that one uses to learn about historical data are 
independent for a particular choice of the behavior one is trying to learn.

i.e. if one is trying to predict whether a given user is shopping or just
surfing using the query words that they use to search. We assume that the
words themselves are all independent and equally likely of occur independent of 
what what other words are used in the query.

Assumption - R and C are independent given B.

P(B|R,C) * P(R,C) = P(R,C|B) * P(B) (Bayes rule)
= P(R|C,B) * P(C|B) * P(B) (Bayes rule)
= P(R|B) * P(C|B) & P(B) (independence)

so, given values r and c for R and C

compute:

p(r|B=y)*p(c|B=y)*p(B=y) / p(r|B=n)*p(c|B=n)*p(B=n)

choose B=y if this is > alpha (usually 1), and B=n otherwise

