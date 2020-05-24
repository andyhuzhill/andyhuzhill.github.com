---
layout: post
title: "中位数选取问题"
description: ""
category: ["algorithm"] 
tags: []
---
{% include JB/setup %}

## 问题定义：

在`O(n)`时间内，在一个长度为n的数组Ｘ中找到x,使得恰有 $$\lfloor \frac{n}{2} \rfloor $$ 个元素小于x.

问题可以更一般化为”选取第ｉ大的元素“问题

从一个长度为n的数组X中找到一个ｘ，使得恰有 i-1 个元素小于ｘ。

## 求解步骤

1. 将数组分组，每组５个数，最后一组可能少于５个数
2. 对上面每组数进行排序，选出每组元素的中位数
3. 递归调用算法求得这些中位数的中位数 (MoM)
4. 使用MoM将原数组X划分为小于MoM的一组和大于MoM的一组
5. 划分完成后，设MoM的下标为k,
   如果 i = k, 则返回ｋ
   如果 i < k, 则在第一个部分递归选取第 i 大的数
   如果 i > k, 则在第三个部分递归选取第 (i-k) 大的数

```
算法Select(A,i)
Input: 数组A[1:n], 1 i n
Output: A[1:n]中的第i-大的数

 1. for j <- 1 to n/5
 2. InsertSort(A[(j-1)*5+1 : (j-1)*5+5]);
 3. swap(A[j], A[[(j-1)*5+3]);
 4. x <- Select(A[1: n/5], n/10 );
 5. k <- partition(A[1:n], x);
 6. if k=i then return x;
 7. else if k>i then retrun Select(A[1:k-1],i);
 8. else retrun Select(A[k+1:n],i-k);
 ```