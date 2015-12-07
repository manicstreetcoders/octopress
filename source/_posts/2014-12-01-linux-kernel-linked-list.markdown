---
layout: post
title: "Linux Kernel - Linked List"
date: 2014-12-01 07:36:02 +0900
comments: true
categories: 
---

### Linked List

`linux/list.h` 에 구현되어 있다. static inline 으로 헤더 화일에 짧게 구현되어있다.

``` c
struct list_head {
    struct list_head *prev;
    struct list_head *next;
}

static inline void INIT_LIST_HEAD(struct list_head *list)
{
    list->prev = list;
    list->next = list;
}

struct cat {
    char *name;
    struct list_head list;
}

int main()
{
    struct cat babo;

    INIT_LIST_HEAD(&babo.list);
    babo.name = "kakaka";
}
```

구조체에 `struct list_head` 를 넣는 것으로 doubliy-linked list 를 만들 수 있다.

#### container_of()

``` c
#define container_of(ptr,type,member) \
    ((type*)((char*)(const typeof(((type*)0)->member)*)ptr-offsetof(type,member)))

...
struct dog {
    char *name; 
    struct list_head list;
}
...
struct list_head *list_p = &dog.list;
...
struct dog *p;

p = container_of(list_p, struct dog, list);
```

`list_head 포인터` 에서 `스트럭쳐에서의 오프셋` 만큼 뒤로 가는 매크로. 그만큼 뒤로 가면, 스트럭쳐의 시작 위치가 나온다.

#### LIST_HEAD_INIT()

node 를 초기화하기 위해 2 가지 함수가 제공되는데, 매크로 버전과 inline 함수 버전이 있다.

### Priority List

우선순위를 가진 sorted list. 2 개의 linked list 로 구현되어있다. sorted list 이기 때문에, tail 에 붙인다는 개념보다는 head 에 붙인다는 식이다.

``` c
struct plist_head {
    struct list_head node_list;
}

struct plist_node {
    int prio;
    struct list_head prio_list;
    struct list_head node_list;
}
```
