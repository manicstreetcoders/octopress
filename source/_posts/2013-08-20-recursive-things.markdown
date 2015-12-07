---
layout: post
title: "Recursive Things"
date: 2013-08-20 12:15
comments: true
categories: 
---

"알고리즘 문제 해결 전략" 이라는 책을 읽고 있다. algospot.com 의 운영자가 쓴 책이다.

### Combination

1 ~ 9 사이의 숫자를 4 개 뽑는 combination 구하기.

    def pick(selection, last):
        if len(selection) == 4:
            print selection
            return
        for i in range(last+1,9):
            selection.append(i)
            pick(selection, i)
            selection.remove(i)

    if __name__ == "__main__":
        selection = []
        pick(selection, 0)

### 음식 대접

음식을 가리는 4 명의 사람들에게 6 가지의 메뉴 중, 최소한의 메뉴를 준비해서 대접하는 문제.

    answer = []

    def OK(menu):
        bit = 0
        if 1 in menu:
            bit = bit | 0b1100
        if 2 in menu:
            bit = bit | 0b1001
        if 3 in menu:
            bit = bit | 0b0101
        if 4 in menu:
            bit = bit | 0b0001
        if 5 in menu:
            bit = bit | 0b0110
        if 6 in menu:
            bit = bit | 0b1010

        if bit == 0b1111:
            return True
        else:
            return False

    def Explore(menu, depth):
        if depth == 7:
            if OK(menu):
                answer.append( (len(menu), list(menu)) )
                return len(menu)
            else:
                return 987654321
        menu.append(depth)
        size = Explore(menu, depth + 1)
        menu.pop()
        size2 = Explore(menu, depth + 1)
        if size < size2:
            return size
        else:
            return size2

    if __name__ == "__main__":
        menu = []
        min = Explore(menu,1)
        for (x,y) in answer:
            if x == min:
                print y

### 구간 max

Recursive algorithm 은 아니지만.

{ 5, -2, 4, -18, 8, 10, -7, 5, 8, 4, 3, -7, -2, 5, 8, -28, 9, 10, -12, 13, 5 } 라는 어레이에서 summation_from_i_to_j 가 가장 큰 index i, j 를 찾는 문제. 무식하지 않게 하는 것이 포인트.

우선 O(N^2) 으로 하는 방법은 간단하다.

    #include <stdio.h>
    #include <stdlib.h>

    main()
    {
        int a[] = { 5, -2, 4, -18, 8, 10, -7, 5, 8, 4, 3, -7, -2, 5, 8, 0, 9, 10, -28, 13, 5 };
        int N = sizeof a / sizeof (int);

        int i,j;
        int max = 1 << (sizeof(int) * 8 - 1);
        int sum;
        int _i,_j;

        for (i = 0; i < N; i++) {
            sum = 0;
            for (j = i; j < N; j++) {
                sum += a[j];
                if (sum > max) {
                    max = sum;
                    _i = i;
                    _j = j;
                }
            }
        }
        printf("sum(%d, %d) = %d\n", _i, _j, max);
    }

O(N) 알고리즘.

1. 과거를 잊고 새로 summation 을 시작해야 할지를 계속 결정해야 한다.
2. 과거가 minus 라면 싹 잊고 새로 시작하는 것이 낫다.
3. 과거가 그래도 plus 라면 구간합을 올리는데 도움은 된다.
4. 음수인 구간합은 의미가 없다. 0 이 낫다.
5. i 까지의 최대구간합은 maxAt(i) = max( 0, maxAt(i-1) ) + a[i]

등을 가지고 만들어본다.

    #include <stdio.h>
    #include <stdlib.h>

    #define MAX(a,b) (a > b ? a : b)
    sum2() 
    {
        int a[] = { 5, -2, 4, -18, 8, 10, -7, 5, 8, 4, 3, -7, -2, 5, 8, 0, 9, 10, -28, 13, 5 };
        int N = sizeof a / sizeof (int);
        int partial_sum = 0;
        int max = 1 << (sizeof(int) * 8 - 1);
        int i;

        for (i = 0; i < N; ++i) {
            partial_sum = MAX(partial_sum, 0) + a[i];
            max = MAX(partial_sum, max);
        }
        printf("max %d\n", max);
        return max;
    }

여기서 구간을 알아내도록 고쳐보면,

    #include <stdio.h>
    #include <stdlib.h>

    int _start = 0;
    int _end = 0;

    sum2() 
    {
        int a[] = { 5, -2, 4, -18, 8, 10, -7, 5, 8, 4, 3, -7, -2, 5, 8, 0, 9, 10, -28, 13, 5 };
        int N = sizeof a / sizeof (int);
        int partial_sum = 0;
        int max = 1 << (sizeof(int) * 8 - 1);
        int i;

        for (i = 0; i < N; ++i) {
            if (partial_sum < 0) {
                partial_sum = a[i];
                _start = i;
            } else
                partial_sum = partial_sum + a[i];

            if (partial_sum > max) {
                max = partial_sum;
                _end = i;
            } 
        }
        return max;
    }
    main(){printf("sum(%d-%d) = %d\n",_start,_end,sum2());}
