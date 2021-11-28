---
title: ACWing weekly contest 27
date: 2021-11-28 14:19:04
tags: [二分, DP, 背包问题]
categories: 算法
katex: true
description: ACWing 周赛第27场题解
---

![LC](/images/ACWing_logo.png)

<!--more-->

###  **Solution**

#### [数字串](https://www.acwing.com/activity/content/problem/content/5823/)

Since $n$ is only $1000$, we can simply convert all numbers to string and concat them together.

因为$n$只有$1000$，我们只需要将所有的数字转换成字符串并且拼接起来即可。

##### **Code Q1**
```cpp
#include<iostream>
#include<cstring>

using namespace std;

string s;

void prep()
{
    for (int i = 0; i < 1000; i++) {
        s += to_string(i);
    }
}

int main()
{
    int T, n;
    prep();
    cin >> T;
    while (T--) {
        cin >> n;
        cout << s[n] << endl;
    }
    return 0;
}
```

#### [第k个数](https://www.acwing.com/activity/content/problem/content/5824/)

Since $n \times m$ is up to $2.5 \times 10^{11}$, brute-forcing the result of all the product ($i \times j$) is computationally impractical. Given that the final product array of length $n \times m$ is **sorted**, a fast strategy to find a number in sorted array is binary search. Our goal is to find the $k_{th} $ number (noted as $t$), which means there must be $k - 1$ numbers less or equal to $t$. As $t$ grows, the numbers that less or equal to $t$ also grow; while as $t$ reduces, the numbers that less or equal to $t$ reduce, too. Therefore, we can always find the smallest $t$ that satisfies the condition ($\ge k$) by binary search.

To count the numbers that are less or equal to $t$, we can iterate each row to count how many numbers satisfy the condition and add them up. The time complexity is $O(n)$

The time complexity of the whole algorithm is $O(nlog{nm})$.

因为$n \times m$高达$2.5 \times 10^{11}$，暴力枚举所有乘积是在计算上不可行。鉴于最后的乘积数组是一个**排好序**的长度为$n \times m$的数组，一个在排序数组找数的快速策略就是二分搜索了。我们的目标是找到第$k$个数(记为$t$)，这意味着一定会有$k-1$个数小于等于$t$。当$t$增大时，小于等于$t$的数字的数量也会增加；当$t$减小时，小于等于$t$的数字的数量也会减小。因此，我们总可以二分找到最小的$t$，使得小于等于它的数字的数量大于等于$k$。

对于统计小于$t$有多少个数，我们可以按行遍历一下，统计每行有几个数小于$t$，然后加起来。时间复杂度是$O(n)$的。

总的时间复杂度是 $O(nlog{nm})$。

##### **Code Q2**
```cpp
#include<iostream>

using namespace std;

using LL = long long;

LL n, m, k;

bool check(LL mid) {
    LL res = 0;
    for (int i = 1; i <= n; i++) 
        res += min(m, mid / i);
    return res >= k;
}

int main()
{
    cin >> n >> m >> k;
    LL l = 1, r = n * m;
    while (l < r) {
        LL mid = l + r >> 1;
        if (check(mid)) r = mid;
        else l = mid + 1;
    }
    
    cout << r << endl;
    
    return 0;
}

```

#### [选数](https://www.acwing.com/activity/content/problem/content/4084/)

Selecting $k$ numbers from an array can be regarded as selection problem with limitation, which is equivalent to the 0/1 Knapsack problem.

从数组中选$k$个数可以被视为带限制的选择问题，这跟0/1背包问题是等价的。

We can define $f[i][j][k]$ as: select $j$ numbers from an array of length $i$, the maximum number of factor $2$ when the number of factor $5$ is $k$. Here, we treat the number of factor $5$ as the volumn of the knapsack and treat the number of factor $2$ as value for each product. For each product (number), we have two choices -- take it or do not take it, so the transition of states is $f[i][j][k] = max(f[i - 1][j][k], f[i - 1][j - 1][k - v] + w)$, where $v$ is the volumn of the product, and $w$ is its value. For the initial state, $f[0][0][0] = 0$ and $f[x][0][y] = -\infty$ because they are invalid states. 

我们可以定义$f[i][j][k]$为：从长度为$i$的数组中选$j$个数，当因子$5$的个数为$k$时，因子$2$的最大数量。这里，我们把因子$5$的数量视为背包容量，把$2$的数量视为每个物品的价值。对于每个物品（数字），我们有2种选择--选或者不选，所以状态转移就是$f[i][j][k] = max(f[i - 1][j][k], f[i - 1][j - 1][k - v] + w)$，$v$代表物品的体积，$w$代表物品价值。对于初始状态，$f[0][0][0] = 0$， $f[x][0][y] = -\infty$因为这些都是不合法的状态。

To solve the 0/1 Knapsack problem with limitation, the time complexity is $O(200 \times 200 \times 200 \times log_510^{18} \approx O(2 \times 10^8)$, the space complexity is also $ O(2 \times 10^8)$ but we can optimize it to $O(10^6)$ by scrolling the 1st dimension of $f$.

求解带限制的0/1背包问题，时间复杂度是$O(200 \times 200 \times 200 \times log_510^{18} \approx O(2 \times 10^8)$，空间复杂度也是$ O(2 \times 10^8)$ ，但是可以优化掉第一维从而降到$O(10^6)$。

##### **Code Q3**
```cpp
#include<iostream>
#include<cstring>

using namespace std;

using LL = long long;

const int N = 210, M = 5100; // 5100 = log_5(10^18)

int f[N][M], w[N], v[N];
int n, m;

int main()
{
    cin >> n >> m;
    for (int i = 1; i <= n; i++) {
        LL x;
        cin >> x;
        while (x % 5 == 0) x /= 5, v[i]++;
        while (x % 2 == 0) x /= 2, w[i]++;
    }
    
    memset(f, -0x3f, sizeof f);
    f[0][0] = 0;
    
    for (int i = 1; i <= n; i++) {
        for (int j = m; j; j--) {
            for (int k = i * 26; k >= v[i]; k--) {
                f[j][k] = max(f[j][k], f[j - 1][k - v[i]] + w[i]);
            }
        }
    }
    
    int res = 0;
    for (int i = 1; i < M; i++)
        res = max(res, min(f[m][i], i)); // min of number factor 5 and 2
    cout << res << endl;
    
    return 0;
}
```