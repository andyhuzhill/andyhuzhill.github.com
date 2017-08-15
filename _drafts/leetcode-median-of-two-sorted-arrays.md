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