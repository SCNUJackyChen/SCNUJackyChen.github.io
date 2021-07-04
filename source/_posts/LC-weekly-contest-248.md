---
title: LC 周赛第248场
date: 2021-07-04 15:47:19
tags: [Leetcode, 模拟, 贪心, 快速幂, 字符串哈希, 二分]
categories: Leetcode周赛
katex: true
description:  LC周赛第248场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **感想**
在Singapore做的第一场周赛，难度适中，虽然第三题WA了三发排名直接掉出top500但是还是挺开心滴~明天就开学啦，希望学习&生活都顺利🌴


### **题解**

#### [基于排列构建数组](https://leetcode-cn.com/problems/build-array-from-permutation/)

直接模拟即可。

##### **Code Q1**
```cpp
class Solution {
public:
    vector<int> buildArray(vector<int>& nums) {
        int n = nums.size();
        vector<int> res(n);
        for(int i = 0; i < n; i++) {
            res[i] = nums[nums[i]];
        }
        return res;
    }
};
```

#### [消灭怪物的最大数量](https://leetcode-cn.com/problems/eliminate-maximum-number-of-monsters/)

题意有点类似于植物大战僵尸。一个很直观的想法就是优先枪毙较早到家的怪兽。可以用反证法+调整最优的思路来证明。假设优先枪毙的怪兽不是最早到家的，那可以把这次枪毙调整为枪毙最早到家的怪兽，由于次早到家的怪兽一定在枪毙的（最早到家的）怪兽之后到家，那么不会使得答案变差。因此，如果优先枪毙的怪兽不是最早到家是最优解，那么我们亦可以构造出一个最优解，说明优先枪毙最早到家的策略可以得到最优解。

##### **Code Q2**

```cpp
class Solution {
public:
    int eliminateMaximum(vector<int>& dist, vector<int>& speed) {
        int cnt = INT_MAX;
        int n = dist.size();
        vector<int> step(n);
        for(int i = 0; i < n; i++) {
            int c = (dist[i] + speed[i] - 1) / speed[i];
            step[i] = c;
        }
        sort(step.begin(), step.end());
        int res = 0, last = 1;
        for(int i = 0; i < n; i++) {
            if (step[i] >= last) res ++;
            else break;
            last ++;
        }
        return res;
    }
};
```

#### [统计好数字的数目](https://leetcode-cn.com/problems/count-good-numbers/)

观察得，如果$n$是奇数，则答案为$20^{n / 2} * 5$；如果为偶数，答案为$20^{n / 2}$。由于$n$比较大，可以采用快速幂求解。

##### **Code Q3**
```cpp
typedef long long LL;
const int MOD = 1e9 + 7;

class Solution {
public:
    
    int qmi(int a, LL b) {
        int res = 1;
        while (b) {
            if (b & 1) res = (LL)res * a % MOD;
            a = (LL)a * a % MOD;
            b >>= 1;
        }
        return res;
    }
    
    int countGoodNumbers(long long n) {
        if (n % 2 == 0) return qmi(20, n / 2);
        return qmi(20, n / 2) * 5ll % MOD;
    }
};
```
#### [最长公共子路径](https://leetcode-cn.com/problems/longest-common-subpath/)

保险的解法是后缀自动机/后缀数组（然并不会orz，有空再来补坑吧）。

这个题也可以用字符串哈希 + 二分来做。
首先二分的对象是答案，即公共子串的长度。字符串哈希可以在常数时间内求出区间哈希值，根据哈希值是否相同可以判断字符串是否相同（当然，总可以找到hack数据，不能保证完美哈希，所以这算是一个伪算法）。

时间复杂度：$O(nlogn)$
##### **Code Q4**

```cpp
typedef unsigned long long ULL;

const int N = 100010;
ULL h[N], p[N];

ULL get(int l, int r) {
    return h[r] - h[l - 1] * p[r - l + 1];
}

class Solution {
public:
    vector<vector<int>> paths;
    unordered_map<ULL, int> cnt, S;

    bool check(int mid) {
        cnt.clear(), S.clear();
        p[0] = 1;
        for(int i = 0; i < paths.size(); i++) {
            auto str = paths[i];
            int n = str.size();
            for(int j = 1; j <= n; j++) {
                p[j] = p[j - 1] * 1333331;
                h[j] = h[j - 1] * 1333331 + str[j - 1];
            }
            for(int j = mid; j <= n; j++) {
                ULL t = get(j - mid + 1, j);
                if (!S.count(t) || S[t] != i) {
                    S[t] = i;
                    cnt[t] ++;
                }
            }
        }
        int res = 0;
        for(auto& [c, s] : cnt) res = max(res, s);
        return res == paths.size();
    }

    int longestCommonSubpath(int n, vector<vector<int>>& _paths) {
        paths = _paths;
        int l = 0, r = INT_MAX;
        for(auto& p : paths) {
            r = min(r, (int)p.size());
        }

        while (l < r) {
            int mid = l + r + 1 >> 1;
            if (check(mid)) l = mid;
            else r = mid - 1;
        }
        return r;
    }
};
```
