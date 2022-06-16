---
title: LC biweekly contest 80
date: 2022-06-16 18:01:20
tags: [Leetcode, 滑动窗口, 哈希, 前缀和, 二分]
categories: Leetcode周赛
katex: true
description:  🎉解锁新成就 Knight🎉
---

![ranking](/images/LC-biweekly-contest-80/ranking.png)

<!--more-->


###  **Preface**

从去年开始参加断断续续参加了60场Leetcode周赛，终于在这次解锁了Knight勋章！一年前偶然间重新拾起算法竞赛，没想到已经坚持了这么久。这个博客也记录了历次周赛的总结和成长，回头看起来还真是感慨万千。以后进入职场了或许没有太多的空闲时间来做题了，但如果有时间肯定会继续写题解。Knight往上是Guardian，大概需要2200分以上，很遥远，但希望有朝一日也可以摘下那颗闪耀的星星~




###  **Solution**

#### [替换字符后匹配](https://leetcode.cn/problems/match-substring-after-replacement/)

由于`sub`串的长度是固定的，而替换后的字符串长度不变，因此可以直接用大小为`sub`长度的滑窗遍历一遍`s`串，同时判断每个窗口内的`s`子串是否能够由`sub`替换得到。时间复杂度为$O(n^2)$，由于长度最多只有$5000$，因此是满足要求的。

至于如何快速判断是否有从字符`a`到`b`的转换操作，我的做法是存一个长度为2的字符串`ab`，并且存到哈希表里面方便查询。

P.S. 比赛的时候在构造字符串时采用了to_string(a) + b的写法，然后超时...，改成两次push_back就过了。这个故事告诉我们字符串相加的效率是很慢的。

##### **Code Q3**

```cpp
class Solution {
public:
    bool matchReplacement(string s, string sub, vector<vector<char>>& mappings) {
        unordered_set<string> hash;
        for (auto& p : mappings) {
            string a;
            a.push_back(p[0]);
            a.push_back(p[1]);
            hash.insert(a);
        }
        int m = sub.size(), n = s.size();
        for (int i = 0; i + m <= n; i++) {
            int j = 0;
            for (int k = i; j < m; j++, k++) {
                if (s[k] != sub[j]) {
                    string a;
                    a.push_back(sub[j]);
                    a.push_back(s[k]);
                    if (!hash.count(a)) {
                        break;
                    }
                }
            }
            
            if (j == m) return true;
        }
        
        return false;
    }
};
```

#### [统计得分小于 K 的子数组数目](https://leetcode.cn/problems/count-subarrays-with-score-less-than-k/)

涉及到区间和，考虑采用前缀和预处理。根据题意，分数的大小随着长度的增加而变大，具有单调性，因此可以用二分来优化。假设左右端点为$l, r$，$[l,r]$表示以$l$为左端点，区间得分小于$k$的最长区间。可以枚举左端点，右端点可以二分得到,由于$[l,l+1]$到$[l,r]$都是满足条件的答案，因此累加所有的区间长度即为最终答案。

##### **Code Q4**

```cpp
class Solution {
public:
    long long countSubarrays(vector<int>& nums, long long k) {
        int n = nums.size();
        vector<long long> s(n + 1, 0);
        
        for (int i = 1; i <= n; i++) s[i] = s[i - 1] + nums[i - 1];
        
        long long res = 0;
        
        for (int i = 0; i <= n; i++) {
            int l = i, r = n;
            while (l < r) {
                int mid = l + r + 1 >> 1;
                if ((long long)(s[mid] - s[i]) * (mid - i) < k) l = mid;
                else r = mid - 1;
            }
            res += l - i;
        }
        return res;
    }
};
```
