---
title: LC 双周赛第57场
date: 2021-07-25 12:25:05
tags:  [Leetcode, 模拟, 数据结构, 差分, 前缀和, 单调栈]
categories: Leetcode周赛
katex: true
description:  LC双周赛第57场复盘+题解
---
![LC](/images/Leetcode.jpg)

<!--more-->

### **题解**

#### [检查是否所有字符出现次数相同](https://leetcode-cn.com/problems/check-if-all-characters-have-equal-number-of-occurrences/)
简单模拟即可。

##### **Code Q1**
```cpp
class Solution {
public:
    bool areOccurrencesEqual(string s) {
        unordered_map<char, int> cnt;
        for (auto c : s) {
            if (!cnt.count(c)) cnt[c] = 1;
            else cnt[c]++;
        }
        int a = -1;
        for (auto [c, v] : cnt) {
            if (a == -1) {
                a = v;
            } else if (v != a) {
                return false;
            }
        }
        return true;
    }
};
```
#### [最小未被占据椅子的编号](https://leetcode-cn.com/problems/the-number-of-the-smallest-unoccupied-chair/)
本质上也是一个模拟，但是顺便考察了set，如果直接暴力枚举会超时。
在实现上有一个小trick，在插入集合时，如何判断一个人在某个时间点是到达还是离开？
这里可以将离开时间对应的id变成相反数-1，作为标识，在遍历时通过正负号就可以判断某个时间是进入还是离开。

##### **Code Q2**
```cpp
class Solution {
public:
    int smallestChair(vector<vector<int>>& times, int target) {
        int n = times.size();
        set<pair<int, int>> S;
        for (int i = 0; i < n; i++) {
            auto t = times[i];
            S.insert({t[0], i});
            S.insert({t[1], -i - 1});
        }
        vector<int> res(n);
        set<int> chair;
        for (int i = 0; i < n; i++) chair.insert(i);
        for (auto [c, s] : S) {
            if (s >= 0) {
                res[s] = *chair.begin();
                chair.erase(chair.begin());
            } else {
                chair.insert(res[-s - 1]);
            }
        }
        return res[target];
    }
};
```
#### [描述绘画结果](https://leetcode-cn.com/problems/describe-the-painting/)
本题可以把题意抽象为：
给定若干组操作，每组操作可以将一段区间所有元素加上某个数。问操作完之后一共有多少段不同的区间，并把这些区间及其区间和输出。

对于区间加减可以利用差分的方法线性时间进行操作 ($O(n)$)
对于找出所有区间，首先记录下所有区间端点然后去重，可以得到每个分割点
对于求区间和，只需要预先对差分数组再做一遍前缀和即可。

##### **Code Q3**
```cpp
using LL = long long;
const int N = 100010;
class Solution {
public:
    vector<vector<long long>> splitPainting(vector<vector<int>>& seg) {
        vector<LL> s(N + 1);
        vector<int> pos;
        int n = seg.size();
        for (int i = 0; i < n; i++) {
            auto t = seg[i];
            s[t[0]] += t[2];
            s[t[1]] -= t[2];
            pos.push_back(t[0]);
            pos.push_back(t[1]);
        }
        // 去重
        sort(pos.begin(), pos.end());
        pos.resize(unique(pos.begin(), pos.end()) - pos.begin());
        // 前缀和
        for (int i = 1; i <= N; i++) s[i] += s[i - 1];

        vector<vector<LL>> res;
        for (int i = 0; i + 1 < pos.size(); i++) {
            if (s[pos[i]])
                res.push_back({pos[i], pos[i + 1], s[pos[i]]});
        }
        return res;
    }
};
```
#### [队列中可以看到的人数](https://leetcode-cn.com/problems/number-of-visible-people-in-a-queue/)
经典的单调栈问题。
采用从后往前遍历的方式， 维护一个单调增的栈，栈元素代表高度。
 - 核心点：如果当前高度高于栈顶，则说明栈顶元素对于后续元素没有任何作用（直观看就是被当前这个人挡住了，所以栈顶这个人不再可能被左边的人看到），所以可以弹栈。这样始终维护了一个单调增的栈。在计算答案时，需要找到右边第一个高于自己的人，并且中间的人的身高是递增的（单调栈保证这一点），统计一下答案即可。


##### **Code Q4**
```cpp
class Solution {
public:
    vector<int> canSeePersonsCount(vector<int>& h) {
        int n = h.size();
        stack<int> stk;
        vector<int> res(n, 0);
        for (int i = n - 1; i >= 0; i--) {
            while(stk.size() && stk.top() < h[i]) {
                res[i] ++;
                stk.pop();
            }
            res[i] += !stk.empty(); // 算上边界
            stk.push(h[i]);
        }
        return res;
    }
};
```