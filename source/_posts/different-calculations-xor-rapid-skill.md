---
title: 异或运算 XOR 快速上手
date: 2022-10-31 23:31:38
tags: ["算法"]
categories: ["算法"]
---

运算符有多种类型，常见的有：
* 算术运算符
* 关系运算符
* 逻辑运算符
* 位运算
* ...

在位运算中，异或运算虽然不常用，但是也非常重要，某些场景下，使用异或运算非常方便。

<!-- more -->

## 概念
位运算符作用于位，并逐位执行操作。&、 | 和 ^ 的真值表如下所示：

| p    | q    | p & q（与运算） | p \| q（或运算） | p ^ q（异或运算） |
| :--- | :--- | :---- | :----- | :---- |
| 0    | 0    | 0     | 0      | 0     |
| 0    | 1    | 0     | 1      | 1     |
| 1    | 1    | 1     | 1      | 0     |
| 1    | 0    | 0     | 1      | 1     |

**与运算（AND）** 和 **或运算（OR）** 都比较好理解，所以这篇笔记重点介绍 **异或运算（XOR）**。

异或，英文为 exclusive OR，缩写成xor，**XOR 主要用来判断两个值是否不同**。

## 运算法则

归零律，一个值与自身的运算，总是为 0：
```
x ^ x = 0
```

恒等律，一个值与 0 的运算，总是等于其本身：
```
x ^ 0 = x
```

交换律：
```
x ^ y = y ^ x
```

结合律：
```
x ^ y ^ z = x ^ (y ^ z) = (x ^ y) ^ z
```

自反：
```
x ^ y ^ x = y
```

## 应用
根据上面的这些运算法则，可以得到异或运算的很多重要应用。

### 简化计算

多个值的异或运算，可以根据运算定律进行简化。
```
x ^ y ^ z ^ y ^ x
= x ^ x ^ y ^ y ^ z
= 0 ^ 0 ^ z
= z
```

### 交换值

两个变量连续进行三次异或运算，可以互相交换值。

假设两个变量是x和y，各自的值是a和b。下面就是x和y进行三次异或运算：
```php
x = x ^ y;  // 第一次运算之后，x 的值是 a ^ b，y 的值是 b
y = x ^ y   // 第二次运算之后，x 的值是 a ^ b，y 的值是 a ^ b ^ b，也就是 a
x = x ^ y   // 第三次运算之后，x 的值是 a ^ b ^ a，y 的值是 a

// 所以最终 x 的值是 b， y 的值是 a
```

### 加密
异或运算可以用于加密。

第一步，明文（text）与密钥（key）进行异或运算，可以得到密文（cipherText）。

```
text ^ key = cipherText
```

第二步，密文与密钥再次进行异或运算，就可以还原成明文。

```
cipherText ^ key = text
```

原理很简单，如果明文是 x，密钥是 y，那么 x 连续与 y 进行两次异或运算，得到自身。

```
(x ^ y) ^ y
= x ^ (y ^ y)
= x ^ 0
= x
```

### 数据备份

异或运算可以用于数据备份。

文件 x 和文件 y 进行异或运算，产生一个备份文件 z。

```
x ^ y = z
```

以后，无论是文件 x 或文件 y 损坏，只要不是两个原始文件同时损坏，就能根据另一个文件和备份文件，进行还原。

```
x ^ z
= x ^ (x ^ y) 
= (x ^ x) ^ y
= 0 ^ y
= y
```

上面的例子是 y 损坏，x 和 z 进行异或运算，就能得到 y。

## 一道算法题
[只出现一次的数字——LeetCode](https://leetcode.cn/problems/single-number/)

给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

要求：算法的复杂度是线性的，且不允许使用额外的空间。

示例：
```
输入: [4,1,2,1,2]
输出: 4
```

解题思路：这里就可以利用异或运算的自反特性，可以将所有相同的数字全部抵消掉，最后留下的就是只出现了一次的元素：
```php
function func($array)
{
    $result = 0;
    for ($i = 0; $i < count($array); $i++) {
        $result ^= $array[$i];
    }

    return $result;
}

$array = [4, 1, 2, 1, 4];
$result = func($array);
print_r($result);   // 0 ^ 4 ^ 1 ^ 2 ^ 1 ^ 4 = 2
```

## 参考链接
* [异或运算 XOR 教程](https://www.ruanyifeng.com/blog/2021/01/_xor.html)