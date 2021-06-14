---
title: LC 双周赛第54场
date: 2021-06-13 22:18:06
tags:  [Leetcode, 模拟, 前缀和, 树形DP]
categories: Leetcode周赛
katex: true
description:  LC双周赛第54场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **题解**

#### [检查是否区域内所有整数都被覆盖](https://leetcode-cn.com/problems/check-if-all-the-integers-in-a-range-are-covered/)

开一个数组记录每个数是否出现过，枚举所有区间里面的数，逐一标记，最后判断目标区间内的数是否都出现过即可。

##### **Code Q1**
```cpp
class Solution {
public:
    bool isCovered(vector<vector<int>>& ranges, int left, int right) {
        vector<bool> f(100);
        for(auto r : ranges) {
            for(int i = r[0]; i <= r[1]; i++) {
                f[i] = true;
            }
        }
        for(int i = left; i <= right; i++) {
            if (!f[i]) return false;
        }
        return true;
    }
};
```

#### [找到需要补充粉笔的学生编号](https://leetcode-cn.com/problems/find-the-student-that-will-replace-the-chalk/)

这个题直接模拟会超时，因为$k$可以取到$10^9$。但仔细一看发现我们可以先算出来一圈所需要的粉笔数，然后取余数可以得到：到最后一圈时，所剩的粉笔数。这时候再模拟即可。


##### **Code Q2**
```cpp
class Solution {
public:
    int chalkReplacer(vector<int>& nums, int k) {
        long long kk = k;
        long long tot = 0;
        for(auto x : nums) tot += x;
        kk %= tot;
        
        for(int i = 0; i < nums.size(); i++) {
            kk -= nums[i];
            if (kk < 0) return i;
        }
        return nums.size() - 1;
    }
};
```

####  [最大的幻方](https://leetcode-cn.com/problems/largest-magic-square/)

这个题要算好复杂度，直接暴力需要$O(n^5)$，是会超时的。考虑到算行和列的和这一步可以做前缀和优化，因此可以把复杂度降到$O(n^4)$。

##### **Code Q3**
```cpp
class Solution {
public:
    
    vector<vector<int>> g;
    vector<vector<int>> f;
    int ans;
    
    bool cal(int x, int y, int len) {
        long long val = 0;
        for(int i = x; i < x + len; i++) val += g[i][y];
        
        // 行
        for(int i = x; i < x + len; i++) {
            int a = f[i + 1][y + len] - f[i + 1][y] - f[i][y + len] + f[i][y];
            if (a != val) return false;
        }
        
        // 列
        for(int i = y; i < y + len; i++) {
            int a = f[x + len][i + 1] - f[x][i + 1] - f[x + len][i] + f[x][i];
            if (a != val) return false;
        }
        
        long long d = 0;
        int dy = y;

        // 主对角线
        for(int i = x; i < x + len; i ++) {
            d += g[i][dy];
            dy ++;
        }
        if (d != val) return false;
        d = 0;
        dy = y + len - 1;
        
        //副对角线
        for(int i = x; i < x + len ; i++) {
            d += g[i][dy];
            dy --;
        }
        if (d != val) return false;
        return true;
    }
    
    void get(int len) {
        int n = g.size(), m = g[0].size();
        for(int i = 0; i + len <= n; i++) {
            for(int j = 0; j + len <= m; j++) {
                if (cal(i, j, len)) {
                    ans = max(ans, len);
                    return;
                }
            }
        }    
    }
    
    int largestMagicSquare(vector<vector<int>>& grid) {
        g = grid;
        int n = g.size(), m = g[0].size();
        int L = min(n, m);
        ans = 0;
        f = vector<vector<int>>(n + 1, vector<int>(m + 1, 0));
        for(int i = 1; i <= n; i++) {
            for(int j = 1; j <= m; j++) {
                f[i][j] = f[i - 1][j] + f[i][j - 1] - f[i - 1][j - 1] + g[i - 1][j - 1];
            }
        }
        for(int i = 1; i <= L; i++) {
            get(i);
        }
        return ans;
        
    }
};
```
#### [反转表达式值的最少操作次数](https://leetcode-cn.com/problems/minimum-cost-to-change-the-final-value-of-expression/)

本题考察的是树形DP。可以把表达式看成解析树的形式，比如：
`1 & (0 | 1)` 可以表示成：
```
   &
 /   \
1     |
     / \
    0   1
```
我们最终要求的是将这棵树所表示的表达式的值反转所需的最小操作次数。很明显，表达式的值不是1就是0。因此可以用`dp[x][0]`表示以当前节点`x`为根节点的子树对应的表达式返回0的操作次数，`dp[x][1]`表示返回1的操作次数。则状态的转移如下：
1. 如果`x`节点的符号为`$`，那么分类讨论一下左右子树（记为a，b）的返回值：
1.1  左右子树都返回0，  则`dp[x][0] = dp[a][0] + dp[b][0]`;
1.2  左子树返回0， 右子树返回1， 则可以将`$`变成`|`，则`dp[x][0] = dp[a][0] + dp[b][1] + 1`;
1.3  左子树返回1，右子树返回0，则可以将`$`变成`|`，则`dp[x][0] = dp[a][1] + dp[b][0] + 1`;
1.4  左右都是1，这种情况不可能返回0，所以不需要讨论。

`x`为`|`的情况类似，见代码。

##### **Code Q4**
```cpp
class Solution {
public:
    stack<vector<int>> num;
    stack<char> op;

    void eval() {
        auto a = num.top(); num.pop();
        auto b = num.top(); num.pop();
        char c = op.top(); op.pop();

        vector<int> t(2);
        if (c == '&') {
            int m0 = min(a[0] + b[0], min(a[0] + b[1], a[1] + b[0]));
            int m1 = min(a[0] + b[1] + 1, min(a[1] + b[0] + 1, a[1] + b[1]));
            t = {m0, m1};
        } else {
            int m0 = min(a[0] + b[0], min(a[0] + b[1] + 1, a[1] + b[0] + 1));
            int m1 = min(a[0] + b[1], min(a[1] + b[0], a[1] + b[1]));
            t = {m0, m1};
        }
        num.push(t);
    }

    int minOperationsToFlip(string str) {
        for(auto c: str) {
            if (isdigit(c)) {
                if (c == '0') num.push({0, 1});
                else num.push({1, 0});
            } 
            else if (c == '(') {
                op.push(c);
            }
            else if (c == ')') {
                while(op.top() != '(') eval();
                op.pop();
            }
            else {
                while(op.size() && op.top() != '(') eval();
                op.push(c);
            }
        }
        while(op.size()) eval();
        return max(num.top()[0], num.top()[1]); // 必然有一个是0
    }
};
```

表达式求值这模板还是得多写写呀~