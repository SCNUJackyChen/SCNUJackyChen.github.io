---
title: ACWing weekly contest 29
date: 2021-12-12 13:41:58
tags: [离散化, 差分]
categories: 算法
katex: true
description: ACWing 周赛第29场题解
---

![LC](/images/ACWing_logo.png)

<!--more-->

###  **Solution**

T1是一道签到题，T3一道模板题，所以这次只写T2的题解。

#### [线段覆盖](https://www.acwing.com/activity/content/problem/content/5877/)

An intuitive idea is to create an array to record each segment, e.g, for segment $[l, r]$, plus one in the interval of $a_l$ ~ $a_r$. We can preprocess via computing a differential array and update the intervals in $O(1)$. One more issue is that the range of $l, r$ is too big (up to $10^{18}$) to be held in an array and computational impractical. Although the range of $l, r$ is big, they are sparse (only $2 \times 10^5$ numbers). Therefore, the original big array could be compressed into a small one with length up to $2 \times 10^5$, with the help of discretization. There are several ways to implement discretization method, and one simple way is to utilize the `std::map`. But in the contest I implement it from scratch 😅 (not familiar with map...)

One detail that easy to introduce bug is that, the index $r$ need to be stored as $r + 1$, otherwise, bugs may appear in the case one index acts as both beginning and endding of some segments.

一个直观的想法是开一个数组来记录每一个线段，比如，对于线段$[l, r]$，在区间$a_l$ ~ $a_r$加1。我们可以通过计算差分数组来做预处理，并且可以在常数时间内更新区间。另外一个问题是$l, r$的范围很大（达到$10^{18}$），以至于无法存在一个数组中并且时间复杂度爆炸。尽管$l, r$的范围比较大，但是它们是稀疏的（只有 $2 \times 10^5$ 个数）。因此，可以借助离散化把原始的大数组可以压缩成一个小数组，它的长度最多只有$2 \times 10^5$。有许多方法可以实现离散化，一种简单的方式是利用`std::map`。然而比赛中我是从头实现了一个😅（还是对map不太熟...)

本题容易出错的点是存下标的时候，$r$要存成$r + 1$，否则在某个下标既是线段开头又是线段结尾的情况下会出错。

##### **Code Q1**
```cpp
#include <iostream>
#include <cstring>
#include <algorithm>
#include <vector>
#include <map>


using namespace std;

const int N = 400010;
using LL = long long;
using PII = pair<LL, LL>;

int a[N], d[N];
vector<PII> add;
vector<LL> alls;

void insert(int l, int r, LL c) {
    d[l] += c;
    d[r] -= c;
}

int find(LL x) {
    int l = 0, r = alls.size() - 1;
    while (l < r) {
        int mid = l + r >> 1;
        if (alls[mid] >= x) r = mid;
        else l = mid + 1;
    }
    return l;
}

int main()
{
    int n;
    cin >> n;
    for (int i = 0; i < n; i ++ ) {
        LL l, r;
        cin >> l >> r;
        add.push_back({l + 1, r + 2});
        alls.push_back(l + 1);
        alls.push_back(r + 2);
    }
    sort(alls.begin(), alls.end());
    alls.erase(unique(alls.begin(), alls.end()), alls.end());
    
    for (auto p : add) {
        int l = find(p.first), r = find(p.second);
        insert(l, r, 1);    
    }
    unordered_map<int, LL> cnt;
    for (int i = 1; i < alls.size(); i ++ ) {
        d[i] += d[i - 1];
        cnt[d[i - 1]] += alls[i] - alls[i - 1];
        
    }
    for (int i = 1; i <= n; i++)
        cout << cnt[i] << ' ';
    
    return 0;
}
```
