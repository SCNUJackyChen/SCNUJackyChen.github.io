---
title: LC biweekly contest 78
date: 2022-05-21 14:01:03
tags: [Leetcode, 前缀和, 二分, 贪心, DP]
categories: Leetcode周赛
katex: true
description:  LC 双周赛第78场题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

###  **Solution**

#### [找到一个数字的 K 美丽值](https://leetcode.cn/problems/find-the-k-beauty-of-a-number/)

根据题意模拟即可，注意除0特判。

Simulate according to the question description. Note to check zero division.

##### **Code Q1**
```cpp
class Solution {
public:
    int divisorSubstrings(int num, int k) {
        int res = 0;
        string s = to_string(num);
        for (int i = 0; i + k <= s.size(); i++) {
            if (stoi(s.substr(i, k)) == 0) continue;
            if (num % stoi(s.substr(i, k)) == 0) res++;
        }
        return res;
    }
};
```


#### [分割数组的方案数](https://leetcode.cn/problems/number-of-ways-to-split-array/)

本题数据范围比较大，直接枚举分割点 + 统计区间和的时间复杂度为$O(n^2)$，会超时。可以先预处理出前缀和，然后再枚举分割点，从而将时间降到$O(n)$。

Since the range of data in this question is large, directly iterating the splitting point and calculating the interval sum leads to $O(n^2)$ time complexity. We can preprocess a prefix sum array and then iterate all splitting points, thus reducing the time to $O(n)$. 

##### **Code Q2**
```cpp
class Solution {
public:
    int waysToSplitArray(vector<int>& nums) {
        int n = nums.size();
        vector<long long> s(n + 1);
        long long sum = 0;
        for (int i = 1; i <= n; i++) s[i] = s[i - 1] + nums[i - 1], sum += nums[i - 1];
        int res = 0;
        for (int i = 1; i < n; i++) {
            if (s[i] >= sum - s[i])
                res ++;
        }
        return res;
    }
};
```

#### [毯子覆盖的最多白色砖块数](https://leetcode.cn/problems/maximum-white-tiles-covered-by-a-carpet/)

本题需要利用贪心的思想推出一个比较强的结论：**答案一定可以是某个白砖区间左端点开始的一段区间**。

> 证明：调整法。假设最优解所对应的区间左端点$l_0$不在某个白色砖块区间的左端点，那么会有2种情况：1）$l_0$落在某个白砖区间内部，我们可以将整个区间向左平移，直到和白砖区间左端点$l_1$重合。此时得到的区间至少是最优解，因为左移操作会贡献$l_1-l_0$个格子，而区间右端点移出的格子不可能超过这个值。2）$l_0$不落在白砖区间内部，我们可以将整个区间向右平移，直到和右边最近的白砖区间的左端点$l_2$重合。此时得到的区间至少是最优解，因为右移操作会贡献$0\sim (l_2-l_0)$个格子，且原来的格子均不会移出区间。


可以按区间从小到大枚举所有白砖区间，对于每个左端点，计算其对应的右端点，统计中间共有多少个砖块。区间和可以通过前缀和先做预处理。找到右端点最近的区间，可以通过二分搜索来快速查找。

##### **Code Q3**
```cpp
class Solution {
public:
    int maximumWhiteTiles(vector<vector<int>>& tiles, int k) {
        sort(tiles.begin(), tiles.end());
        unordered_map<int, int> cnt, lr;
        int s = 0;
        for (auto t : tiles) {
            lr[t[0]] = t[1];
            cnt[t[0]] = s;
            s += t[1] - t[0] + 1;
        }
        
        int res = 0;
        for (auto t : tiles) {
            int x = t[0] + k - 1;
            int l = 0, r = tiles.size() - 1;
            while (l < r) {
                int mid = l + r + 1 >> 1;
                if (tiles[mid][0] <= x) l = mid;
                else r = mid - 1;
            }
            res = max(res, cnt[tiles[l][0]] - cnt[t[0]] + min(lr[tiles[l][0]], x) - tiles[l][0] + 1);

        }
        return res;
        
    }
};
```


#### [最大波动的子字符串](https://leetcode.cn/problems/substring-with-largest-variance/)

本题的难点在于如何将问题进行转换。直观上会很想枚举子字符串然后去考虑每个子字符串的波动值。但这么做复杂度比较高。

注意到，最大波动值只有2种字符决定（出现次数最多和最少），虽然对于某个区间，我们不知道出现次数最多和最少的字符是什么，但其实不需要关心，因为我们只需要求出全部区间的最大波动值。换言之，我们可以枚举这2种字符的所有可能情况，而最优答案一定包含在里面。

我们需要枚举$C_{26}^2$种可能的组合，不妨记每一组为$(a, b)$，其中$a$为出现次数最多的字符，$b$为最少的字符。则对于整个字符串$s$，一共有3种字符：$a, b$ 和剩下的其他字符。问题转换成：求$s$的所有子串中，$a,b$的频数差的最大值。将$a$视为1，$b$视为-1，问题变成：求数组的最大子数组和（且必须包含-1）。最大子数组是一个经典问题（[LC53](https://leetcode.cn/problems/maximum-subarray/)）。但是本题还有一个附加条件：必须包含-1，即不能将只包含1的子数组算入最优解中。我们可以另开一个变量记录$g$包含-1的最大值，该变量只会在遇到-1时才更新，且初始值为$-\infin$，因为不含-1。最终答案为最大的$g$。

枚举所有字母组合 + 遍历字符串的时间复杂度为$O(26^2n)$。我们可以调整遍历顺序，先遍历字符串，对于每个字符，将其视为$a$或$b$，这样可以少一层循环，时间降为$O(26n)$。

##### **Code Q4**

```cpp
class Solution {
public:
    int largestVariance(string s) {
        int n = s.size();
        vector<vector<int>> f(26, vector<int>(26, 0)), g(26, vector<int>(26, -1e9));
        int res = 0;

        for (int i = 0; i < n; i++) {
            int c = s[i] - 'a';
            for (int j = 0; j < 26; j++) {
                f[c][j]++, g[c][j]++; // s[i] = a, j = b
                g[j][c] = --f[j][c]; // s[i] = b, j = a
                f[j][c] = max(f[j][c], 0);
                res = max({res, g[c][j], g[j][c]});
            }
        }
        return res;
    }
};
```