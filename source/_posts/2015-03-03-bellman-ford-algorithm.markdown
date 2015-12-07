---
layout: post
title: "Bellman-Ford Algorithm"
date: 2015-03-03 21:26:51 +0900
comments: true
categories: 
---

Bellman-Ford 알고리즘에서 negative edge 체크가 빠진 버전.

파이썬으로 만들어봤다.

```
class Vertex:
    def __init__(self,n):
        self.number = n
        self.adj = []
        self.d = float('inf')

    def addNeighbor(self,v,weight):
        self.adj.append( (v,weight) )

    def p(self):
        print("Vertex %d, depth %f ->" % (self.number,self.d)),
        for (v,weight) in self.adj:
            print("%d:%d " % (v.number,weight)),
        print

def relax(v):
    for i in v:
        for (nv,weight) in i.adj:
            if i.d == float('inf'):
                print "skip..."
                continue
            if (i.d + weight < nv.d):
                print "relaxing vertex %d" % (nv.number)
                nv.d = i.d + weight
    return

if __name__ == "__main__":
    v = []
    for i in range(0,10):
        v.append(Vertex(i))

    v[9].addNeighbor(v[8],5)
    v[8].addNeighbor(v[7],5)
    v[7].addNeighbor(v[6],5)
    v[6].addNeighbor(v[5],5)
    v[5].addNeighbor(v[4],5)
    v[4].addNeighbor(v[3],5)
    v[3].addNeighbor(v[2],5)
    v[2].addNeighbor(v[1],5)
    v[1].addNeighbor(v[0],5)

    v[7].addNeighbor(v[2],5)

    v[9].d = 0

    for i in v:
        i.p()

    count = 0
    for i in range(0,10-1):
        count += 1
        relax(v)

    print "performed relaxing operation %d times." % (count)

    for i in v:
        i.p()
```
