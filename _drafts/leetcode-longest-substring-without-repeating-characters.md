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

### Brute-Force 暴力搜索

直接遍历字符串，查找所有无重复的子串，计算出最长的长度。

{% highlight cpp %}
class Solution {
public:
  bool uniqueString(string s, int startIdx, int endIdx) {
    unordered_set<char> charSet; // 使用一个 Set 保存字符
    for (int i = startIdx; i < endIdx; ++i) {
      if (charSet.find(s.at(i)) != end(charSet)) { 
        // 当 Set 中已经有该字符了, 代表有重复的字符，返回 false
        return false;
      } else {
        charSet.emplace(s.at(i));
      }
    }
    // 执行到这里，说明没有重复的字符，返回 true
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

该暴力算法的时间复杂度是 $$ O(n^3) $$, 在计算比较长的字符串时，会遇到*Time Limit Exceeded* 错误。

### 滑动窗口

暴力法很直观，但是太慢了。在暴力法中，我们重复的检测子串时候有重复的字符，如果一个子串 $$S_{ij}$$ 已经检查了 i 到 j-1 位的字符没有重复，那么我们就只需要检查　j 位的字符是否在子串 $$ S_{ij}$$ 中就可以了。

{% highlight cpp %}
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
      int n = s.length();
      unordered_set<char> charSet;
      int ans = 0;
      int i = 0; 
      int j = 0;
      while (i < n && j < n) {
        if (charSet.find(s.at(j)) == end(charSet)) {
          charSet.emplace(s.at(j++));
          ans = max(ans, j - i);
        } else {
          charSet.erase(s.at(i++));
        }
      }
      return ans;
    }
};
{% endhighlight %}

### 哈希表保存字符位置

用一个哈希表保存字符的位置。
一个变量保存子串的起始位置。如果查找到表中的字符位置在子串起始位置前面，说明没有重复字符，如果在子串起始位置后面，说明有重复字符，修改起始位置，计算子串长度。

{% highlight cpp %}
class Solution {
public:
  int lengthOfLongestSubstring(string s) {
    unordered_map<char, int> pos_map;
    int start = 0;
    int max_length = 0;
    for (int i = 0; i < s.length(); ++i) {
      char ch = s.at(i);

      auto iter = pos_map.find(ch);

      if (iter != end(pos_map)) {
        if (start <= (*iter).second) {
          start = (*iter).second + 1;
        }
        pos_map.erase(iter); // 删除之前的字符所在的位置
      }
      max_length = max(max_length, i - start + 1);
      pos_map.insert(make_pair(ch, i)); 
    } 
    return max_length;
  }
};
{% endhighlight %}