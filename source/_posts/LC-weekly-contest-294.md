---
title: LC weekly contest 294
date: 2022-06-05 23:40:23
tags: [Leetcode, 模拟, 前缀和,单调栈, 数学]
categories: Leetcode周赛
katex: true
description:  LC 周赛第294场题解
---

![LC](/images/Leetcode.jpg)

<!--more-->


### **Preface**

之后的周赛就只更新T3 & T4了，因为一般前两题都比较简单，还是把时间多分配给后面有意思的题目上（如果前两题也很有质量/比赛时做错，也会放进来）。


###  **Solution**

#### [表示一个折线图的最少线段数](https://leetcode.cn/problems/minimum-lines-to-represent-a-line-chart/)

思路不难但是却有坑点的一道题。很容易看出来只需要从左往右遍历，对于每一段，判断其斜率是否和前面维护的斜率一致，如果一致，则将其加入维护的集合中（集合中存着连续的斜率相同的线段）；如果不同，则答案加一，并更新斜率。

坑点就在于**判断斜率一致**这一步。根据初中数学知识，过$(x_1, y_1), (x_2, y_2)$和$(x_3, y_3), (x_4, y_4)$所形成的两条直线相互平行当且仅当$\frac{y_1 - y_2}{x_1 - x_2} = \frac{y_3 - y_4}{x_3 - x_4}, (x_1 - x_2)(x_3 - x_4) \neq 0$. 很容易想到用浮点数除法来判断相等，但在计算机中，浮点数类型是有精度限制的，也就是说，存在某些很小的数是计算机无法表示的。根据[IEEE 754标准](https://en.wikipedia.org/wiki/IEEE_754)，64位浮点数（也就是C++中的double类型）小数点后面用53个bit表示，换算成10进制大约是$10^{-16}$。对于本题，数据范围是$[1,10^9]$，可能存在浮点数下溢的情况，比如：$\frac{1}{10^9} - \frac{1}{10^9+1}$。为了避免精度问题，可以将比较等式调整为乘法形式。

##### **Code Q3**
```cpp
class Solution {
public:
    int minimumLines(vector<vector<int>>& p) {
        sort(p.begin(), p.end());
        int n = p.size();
        if (n == 1) return 0;
        int res = 1;
        int last_x = p[0][0], last_y = p[0][1];
        
        for (int i = 1; i < n; i++) {
            int a = p[i][1] - last_y, b = p[i][0] - last_x, c = p[i][1] - p[i - 1][1], d = p[i][0] - p[i - 1][0];
            if((long long)a * d == (long long)b * c) {
                continue;
            } else {
                res ++;
                last_x = p[i - 1][0], last_y = p[i - 1][1];
            }
        }
        return res;
    }
};
```

#### [巫师的总力量和](https://leetcode.cn/problems/sum-of-total-strength-of-wizards/)

本题据说是Amazon的一道高频OA，参考[这里](https://mp.weixin.qq.com/s?__biz=MzI2NzQ3OTQ1Mw==&mid=2247485030&idx=1&sn=b648c8a3c8a9b8fce625765fb345d233&chksm=eaff7694dd88ff825dd71a9b740dcba6711f192220c718361382b70534718b0d98936b5514c4&token=1764672036&lang=zh_CN#rd)。本题难点在于数学推导，我觉得在笔试题里面考察推公式属实有些恶心了。

我们可以考察每一个巫师作为最小力量时，他所在的所有组，进而可以算出这些组的总力量，最后将所有的总力量累加得到答案。

**难点1：** 如何找到某一个巫师最为最小力量时，他所在的所有组？
把巫师们的力量值想象成一堆柱子（力量越高柱子越高），那么当某个柱子作为最矮柱子时，意味着它周围的柱子都比它高。具体来说，从柱子左边第一个比它矮的柱子到右边第一个比它矮的柱子的这一段区间（不包括左右端点），就是它作为最矮柱子所在的**最大组**，而其他组必然是这个最大组的子数组。为了实现这一点，可以利用单调栈来维护每根柱子左边/右边第一个比它矮的柱子的下标。

**难点2：** 如何计算这些组的总力量？
为了方便讨论，定义一下当巫师$x$作为最小力量的巫师时，其所在的最大组的区间为$[L, R]$, 显然应该满足$l \le x \le r$。定义某一个满足条件的组的区间为$[i, j]$，则应该满足$L \le i \le x \le j \le R$. 那么可以写出如下的求和公式：

$$\sum_{j = x}^R\sum_{i = L}^x S_{ij}$$

其中，$S$是$[i, j]$的区间和。涉及到区间和，可以利用前缀和先做预处理。记前缀和数组为$s$, 则$S_{ij} = s[j + 1] - s[i]$。将其代入上式，得到：

$$\sum_{j=x+1}^{R+1}\sum_{i=L}^{x}(s[j]-s[i])=\left(\sum_{j=i+1}^{R+1}\sum_{i=L}^{x} s[j]\right)-\left(\sum_{j=i+1}^{R+1} \sum_{i=L}^{x} s[i]\right)=(x-L+1)\sum_{j=x+1}^{R+1} s[j]-(R-x+1) \sum_{i=L}^{x} s[i]$$



注意到，这里出现了$\sum_{j=x+1}^{R+1} s[j]$ 和 $\sum_{i=L}^{x} s[i]$，其实是对前缀和$s$再求和。与前缀和类似，我们也可以如法炮制地预处理出**前缀和的前缀和**（记为$ss$），从而加速计算。引入$ss$之后，上式可以进一步写为：

$$(x-L+1)\sum_{j=x+1}^{R+1}s[j]-(R-x+1)\sum_{i=L}^{x}s[i]=(x-L+1)(ss[R+2] - ss[x+1]) - (R-x+1)(ss[x+1]-ss[L])$$

总结一下，我们考察每一位巫师作为最小力量时，其所在的组的所有情况，这一类情况的总力量值可以由$(x-L+1)(ss[R+2] - ss[x+1]) - (R-x+1)(ss[x+1]-ss[L])$得到，将所有的总力量值加起来即得最终结果。


```cpp
class Solution {
public:

    using LL = long long;
    const int MOD = 1e9 + 7;

    int totalStrength(vector<int>& a) {
        int n = a.size();
        vector<int> left(n + 1, -1), right(n + 1, n);
        stack<int> stk;

        for (int i = 0; i < n; i++) {
            while (stk.size() && a[stk.top()] >= a[i]) stk.pop();
            if (stk.size()) left[i] = stk.top();
            stk.push(i);
        }

        stk = stack<int>();
        for (int i = n - 1; i >= 0; i--) {
            while (stk.size() && a[stk.top()] > a[i]) stk.pop();
            if (stk.size()) right[i] = stk.top();
            stk.push(i);
        }

        vector<LL> s(n + 1), ss(n + 2);
        for (int i = 1; i <= n; i++) s[i] = (s[i - 1] + a[i - 1]) % MOD;
        for (int i = 2; i <= n + 1; i++) ss[i] = (ss[i - 1] + s[i - 1]) % MOD;


        LL res = 0;
        for (int x = 0; x < n; x++) {
            int L = left[x] + 1, R = right[x] - 1;
            LL A = (x - L + 1) * ((ss[R + 2] - ss[x + 1] + MOD) % MOD) % MOD;
            LL B = (R - x + 1) * ((ss[x + 1] - ss[L] + MOD) % MOD) % MOD;
            res = (res + (a[x] * (A - B + MOD) % MOD) % MOD) % MOD;
        }

        return res;
    }
};
```