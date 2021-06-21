---
title: 完美矩阵
date: 2021-06-22 00:07:35
tags: [绝对值不等式, 思维题]
description: 一道精妙的思维题
categories: 算法
katex: true
---

### **题目**

[完美矩阵](https://www.acwing.com/problem/content/3568/)

如果一个矩阵能够满足所有的行和列都是回文序列，则称这个矩阵为一个完美矩阵。

一个整数序列 $a_1,a_2,…,a_k$，如果满足对于任何整数 $i（1≤i≤k）$，等式 $a_i=a_{k−i+1}$ 均成立，则这个序列是一个回文序列。

给定一个 $n×m$ 的矩阵$ a$，每次操作可以将矩阵中的某个元素加一或减一，请问最少经过多少次操作后，可以将矩阵 $a$ 变为一个完美矩阵？

**输入格式**
第一行包含整数 $T$，表示共有$ T $组测试数据。

每组数据第一行包含整数 $n$ 和 $m$，表示矩阵的大小。

接下来 $n$ 行，每行包含 $m $个整数 $a_{ij}$，表示矩阵中的元素。

**输出格式**
每组数据输出一行，一个答案，表示最少操作次数。

**数据范围**
$1≤T≤10,$
$1≤n,m≤100,$
$0≤a_{ij}≤10^9$

**输入样例：**
```
2
4 2
4 2
2 4
4 2
2 4
3 4
1 2 3 4
5 6 7 8
9 10 11 18
```
**输出样例：**
```
8
42
```
**样例解释**
第一组数据可以通过 8 步操作得到以下矩阵：
```
2 2
4 4
4 4
2 2
```
第二组数据可以通过 42 步操作得到以下矩阵：
```
5 6 6 5
6 6 6 6
5 6 6 5
```

### **思路**

本题有2个难点。
 -  第一个是如何处理所有行列都是回文序列。这里需要观察出一个性质：如果一个矩阵是完美矩阵（记为$w$），那么对于某个位置$(i,j)$，必然有$w_{i,j} = w_{n - i, j} = w_{i, m - j} = w_{n - i, m - j}$，其中$n, m$为矩阵的长宽。由这个性质我们可以将原矩阵分割成4部分（左上，右上，左下，右下），每确定一个位置，另外3个位置唯一确定，并且不同位置所对应的4元组之间相互独立。

 - 第二个是如何在一个4元组中找到一个值，使得4个数相等的操作数最小。这里用到了绝对值不等式。$|a - x| + |b - x| + |c - x| +|d - x|$的最小值在$x$取$a, b, c, d$的中位数时取到。

### **代码实现**

```cpp
#include<iostream>
#include<set>
#include<algorithm>
#include<vector>

using namespace std;

typedef long long LL;
typedef pair<int, int> PII;

const int N = 110;
int w[N][N];


LL cal (set<PII> S)
{
    vector<int> a;
    for(auto c : S) a.push_back(w[c.first][c.second]);
    sort(a.begin(), a.end());
    int n = a.size();
    LL res = 0;
    for(int i = 0; i < n; i++)
        res += abs(a[i] - a[n / 2]);
    return res;
}   

int main()
{
    int T, n, m;
    cin >> T;
    while (T--) {
        cin >> n >> m;
        for(int i = 0; i < n; i++)
            for(int j = 0; j < m; j++)
                scanf("%d", &w[i][j]);
        LL res = 0;
        for(int i = 0; i <= n - 1 - i; i++)
            for(int j = 0; j <= m - 1 - j; j++) 
                res += cal({ {i, j}, {n - 1 - i, j}, {i, m - 1 - j}, {n - 1 - i, m - 1 - j}});
        cout << res << endl;
    }
    
    return 0;
}
```