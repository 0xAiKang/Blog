---
title: 数据结构（三）栈
date: 2022-05-27 00:02:07
tags: ["数据结构"]
categories: ["数据结构"]
---

## 栈的定义
栈是一种只能从表的一端存取数据且遵循 "后进先出 LIFO（Last In First Out）" 原则的线性存储结构。

<!-- more -->

![](http://c.biancheng.net/uploads/allimg/190426/1I0526392-0.gif)

通常，栈的开口端被称为栈顶，允许进行插入和删除；相应地，封口端被称为栈底，不允许进行插入和删除。

![](http://c.biancheng.net/uploads/allimg/190426/1I0523601-1.gif)

基于栈结构的特点，在实际应用中，通常只会对栈执行以下两种操作：
* 向栈中添加元素，此过程被称为"进栈"（入栈或压栈）；
* 从栈中提取出指定元素，此过程被称为"出栈"（或弹栈）；

栈的基本操作：
* `void InitStack(SqStack &S);`         // 初始化一个空栈
* `bool StackEmpty(SqStack S);`         // 判断一个栈是否为空， 若找 s 为空则返回 true，否则返回 false。
* `bool Push(SqStack &S, ElemType x);`  // 进栈，若栈 s 未满，则将 x 压入栈
* `bool Pop(SqStack &s, ElemType &x);`  // 出栈，若栈 S 非空，则弹出栈顶元素， 并用 x 返回。
* `bool GetTop(SqStack S, ElemType &x);`// 读栈顶元素，若栈 s 非空，则用 x 返回栈顶元素。
* `void DestroyStack(SqStack&S);`       // 销毁栈，并释放栈 s 占用的存储空间。
* `bool StackOverflow(SqStack S);`      // 判断栈是否满
* `int StackLength(SqStack S);`         // 栈元素个数

## 栈的顺序存储

采用顺序存储结构可以模拟栈存储数据的特点，从而实现栈存储结构。
```cpp

#define MaxSize 100                 // 定义栈中元素的最大个数

/**
 * 栈的顺序存储
 */
typedef int ElemType;               // 元素数据类型
typedef struct SqStack {
    ElemType data[MaxSize];         // 存放栈中元素
    int top;                        // 栈顶指针
} SqStack;

/**
 * 初始化栈
 *
 * @param S
 */
void InitStack(SqStack &S){
    // 将栈顶赋值为 -1
    S.top = -1;
};

/**
 * 判断栈是否为空
 *
 * @param S
 * @return
 */
bool StackEmpty(SqStack S){
    return S.top == -1;
};

/**
 * 入栈
 *
 * @param S
 * @param x
 * @return
 */
bool Push(SqStack &S, ElemType x){
    if (StackOverflow(S)) {
        return false;
    }

    //  先给S.top +1，然后给 S.data[S.top +1] 赋值
    S.data[++S.top] = x;
    return true;
};

/**
 * 出栈
 *
 * @param S
 * @param x
 * @return
 */
bool Pop(SqStack &S, ElemType &x){
    // 判断栈空
    if (StackEmpty(S)) {
        return false;
    }

    // 先把S.data[S.top] 的值赋值给 x，然后 S.top -1
    x = S.data[S.top--];

    return true;
};

/**
 * 获取栈顶元素
 *
 * @param S
 * @param x
 * @return
 */
bool GetTop(SqStack S, ElemType &x){
    // 判断栈空
    if (StackEmpty(S)) {
        return false;
    }

    x = S.data[S.top];
    return true;
};

/**
 * 销毁栈
 *
 * @param S
 */
void DestroyStack(SqStack &S){
    S.top = -1;
    // 释放栈 S 所占存储空间
    free(S.data);
};

/**
 * 判断桟满
 *
 * @param S
 * @return
 */
bool StackOverflow(SqStack S){
    if (S.top >= MaxSize-1) {
        return false;
    }
};

/**
 * 获取栈的元素个数
 *
 * @param S
 * @return
 */
int StackLength(SqStack S){
    return S.top+1;
};
```

## 栈的链式存储

采用链式存储结构实现栈结构。
```cpp
/**
 * 栈的链式存储
 */
typedef struct LinkNode {
    ElemType data;
    LinkNode * next;
}LinkNode, *LiStack;

/**
 * 初始化栈
 *
 * @param LS
 * @return
 */
LiStack InitLiStack(LiStack &LS){
    LS = (LiStack) malloc(sizeof(LinkNode));
    LS->next = NULL;
    return LS;
};

/**
 * 判断栈是否为空
 *
 * @param LS
 * @return
 */
bool LiStackEmpty(LiStack LS){
    return LS->next == NULL;
};

/**
 * 入栈
 *
 * @param LS
 * @param x
 */
void LiPush(LiStack &LS, ElemType x){
    LiStack Lhead = (LiStack) malloc(sizeof(LinkNode));
    Lhead->data = x;
    Lhead->next = LS->next;
    LS->next = Lhead;
};

/**
 * 出栈
 *
 * @param LS
 * @return
 */
ElemType LiPop(LiStack &LS){
    if (LiStackEmpty(LS)) {
        return false;
    }

    ElemType x;
    LiStack Lhead = LS->next;
    LS->next = Lhead->next;
    x = Lhead->data;
    free(Lhead);
    return x;
};

/**
 * 获取栈的元素个数
 *
 * @param LS
 * @return
 */
int LiStackLength(LiStack LS){
    int i = 0;
    while (LS->next != NULL) {
        i ++;
    }
    return i;
};
```