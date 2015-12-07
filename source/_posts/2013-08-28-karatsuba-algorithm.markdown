---
layout: post
title: "Karatsuba Algorithm"
date: 2013-08-28 02:19
comments: true
categories: 
---

### Karatsuba Algorithm 

Karatsuba 가 23 살때 발명한 알고리즘. 

    def kara(x,y):
        n = len(str(x))
        n2 = pow(10,n/2)
        a = x / n2
        b = x % n2
        c = y / n2
        d = y % n2
        ans = pow(10,n)*a*c + pow(10,n/2)*(a*d+b*c) + b*d

        return ans
        
    if __name__ == "__main__":
        x = 4586
        y = 1234
        ans = kara(x,y)
        print "%d == %d" % (ans, x*y)

여기서 n-digit 숫자간의 곱셈을 n/2-digit 숫자간의 곱셈으로 divide 할 수 있다.
recursive 하게 적용하면, n-digit by n-digit 을 곱하는 일반적인 곱셈의 복잡도 Theta(N^2) 
를 Theta(N^\log_2 3) 으로 낮출 수 있다.
