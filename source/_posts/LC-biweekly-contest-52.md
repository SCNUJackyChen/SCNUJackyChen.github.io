---
title: LC 双周赛第52场
date: 2021-05-16 20:27:19
tags: [Leetcode, 模拟, 思维题, 前缀和]
categories: 算法
katex: true
description:  LC双周赛第52场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

###  **题目**
[将句子排序](https://leetcode-cn.com/problems/sorting-the-sentence/)
[增长的内存泄露](https://leetcode-cn.com/problems/incremental-memory-leak/)
[旋转盒子](https://leetcode-cn.com/problems/rotating-the-box/)
[向下取整数对和](https://leetcode-cn.com/problems/sum-of-floored-pairs/)

### **题解**

第一题由于数字只有1-9，直接开个长度为10的数组存一下每个字符串即可。
#### **Code Q1**
```cpp
class Solution {
public:
    string sortSentence(string s) {
        vector<string> a(10);
        for(int i = 0; i < s.size(); i++) {
            int j = i;
            while(j < s.size() && s[j] != ' ') j++;
            int num = s[j - 1] - '0';
            a[num] = s.substr(i, j - i - 1);
            i = j;
        }
        string res;
        for(string t : a) if(t != "") res += t + ' ';
        return res.substr(0, res.size() - 1);
    }
};
```
第二题根据题意模拟即可。
#### **Code Q2**
```cpp
class Solution {
public:
    vector<int> memLeak(int m1, int m2) {
        int cost = 1;
        while(m1 >= cost || m2 >= cost) {
            if (m1 >= m2) {
                m1 -= cost;
                cost ++;
            } else if (m2 > m1) {
                m2 -= cost;
                cost ++;
            } else {
                break;
            }
        }
        return {cost, m1, m2};
    }
};
```

第三题改变自如何将矩阵旋转90°那个题，旋转后再考一个小模拟。个人感觉这个题出得还蛮有意思的。
矩阵旋转90°可以分解成：
1. 先将原矩阵沿对角线翻转；
2. 再将矩阵左右镜面翻转

#### **Code Q3**
```cpp
class Solution {
public:
    vector<vector<char>> rotateTheBox(vector<vector<char>>& box) {
        int m = box.size();
        int n = box[0].size();
        
        vector<vector<char>> res(n, vector<char>(m));
        
        // 对角线翻转
        for(int i = 0; i < m; i++) {
            for(int j = 0; j < n; j++) {
                res[j][i] = box[i][j];
            }
        }
        
        // 左右翻转
        for(int i = 0; i < n; i++) {
            for(int j = 0; j < m / 2; j++) {
                swap(res[i][j], res[i][m - j - 1]);
            }
        }
        
        // 模拟石子下落
        for(int i = n - 1; i >= 0; i--) {
            for(int j = 0; j < m; j++) {
                if(res[i][j] == '#') {
                    int k = i + 1;
                    while(k < n && res[k][j] == '.') {
                        swap(res[k][j], res[k - 1][j]);
                        k++;
                    }
                }
            }
        }
        return res;
    }
};
```

第四题考试的时候没想出来（前三题梦幻开局，可惜最后一题还是没能拿下）
数据范围$10^5$说明本题要用$O(n)$或者$O(nlogn)$做法。直接暴力是$O(n^2)$显然不行。
因为要求：
$$\sum {\lfloor\frac{y}{x}\rfloor}  (y,x \in nums) $$
不妨令：$k = \lfloor\frac{y}{x}\rfloor$, 对于每一个$k$，$y$的取值范围是$[kx, (k + 1)x - 1)$。那么我们可以枚举$k$，然后算出每个$k$下$y$的取值范围中，有多少个$y$是出现在数组中的。而统计多少个$y$这一步可以利用前缀和做到$O(1)$。

**时间复杂度**：
当$k = 1$时，一共有$n$个区间要算；
当$k = 2$时，一共有$\frac{n}{2}$个区间要算；
当$k = 3$时，一共有$\frac{n}{3}$个区间要算；
......
于是整体复杂度是：$n + \frac{n}{2} + \frac{n}{3} + ... + 1 = n(1 + \frac{1}{2} + \frac{1}{3} + ... + \frac{1}{n}) \approx O(nlogn)$

#### **Code Q4**
```cpp
class Solution {
public:
    int sumOfFlooredPairs(vector<int>& nums) {
        int n = nums.size();
        const int N = 100010, MOD = 1e9 + 7;
        vector<long long> s(N, 0);
        unordered_set<int> S;
        for(auto x : nums) s[x] ++, S.insert(x);
        for(int i = 1; i < N; i++) s[i] += s[i - 1];

        long long res = 0;
        for(auto x : S) {
            for(int i = 1; i * x < N; i++) {
                int l = i * x, r = min((i + 1) * x - 1, N - 1);
                long long sum = (i * (s[r] - s[l - 1])) % MOD;
                res = (res + (sum * (s[x] - s[x - 1]))) % MOD;
            }
        }
        return res;
    }
};
```