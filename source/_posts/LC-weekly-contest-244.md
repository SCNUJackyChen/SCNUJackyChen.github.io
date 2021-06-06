---
title: LC 周赛第244场
date: 2021-06-06 16:03:14
tags: [Leetcode, 贪心, 前后缀分解, 二分]
katex: true
description: LC周赛第244场复盘+题解
categories: Leetcode周赛
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **题解**

#### [1. 判断矩阵经轮转后是否一致](https://leetcode-cn.com/problems/determine-whether-matrix-can-be-obtained-by-rotation/)

直接模拟即可，将原矩阵旋转90°并于目标矩阵比较。
##### **Code Q1**
```cpp
class Solution {
public:
    
    vector<vector<int>> rotate(vector<vector<int>>& mat) {
        int n = mat.size();
        vector<vector<int>> res(n, vector<int>(n));
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < n; j++) {
                res[i][j] = mat[j][i];
            }
        }
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < n / 2; j++) {
                swap(res[i][j], res[i][n - j - 1]);
            }
        }
        return res;
    }
    
    bool check(vector<vector<int>>& mat, vector<vector<int>>& target ) {
        int n = mat.size();
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < n; j++) {
                if (mat[i][j] != target[i][j]) 
                    return false;
            }
        }
        return true;
    }
    
    bool findRotation(vector<vector<int>>& mat, vector<vector<int>>& target) {
        if (check(mat, target)) return true;
        for(int i = 0; i < 3; i++) {
            mat = rotate(mat);
            if(check(mat, target)) return true;
        }
        return false;
    }
};

```

#### [2. 使数组元素相等的减少操作次数](https://leetcode-cn.com/problems/reduction-operations-to-make-the-array-elements-equal/)

这是一道贪心题，依据题意，每次把最大的数变成第二大的数，直到所有的数都变成最小的数。可以把原数组看成很多段连续的相同数字（排序后）,则每次操作后，操作次数一定是当前最大数的个数，我们只需将每次操作后最大数的个数统计一下即可。

##### **Code Q2**
```cpp
class Solution {
public:
    int reductionOperations(vector<int>& nums) {
        sort(nums.begin(),nums.end());
        int n = nums.size();
        int res = 0, seg_len = 0;
        for(int i = n - 1; i >= 0; i--) {
            if (nums[i] == nums[0]) break;
            int t = 0;
            int j = i;
            while(nums[j] == nums[i]) t++, j--;
            seg_len += t;
            res += seg_len;
            i = j + 1;
        }
        return res;
    }
};
```
#### [3. 使二进制字符串字符交替的最少反转次数](https://leetcode-cn.com/problems/minimum-number-of-flips-to-make-the-binary-string-alternating/)

这题考试的时候思路对了，但是无奈这类题做得少，写的时候很慢。。最后考试结束后几分钟过的Orz

如果字符串长度为偶数，可以发现其实不论怎么旋转都不会影响次数（也就是操作1没啥用），那么对于这种情况，我们只需要统计一下和目标串（101010...和010101...）的差异字符；

难点在于奇数情况，奇数和偶数最大的区别在于，奇数情况在旋转之后，会构造出相邻元素相同的串（比如：010101101，将它右旋3位可以得到目标串）。可以发现我们其实是要寻找一个分割点，使得原串分割成2部分，并且这两部分的操作次数之和最小。本质上就是把原串进行前后缀分解。

具体来说，可以分别从左到右、从右到左预处理出子串转换到两个目标串的操作次数，时间复杂度为$O(n)$。然后再枚举分割点，统计最小操作次数，时间也是$O(n)$的。

举个例子：

原始串  ： `0 1 0 0 1 0 0 1 1 0 1`
目标串1：`1 0 1 0 1 0 1 0 1 0 1`
目标串2：`0 1 0 1 0 1 0 1 0 1 0`
串1前缀：`1 2 3 3 3 3 4 5 5 5 5`
串1后缀：`5 4 3 2 2 2 2 1 0 0 0`
串2前缀：`0 0 0 1 2 3 3 3 4 5 6`
串2后缀：`6 6 6 6 5 4 3 3 3 2 1`

##### **Code Q3**
```cpp
class Solution {
public:
    vector<vector<int>> f, g;
    
    int minFlips(string s) {
        int n = s.size();
        f = vector<vector<int>> (n + 1, vector<int>(2, 0));
        g = vector<vector<int>> (n + 1, vector<int>(2, 0));
        
        f[0][1] = s[0] == '0';
        f[0][0] = s[0] == '1';
        
        string s10, s01;
        for(int i = 0; i < n; i++) {
            if (i % 2 == 0) {
                s10.push_back('1');
                s01.push_back('0');
            } else {
                s10.push_back('0');
                s01.push_back('1');
            }
        }
        
        for(int i = 1; i < n; i++) {
            f[i][1] = f[i - 1][1] + (s[i] != s10[i]);
            f[i][0] = f[i - 1][0] + (s[i] != s01[i]);
        }
        
        g[n - 1][1] = s[n - 1] != s10[n - 1];
        g[n - 1][0] = s[n - 1] != s01[n - 1];
        for(int i = n - 2; i >= 0; i--) {
            g[i][1] = g[i + 1][1] + (s[i] != s10[i]);
            g[i][0] = g[i + 1][0] + (s[i] != s01[i]);
        }
        
        int res = min(f[n - 1][1], f[n - 1][0]);
        for(int i = 0; i < n; i++) {
            if (i % 2 == 0) { // odd
                if (n % 2) {
                    res = min(res, f[i][0] + g[i + 1][1]);
                    res = min(res, f[i][1] + g[i + 1][0]);   
                }
                else {
                    res = min(res, f[i][0] + g[i + 1][0]);
                    res = min(res, f[i][1] + g[i + 1][1]);   
                }
            } else { // even
                if (n % 2) {
                    res = min(res, f[i][0] + g[i + 1][1]);
                    res = min(res, f[i][1] + g[i + 1][0]);   
                }
                else {
                    res = min(res, f[i][0] + g[i + 1][0]);
                    res = min(res, f[i][1] + g[i + 1][1]);   
                }
            }
        }
        
        return res;
    }
};
```


#### [4. 装包裹的最小浪费空间](https://leetcode-cn.com/problems/minimum-space-wasted-from-packaging/)

这个题考察二分，但对谁二分需要斟酌。如果考虑固定包裹，对箱子二分，由于箱子被分在不同的厂家中，厂家最多有$10^5$个，而箱子也有$10^5$个，因此最坏情况是$O(n^2logn)$的。

但是如果固定箱子，对包裹二分，由于箱子总数是限制在$10^5$以内的，而计算代价只需要$O(1)$的时间，因此总时间可以控制在$O(nlogn)$以内。

计算代价时可以维护上一个箱子所能容纳的最大包裹的下标`last`，然后对于当前箱子，计算其能容纳的最大包裹的下标`pos`，则使用这种箱子所用的总空间为`(pos-last) * x`，`x`是箱子的体积。由于总代价 = 总箱子体积 - 总包裹体积，后者为定值，前者可以在二分出包裹后$O(1)$时间算出。因此计算代价的时间是常数的。


##### **Code Q4**
```cpp
class Solution {
public:
    int minWastedSpace(vector<int>& a, vector<vector<int>>& bs) {
        long long MOD = 1e9 + 7, INF = 1e18, sum = 0;
        for(auto x : a) sum += x;
        sort(a.begin(), a.end());
        int n = a.size();
        
        long long res = INF;
        for(auto b: bs) {
            sort(b.begin(), b.end());
            if (b.back() < a.back()) continue;
            int last = -1;
            long long t = -sum;
            for(int x : b) {
                int l = last, r = n - 1;
                while(l < r) {
                    int mid = l + r + 1 >> 1;
                    if (a[mid] <= x) l = mid;
                    else r = mid - 1;
                }
                if (l == last) continue;
                t += (l - last) * (long long)x;
                last = l;
            }
            res = min(res, t);
        }
        if (res == INF) res = -1;
        return res % MOD;
    }
};
```