---
layout: post
title: "Relatively Prime Probability"
date: 2015-03-08 14:38:05 +0900
comments: true
categories: 
---

### 코드로 돌려보기

(a,b) 가 서로소일 확률은 어떻게 될까? 일단 프로그램으로 돌려보니 대략 60% ~ 61% 가 나온다. 1 은 계산에서 제외하였다.

```
def gcd(a,b):
    while b:
        a,b = b,a % b
    return a

if __name__ == "__main__":
    size = 0
    coprimes = 0
    for i in range(2,1000):
        for j in range(i+1,1000+1):
            size += 1
            if gcd(i,j) == 1:
                coprimes += 1
    print "%d/%d = %f" % (coprimes,size, float(coprimes)/float(size))            
```

### 확률로 따져보기

a,b 가 2 를 factor 로 가지고 있을 확률은 얼마나 될까? a 가 2 의 배수일 확률은 1/2 이다. b 역시 2 의 배수일 확률은 1/2 이다. a,b 가 동시에 2 의 배수일 확률은 1/2 * 1/2 이다.

그럼 a 가 3 을 factor 로 가지고 있을 확률은 얼마나 될까? 이건 1/3 이다. b 역시 3 을 factor 로 가지고 있을 확률은 1/3. a,b 가 동시에 3 을 팩터로 가지고 있을 확률은 1/3 * 1/3 이다.

이런 식으로 생각하면 (a,b) 가 prime p 를 coprime 으로 가지고 있을 확률은 1/(p^2) 이다.

위의 코드에서는 1000 이하에서 (a,b) 를 골랐을때 서로소일 확률을 구했었다. 그렇다면, 1000 을 무한으로 보냈을때, 서로소일 확률을 구하려면, 모든 prime 에 대해서 확률을 구해줘야한다. 
