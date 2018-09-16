---
layout: post
title:  "LeetCode #3 无重复字符的最长子串 Longest Substring Without Repeating Characters"
date:   2018-09-16 23:12:20 +0800
categories: 算法
---

## LeetCode #3 无重复字符的最长子串 Longest Substring Without

给定一个字符串，找出不含有重复字符的最长子串的长度。

**示例 1**:

```
输入: "abcabcbb"
输出: 3 
解释: 无重复字符的最长子串是 "abc"，其长度为 3。
```

**示例 2**:

```
输入: "bbbbb"
输出: 1
解释: 无重复字符的最长子串是 "b"，其长度为 1。
```

**示例 3**:

```
输入: "pwwkew"
输出: 3
解释: 无重复字符的最长子串是 "wke"，其长度为 3。
     请注意，答案必须是一个子串，"pwke" 是一个子序列 而不是子串。
```

一般对这类题目，都需要保存我们的遍历信息。我们可以使用一定的内存辅助，在一个哈希表中存储出现过的字符和它最近一次出现的角标，遍历时如果在哈希表中发现了该字符已存在，就说明之前肯定存在过，我们再利用已存储的当前子字符串的起始角标，以及当前已发现的最大子字符串的长度，就可以一次遍历而求出答案。

对哈希表而言，由于字符数量有限，即128个，所以哈希表的空间复杂度最大是128，最小是使用到的字符数量（实际上是大于或等于这个数量的2的幂）。另外，可以利用字符数量恒定为128这个特性，直接初始化一个`int`型的数组，`char`的值即为角标，存储的是该字符在字符串中最近一次出现的角标加1，因为`int`型数组中元素初始化值为0，所以就给加了个1。

针对以上分析，可以很快写出以下代码：

```java
class Solution {
    public int lengthOfLongestSubstring(String s) {
        if (s == null || s.length() == 0) {
			return 0;
		}
		int[] index = new int[128];
		int maxLength = 0;
		int curIndex = 0;
		for (int i = 0; i < s.length(); i++) {
			curIndex = Math.max(curIndex, index[s.charAt(i)]);
			maxLength = Math.max(maxLength, i - curIndex + 1);
			index[s.charAt(i)] = i + 1;
		}
		return maxLength;
    }
}
```