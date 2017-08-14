---
layout: post
title: "LeetCode #1 Two Sum"
description: ""
category: algorithm
tags: [algorithm, leetcode]
---
{% include JB/setup %}

* TOC 
{:toc}

## 题目

Given an array of integers, return indices of the two numbers such that they add up to a specific target.

You may assume that each input would have exactly one solution, and you may not use the same element twice.

**Example:**

    Given nums = [2, 7, 11, 15], target = 9,

    Because nums[0] + nums[1] = 2 + 7 = 9,
    return [0, 1].

*翻译*:
  给出一个整数数组，找出数组中相加和为指定目标的两个数在数组中的索引。

## Brute-Force 暴力法

最简单的想法就是一个二重循环，找到两个数相加之和为目标数。

{% highlight cpp %}
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        for (int i = 0; i < nums.size() - 1; ++i) {
            for (int j = i + 1; j < nums.size(); ++j) {
                if ((nums[i] + nums[j]) == target) {
                    return vector<int>{i, j};
                }
            }
        }
    }
};
{% endhighlight %}

显而易见，暴力法的最差时间复杂度是 $$O(n^2)$$ 。

## HashMap 哈希表查询法

我们分析暴力法可以知道，数组中的每一个元素都被访问了n次。如果我们在第一次访问的时候就记录下该元素的值和索引，就不用再次使用循环去遍历数组了。于是我们可以使用一个 HashMap 来存储该信息。

{% highlight cpp %}
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> pos_map;

        for (int i = 0; i < nums.size(); ++i) {
            auto result = pos_map.find(target - nums[i]);
            if (result != end(pos_map)) {
                return vector<int>{i, (*result).second};
            } else {
                pos_map.emplace(make_pair(nums[i], i));
            }
        }
    }
};
{% endhighlight %}

unordered_map 是 C++11 标准中新增加的关联容器，它的内部实现是哈希表，它的查询时间复杂度是 $$O(1)$$。因此可以计算出使用 unordered_map 之后，算法时间复杂度从 $$O(n^2)$$ 降到了 $$O(n)$$。