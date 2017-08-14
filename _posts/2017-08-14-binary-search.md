---
layout: post
title: "Binary Search二分法查找"
description: ""
category: algorithm
tags: [algorithm]
---
{% include JB/setup %}

* TOC 
{:toc}


二分法查找是一种在有序的序列中查找元素的有效方法。

最坏时间复杂度是  O(log(n)) , 最好时间复杂度是 O(1)。


## 朴素查找法

如果一个序列是无序的，就不能使用二分查找了，只能通过一个一个遍历序列来查找。

{% highlight cpp %}
template<typename T>
int naive_search(T key, std::vector<T> data)
{
    for (int i = 0; i < data.size(); ++i) {
        if (key == data[i]) { // 找到元素，返回元素在序列中的位置
            return i;
        }
    }
    return -1; // 未找到元素，返回-1
}
{% endhighlight %}

## 二分查找思想

二分查找，首先要保证查找的序列是有序的。

* 第一步, 找出序列中间值，将序列分为左半边和右半边。
* 第二步，比较中间值与要查找的键值.
  如果相等，即找到对象，返回中间值索引即可。
  如果中间值小于键值，将左边界更新为中间值，继续在右半边重复第一到第二步。 
  如果中间值大于键值, 将右边界更新为中间值，继续在左半边重复第一到第二步。

反复执行以上循环直到找到键值或者序列为空(未找到)。


## 递推方式实现

{% highlight cpp %}
/**
 *  return the index of key in sorted vector data,
 *  return -1 if not found
 */
template<typename T>
int binary_search(T key, const std::vector<T> data)
{
    int left = 0;
    int right = data.size() - 1;
    int mid = 0;
    while (left <= right) {
        mid = (left + right) / 2;
        if (data[mid] < key) {
            left = mid + 1;
        } else if (data[mid] > key) {
            right = mid - 1;
        } else {
            return mid;
        }
    }

    // not found return -1
    return -1; 
}
{% endhighlight %}

## 递归方式实现

{% highlight cpp %}
template<typename T>
int binary_search_subrecursion(T key, int left, int right, const std::vector<T> data) 
{
    if (left > right) {
        // not found return -1
        return -1;
    }
    
    int mid = (left + right) / 2;

    if (data[mid] == key) {
        return mid;
    } else if (data[mid] < key) {
        return binary_search_subrecursion(key, mid + 1, right, data);
    } else if (data[mid] > key) {
        return binary_search_subrecursion(key, left, mid - 1, data);
    }
}

/**
 *  return the index of key in sorted vector data,
 *  return -1 if not found
 */
 template<typename T>
 int binary_search_recursion(T key, const std::vector<T> data) 
 {
    return binary_search_subrecursion(key, 0, data.size() - 1, data);
 }
{% endhighlight %}


