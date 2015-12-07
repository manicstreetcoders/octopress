---
layout: post
title: "Peak Finder"
date: 2013-09-06 11:50
comments: true
categories: 
---

Peak Finding 을 recursive 하게 만들어 보기.

peak 의 정의

1. Consider an array a[1...n]:
2. An element a[i] is a peak if it is not smaller than its neighbor(s). I.e.,
- if i != 1, n: a[i] >= a[i-1] and a[i] >= a[i+1]
- if i == 1, a[1] >= a[2]
- if i == n, a[n] >= a[n-1]

처음에는 straight forward 하게 만들기

    def peak(a):
        n = len(a)
        for i in range(0,n):
            if i == 0:
                if a[0] >= a[1]:
                    return 0
            elif i == n:
                if a[n] >= a[n-1]:
                    return n
            else
                if a[i] >= a[i-1] and a[i] >= a[i+1]:
                    return i
        return -1

    if __name__ == "__main__":
        a = [ 10, 13, 5, 8, 3, 2, 1 ]
        print a[peak(a)]

divide & conquer 로 찾아보기.

    def peak(a,abs):
        n = len(a)

        if n == 1:
            return abs
        if n == 2:
            if a[0] > a[1]:
                return abs
            elif a[1] > a[0]:
                return abs+1
            else
                return abs

        right = n/2+1
        left = n/2-1
        mid = n/2

        if a[mid] < a[left]:
            half = a[:mid]
            return peak(half,abs)
        elif a[mid] < a[right]:
            half = a[mid+1:]
            return peak(half,abs+mid+1)
        else:
            return abs+mid

    if __name__ == "__main__":
        a = [ 2, 2 ]
        print a[peak(a,0)]

이번에는 2D 어레이에서 Peak 를 찾아본다. straight forward 하게 작성하면 O(n^2) 이 된다.

[Comparison of public peak detection algorithms for MALDI mass spectrometry data analysis](http://www.ncbi.nlm.nih.gov/pmc/articles/PMC2631518/)

- Smoothing
- baseline correction
- peak finding
