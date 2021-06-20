---
title: LC 周赛第246场
date: 2021-06-20 17:27:44
tags: [Leetcode, 模拟, DFS, 前缀和]
categories: Leetcode周赛
katex: true
description:  LC周赛第246场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **反思**

这次周赛其实总体难度不大，但是成绩不太理想。原因主要在T2和T3。T2对于日期字符串问题不熟悉，写得很慢并且代码也比较冗长，27min才过的T2。T3思路比较直接，就是一个flood fill 算法，但是卡了2个地方，第一处是拷贝了一份矩阵但是却用了原来的形参的矩阵，导致debug了一阵子。第二处地方是dfs内部的一个细节没有考虑清楚。第二处bug调了半小时orz，最后3T混了个垫底排名。赛后看了一下T4，感觉时间充裕说不定可以做出来，应该是近期比较容易的一道T4了。临近开学了，可能能够打的周赛也不多了吧，继续加油吧💪💪💪

### **题解**

#### [字符串中的最大奇数](https://leetcode-cn.com/problems/largest-odd-number-in-string/)

奇数的个位一定也是奇数，只需要从后往前找到第一个奇数，然后截取子串即可。

##### **Code Q1**
```cpp
class Solution {
public:
    string largestOddNumber(string num) {
        int i;
        for(i = num.size() - 1; i >= 0; i--) {
            if (num[i] % 2) break;
        }
        if (i < 0) return "";
        return num.substr(0, i + 1);
    }
};
```

#### [你完成的完整对局数](https://leetcode-cn.com/problems/the-number-of-full-rounds-you-have-played/)

这是一个涉及到时间的字符串处理问题，思路是把`小时:分钟`的格式先转换成分钟形式，然后再求2个时间点中间有多少完整的15分钟。比赛的时候用了比较麻烦的模拟方法，导致码量冗长。


##### **Code Q2**
```cpp
class Solution {
public:
    int get(string s) {
        int h, m;
        sscanf(s.c_str(), "%d:%d",&h, &m);
        return h * 60 + m;
    }

    int numberOfRounds(string a, string b) {
        int x = get(a), y = get(b);
        if (x > y) y += 24 * 60;
        x = (x + 15 - 1) / 15, y /= 15;
        return max(0, y - x);
    }
};
```


#### [统计子岛屿](https://leetcode-cn.com/problems/count-sub-islands/)

这题是一个flood fill算法，对grid2进行dfs，同时判断一下每个1的点是否也在grid1中，一旦不是，返回false并回溯向上层传导，说明一整个块都不是子岛屿。注意这里是一出现不匹配即需要返回false，考试的时候这里逻辑写错，导致调了半小时bug😭。

##### **Code Q3**
```cpp
class Solution {
public:

    vector<vector<int>> G1, G2;
    int dx[4] = {-1, 0, 1, 0}, dy[4] = {0, 1, 0, -1};

    bool dfs(int x, int y) {
        G2[x][y] = 0;
        int n = G1.size(), m = G1[0].size();
        bool ans = true;
        for(int i = 0; i < 4; i++) {
            int a = x + dx[i], b = y + dy[i];
            if (a >= 0 && a < n && b >= 0 && b < m && G2[a][b]) {
                if (!dfs(a, b)) ans = false;
            }
        }
        return ans && G1[x][y];
    }

    int countSubIslands(vector<vector<int>>& g1, vector<vector<int>>& g2) {
        int n = g1.size(), m = g1[0].size();
        G1 = g1, G2 = g2;
        int res = 0;
        for(int i = 0; i < n; i++) 
            for(int j = 0; j < m; j++) {
                if (G2[i][j]) {
                    res += dfs(i, j);
                }
            }
        return res;
    }
};
```

#### [查询差绝对值的最小值](https://leetcode-cn.com/problems/minimum-absolute-difference-queries/)

突破点在于数组中每个数的取值仅在$[0, 100]$。对于一组查询来说，我们可以用类似于桶排序的思路把区间内的数排序，然后从小到大枚举每个数，算出两两之间的差，更新答案即可。对于任意区间是否存在某数，可以用前缀和作预处理，使得查询时间降到$O(1)$。总的时间复杂度为$O(100m)$，预处理时间复杂度$O(100n)$。

##### **Code Q4**
```cpp
const int N = 100010, M = 110;

int s[N][M];

class Solution {
public:
    vector<int> minDifference(vector<int>& nums, vector<vector<int>>& q) {
        int n = nums.size(), m = q.size();
        for(int i = 1; i <= n; i++) 
            for(int j = 1; j <= 100; j++) {
                int t = 0;
                if (nums[i - 1] == j) t++;
                s[i][j] = s[i - 1][j] + t;
            }
        
        vector<int> res(m, -1);
        for(int i = 0; i < m; i++) {
            int l = q[i][0] + 1, r = q[i][1] + 1;
            int last = -1, mi = 110;
            for(int j = 1; j <= 100; j++) {
                if (s[r][j] - s[l - 1][j] > 0) {
                    if (last == -1) last = j;
                    else {
                        mi = min(mi, j - last);
                        last = j;
                    }
                }
            }
            if (mi != 110) res[i] = mi;
        }
        return res;
    }
};
```

