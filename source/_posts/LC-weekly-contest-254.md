---
title: LC 周赛第254场
date: 2021-08-15 16:21:04
tags: [Leetcode, 模拟, 贪心, 快速幂, BFS, 二分]
categories: Leetcode周赛
katex: true
description:  LC周赛第254场复盘+题解
---


![LC](/images/Leetcode.jpg)

<!--more-->

### **反思**
> 从近几次周赛都卡了贪心题这一现象可以看出，jacky的脑瓜子儿是越来越不灵活了

###  **题解**

#### [作为子字符串出现在单词中的字符串数目](https://leetcode-cn.com/problems/number-of-strings-that-appear-as-substrings-in-word/)
偷懒直接用api了

##### **Code Q1**
```cpp
class Solution {
public:
    int numOfStrings(vector<string>& p, string w) {
        int res = 0;
        for (auto &s : p) {
            if (w.find(s) != w.npos)
                res ++;
        }
        return res;
    }
};
```
#### [构造元素不等于两相邻元素平均值的数组](https://leetcode-cn.com/problems/array-with-elements-not-equal-to-average-of-neighbors/)
观察一下（然而我是亿下）可以得出，当相邻两个数都比中间数大，或者都比中间数小时，其平均值一定不等于中间值。因此可以将数组排序后分为2半，满足前一半都小于后一半，然后交错排列即可。

##### **Code Q2**

```cpp
class Solution {
public:
    vector<int> rearrangeArray(vector<int>& nums) {
        int n = nums.size();
        sort(nums.begin(), nums.end());
        vector<int> res = vector<int>(n);
        int k = 0;
        for (int i = 0; i < n; i += 2) res[i] = nums[k++];
        for (int i = 1; i < n; i += 2) res[i] = nums[k++];
        return res;
    }
};
```
#### [数组元素的最小非零乘积](https://leetcode-cn.com/problems/minimum-non-zero-product-of-the-array-elements/)
假设数组长度为$n$，把二进制数两两分为一组， 对于一个数, 总可以找到它的翻转数（除最后一位)。比如00011的翻转数为11101，对这两个数进行操作，可以操作成11110与00001，并且这种操作得到的乘积是最小的（可以用反证法证明）。因此两两配对后会得到$\lfloor \frac{n}{2} \rfloor$个1111...10，以及剩余的一个全1数。

$p$是位数，分别用$p$表示配对乘积以及全1数，如下所示：
$(2^p - 2)^{\lfloor \frac{2^p - 1}{2} \rfloor}(2^p - 1)$
因为$p$最大只有60，所以$2^p$可以用`long long`来存，剩下的交给快速幂。

##### **Code Q3**
```cpp
using LL = long long;

class Solution {
public:

    LL qmi(LL a, LL b, int q) {
        LL res = 1;
        while(b) {
            res = (res % q) * (a % q);
            b >>= 1;
            a = (a % q) * (a % q);
        }
        return res;
    }

    int minNonZeroProduct(int p) {
        int mod = 1e9 + 7;
        LL base = 1ll << p;  // 2^p
        LL a = qmi(base - 2, (base - 1) >> 1, mod);
        return (a % mod) * ((base - 1) % mod) % mod;
    }
};
```
#### [你能穿过矩阵的最后一天](https://leetcode-cn.com/problems/last-day-where-you-can-still-cross/)
比较裸的二分+BFS，比较贴近面试题了

##### **Code Q4**
```cpp
class Solution {
public:

    vector<vector<int>> cells;
    vector<vector<int>> f;
    int row, col;
    int dx[4] = {-1, 0, 1, 0}, dy[4] = {0, 1, 0, -1};

    bool check(int mid) {
        f = vector<vector<int>>(row, vector<int>(col, 0));
        for (int i = 0; i <= mid; i++) {
            auto x = cells[i];
            f[x[0] - 1][x[1] - 1] = 1;
        }
        queue<pair<int, int>> q;
        for (int i = 0; i < col; i++)
            if (!f[0][i]) {
                q.push({0, i});
                f[0][i] = 1;
            }
        while (q.size()) {
            auto t = q.front();
            q.pop();

            for (int i = 0; i < 4; i++) {
                int a = t.first + dx[i], b = t.second + dy[i];
                if (a >= 0 && a < row && b >= 0 && b < col && f[a][b] != 1) {
                    if (a == row - 1) return true;
                    f[a][b] = 1;
                    q.push({a, b});
                }
            }
        }
        return false;
    }

    int latestDayToCross(int _row, int _col, vector<vector<int>>& c) {
        int n = c.size();
        cells = c;
        row = _row, col = _col;

        int l = 0, r = n - 1;
        while (l < r) {
            int mid = l + r + 1 >> 1;
            if (!check(mid)) r = mid - 1;
            else l = mid;
        }
        return r + 1;
    }
};
```
