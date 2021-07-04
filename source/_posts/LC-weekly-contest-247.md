---
title: LC 周赛第247场
date: 2021-07-01 13:42:05
tags: [Leetcode, 前缀和, 状压, 树形DP, 数论]
categories: Leetcode周赛
katex: true
description:  LC周赛第247场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **题解**

#### [两个数对之间的最大乘积差](https://leetcode-cn.com/problems/maximum-product-difference-between-two-pairs/)

排序取头尾即可。

##### **Code Q1**
```cpp
class Solution {
public:
    int maxProductDifference(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        int n = nums.size();
        return nums[n - 1] * nums[n - 2] - nums[0] * nums[1];
    }
};
```

#### [循环轮转矩阵](https://leetcode-cn.com/problems/cyclically-rotating-a-grid/)

一道码量略大的模拟题，按照题意从外往内旋转即可。

##### **Code Q2**
```cpp
class Solution {
public:

    vector<vector<int>> A;
    int dx[4] = {1, 0, -1, 0}, dy[4] = {0, 1, 0, -1};

    void rotate(int x, int k, int len) {
        if (len < 0) return;
        if (k == 0) return;
        int n = A.size(), m = A[0].size();

        for(int i = 0; i < k; i++) {
            int d = 0;
            int t = A[x][x];
            int a = x, b = x;
            for(int j = 0; j < len; j++) {
                a += dx[d], b += dy[d];
                if (a > n - x - 1 || a < x || b > m - x - 1 || b < x) {
                    a -= dx[d], b -= dy[d];
                    d ++;
                    a += dx[d], b += dy[d];
                }
                swap(t, A[a][b]);
                
            }
        }
    }


    vector<vector<int>> rotateGrid(vector<vector<int>>& grid, int k) {
        A = grid;
        int n = A.size(), m = A[0].size();
        int L = min(n, m);
        for(int i = 0; i < (L + 1) / 2; i++) {
            int C = 2 * (n + m - 4 * i) - 4;
            rotate(i, k % C, C);
        }
        return A;
    }
};
```


#### [最美子字符串的数目](https://leetcode-cn.com/problems/number-of-wonderful-substrings/)

可以把原问题拆解为以下几个小问题：
- 如何快速求出某段连续的区间内，某字符出现次数是否为奇数
- 如何快速地枚举下标`i`作为结尾的所有子串的合法方案

针对问题1，可以用前缀和方法进行预处理，然后可以在常数时间内得到任意区间内某字符的出现个数，即得到其奇偶性。此处，不妨记奇数为1，偶数为0。

针对问题2，可以用10位二进制数来表示某类状态，例如，`0000000001`表示字母`a`出现了奇数次，其他9个字母出现了偶数次的状态。可以注意到，一共只有$2^{10}$种状态。可以开一个长度为1024的数组来存储每种状态出现的次数。通过枚举下标，可以得到每个位置的状态，根据题意，合法的方案应该满足：最多只有一位比特与当前状态不同，又因为已经计算出之前的状态数并记录到数组中，因此可以在常数时间内累加出答案。

##### **Code Q3**
```cpp
const int N = 100010;
int s[N][10];
int cnt[1024];

class Solution {
public:
    long long wonderfulSubstrings(string word) {
        int n = word.size();
        word = ' ' + word;
        long long res = 0;
        memset(cnt, 0, sizeof cnt);
        cnt[0] ++;
        for(int i = 1; i <= n; i++) {
            for(int j = 0; j < 10; j++) {
                if (word[i] - 'a' == j) 
                    s[i][j] = s[i - 1][j] + 1;
                else
                    s[i][j] = s[i - 1][j];
            }
            int state = 0;
            for(int j = 0; j < 10; j++) {
                state = state * 2 + s[i][j] % 2;
            }
            res += cnt[state];
            for(int j = 0; j < 10; j++) {
                res += cnt[state ^ (1 << j)];
            }
            cnt[state] ++;
        }
        return res;
    }
};
```


#### [统计为蚁群构筑房间的不同顺序](https://leetcode-cn.com/problems/count-ways-to-build-rooms-in-an-ant-colony/)

本题是一道计数问题，涉及到基础数论（费马小定理求乘法逆元）以及树形DP的思想。

状态表示：$dp(u)$表示以$u$为根节点的所有子树的所有方案数之和

状态计算：假设根结点$u$有$n$棵子树，记每棵子树的结点数目为$S_1, S_2, ... , S_n$。问题可以看成将$n$组的物品进行分配，不考虑组内部的顺序。在不考虑顺序的情况下，相当于对$\sum_{i = 1}^{n} S_i$进行全排列；如果考虑顺序，则需要除去每棵子树内部的排列数。所以最终的方案数为：
$$\frac{(S_1 + S_2 + ... + S_n)!}{S_1!S_2!...S_n!}$$

所以转移方程为：
$$dp(u) = \prod_{i = 1}^{n}dp(i) \frac{(S_1 + S_2 + ... + S_n)!}{S_1!S_2!...S_n!}$$

##### **Code Q4**
```cpp
typedef long long LL;
const int N = 100010, MOD = 1e9 + 7;

int fact[N], inv[N];
int dp[N], s[N];
int h[N], e[N], ne[N], idx;

class Solution {
public:

    void add(int a, int b) {
        e[idx] = b;
        ne[idx] = h[a];
        h[a] = idx ++;
    }


    int dfs(int u) {
        s[u] = 0;  
        for(int i = h[u]; ~i; i = ne[i]) {
            int j = e[i];
            dfs(j);
            s[u] += s[j];
        }
        dp[u] = fact[s[u]];
        for(int i = h[u]; ~i; i = ne[i]) {
            int j = e[i];
            dp[u] = (LL)dp[u] * inv[s[j]] % MOD * dp[j] % MOD; // 有2次乘法，可能爆long long，需要及时取模
        }
        s[u]++; // 加上根节点
        return dp[u];
    }


    int qmi(int a, int b) {
        int res = 1;
        while (b) {
            if (b & 1) res = (LL)res * a % MOD;
            a = (LL)a * a % MOD;
            b >>= 1;
        }
        return res;
    }

    int waysToBuildRooms(vector<int>& p) {
        int n = p.size();
        idx = 0;
        memset(h, -1, sizeof h);

        fact[0] = inv[0] = 1;
        for(int i = 1; i < N; i++) {
            fact[i] = (LL)fact[i - 1] * i % MOD;
            inv[i] = qmi(fact[i], MOD - 2) % MOD;
        }

        for(int i = 1; i < n; i++) add(p[i], i);
        return dfs(0);
    }
};
```

时间复杂度：$O(nlog MOD)$，每个结点遍历一次，求乘法逆元需要$log MOD$的时间。

##### **数论知识补充**
设模数为$m$，对于整数$a$，如果存在整数$a^{-1}$，使得
$$aa^{-1} \equiv 1 \pmod m$$
则称$a^{-1}$是$a$的乘法逆元。

- 何时存在乘法逆元？
当$gcd(a, m) = 1$时，$a^{-1}$存在。

因为本题$m$为质数，所以逆元一定存在。

不妨令$aa^{-1} = km + 1$，整理得$aa^{-1} - km = 1, k \in \mathbb{N}$

- 裴蜀定理
> 设$a, b$是不全为0的整数，则存在整数$x, y$使得$ax + by \equiv gcd(a, b)$

裴蜀定理可以推出$k$一定是存在的，也就是存在一组解$(a_0^{-1}, k_0)$。并且，$(a_0^{-1} + ca, k_0 + cm), c \in \mathbb{Z}$。

因此一定存在$a^{-1} \in (1, m)$。

- 费马小定理
> 如果$m$是质数，则$a^{m - 1} \equiv 1 \pmod m$

进行等价变形可得，
$$a^{m - 2} \equiv a^{-1} \pmod m$$

因此我们将求除法的问题变成求乘法的问题，利用快速幂可以在对数时间求出逆元。
