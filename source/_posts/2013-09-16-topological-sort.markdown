---
layout: post
title: "Topological Sort"
date: 2013-09-16 13:05
comments: true
categories: 
---

### Breadth first search.

우선 BFS.

    class Vertex:
        def __init__(self, i):
            self.index = i
            self.adjacency = []
            self.visited = False
            self.distance = 0
        def addEdge(self, e):
            self.adjacency.append(e)
        def listEdge(self):
            for e in self.adjacency:
                print "(%d,%d)" % (self.index, e.index)

    if __name__ == "__main__":
        va = []
        for i in range(0,10):
            va.append( Vertex(i) )

        def e(v, e):
            va[v].addEdge( va[e] )
            pass

        e(0,1)
        e(0,2)

        e(1,3)
        e(2,4)

        e(3,5)
        e(4,5)

        for v in va:
            v.listEdge()

        import Queue
        q = Queue.Queue()
        
        va[0].visited = True
        q.put(item = va[0], block=False)

        while ( q.empty() == False):
            v = q.get(block=False)
            for adj in v.adjacency:
                if ( adj.visited == False):
                    adj.visited = True
                    adj.distance = v.distance + 1
                    q.put(item = adj, block=False)

        for v in va:
            print "%d -> %d: (%d)" % (0, v.index, v.distance)

### DFS

그 다음 DFS

    class Vertex:
        def __init__(self, i):
            self.index = i
            self.adjacency = []
            self.visited = False
            self.distance = 0
        def addEdge(self, e):
            self.adjacency.append(e)
        def listEdge(self):
            for e in self.adjacency:
                print "(%d,%d)" % (self.index, e.index)

    def dfs(v):
        for n in v.adjacency:
            if n.visited == False:
                n.visited = True
                n.distance = v.distance + 1
                dfs(n)
            else:
                pass

    if __name__ == "__main__":
        va = []
        for i in range(0,10):
            va.append( Vertex(i) )

        def e(v, e):
            va[v].addEdge( va[e] )
            pass

        e(0,1)
        e(0,2)

        e(1,3)
        e(2,4)

        e(3,5)
        e(4,5)

        for v in va:
            v.listEdge()

        dfs(va[0])
        
        for v in va:
            print "%d -> %d: (%d)" % (0, v.index, v.distance)

### Dijkstra's Shortest-Path

다익스트라 알고리즘의 요체는 다음과 같다.

    1. 그래프를 두개의 컷으로 나눈다.
    2. 한개는 explored, 다른 한개는 unexplored.
    3. 목표는 모든 vertex 를 explore 하는 것.
    4. unexplored set 을 explore 한다는 것은 무슨 의미일까?
    5. unexplored set 의 vertex 하나를 explored set 과 연결해야 한다.
    6. 그 연결하는 edge 는 어떤 edge 를 선택해야 하는 것일까?
    7. explored 된 vertex 마다 unexplored 로 가는 edge 가 있는지, 있다면 그 weight 는 얼마인지?
    7. A[v] + l_vw 를 minimize 하는 v-w (v from explored set, w from unexplored set) 를 선택한다. l_vw 는 weight.
    8. 이 부분이 greedy 한 부분이다. 단계별로 행하는 myoptic decision 이 뒤집히지 않을 것이라는 가정하에 선택하는.

다익스트라 알고리즘의 가정은 edge 의 weight 가 non-negative 해야 한다는 것. greedy 한 nature 때문에 어쩔 수 없다.

### Minimum Spanning Tree

Prim's Algoritm: "The plan is to grow a tree one edge at a time"

주어진 input graph 가 connected graph 라는 가정하에.

Tree 를 한 개의 edge 씩 더해가며 키워가는 것이다.

    1. Vertex 중에서 하나를 임의로 고른다.
    2. 새로운 Vertex 를 acquire 해야 한다.
    3. 그러기위해서, 어떤 edge 를 acquire 해야 할까?
    4. greedy 하게 결정한다. 가장 weight 가 적은 edge 를 선택하여 새로운 vertex 를 acquire 한다.
    5. 모든 vertex 를 acquire 하면 종료.

### Topological Sort

Kahn algorithm.

    L <- Empty list that will contain the sorted elements
    S <- Set of all nodes with no incoming edges
    while S is non-empty do
        remove a node n from S
        insert n into L
        for each node m with an edge e from n to m do
            remove edge e from the graph
            if m has no other incoming edges then
                insert m into S
    if graph has edges then
        return error (graph has at least one cycle)
    else
        return L (a topological sorted order)
