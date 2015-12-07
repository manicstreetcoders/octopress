---
layout: post
title: "Picking n numbers"
date: 2014-03-26 11:13:19 +0900
comments: true
categories: 
---
아이의 숙제를 해주기 위해...

0 ~ 6 중에서 4 개의 수를 뽑는 프로그램.

    depth = 4
    list = []
    def pick(d):
        global depth
        if (d > depth):
            print list
            return
        if (len(list) == 0):
            smallest = -1
        else:
            smallest = list[-1]

        for i in range(smallest+1,6+1):
            list.append(i)
            pick(d+1)
            del list[-1]

    if __name__ == "__main__":
        pick(1)
