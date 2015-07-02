---
layout: post
title: "LeetCode: Two Sum 解题报告"
date: 2015-06-19 23:55:27 +0800
comments: true
categories: 解题报告
tags:
- leetcode
- hash
---
[题目描述](https://leetcode.com/problems/two-sum/):
*Given an array of integers, find two numbers such that they add up to a specific target number.*

*The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2. Please note that your returned answers (both index1 and index2) are not zero-based.*

*You may assume that each input would have exactly one solution.*

*Input: numbers={2, 7, 11, 15}, target=9*
*Output: index1=1, index2=2*

---
分析：题中要求找到数组`number`中“和”为`target`的两个元素对应的索引。

题解：首先构造一个`Hash{ value->idx }`，之后遍历每个元素`Numbers[i]`对应的`Target - Numbers[i]`是否在`Hash Table`中，若存在则输出解。
```cpp
#include <vector>
#include <unordered_map>

class Solution {
public:
    vector<int> twoSum(vector<int> &numbers, int target) {
        std::unordered_map<int, int> num_idx_map;
        auto n = numbers.size();
        for (int i = 0; i < n; i++) num_idx_map[numbers[i]] = i + 1;

        for (int i = 0; i < n - 1; i++) {
            if ( num_idx_map.find(target - numbers[i]) != num_idx_map.end() ) {
                int current_num_idx = i + 1;
                int match_num_idx   = num_idx_map[target - numbers[i]];

                if (match_num_idx != current_num_idx) return { current_num_idx,  match_num_idx };
            }
        }
    }
};
```