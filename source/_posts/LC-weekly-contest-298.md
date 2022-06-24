---
title: LC weekly contest 298
date: 2022-06-24 12:06:04
tags: [Leetcode, 数学, 贪心, DP]
categories: Leetcode周赛
katex: true
description:  LC 周赛第298场题解
---

![LC](/images/Leetcode.jpg)

<!--more-->


###  **Solution**

#### [个位数字为 K 的整数之和](https://leetcode.cn/problems/sum-of-numbers-with-units-digit-k)

组成$num$的所有数的个位都是$k$，存在如下的等式：
$$num-nk \equiv 0 \pmod {10}$$

答案$n$的取值范围是$[0, 10]$，由于范围区间较小，可以直接枚举答案，判断是否满足上述等式即可。同时，因为$num - nk$不可能为负数，因此还需加上判断$num-nk<0$的情况。

##### **Code**

```cpp
class Solution {
public:
    int minimumNumbers(int num, int k) {
        int res = 0;
        if (!num) return res;

        for (res = 1; res <= 10; res ++) {
            if (k * res > num) return -1;
            if ((num - k * res) % 10 == 0) return res;
        }
        return -1;
    }
};
```

#### [小于等于 K 的最长二进制子序列](https://leetcode.cn/problems/longest-binary-subsequence-less-than-or-equal-to-k/)

直观地看，我们希望子序列尽可能长，就应该将含有1的部分尽量往后放（因为这样才有更多的空间去填充前导0，从而增加长度）。于是我们可以考察字符串$s$的末尾一段对应的二进制数与$k$的大小关系。

记$s$的长度为$n$，$k$的二进制长度为$m$，有以下三种情况：

1. $n < m$，此时$s$对应的二进制数恒小于等于$k$，所以答案是$n$；
2. $n > m$，且末尾$m$长度的二进制数大于$k$，此时只有末尾$m-1$长度的二进制数小于$k$，所以答案是区间$[0,n - (m - 1)]$的0的个数$+(m-1)$；
3. $n > m$，且末尾$m$长度的二进制数大于$k$，此时答案是区间$[0,n - m]$的0的个数$+m$.


##### **Code**
```cpp
class Solution {
public:
    int longestSubsequence(string s, int k) {
        int n = s.size(), m = 0;
        int t = k;
        while (t) {
            t >>= 1;
            m++;
        }
        if (n < m) return n;
        
        // 求末尾长为m的子串对应的二进制数
        t = 0;
        for (int i = n - m; i < n; i++) {
            if (s[i] == '1') t = t * 2 + 1;
            else t = t * 2;
        }
    
        if (t > k) m--;
        return m + count(s.begin(), s.begin() + n - m, '0');
    }
};
```

#### [卖木头块](https://leetcode.cn/problems/selling-pieces-of-wood/)

二维区间DP。记$f[i][j]$为高度为$i$，宽度为$j$的木板分割后产生的最大价值。木板可以水平切割或者垂直切割，切割之后会产生两个小木板，即为2个子问题。我们只需要枚举所有的高度和宽度，然后再枚举切割点，可得状态转移方程：

- 水平切割
$$f[i][j] =max_{k=1}^{i}(f[i-k][j]+f[i][j])$$

- 垂直切割
$$f[i][j] =max_{k=1}^{j}(f[i][k]+f[i][j-k])$$

合起来就是：
$$f[i][j] = max \left\\{ max_{k=1}^{j}(f[i][k]+f[i][j-k]), max_{k=1}^{i}(f[i-k][j]+f[i][j]) \right\\} $$



##### **Code**

```cpp
class Solution {
public:
    using LL = long long;
    LL f[210][210];

    long long sellingWood(int n, int m, vector<vector<int>>& prices) {
        for (auto& p : prices) f[p[0]][p[1]] = p[2];

        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= m; j++) {
                for (int k = 1; k <= j / 2; k++) f[i][j] = max(f[i][j], f[i][j - k] + f[i][k]);
                for (int k = 1; k <= i / 2; k++) f[i][j] = max(f[i][j], f[i - k][j] + f[k][j]);
            }
        }

        return f[n][m];

    }
};
```