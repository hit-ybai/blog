---
layout: post
title: "LeetCode: Longest Substring Without Repeating Characters 解题报告"
date: 2015-06-20 00:20:27 +0800
comments: true
categories: 解题报告
tags:
- leetcode
- hash
---
[题目描述](https://leetcode.com/problems/longest-substring-without-repeating-characters/):
*Given a string, find the length of the longest substring without repeating characters. For example, the longest substring without repeating letters for "abcabcbb" is "abc", which the length is 3. For "bbbbb" the longest substring is "b", with the length of 1.*

---
分析：找到最长的不包含相同字母的字串

题解：用一个队列来存储不含相同字母的子串，用一个hash来标记队列中已经使用过的字符。

1. 当发现即将入队的字符未被使用过时，入队，并在hash中标记下来；
2. 当发现即将入队的字符已经被使用过时，记下当前队列中字串的长度，取其与历史最大长度之间的最大值作为新的解。之后将队首的字符逐一出队，并在hash中清除标记，直道清除了即将入队的字符为止，最后字符入队。
3. 循环这个过程，直到所有的字符都被处理完成。

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        queue<char> current_sub_str;
        int sub_string_max_length = 0;
        
        for (auto it = s.begin() ; it < s.end(); it++) {
            char current_alphabet = *it;
            
            if (isAlphabetExist(current_alphabet)) {
                syncMaxValue( sub_string_max_length, current_sub_str.size() );
                popSubStringUntilAlphabet( current_alphabet, current_sub_str );
            } else {
                markAlphabet(current_alphabet);
            }
            
            current_sub_str.push(current_alphabet);
        }
        syncMaxValue( sub_string_max_length, current_sub_str.size() );
        return sub_string_max_length;
    }
private:
    static const int SIZE_OF_ASCII    = 128;
    bool alphabet_flag[SIZE_OF_ASCII] = { false };
    
    bool isAlphabetExist(const char alphabet) const {
        return alphabet_flag[int(alphabet)];
    }
    void syncMaxValue(int &sub_string_max_length, const size_t current_sub_string_length) const {
        if (current_sub_string_length > sub_string_max_length) {
            sub_string_max_length = int(current_sub_string_length);
        }
    }
    void popSubStringUntilAlphabet(const char alphabet, queue<char> &current_sub_str) {
        for ( char current_alphabet = current_sub_str.front(); current_alphabet != alphabet;
            alphabet_flag[int(current_alphabet)] = false,
            current_sub_str.pop(),
            current_alphabet = current_sub_str.front()
        );
        current_sub_str.pop();
    }
    void markAlphabet(const char alphabet) {
        alphabet_flag[int(alphabet)] = true;
    }
};
```