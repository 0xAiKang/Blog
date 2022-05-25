---
title: 数据结构（二）线性表
date: 2022-05-25 23:49:45
tags: ["数据结构"]
categories: ["数据结构"]
---

## 线性表的逻辑结构
定义：由 N（N >= 0）个**相同类型**的元素组成的**有序**集合。

<!-- more -->

```
L = (a1, a2, , ai-1, ai, ai+1, , an);
```

数组是一种线性表，但是不能说线性表就是数组。\
线性表可以用数组实现（顺序表），也可以用链表实现（单链表、双链表）。

1. 线性表中元素个数 n，称为线性表的长度。当 `n=0` 时，为空表。
2. a1 是唯一一个 **第一个** 数据元素，an 是唯一的 **最后一个** 数据元素。
3. ai-1 是 ai 的直接前驱，ai+1 为 ai 的直接后继。

线性表的特点：
1. 表中元素是有限的
2. 表中元素的数据类型都是相同的，意味着每一个元素占用相同大小的空间
3. 表中元素具有逻辑上的顺序性，在序列中各元素排序有其先后顺序

## 线性表的顺序存储

使用顺序储存实现线性表的优点：
1. 可以随机存取（根据表头任意元素地址和元素序号）表中任意一个元素
2. 存储密度高，每个结点只存储数据元素

缺点：
1. 插入和删除时都需要移动大量元素
2. 线性表变化较大时，难以确定存储空间的容量
3. 存储分配需要一段连续的存储空间，不够灵活

线性表基本操作主要有：

* `bool InitList(SqList &L)`：初始化表。构造一个空的线性表。
* `int LocateElem(SqList L, ElemType e)`：按值查找操作。在表 L 中查找具有给定关键宇值的元素。
* `ElemType GetElem(SqList L, int i)`：按位查找操作。获取表 L 中第 i 个位置的元素的值。
* `bool ListInsert(SqList &L, int i, ElemType e)`：插入操作。在表 L 中的第 i 个位置上插入指定元素 e。
* `bool ListDelete(SqList &L, int i, ElemType &e)`：删除操作。删除表 L 中第 i 个位置的元素，并用 e 返回删除元素的值。
* `int Length(SqList L)`：返回线性表 L 的长度，即 L 中数据元素的个数。
* `bool Empty(SqList L)`：判空操作。若 L 为空表， 则返回 true，否则返回 false。
* `bool DestroyList(SqList &L)`：销毁操作。销毁线性表，井释放线性表 L 所占用的内存空间。

```cpp
#include "SqList.hpp"

/**
 * 初始化。构造一个空的线性表。
 *
 * @param L
 * @return
 */
bool InitList(SqList &L){
    L.data = (ElemType *) malloc(sizeof (ElemType) * MaxSize);
    L.length = 0;
    return true;
};

/**
 * 插入操作，在表 L ，第 i 个位置插入指定元素 e
 *
 * @param L
 * @param i
 * @param e
 * @return
 */
bool ListInsert(SqList &L, int i, ElemType e)
{
    // 判断 i 的取值范围是否有效（ 1<=i<=L.length)
    if (i < 1 || i > L.length) {
        return false;
    }

    // 当存储空间已满，不能插入
    if (i >= MaxSize) {
        return false;
    }

    for (int j = L.length; j >= i; j--) {
        L.data[j] = L.data[j-1];
    }

    L.data[i-1] = e;
    L.length ++;
    return true;
}

/**
 * 删除操作，删除线性表 L 第 i 个位置的元素，并用 e 返回被删除元素的
 *
 * @param L
 * @param i
 * @param e
 * @return
 */
bool ListDelete(SqList &L, int i, ElemType &e) {
    // i 的值必须合法
    if (Empty(L)) {
        return false;
    }

    // 判断 i 的取值范围是否有效（ 1<=i<=L.length)
    if (i < 1 || i > L.length) {
        return false;
    }

    // 需要被删除元素的值
    e = L.data[i-1];
    // 将剩余的元素的值往前移动
    for (int j = i; j < L.length; j++) {
        L.data[j-1] = L.data[j];
    }

    L.length--;
    return true;
}

/**
 * 返回线性表的长度
 *
 * @param L
 * @return
 */
int Length(SqList L) {
    return L.length;
};

/**
 * 按值查找
 *
 * @param L
 * @param e
 * @return
 */
int LocateElem(SqList L, ElemType e) {
    for (int i = 0; i < L.length; ++i) {
        if (L.data[i] == e) {
            // 下标为 i 的元素等于 e，返回其位序为 i+1
            return i+1;
        }
    }

    return 0;
};

/**
 * 按位查找
 *
 * @param L
 * @param i
 * @return
 */
ElemType GetElem(SqList L, int i) {
    // 判断 i 的取值范围是否有效（ 1<=i<=L.length)
    if (i < 1 || i > L.length) {
        return -1;
    }

    return L.data[i-1];
};

/**
 * 打印线性表所有元素
 *
 * @param L
 */
void PrintList(SqList L){
    for (int i = 0; i < L.length; ++i) {
        printf("%d ", L.data[i]);
    }
    printf("\n");
};

/**
 * 判断是否为空
 *
 * @param L
 * @return
 */
bool Empty(SqList L){
    if (L.length == 0) {
        return true;
    }

    return false;
};

/**
 * 销毁操作，释放线性表所占内存空间
 *
 * @param L
 * @return
 */
bool DestroyList(SqList &L){
    free(L.data);
    L.length = 0;
    return true;
};
```

最好情况：在表尾插入元素，不需要移动元素，时间复杂度为 O(1)
最坏情况：在表头插入元素，所有元素依次后移，时间复杂度为 O(n)
平均情况：在插入位置概率均等的情况下，平均移动元素的次数为 n/2，时间复杂度为 O(n)

时间复杂度的计算忽略高阶项的系数，

---
scanf 传递时，为什么后台需要给一个地址？\
其实就是指针传递的使用场景。

## 线性表的链式存储
头指针：链表中第一个结点的存储位置，用来标识单链表
头结点：在单链表第一个结点之前附加的一个结点，为了操作上的方便

若链表有头结点，则头指针永远指向头结点，不论链表是否为空，头指针均不为空，头指针是链表的必须元素，他标识一个链表。

头结点是为了操作的方便而设立的，其数据域一般为空，或者存放链表的长度。
有了头结点之后，对在第一结点钱插入和删除第一结点的操作就统一了，不需要频繁重置头指针。但头结点不是必须的。

### 单向链表

```cpp
#include "LinkList.hpp"

/**
 * 头插法建立单链表
 * 思路：建立新的结点分配内存空间，将新结点插入到当前链表的表头
 *
 * @param L
 * @param a
 * @param n
 * @return
 */
LinkList List_HeadInsert(LinkList &L, int a[], int n) {
    // 创建头结点
    L = (LinkList) malloc(sizeof (LNode));
    // 初始化链表
    L->next = NULL;

    LNode *s;
    for (int i = 0; i < n; i++) {
        // 申请一个新空间给 s
        s = (LNode *) malloc(sizeof (LNode));
        s->data = a[i];
        s->next = L->next;
        L->next = s;
    }
    return L;
};

/**
 * 尾插法建立单链表
 * 思路：建立新的结点分配内存空间，将新结点插入到当前链表的表尾
 *
 * @param L
 * @return
 */
LinkList List_TailInsert (LinkList &L, int a[], int n){
    L = (LinkList) malloc(sizeof(ElemType));
    L->next = NULL;

    LinkList s, r = L;
    for (int i = 0; i < n; ++i) {
        // 创建新结点
        s = (LNode*)malloc(sizeof(LNode));
        s->data = a[i];

        // r 指向新的表尾结点
        r->next = s;  // 这一步的目的是保证 r 的 next 指针指向新空间
        r = s;        // 这一步的目的是保证 r 是链表的尾部
    }
    r->next = NULL;
    return L;
}

/**
 * 按位查找操作。获取表 L 中第 i 个位置的元素的值。
 * 思路：在单链表中从第一个结点出发，顺指针next域逐个往下搜索，直到找到第i个结点为止,否则返回最后一个结点指针域NULL。
 *
 * @param L
 * @param i
 * @return
 */
LNode *GetElem(LinkList L, int i){
    int j = 1;
    // 让 P 指向第一个指针
    LNode *p = L->next;
    if (i == 0) {
        // 如果 i = 0，返回头结点
        return L;
    }

    if (i < 1) {
        // 如果 i 小于零，返回 NULL
        return NULL;
    }

    while (p && j < i) {
        p = p->next;
        j ++;
    }
    return p;
};

/**
 * 按值查找操作。在表 L 中查找具有给定关键宇值的元素。
 * 思路：从单链表第一个结点开始，由前往后依次比较表中各结点数据域的值，若某结点数据域的值等于给定值e，则返回该结点的指针
 * 若整个单链表中没有这样的结点，则返回NULL。
 *
 * @param L
 * @param e
 * @return
 */
LinkList LocateElem(LinkList L, ElemType e) {
    L = L->next;
    while (L) {
        if (L->data == e) {
            break;
        }
        L = L->next;
    }
    return L;
}

/**
 * 获取链表长度
 *
 * @param L
 * @return
 */
int Length(LNode *L){
    L = L->next;
    int i = 0;
    while (L) {
        i++;
        L = L->next;
    }
    return i;
}

/**
 * 打印链表
 *
 * @param L
 */
void PrintLinkList(LinkList L)
{
    L = L->next;
    while (L != NULL)
    {
        printf("%3d", L->data);
        L = L->next;
    }
    printf("\n");
}
```


### 双向链表

```c
#include "LinkList.hpp"

/**
 * 头插法建立双链表
 * 思路：从四个方向依次进行关联（头结点的前驱、新结点的前驱、新结点的后继、头结点的 next 的前驱）
 *
 * @param DL
 * @param a
 * @param n
 * @return
 */
DLinkList DList_HeadInsert(DLinkList &DL, int a[], int n){
    // 创建头结点
    DL = (DLinkList) malloc(sizeof(DNode));
    // 前驱和后继指针都是NULL
    DL->next = NULL;
    DL->prior = NULL;

    DNode *s;
    for (int i = 0; i < n; ++i) {
        s = (DLinkList)malloc(sizeof(DNode));   // 申请新结点
        s->data = a[i];
        s->next = DL->next;
        if (DL->next != NULL) {
            DL->next->prior = s;
        }

        s->prior = DL;
        DL->next = s;
    }

    return DL;
};

/**
 * 尾插法建立双链表
 *
 * @param DL
 * @param a
 * @param n
 * @return
 */
DLinkList DList_TailInsert(DLinkList &DL, int a[], int n){
    DL = (DNode *) malloc(sizeof(DNode));

    DLinkList s, r = DL;
    DL->prior = NULL;  // 前驱指针是NULL
    for (int i = 0; i < n; ++i) {
        s = (DLinkList) malloc(sizeof(DNode));   // 申请新结点
        s->data = a[i];
        r->next = s;
        s->prior = r;
        r = s;          // 这一步的目的是为了保证 r 始终是尾指针
    }
    r->next = NULL;
    return DL;
};

/**
 * 按位查找操作
 *
 * @param L
 * @param i
 * @return
 */
DNode *GetElemDList(DLinkList DL, int i) {
    if (i < 0) {
        return NULL;
    }

    int j = 0;
    while (DL && j < i) {
        DL = DL->next;
        j++;
    }
    return DL;
}

/**
 * 按值查找操作
 *
 * @param L
 * @param e
 * @return
 */
DNode *LocateElemDList(DLinkList DL, ElemType e){
    DL = DL->next;
    while (DL) {
        if (e == DL->data) {
            break;
        }
        DL = DL->next;
    }
    return DL;
};

/**
 * 打印双链表
 *
 * @param L
 */
void PrintDLinkList(DLinkList DL)
{
    DL = DL->next;
    while (DL != NULL)
    {
        printf("%3d", DL->data);
        DL = DL->next;
    }
    printf("\n");
}
```