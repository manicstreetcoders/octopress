---
layout: post
title: "Job Scheduling Algorithm"
date: 2013-09-06 11:02
comments: true
categories: 
---

- Job 1,2,3,... 가 있고,
- 각각의 weight 가 w1,w2,w3,...
- 각각의 length 가 l1,l2,l3,...
- Job completion time 이 c1,c2,c3,...

가 있을때, w_i * c_i 의 합을 최소화하는 문제.

w_i/l_i 의 decreasing order 로 job 을 스케쥴링하는 알고리즘을 디자인하고, 옵티멀하다는 것을 증명
 
assumptions

1. w_i/l_i 가 같은 Job 은 없다고 가정
2. w_i/l_i 의 desc order 로 정렬하고, 각각을 Job 1,2,3,... 로 명명
3. 약속에 따라 w_i/l_i > w_j/l_j if i > j
4. w_1/l_1 > w_2/l_2 > w_3/l_3 ...

proof by contradiction 으로 증명.
