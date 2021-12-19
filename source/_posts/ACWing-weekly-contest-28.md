---
title: ACWing weekly contest 28
date: 2021-12-05 23:07:37
tags: [数论, 并查集]
categories: 算法
katex: true
description: ACWing 周赛第28场题解
---

![LC](/images/ACWing_logo.png)

<!--more-->

###  **Solution**

#### [最大公约数](https://www.acwing.com/problem/content/4086/)

Since the range of number $a_i$ is small, i.e $[1,10^5]$, we can iterate each factor, which should also be in the range of $[1,10^5]$. For each factor, we can count the number of its multiples in the given array and update the max count. The time complexity is $O(n + \frac{n}{2}+\frac{n}{3}+...+\frac{n}{n}) \approx O(nlnn)$ (Harmonic series)

因为$a_i$的取值，即$[1,10^5]$，是比较小的，我们可以枚举每个公约数，这些公约数也落在$[1,10^5]$的范围内。对于每一个公约数，我们可以统计它的倍数在给定的数组中出现的次数，并更新最大值。时间复杂度为$O(n + \frac{n}{2}+\frac{n}{3}+...+\frac{n}{n}) \approx O(nlnn)$ （调和级数）

> Extension
> How about finding the longest increasing non-prime subsequence? See [Codeforces 264B](https://codeforces.com/problemset/problem/264/B)

##### **Code Q1**
```cpp
#include<iostream>

using namespace std;

const int N = 100010;
int f[N];

int main()
{
    int n, mm = 0;
    cin >> n;
    for (int i = 0; i < n; i++) {
        int x;
        cin >> x;
        f[x]++;
        mm = max(mm, x);
    }
    
    int res = 1;
    for (int i = 2; i <= mm; i++) {
        int cnt = 0;
        for (int j = i; j <= mm; j += i) {
            cnt += f[j];
        }
        res = max(res, cnt);
    }
    cout << res << endl;
    return 0;
}
```

#### [号码牌](https://www.acwing.com/problem/content/4087/)

Based on array $d$, we can find the "neighbours" of a child. Such relationship of whether they can exchange number card can be regarded as  edge. Given such "neighbourhood" relationship, the question can be converted into a graph problem. We notice that children will form groups (or connected component) in the graph. Furthermore, the children can get any number card from their group members, which means that we only need to check for every group, whether the set of number cards the kids want is exactly the same as the set of what they have currently. For example, if there are three children in group A, with number card say $[1,2,3]$; and the cards they want are $[3,2,1]$. Then children in group A can all get their favorite card by several exchanges.

Therefore, the algorithm could be described as the following processes:

1. Treat child as node and whether they can exchange number card as edge;
2. Divide the graph into several connected components (can be done by disjoint set);
3. For each group, check whether the set of number cards the children have is equal to the set of number cards they want.


根据数组$d$，我们可以找到小孩的“邻居”。这种关系可以视为一条边，也就是他们是否可以相互交换号码牌的关系。给定这些“邻居”的关系，这个问题可以转化成一个图问题。注意到，孩子们会在图中形成若干组（或者叫联通块）。并且，孩子们可以拿到他们同一组中任意一位小孩手中的卡片，这意味着，我们只需要检查一下同一个组中，孩子们手上的卡片的集合是否等于他们手中卡片的集合。例如，如果有3个小孩在A组中，他们手上的号码牌是$[1,2,3]$；并且他们想要的牌是$[3,2,1]$。那么通过几次交换，A组的孩子们都可以拿到他们心仪的牌。

因此，算法可以描述为以下的流程：
1. 将孩子们当初节点，孩子们是否可以交换牌这一关系视为边；
2. 在图中划分连通块(可以通过并查集来完成)；
3. 对于每一个组，检查孩子们想要的号码牌的集合是否等于他们手上有的牌的集合。


##### **Code Q2**
```cpp
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

const int N = 110;
int n;
int a[N], d[N], p[N];
vector<int> A[N], B[N];

int find(int x) {
    if (p[x] != x) p[x] = find(p[x]);
    return p[x];
}

void merge(int a, int b) {
    if (a < 0 || a >= n) return;
    p[find(a)] = find(b);
}

int main()
{
    cin >> n;
    for (int i = 0; i < n; i ++ ) cin >> a[i];
    for (int i = 0; i < n; i ++ ) cin >> d[i];
    
    for (int i = 0; i < n; i ++ ) p[i] = i;
    
    for (int i = 0; i < n; i ++ ) {
        merge(i - d[i], i);
        merge(i + d[i], i);
    }
    
    for (int i = 0; i < n; i ++ ) {
        A[find(i)].push_back(a[i]); // current 
        B[find(i)].push_back(i+1); // looking-for
    }
    
    for (int i = 0; i < n; i ++ ) {
        sort(A[i].begin(), A[i].end());
        if (A[i] != B[i]) {
            cout << "NO";
            return 0;
        }
    }
    cout << "YES";
    return 0;
}
```