---
layout: post
title: "AdSense: Inverse Search"
date: 2013-09-10 18:08
comments: true
categories: 
---

search 는 주어진 키워드와 가장 relevant 한 document 를 찾아 사용자에게 제시한다.
AdSense 는 역으로 주어진 document 에 대해, 가장 relevant 한 키워드를 찾아내, 그 키워드에
비딩한 광고를 실어주어야 한다.

### TF-IDF

* rarer words make better keywords

IDF = inverse document frequency of word w = log_2 (N/N_w)

(N total documents, with N_w containing w)

* more frequent words make better keywords

TF (Term Frequency) = frequency of w in document d 

TF-IDF (w) = Term Frequency (w) * Inverse Document Frequency (w)

'the' 처럼 의미없는 말이 문서에 자주 반복되어봐야 별 가치 없으니,
어느정도 rarer 한 단어가 어느 정도 자주나오는가? 라는 의미로 보면 될 것.

["An information-theoretic perspective of TF-IDF measures", Kiko Aizawa, Journal of Information Processing and Management, Vol. 39 (1), 2003](http://comminfo.rutgers.edu/~muresan/IR/Docs/Articles/ipmAizawa2003.pdf)

