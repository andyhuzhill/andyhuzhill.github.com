---
layout: post
title: "LeetCode #4 Median of Two Sorted Arrays"
description: ""
category: algorithm
tags: [algorithm, leetcode]
---
{% include JB/setup %}

## 题目

There are two sorted arrays **nums1** and **nums2** of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

**Example 1:**

    nums1 = [1, 3]
    nums2 = [2]
    
    The median is 2.0

**Example 2:**

    nums1 = [1, 2]
    nums2 = [3, 4]
    
    The median is (2 + 3)/2 = 2.5

## 解答

如果不考虑题目要求的时间复杂度的话，我们可以先将两个数组合并为一个数组，然后找到中间数即可。

### Wrong Answer 1
{% highlight cpp %}
double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
    vector<int> all;
    for (auto iter = begin(nums1); iter != end(nums1); ++iter) {
        all.emplace_back(*iter);
    }
    for (auto iter = begin(nums2); iter != end(nums2); ++iter) {
        all.emplace_back(*iter);
    }
    sort(begin(all), end(all));
    
    if ((all.size() % 2) == 1) {
        return all.at(all.size() / 2);
    } else {
        return (all.at(all.size() / 2) + all.at(all.size() / 2 - 1)) / 2.0;
    }
}
{% endhighlight %}

### Wrong Answer 2
{% highlight cpp %}
double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
    vector<int> all;
    int i = 0;
    int j = 0;
    while ((i < nums1.size()) || (j < nums2.size())) {
        int num1 = (i < nums1.size()) ? nums1.at(i) : INT_MAX;
        int num2 = (j < nums2.size()) ? nums2.at(j) : INT_MAX;
        if (num1 < num2) {
            all.emplace_back(num1);
            i++;
        } else if (num1 > num2) {
            all.emplace_back(num2);
            j++;
        } else {
            all.emplace_back(num1);
            all.emplace_back(num2);
            i++;
            j++;
        }
    }

    if ((all.size() % 2) == 1) {
        return all.at(all.size() / 2);
    } else {
        return (all.at(all.size() / 2) + all.at(all.size() / 2 - 1)) / 2.0;
    }
}
{% endhighlight %}

### 递归方法 

　要解决该问题，我们首先要理解中位数有什么用。在统计学中，中位数用于将一个数据集分割成两个长度相等的子集，其中一个子集总是大于另一个。

　如果我们理解了中位数是用于分割，我们就接近答案了。

　首先，我们在Ａ中随机位置i将Ａ分割为两个部分。

```
      left_A             |        right_A
A[0], A[1], ..., A[i-1]  |  A[i], A[i+1], ..., A[m-1]
```

因为A有m个元素，所以这里就有 m+1 中分割方式(i = 0 ~ m)

同样的我们可以将Ｂ也使用一个随机位置ｊ分割为两个部分。将 leftA和leftB 放到一个集合，rightA和rightB放入另一个集合，我们把他们称为 left_part 和 right_part.

```
         left_part          |        right_part
   A[0], A[1], ..., A[i-1]  |  A[i], A[i+1], ..., A[m-1]
   B[0], B[1], ..., B[j-1]  |  B[j], B[j+1], ..., B[n-1]
```

如果我们能保证

1. len(left_part)=len(right_part)
2. $$\max(\text{left\_part}) \leq \min(\text{right\_part})max(left_part)≤min(right_part)$$

然后，我们将所有在{A, B}中的元素分割为长度相等的两个部分，而且其中一部分总是大于另一部分。则有

　$$ median = \frac{max(left_part) + min(right_part)}{2} $$

要满足这两个条件，我们只需要保证

1. i+j=m−i+n−j (or: m - i + n - j + 1m−i+n−j+1)
   if $$ n \geq m $$, we just need to set:  $$ i=0∼m, j=\frac{m+n+1}{2}−i $$

2. $$\text{B}[j-1] \leq \text{A}[i] and \text{A}[i-1] \leq \text{B}[j]$$

