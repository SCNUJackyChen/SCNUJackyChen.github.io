---
title: LC weekly contest 293
date: 2022-05-21 15:15:38
tags: [Leetcode, 模拟, 区间合并]
categories: Leetcode周赛
katex: true
description:  LC 周赛第293场题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **Summary**

T3一开始往Trie树那边去想了然后就掉进坑里挣扎了好久，最后发现通过人数多得离谱，重新审视了一下题目，发现想复杂了，梦中惊醒，然后就过了。T4思路比较简单，但是没有时间写完。好好反思一下T3为啥想错了555(ಥ _ ಥ)

###  **Solution**

#### [移除字母异位词后的结果数组](https://leetcode.cn/problems/find-resultant-array-after-removing-anagrams/)

判断相邻2个单词是否是异位词，可以通过哈希表来做。


##### **Code Q1**
```cpp
class Solution {
public:
    vector<string> removeAnagrams(vector<string>& words) {
        vector<string> res;
        unordered_map<char, int> hash;
        
        for (string word : words) {
            unordered_map<char, int> t;
            for (char c : word) t[c] ++;
            bool f = false;
            for (auto [c, v] : t) {
                if (!hash.count(c) || hash[c] != v){
                    f = true;
                    break;
                }
            }
            if (t.size() != hash.size()) f = true;
            hash = t;
            
            if (f) res.push_back(word);
        }
        return res;
    }
};
```
#### [不含特殊楼层的最大连续楼层数](https://leetcode.cn/problems/maximum-consecutive-floors-without-special-floors/)

从楼底到楼顶依次计算连续楼层的区间，更新答案即可。

##### **Code Q2**
```cpp
class Solution {
public:
    int maxConsecutive(int bottom, int top, vector<int>& special) {
        int res = 0;
        int last = bottom - 1;
        sort(special.begin(), special.end());
        for (int x : special) {
            res = max(res, x - last - 1);
            last = x;
        }
        res = max(res, top - last);
        return res;
    }
};
```
#### [按位与结果大于零的最长组合](https://leetcode.cn/problems/largest-combination-with-bitwise-and-greater-than-zero/)

涉及位运算的题目一般要按位来看。对于每一位bit，按位与的结果只有0和1，如果要让两个数的按位与大于0，则需要至少1位数字相同。我们可以枚举每一位，然后统计该位上的1的个数，答案应该取决于所有位上1的个数的最大值。

##### **Code Q3**
```cpp
class Solution {
public:
    int largestCombination(vector<int>& A) {
        int n = A.size(), m = 32;
        vector<vector<int>> f(n + 1, vector<int>(m, 0));
        for (int i = 0; i < n; i++) {
            for (int j = 0; j <31; j++) {
                if (A[i] >> j & 1) {
                    f[i][j] = 1;
                }
            }
        }
        
        int res = 0;
        for (int i = 0; i < 32; i++) {
            int t = 0;
            for (int j = 0; j < n; j++) {
                if (f[j][i]) t++;
            }
            res = max(res, t);
        }
        return res;
    }
};
```
#### [统计区间中的整数数目](https://leetcode.cn/problems/count-integers-in-intervals/)

本题是一道区间合并的模板题，直接用map模拟即可。

##### **Code Q4**

```cpp
class CountIntervals {
public:

    map<int, int> hash;
    int cnt;

    CountIntervals() {
        hash.clear();
        cnt = 0;
    }
    
    void add(int left, int right) {
        int L = left, R = right;
        auto it = hash.lower_bound(left - 1); // find the cloest interval whose right end point is greater or equal to "left"
        while (it != hash.end()) { // merge all intervals in [L, R], and maintain the answer
            int r = it->first, l = it->second;
            if (l > R) break;
            L = min(L, l);
            R = max(R, r);
            cnt -= r - l + 1;
            hash.erase(it++);
        }
        cnt += R - L + 1;
        hash[R] = L;
    }
    
    int count() {
        return cnt;
    }
};

/**
 * Your CountIntervals object will be instantiated and called as such:
 * CountIntervals* obj = new CountIntervals();
 * obj->add(left,right);
 * int param_2 = obj->count();
 */
```