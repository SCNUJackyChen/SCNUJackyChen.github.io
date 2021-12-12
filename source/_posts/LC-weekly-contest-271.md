---
title: LC-weekly-contest-271
date: 2021-12-12 14:27:26
tags: [Leetcode, 模拟，前缀和]
categories: Leetcode周赛
katex: true
description:  LC 周赛第271场题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **Summary**

可能是近期最简单的一次周赛了，然而做的时候感觉在梦游。T2想复杂了拿单调队列来做，过了之后发现已经过了1000+人......T4 没想清楚区间长度的关系，gg了😭 所谓”机会来了把握不住也无济于事“，大概就是这样吧。

###  **Solution**

#### [环和杆](https://leetcode-cn.com/problems/rings-and-rods/)

##### **Code Q1**
```cpp
class Solution {
public:
    int countPoints(string rings) {
        vector<vector<int>> cnt(10, vector<int>(3, 0));
        for (int i = 0; i < rings.size(); i += 2) {
            if (rings[i] == 'R') cnt[rings[i + 1] - '0'][0]++;
            else if (rings[i] == 'G') cnt[rings[i + 1] - '0'][1]++;
            else if (rings[i] == 'B') cnt[rings[i + 1] - '0'][2]++;
        }
        
        int res = 0;
        for (int i = 0; i < 10; i++) {
            if (cnt[i][0] && cnt[i][1] && cnt[i][2]) res++;
        }
        return res;
    }
};
```
#### [子数组范围和](https://leetcode-cn.com/problems/sum-of-subarray-ranges/)

We can enumerate all sub-arrays via iterating their beginning and endding points.

我们可以通过遍历所有子数组的左右端点来进行枚举。

##### **Code Q2 - 1**
```cpp
class Solution {
public:
    long long subArrayRanges(vector<int>& nums) {
        int n = nums.size();

        long long res = 0;
        
        for (int i = 0; i < n; i++) { // iterate the max and min in [i, j]
            int mm = INT_MIN, mi = INT_MAX;
            for (int j = i; j < n; j++) {
                mm = max(mm, nums[j]);
                mi = min(mi, nums[j]);
                res += mm - mi;
            }
        }
        return res;
    }
};
```

I think about this question in a more complicate way when I was taking the contest (mainly misled by the sample). The idea is utilizing a monotonous queue. We can iterate the size of a sliding window, and dynamically maintain a maximum and a minimum in the sliding window. The time complexity is also $O(n^2)$.

考试时想复杂了用的单调队列（主要是被样例迷惑了），思路就是枚举一下滑窗的大小，然后对于每个长度为$k$的滑窗去动态维护窗口内的最大值和最小值。时间复杂度也是$O(n^2)$的。

##### **Code Q2 - 2**
```cpp
class Solution {
public:
    
    long long subArrayRanges(vector<int>& a) {
        long long res = 0;
        int n = a.size();
        vector<int> q(n + 1), p(n + 1);
        
        for (int k = 1; k <= n; k++) {
            q = vector<int>(n + 1, 0);
            p = vector<int>(n + 1, 0);
            int hh = 0, tt = -1;
            int hh2 = 0, tt2 = -1;
            for (int i = 0; i < n; i ++ ) {
                if (hh <= tt && i - q[hh] + 1 > k) hh++;
                if (hh2 <= tt2 && i - p[hh2] + 1 > k) hh2++;
                while (hh <= tt && a[q[tt]] >= a[i]) tt--;
                while (hh2 <= tt2 && a[p[tt2]] <= a[i]) tt2--;
                q[++tt] = i;
                p[++tt2] = i;
                if (i - k + 1 >= 0) res += a[p[hh2]] - a[q[hh]]; 
            }
        }
        return res;
    }
};
```

#### [给植物浇水 II](https://leetcode-cn.com/problems/watering-plants-ii/)

##### **Code Q3**

```cpp
class Solution {
public:
    int minimumRefill(vector<int>& p, int ca, int cb) {
        int n = p.size();
        
        int res = 0;
        int a = ca, b = cb;
        for (int i = 0, j = n - 1; i <= j; i++, j--) {
            if (i == j) {
                if (a >= b) {
                    if (a < p[i]) a = ca, res++;
                }
                else {
                    if (b < p[j]) b = cb, res++;
                }
            }
            else {
                if (a < p[i]) a = ca, res++;
                if (b < p[j]) b = cb, res++;
                a -= p[i], b -= p[j];
            }
        }
        return res;
    }
};
```

#### [摘水果](https://leetcode-cn.com/problems/maximum-fruits-harvested-after-at-most-k-steps/)

To quickly get the sum of an interval, we can calculate a prefix sum array.

For each $k$, we can iterate all the possible solutions. Note the distance between most left position we move and our start position as $x$, and the distance between most right and start position as $y$. If go left firstly, we get $2x + y = k$; while if go right firstly, $x + 2y = k$. 

我们可以利用前缀和数组来快速得到区间和。

对于每一个$k$，我们可以枚举所有的可能方案。记我们移动的最左端到起始位置距离为$x$，最右端到起始位置距离为$y$。如果先往左走，我们有 $2x + y = k$；如果先往右走，我们有 $x + 2y = k$. 

##### **Code Q4**

```cpp
class Solution {
public:
    int s[200010];
    int maxTotalFruits(vector<vector<int>>& fruits, int startPos, int k) {
        for (int i = 0; i < 200010; i ++) s[i] = 0;
        for (auto& v : fruits) {
            s[v[0] + 1] += v[1];
        }
        if (k == 0) return s[startPos + 1];
        
        for (int i = 1; i < 200010; i++) s[i] += s[i - 1];

        int res = 0;

        for (int i = 0; i * 2 <= k; i++) {
            int l = max(1, startPos - i + 1), r = min(200005, startPos + (k - 2 * i) + 1);
            res = max(res, s[r] - s[l - 1]);
            r = min(200005, startPos + i + 1), l = max(1, startPos - (k - 2 * i) + 1);
            res = max(res, s[r] - s[l - 1]);
        }
        return res;
        
        
    }
};
```