---
title: 数据结构（四）队列
date: 2022-06-06 23:33:54
tags: ["数据结构"]
categories: ["数据结构"]
---

## 队列的定义
与栈结构不同的是，队列的两端都"开口"，要求数据只能从一端进，从另一端出，遵循 "先进先出 FIFO (First In First Out)" 的原则。

<!-- more -->

![对头出队，队尾入队](http://c.biancheng.net/uploads/allimg/190426/1I33AU8-0.gif)

通常，称进数据的一端为 "队尾"，出数据的一端为 "队头"，数据元素进队列的过程称为 "入队"，出队列的过程称为 "出队"。

> ⚠️ 栈和队列不要混淆，栈结构是一端封口，特点是"先进后出"；而队列的两端全是开口，特点是"先进先出"。

## 队列的顺序存储（循环队列）

在顺序表的基础上实现的队列结构。

```cpp

#define MaxSize 100                 // 定义队列中元素的最大个数

/**
 * 队列的顺序存储 (循环队列)
 */
typedef int ElemType;               // 元素数据类型
typedef struct SqQueue{
    ElemType data[MaxSize];         // 存放队列元素
    int front;                      // 队首
    int rear;                       // 对头
} SqQueue;

/**
 * 初始化队列
 *
 * @param Q
 */
void InitQueue(SqQueue &Q){
   Q.rear = Q.front = 0;
};

/**
 * 判断队列是否为空
 *
 * @param Q
 * @return
 */
bool QueueEmpty(SqQueue Q){
    // 当队列中没有元素时，对首和队尾指向同一块地址
    if (Q.rear == Q.front) {
        return true;
    }

    return false;
};

/**
 * 入队
 *
 * @param Q
 * @param x
 * @return
 */
bool EnQueue(SqQueue &Q, ElemType x){
    // 判断队列是否已满
    if ((Q.rear + 1) %MaxSize == Q.front) {
        return false;
    }

    // 每次新元素入队时，队尾都需要 "+1"，方便下一次可以新元素直接入队
    Q.data[Q.rear] = x;
    Q.rear = (Q.rear + 1) % MaxSize;
    return true;
};

bool DeQueue(SqQueue &Q, ElemType &x){
    // 判断队列是否为空
    if (QueueEmpty(Q)) {
        return false;
    }

    // 每次旧元素出对时，对头都需要 "+1"，指向新的对头
    x = Q.data[Q.front];
    // 取余的目的是为了使循环队列的尾部回到头部
    Q.front = (Q.front + 1) % MaxSize;
    return true;
};

/**
 * 获取队列长度
 *
 * @param Q
 * @return
 */
int QueueLength(SqQueue Q){
    return (Q.rear-Q.front+MaxSize) % MaxSize;
};
```

## 队列的链式存储

在链表的基础上实现的队列结构。

```cpp
/**
 * 队列的链式存储
 */

// 队列结构体
typedef struct {
    LinkNode *front;    // 链表头
    LinkNode *rear;     // 链表尾
}LinkQueue;

/**
 * 初始化队列
 *
 * @param Q
 */
void InitLinkQueue(LinkQueue &Q){
    // 申请一个头结点，头和尾指向同一个节点
    Q.front = Q.rear = (LinkNode *) malloc(sizeof(LinkNode));
    Q.front->next = NULL;
};

/**
 * 判断队列是否为空
 *
 * @param Q
 * @return
 */
bool LinkQueueEmpty(LinkQueue Q){
    // 头和尾指向同一个节点即为空
    if (Q.front == Q.rear) {
        return true;
    }

    return false;
};

/**
 * 入队（尾插法）
 *
 * @param Q
 * @param x
 * @return
 */
void EnLinkQueue(LinkQueue &Q, ElemType x){
    LinkNode *s = (LinkNode*) malloc(sizeof (LinkNode));
    s->data = x;
    Q.rear->next = s;
    Q.rear = s;
};

/**
 * 出队（头部删除法）
 *
 * @param Q
 * @param x
 * @return
 */
bool DeLinkQueue(LinkQueue &Q, ElemType &x){
    if (LinkQueueEmpty(Q)) {
        return false;
    }

    // 因为是先进先出，所以头部删除法，删除的是第一个元素
    LinkNode *p;
    p = Q.front->next;
    x = p->data;
    Q.front->next = p->next;
    // 这里需要判断删除节点是否是最后一个元素，如果是则需要把头和尾指向同一个节点
    if (Q.rear == p)
    {
        Q.rear = Q.front;
    }

    free(p);
    return true;
};
```