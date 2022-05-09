---
title: C 语言学习笔记
date: 2022-05-09 22:59:01
tags: ["C"]
categories: ["C"]
---

## 为什么要学习 C 语言

为什么要学习C 语言？\
C 语言是无可替代的存在，有些事情只能是C 语言来完成，比如写操作系统。

学习C 语言的目的并不是精通C，而是理解C 语言的编译过程及内存变化的原理，使用C 去练习各种数据结构。\
这些才是学习C 语言的目的。

<!-- more -->

## 快速上手

工欲善其事必先利其器，编写 C语言程序的工具非常多：
* VSCode
* Sublime Text
* CLion
* Xcode
* Qt Creator

根据个人喜好进行选择，前期入门建议使用轻量级的 IDE，这里我选择的是 VSCode。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220509214956.png)
`.vscode`：基于当前工作区生成的配置文件目录，其中主要包含以下文件：
* `tasks.json`：编译器构建设置
* `launch.json`：调试器设置
* `c_cpp_properties.json`：编译器路径和IntelliSense设置

每次创建一个新的项目（Demo），建议创建一个目录，因为每次编译运行`.c` 文件，都会额外生成一些文件：
* `main`：对应的可执行文件
* `main.dsYM`：Xcode 生成的文件

### 运行
正式开始之前，确保本地环境已经安装了 `gcc` 或者其他编译器。

使用 VSCode 运行 C程序非常简单，只需要在对应的 C文件下点击运行或者使用 `^F5`即可。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220509214932.png)

### 调试
在正式开始调试之前，需要先安装一个扩展：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220509215050.png)
安装完成之后，将 `launch.json` 中的 `type` 配置项改为 `lldb`，其他部分不用做修改
```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "gcc - 生成和调试活动文件",
      "type": "lldb",
      "request": "launch",
      "program": "${fileDirname}/${fileBasenameNoExtension}",
      "args": [],
      "stopAtEntry": false,
      "cwd": "${fileDirname}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "lldb",
      "preLaunchTask": "C/C++: gcc 生成活动文件"
    }
  ]
}
```

这个表示选择对应的调试器，刚才安装的 `C/C++` 就是一个调试器

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220509215109.png)
调试程序非常简单，只需要在对应代码的前面加上断点，然后点击调试即可。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220509215249.png)

## 从源文件到可执行文件
仅仅只靠编译是没有办法得到可执行文件的，编译器编译仅仅只是得到了本地文件，最终想到得到可执行文件，还需要进行“链接”处理。

### 编译
在Windows 下，编译后生成的并不是`exe`文件，而是扩展名为`.obj` 的目标文件; 在Unix 下，编译后生成的并不是可执行文件，而是扩展名为`.o`  的目标文件。

这些文件无法执行运行，因为编译过只是检查语法（函数、变成声明）是否正确。

在Mac 下编译`main.c` 文件：
```bash
gcc -c main.c

main.c  main.o
```

### 链接
找到所要用到的函数所在的目标文件并结合，最终生成一个可执行文件的过程就是链接。执行链接的程序被成为链接器。

在Mac 下链接 `main.c` 文件：
```bash
gcc main.o -o main

main.c main.o main
```

将编译、链接合并成一步：
```bash
gcc main.c

main.c main
```

总结：
* `main.c`：源代码文件
* `main.o`：源代码文件通过编译之后生成的本地代码（机器语言）
* `main`：可执行文件
* `a.out`：可执行文件（默认名称）

## 数据类型

在C 语言中，数据类型大致可以分为以下几种：
1. 基本类型：整型、字符型、浮点型
2. 构造类型：数组类型、结构类型、联合类型、枚举类型
3. 指针类型
4. void 类型

### 整数类型

| 类型           | 存储大小    | 值范围                                               |
| :------------- | :---------- | :--------------------------------------------------- |
| char           | 1 字节      | -128 到 127 或 0 到 255                              |
| unsigned char  | 1 字节      | 0 到 255                                             |
| signed char    | 1 字节      | -128 到 127                                          |
| int            | 2 或 4 字节 | -32,768 到 32,767 或 -2,147,483,648 到 2,147,483,647 |
| unsigned int   | 2 或 4 字节 | 0 到 65,535 或 0 到 4,294,967,295                    |
| short          | 2 字节      | -32,768 到 32,767                                    |
| unsigned short | 2 字节      | 0 到 65,535                                          |
| long           | 4 字节      | -2,147,483,648 到 2,147,483,647                      |
| unsigned long  | 4 字节      | 0 到 4,294,967,295                                   |

### 浮点类型

| 类型        | 存储大小 | 值范围                 | 精度        |
| :---------- | :------- | :--------------------- | :---------- |
| float       | 4 字节   | 1.2E-38 到 3.4E+38     | 6 位有效位  |
| double      | 8 字节   | 2.3E-308 到 1.7E+308   | 15 位有效位 |
| long double | 16 字节  | 3.4E-4932 到 1.1E+4932 | 19 位有效位 |

> 需要注意的是，各种类型的存储大小与系统位数有关，但目前通用的以64位系统为主。

在 C 语言中，通过取地址符(&) 即可看见变量在内存所占空间大小。

---
C 语言中的数据类型：

* 字符型常量：用单引号括起来的一个字符，如果用双引号或者单引号内有N 个字符，则都不是字符型常量。
* 字符串常量：用双引号括起来的字符序列

在C 语言中，并没有对应的字符串变量，不会像PHP 语言专门有一个`String` 类型来存储对应的字符变量。

那么该如何存储字符串呢？\
在C 语言中，是通过字符数组来存储字符串的。

字符串是由字符组成的，对于计算机而言，字符串是由一个个字符组成的，而一个字符的大小是一个字节。\
China 这个字符串在计算机中，所占的大小是六个字节而不是五个，这是因为最后一个字符是由`\0` 结尾，也需要占用一个字节。

## 类型转换
C 语言在做混合运算时，因为是强类型的语言，当做算术运算时，C 语言会按照变量的数据类型去进行运算。

在以下示例中，如果直接进行运算，得到的结果并不是我们所期望的：
```c
int main
{
    int i = 5;
    float f = i / 2;
    printf("%f \n", f);  // 2.0000
}
```

所以为了能正常输出 `2.5`，需要将这个表达式给转换为为浮点型。
```c
int main
{
    int i = 5;
    float f = (float)i / 2;
    printf("%f \n", f);  // 2.5
}
```

再来看另外一个例子：
```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    double a = 3.33;
    int i = a / 1.11;

    printf("%d \n", i);  // 2
    return 0;
}
```
为什么得到的结果不是 3 ，而是 2？

这是因为C 语言，对于没有声明为变量的浮点型会默认转换为 `double` 双精度类型：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220509221329.png)

而如果另一个变量的并不是浮点类型，比如是 `float` 类型，此时这两个精度不一样的浮点数直接进行运算就会丢失精度。

所以呢，在C 语言中进行算术运算时，需要保证数据类型在预期内。\
对于上面的问题，有两种方案：
1. 将变量a 转换为 `double` 类型
2. 将 1.11 强制转换为 `float` 类型

## 运算符与表达式
运算符的种类：
1. 算术运算符
2. 关系运算符
3. 逻辑运算符
4. 位运算符
5. 赋值运算符
6. 条件运算符
7. 逗号运算符
8. 指针运算符
9. 求字节数运算符
10. 强制类型转换运算符
11. 分量运算符
12. 下标运算符
13. 其他（如函数调用运算符）

---
`i++` 和 `++i` 的区别：
* `i++` 是先进行运算符，最后才对变量 i 进行`+1`
* `++i` 则刚好是相反的，先对变量 i 进行 `+1`，然后进行其他运算

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    int i = 1;
    int j = i ++ < -1;
    prinf("i = %d, j = %d \n", i, j);   // i = 0, j = 0
  
    // int i = 1;
    // int j = ++i < -1;
    // prinf("i = %d, j = %d \n", i, j);   // i = 0, j = 1
  
    return 0;
}
```

---
C 语言中没有布尔类型。\
C 语言认为一切非零的值都是真。

下面这段代码这样写是有问题的，因为 `char` 类型所占空间大小是一个字节，而 `scanf` 获取标准输入的是一个整型，而整型所占空间大小又是四个字节，所以这段代码运行之后会报错。
```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    char c;
    scanf("%d", &c);
    prinf("c = %c \n", c);
    return 0;
}
```

正确的实例，应该是这样，定义变量时，使用 `int` 类型去定义
```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    int c;
    scanf("%d", &c);
    prinf("c = %c \n", c);
    return 0;
}
```

## 获取输入与输出

### scanf
整型、浮点型、字符型需要使用取地址符。

### printf
`printf` 函数：
* `%d`：以整型输出对应数据
* `%f`：以浮点型输出对应数据
* `%c`：以字符型输出对应数据

### gets
当一次读取一行内容时，可以使用 gets

```c
char c[20];
gets(c);
```

使用 scanf 获取标准输入时，会遇到一个问题，当输入的字符中间存在空格时，会结束匹配，这样就没有办法把一行带有空格的字符串存到一个字符数组中了。

gets 的原理：
会从缓冲区中一直进行读取，直到遇到 `\n` 结束符。

而 `scanf` 则会匹配 `\n` 结束符之前的所有内容，也就是它会把 `\n` 结束符留在缓冲区中。所以当 `scanf` 和 `gets` 函数一起使用时，需要主动去掉结束符。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char const *argv[])
{
    int i;
    // char c[20];
    scanf("%d", &i);
    char *p = (char*)malloc(i);
    char c;
    scanf("%c", &c);
    gets(p);
    
    puts(p);
    free(p);
    p = NULL;

    return 0;
}
```
否则 `gets` 获取不到标准输入。

### puts
输出字符串。

```c
puts()、
// 等价于 print_f("%s \n", p); 这里的p 要求是一个字符指针
// puts 访问的是字符指针的地址，然后遍历将里面的值依次 print 出来
```

## 数组

C 语言的数组。

使用C 语言的数组时，需要注意哪些问题？
1. 数组访问越界的问题
2. C 语言会对字符串常量，自动增加一个`\0`，所以当使用数组存在字符串常量时，需要注意数组的索引长度

字符串为什么需要有结束符？\
因为需要有一个结束符，能让C 语言知道这个字符在什么位置结束。

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20220509225407.png)

在Mac 下，数组一旦越界了，编译会过不了。

---
C 语言规定字符串的结束标记为 `\0`，系统会对字符串常量自动加一个 `\0`，所以字符数组存储的字符串长度必须比字符数组少 1 字节。

整型数组在传递实参时，需要一并把数组的长度给传过去。
而字符数组则不用，

## 函数

每一个函数在执行完成之后，都会被栈空间释放掉。

函数间的调用关系是，由主函数调用其他函数，其他函数之间也可以相互调用，同一个函数可以被一个或者多个函数调用 N 次。

函数的声明与定义是有区别的：
1. 函数的定义是指对函数功能的确立，包括指定函数名、函数值类型、形参及其类型、函数体等，它是一个完整的、独立的函数单位。
2. 函数的声明的作用是把函数的名字、函数类型及形参的类型、个数和顺序通知编译系统，以便在调试该函数时编译系统能正确识别函数并检查调用是否合法

C 语言的局部变量、全局变量和其他语言是很像的，没有太多需要注意的地方，只是在 C 语言中尽量不要使用全局变量，程序容易出错。

---
获取字符数组的索引长度使用 `sizeof`，获取字符数组的字符长度使用 `strlen`。

遍历一个字符数组时，尽管输出的是 `1`，但要明确它是一个字符类型，而不是整型。\
所以在进行比较时，需要时刻保证等式两边的数据类型是一致的。

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
   int ret[20];
   printf("%d \n", sizeof(ret));  // 80   
   printf("%d \n", strlen(ret));  // 0    索引长度
}  
```

在做 C 语言字符串拼接、替换相关的题目时，需要注意使用 `\0` 作为结束符。


`scanf` 会从缓冲区中读取对应的内容，直至遇到 `\n` 才会停止读取，不会读取 `\n`。\
而 `gets` 函数也是从缓冲区里面读取，遇到 `\n` 就结束。

所以使用完 `scanf` 之后，如果不主动消除 `\n`，直接使用 `gets` 会导致程序直接向下继续执行，因为 `gets` 读取到的是结束符 `\n`。

## 指针

指针的本质就是地址。

一个变量在内存中，可以分为两部分：编址（变量的地址）和具体的值。

如果想把某个变量的地址保存下来，就需要用到指针。

& 是取地址符号，也称为引用，通过该操作符可以获取一个变量的地址值。* 是取值操作符，也称为解引用，通过这个操作符可以获取一个地址对应的数据。

指针的使用场景总结下来只有两种：
1. 传递
2. 偏移

函数具有自己的内存空间，在某个函数中定义了一个变量之后，就会在这个内存中开辟对应大小的内存空间。\
值传递是不会改变原值的。

`&` 符号的作用是获取变量的地址，`*` 符号的作用是通过变量的地址获取对应的值。

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    int i = 5;    // 定义一个变量 i 值为 5
    int* p = &i;  // 定义一个整型指针变量，值为变量 i 的地址

    printf("i = %d \n", i);  // 直接访问
    printf("*p = %d \n", *p); // 通过取值操作符间接访问变量 p
    return 0;
}
```

---
字符数组的数组名里存的就是字符数组的起始地址。类型是字符指针。

数组名的类型是数组，里面存了一个值，就是数组的起始地址，

### 指针的传递

```c
#include <stdio.h>

void change(int *j)
{
    *j = 10;
}

int main(int argc, char const *argv[])
{
    int i = 5;

    printf("修改之前i = %d \n", i);  // 5

    change(&i);

    printf("修改之后i = %d \n", i);  // 10
    return 0;
}
```

### 指针的偏移

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    int arr[5] = {1, 2, 3, 4, 8};

    int* p;  // 定义一个整型指针变量
    p = arr;  // 因为数组的名称就是数组的起始地址，所以这里直接把对应地址赋值给整型指针 p
    printf("*p = %d \n", *p);  // 对一个指针变量进行取值时（需要取值操作符），得到的是其基类型

    // 指针的偏移
    for (int i = 0; i < 5; i++)
    {
        // 对指针进行偏移时，每次的偏移量是对应的基类型大小
        // 比如这里对指针 p 进行偏移（+ 1），其实对应的内存中的
        printf("%d \n", *(p + i));
    }
    
    return 0;
}
```

### 指针与一维数组

数组在传递时会弱化为指针：
```c
#include <stdio.h>

// 这两种写法是一样的
// void change(char d[])
// {
//     printf("%c \n", *d);   // h
// }

void change(char *d)
{
    printf("%c \n", *d);      // h
}

int main(int argc, char const *argv[])
{
    char c[10] = "hello";
    change(c);

    return 0;
}
```

一维数组在进行函数调用时，为什么它的长度子函数没有办法知道？\
这是因为一位数组的数组名存储的是数组的首地址（也就是索引为零的值的地址），压根就不是数组，所以没有办法直接知道对应长度。

```c
void change(char *d)
{
    *d = 'H';
    // 等价于
    // d[0] = 'H';
    // 这两种方式都可以改变数组内的值，不过需要注意的是，赋值表达式两边的类型需要保持一致
    // d[1] = 'E';
    // *(d+1) = 'E';
    printf("%c \n", *d);  // h
}
```

为什么在子函数内部可以对数组进行访问和修改？\
这里其实用到了指针的传递与偏移。

### 指针与动态申请内存
数组一开始定义好就确定下来了，数组是放在栈空间的。

栈空间的大小在编译时是确定的，如果使用大小不确定，那么就要使用堆空间。

申请堆空间，会把一个连续的 N 个字节的空间给你，返回的是一个起始地址。
`malloc` 申请空间的单位是字节。

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    char *p1;
    int *p2;
    p1 = (char*)malloc(20); // p1 最多可以存储 19 个字符
    p2 = (int*)malloc(20);  // p2 最多可以存在 5 个整型数
    return 0;
}
```

野指针是什么？

当一个指针指向一块空间，而这个空间又不属于它，这就是野指针。

## 结构体

使用结构体之前，需要先声明：
```c
struct student
{
    int num;
    char name[20];
    int age;
};
```

定义一个结构体变量：
```c
struct student student1 = {1001, "boo", 23};
```

定义一个结构体数组变量：
```c
struct student sarr[3] = {
    1001, "boo", 23,
};
```

`.` 操作符用来访问结构体变量，`->` 操作符用来访问指针的成员。

### 结构体指针
结构体指针就是结构体变量所占据的内存段的起始地址。

```c
#include <stdio.h>

int main(int argc, char const *argv[])
{
    // 定义一个结构体
    struct student student1 = {1002, "boo", 23};
    // 定义一个结构体指针
    struct student * p;
    p = &student1;

    // 访问结构体指针
    printf("%d %s %d \n", p->id, p->name, p->age);        // 通过指针的成员选择进行访问
    printf("%d %s %d \n", (*p).id, (*p).name, (*p).age);  // 通过指针对象的成员进行访问
}
```

### typedef

比如如果需要定义一个结构体变量，每次都需要写一个 `struct xxx`，这个显然是很麻烦的事情。

`typedef` 关键字的作用就是声明新的类型名来代替已有的类型名。

```c
#include <stdio.h>

// 声明一个结构体，顺带定义一个别名
typedef struct student
{
    int id;
    char name[20];
    int age;
} stu,* pstu;

// stu 等价于 struct student
// * pstu 等价于 struct student *
```

## C++引用
C++ 的引用其实就是在子函数中改变主函数的某个变量的值。\
C 也可以做到，只不过相比起来 C++ 的写法更简洁一些。

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// 修改整型
void modify_num(int &i)
{
    i = 5;
}

// 修改指针
void modify_pointer(int *&p)
{
    p = (int*)malloc(100);
    // 为什么这里可以直接给 p[0] 赋值？因为使用 malloc 申请的是一块连续的空间，返回的是这块空间的首地址。
    // p[0] = 9;
    *p = 9;       // 指针的传递
    *(p+1) = 2;   // 指针的偏移
}

int main(int argc, char const *argv[])
{
    int i = 1;
    modify_num(i);
    printf("i = %d \n", i);      // 5

    int *p = NULL;
    modify_pointer(p);
    printf("p[0] = %d \n", p[0]);  // 9
    printf("p[1] = %d \n", p[1]);  // 2
    return 0;
}
```