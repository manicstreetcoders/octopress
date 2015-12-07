---
layout: post
title: "The Knapsack Problem"
date: 2013-09-26 11:04
comments: true
categories: 
---

#### Problem 

도둑이 크기 W 의 자루를 들고 남의 집에 들어가, 자루에 물건을 담아 훔쳐가려고 한다.
집에 물건은 n 개가 있고, 각각 다른 크기를 가지고 있다. 편의상 물건들의 크기를 더해서
전체 크기를 계산할 수 있다고 가정할때, 어떻게 집어넣어야 값어치가 가장 크겠는가?

#### Problem Definition

Input: n items. Each has a value:

- value v_i (non-negative)
- size w_i (non-negative and integer)
- capacity W (a non-negative integer)

Output: a subset S of {1,2,3,...,n} that maximize sum v_i subject to sum w_i <= W

#### Developing a Dynamic Programming Algorithm

첫번째, 두번째, ..., n 번째 아이템이 있을때, 

m[i,w] 를 다음과 같이 정의한다.

1. 0 번째부터 i 번째까지의 아이템을 집어 넣을 수 있다.
2. 집어넣은 아이템 전체 weight 는 w 보다 작거나 같아야 한다.
3. 이때 취할 수 있는 최대 value 가 m[i,w] 라고 하자.

그러면 원래 문제는 m[n,W] 가 된다.

sub-problem 으로는 m[1,W] 를 생각해보자. 첫번째 아이템만만 사용할 수 있는 경우인데,
해답은 굉장히 간단하다. 첫번째 아이템의 크기가 W 보다 작다면 무조건 취하고, 크다면 넣지 못한다.
각각의 경우, 자루의 값어치는 v_1 또는 0 이 된다.

m[1,W] 에서 문제 크기를 조금 위로 올려서 sub-problem m[n-1,W] 를 생각해보자.
m[n-1,W] -> m[n,W] 로 확장될 때, n 번째 아이템을 취해야 하는가? 라는 문제가 되는데.
우선, n 번째 아이템이 너무 커서 고려할 가치가 없는 경우가 있을 수 있다.
`weight of n > W` 인 경우 => `m[n,W] = m[n-1,W]` 가 된다.
아니면, n 번째 아이템을 넣었을 경우와 뺐을 경우를 비교해서 큰 쪽을 선택하면 된다.
m[n-1,w] 가 뺐을 경우. m[n-1,w-w_n] + v_n 이 집어 넣었을 경우가 된다.

* m[i,w] = m[i-1,w] if w_i > w (the new item is more than the current weight limit)
* m[i,w] = max ( m[i-1,w] , m[i-1,w-w_i] + v_i) if w_i <= w

memoization 과 sub-problem 을 사용해서.

    for w from 0 to W do
        m[0,w] = 0
    end
    for i from 1 to n do // enlarge available items
        for j from 0 to W do
            if j >= w[i] then
                m[i,j] = max( m[i-1,j], m[i-1,j-w[i]] + v[i] )
            else
                m[i,j] = m[i-1,j]
            end
        end
    end

그런데 증명해야 할 부분이 있다.

> 아이템 n 이 최적해 S 에 들어있다고 가정하자. 그렇다면, S - {n} 은
> first (n-1) 아이템들과 capacity W - w_n 의 문제의 최적해라는 것을 증명해야 한다.
