---
layout: post
title: "Partition function in Quicksort"
date: 2012-12-01 21:18
comments: true
categories: 
---

### Quicksort

Pseudo-code 로 Quicksort 를 기술해보자.

    def qsort(a):
        if len(a) <= 1:
            return a
        pivot = a[0]
        less = []
        greater = []

        for i in range(1,len(a)):
            if a[i] < pivot:
                less.append(a[i])
            else
                greater.append(a[i])

        return qsort(less) + [ pivot ] + qsort(greater)

Python 으로 구현하니까 너무 간단한데, C 나 Java 로 구현할때는 메모리에 신경을 써야 한다. 
별도의 메모리를 쓰지 않으면서, less / greater 로 파티셔닝하는 함수를 구현하는 것이 좀 까다로운데...

Sedgewick 의 알고리즘 책에 나온 파티셔닝 함수가 있다. 근데 이 함수에 골때린 점이 있다.
__Pivot 과 같은 값들이 여러개 있을 경우, left list / right list 양쪽으로 다 갈 수 있다는 점.
다만 recursion 을 거듭하면서, 양쪽에 흩어져 있는 (Pivot 과) 동일 값들이 pivot 쪽으로 모이게 되는 것이다.

    int _partition(int a[],int lo,int hi) {
        int pivot;
        int i,j;
        pivot = a[lo];
        i = lo; j = hi+1;
        while(1) {
            while (a[++i] < pivot) if (i >= hi) break;
            while (a[--j] > pivot) if (j <= lo) break;
            if (i>=j) break;
            swap(&a[i],&a[j]);
        }
        swap(&a[lo],&a[j]);
        return j;
    }

좀 더 직관적인 파티션 함수

    int __partition(int a[],int lo,int hi) {
        int pivot;
        int i,j;
        pivot = a[lo];
        i=j=lo;
        for (++j;j<=hi;j++) {
            if (a[j] < pivot) {
                i++;
                swap(a,i,j);
            }
        }
        swap(a,lo,i);
        return i;
    }
    void __q(int a[],int lo,int hi) {
        int j;
        if (hi-lo<=0) return;
        j = __partition(a,lo,hi);
        __q(a,lo,j-1);
        __q(a,j+1,hi);
    }

### Counting inversions by piggybacking mergesort

곁다리로, Mergesort 의 응용으로 재미난 것이 있는데.
Sort 가 되지 않은 pair 의 갯수를 찾아내는 알고리즘.
즉 inversion pair 를 찾는 알고리즘.
Mergesort 를 piggybacking 해서 카운팅함.

사람들이 1 부터 n 까지 랭킹을 매긴다고 했을때,
Person A 와 Person B 가 각각 매긴 랭킹이 얼마나 유사한가를 측정하는 알고리즘도 응용해서 만들 수 있음.

Recommendation 에서 쓰이는 collaborative filtering.

### Binary Heap

Binary Heap 으로 Minimum priority queue 를 구현해보았다.

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>

    typedef struct _heap {
        int heap[1024];
        int N;
    } heap;

    /* heap index starts from 1 */
    int parent(int idx) { return idx/2; }
    int left(heap * p_heap, int idx) { return idx*2 <= p_heap->N ? idx*2 : 0; }
    int right(heap * p_heap, int idx) { return idx*2+1 <= p_heap->N ? idx*2+1 : 0; }
    swap(int a[],int i,int j) { int t = a[i]; a[i] = a[j]; a[j] = t; }
    bubble_up(heap * p_heap) {
        int i = p_heap->N;

        while (i > 0 && parent(i) > 0) {
            if (p_heap->heap[i] < p_heap->heap[ parent(i) ] ) {
                swap(p_heap->heap, i, parent(i));
                i = parent(i);
            } else
                break;
        }
    }
    sink(heap * p_heap) {
        int i = 1;
        int l,r;
        int * a = p_heap->heap;

        while (i < p_heap->N) {
            l = left(p_heap, i);
            r = right(p_heap, i);
            if (a[i] > a[l]) {
                if (a[l] > a[r]) {
                    swap(a, i, r);
                    i = r;
                } else {
                    swap(a, i, l);
                    i = l;
                }
            } else if (a[i] > a[r]) {
                if (a[r] > a[l]) {
                    swap(a, i, l);
                    i = l;
                } else {
                    swap(a, i, r);
                    i = r;
                }
            } else
                break;
        }
    }
    heap_init(heap * p_heap) {
        p_heap->N = 0;
        p_heap->heap[0] = (0xFFFFFFFF << 1 >> 1);
    }
    heap_add(heap * p_heap, int x) {
        p_heap->heap[ ++p_heap->N ] = x;
        bubble_up(p_heap);
    }
    int heap_remove(heap * p_heap) {
        int min = p_heap->heap[1];
        int end = p_heap->heap[ p_heap->N ];
        p_heap->heap[1] = end;
        p_heap->N--;
        sink(p_heap);

        return min;
    }
    heap_isempty(heap * p_heap) { return p_heap->N == 0; }
    heap_heapify(heap * p_heap, int a[], int N) {
        int i;
        heap_init(p_heap);
        for (i = 0;i < N;i++)
            heap_add(p_heap, a[i]);
    }
    void _d(int a[],int N) {
        int i;
        for (i=0;i<N;i++) printf("%d ", a[i]);
        printf("\n");
    }
    main()
    {
        int a[]={ 123, 1, 0, -4, 45, 38, 24, 99, 88, 43, 23, 12, 9, 8, 7, 46, 77 };
        int N = sizeof a / sizeof(int);
        heap h;

        _d(a,N);
        heap_heapify(&h, a, N);
        while (!heap_isempty(&h)) printf("%d ", heap_remove(&h));
        printf("\n");
    }
