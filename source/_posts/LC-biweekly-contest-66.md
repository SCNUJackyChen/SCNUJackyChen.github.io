---
title: LC biweekly contest 66
date: 2021-11-28 16:38:38
tags: [Leetcode, 贪心, 模拟, DP]
categories: Leetcode周赛
katex: true
description:  LC双周赛第66场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->



### **Solution**

#### [统计出现过一次的公共字符串](https://leetcode-cn.com/problems/count-common-words-with-one-occurrence/)

##### **Code Q1**
```cpp
class Solution {
public:
    
    int countWords(vector<string>& words1, vector<string>& words2) {
        unordered_map<string, int> hash1, hash2;
        for (auto& s: words1) hash1[s] += 1;
        for (auto& s: words2) hash2[s] += 1;
        int res = 0;
        for (auto& [c, s]: hash1) {
            if (s == 1 && hash2[c] == 1) 
                res ++;
        }
        return res;
    }
};
```

#### [从房屋收集雨水需要的最少水桶数](https://leetcode-cn.com/problems/minimum-number-of-buckets-required-to-collect-rainwater-from-houses/)

For each house, firstly look around its neighbors and find out whether a bucket has been placed, if not, try to put the bucket on the right side, if fails (resulting from another house exists or wall), try to put on the left side, if fails, then it is impossible to put bucket next to this house.

对于每一间屋子，首先看一下邻居有没有水桶，如果没有，先尝试在右边放水桶，如果失败（因为右边已经有屋子，或者是墙），则放在左边，如果也失败，说明这间屋子左右都不能放水桶。
##### **Code Q2**
```cpp
class Solution {
public:
    int minimumBuckets(string s) {
        int n = s.size();
        int res = 0;
        if (n == 1) return s[0] == '.' ? 0 : -1;
        for (int i = 0; i < n; i++) {
            if (s[i] == 'H') {
                if (i + 1 < n) {
                    if (s[i + 1] == 'B' || i > 0 && s[i - 1] == 'B') continue;
                    if (s[i + 1] == '.') s[i + 1] = 'B';
                    else if (i > 0 && s[i - 1] == '.') s[i - 1] = 'B';
                    else return -1;
                } else {
                    if (s[i - 1] == 'B') continue;
                    if (i > 0 && s[i - 1] == '.') s[i - 1] = 'B';
                    else return -1;
                }
            }
        }
        for (char c : s) 
            if (c == 'B')
                res ++;
        return res;
    }
};
```

#### [网格图中机器人回家的最小代价](https://leetcode-cn.com/problems/minimum-cost-homecoming-of-a-robot-in-a-grid/)

We can always move vertically and then move horizontally from the start postion to home position, like the following figure.

我们总是可以从起点先垂直移动然后再左右平移到目的地，像下图所示。
![move](/images/LC-biweekly-contest-66/Q3.png)

##### **Code Q3**
```cpp
class Solution {
public:
    int minCost(vector<int>& startPos, vector<int>& homePos, vector<int>& rowCosts, vector<int>& colCosts) {
        int res = 0;
        while (startPos[0] < homePos[0]) {
            startPos[0] += 1;
            res += rowCosts[startPos[0]];
        }
        while (startPos[0] > homePos[0]) {
            startPos[0] -= 1;
            res += rowCosts[startPos[0]];
        }
        while (startPos[1] < homePos[1]) {
            startPos[1] += 1;
            res += colCosts[startPos[1]];
        }
        while (startPos[1] > homePos[1]) {
            startPos[1] -= 1;
            res += colCosts[startPos[1]];
        }
        return res;
    }
};
```


#### [统计农场中肥沃金字塔的数目](https://leetcode-cn.com/problems/count-fertile-pyramids-in-a-land/)

DP is all you need.

![DP](/images/LC-biweekly-contest-66/Q4.png)

##### **Code Q4**
```cpp
class Solution {
public:
    int ans = 0;
    int n, m;
    vector<vector<int>> f, g;

    void dp() {
        for (int i = 0; i < m; i++) 
            f[n - 1][i] = g[n - 1][i];
        for (int i = n - 2; i >= 0; i--) {
            f[i][0] = g[i][0], f[i][m - 1] = g[i][m - 1];
            for (int j = 1; j < m - 1; j++) {
                if (!g[i][j]) f[i][j] = 0;
                else {
                    f[i][j] = min(f[i + 1][j - 1], min(f[i + 1][j], f[i + 1][j + 1])) + 1;
                    ans += f[i][j] - 1;
                }
            }
        }
    }

    void flip() {
        for (int i = 0; i < n / 2; i++) {
            for (int j = 0; j < m; j++) {
                swap(g[i][j], g[n - 1 - i][j]);
            }
        }
    }

    int countPyramids(vector<vector<int>>& grid) {
        n = grid.size(), m = grid[0].size();
        g = grid;
        f = vector<vector<int>> (n + 1, vector<int>(m + 1, 0));
        dp();
        flip(); 
        dp();
        return ans;
    }
};
```