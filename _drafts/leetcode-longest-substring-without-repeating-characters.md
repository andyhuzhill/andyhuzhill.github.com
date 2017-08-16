---
layout: post
title: "LeetCode #3 Longest Substring Without Repeating Characters"
description: ""
category: algorithm
tags: [algorithm, leetcode]
---
{% include JB/setup %}

## 题目

Given a string, find the length of the **longest substring** without repeating characters.

**Examples:**

Given "abcabcbb", the answer is "abc", which the length is 3.

Given "bbbbb", the answer is "b", with the length of 1.

Given "pwwkew", the answer is "wke", with the length of 3. Note that the answer must be a **substring**, "pwke" is a subsequence and not a substring.

*翻译:*
  给出一个字符串，找出字符串中最长的无重复子串的长度。

## 解答

### Brute Force 暴力搜索

直接遍历字符串，找出每一位开始的最长无重复子串长度，计算最大值。

{% highlight cpp %}
class Solution {
public:
    bool uniqueString(string s, int startIdx, int endIdx) {
        unordered_set<char> charSet;
        for (int i = startIdx; i < endIdx; ++i) {
            if (charSet.find(s.at(i)) != end(charSet)) {
                return false;
            } else {
                charSet.emplace(s.at(i));
            }
        }
        return true;
    }

    int lengthOfLongestSubstring(string s) {
        int max_length = 0;
        for (int i = 0; i < s.length(); ++i) {
            for (int j = i + 1; j <= s.length(); ++j) {
                if (uniqueString(s, i, j)) {
                    max_length = max(max_length, j - i); 
                }
            }
        }
        return max_length;
    }
};
{% endhighlight %}

暴力搜索的时间复杂度是 $$ O(n^3) $$, 因此，在输入比较长的字符串时，该算法很容易就会 *Time Limit Exceeded*。

### 
