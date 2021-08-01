---
title: LC 周赛第252场
date: 2021-08-01 16:43:05
tags: [Leetcode, 模拟, 贪心, DP]
categories: Leetcode周赛
katex: true
description:  LC周赛第252场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **反思**
平时没有常规刷题到了周赛就明显感觉手生了，加上早上没睡醒，签到题都做了10min。今天的题难度不大，但是却做得很烂。希望能给自己打一剂鸡血，常规练习不能偷懒。

### **题解**

#### [三除数](https://leetcode-cn.com/problems/three-divisors/)
直接模拟，统计一共有多少个除数，判断是否为3。

##### **Code Q1**
```cpp
class Solution {
public:
    bool isThree(int n) {
        int cnt = 0;
        for (int i = 1; i <= n; i++) {
            if (n % i == 0) cnt ++;
        }
        return cnt == 3;
    }
};
```

#### [你可以工作的最大周数](https://leetcode-cn.com/problems/maximum-number-of-weeks-for-which-you-can-work/)
这个贪心以前没有见过，考试想了很久也没有想出来。
可以看最大值和余下值的关系。
 - 如果最大值小于等于总和的一半，也就是说最大值小于等于余下值之和，那么总可以完成所有任务。如何完成呢？我们采用以下策略。
1. 记最大值为$m$，余下值中的最大值为$k$，当$m > k$时，将全体值分为2组，一组为最大值，一组为余下值，先执行$m$所在的任务，再执行$k$所在任务（注意，这里的$k$所在任务可能会变化），直到$m = k$；
2. 当$m = k$时，先执行$m$所在任务，然后将$m$任务与$k$交换位置。这样操作可以保证，在执行最大值这一组时$m$不变，亦保证了最大值组始终为全局最大值。
3. 重复1，2直到结束，所有的任务都将完成。

 - 如果最大值大于总和一半。那么同样将所有值分为最大值组和余下值组，先执行最大值组，这样最后会剩下最大值组没有执行完。执行次数应该为$2 * (sum - m) + 1$。

##### **Code Q2**
```cpp
using LL = long long;
class Solution {
public:
    long long numberOfWeeks(vector<int>& milestones) {
        LL sum = 0, m = 0;
        for (LL x : milestones) {
            m = max(m, x);
            sum += x;
        }
        if (m > sum - m) return 2 * (sum - m) + 1; // 不能完成
        return sum;  // 能完成
    }
};
```
#### [收集足够苹果的最小花园周长](https://leetcode-cn.com/problems/minimum-garden-perimeter-to-collect-enough-apples/)
简单的推公式题，可以推出第$n$层外围的苹果数量为$12n^2$。

##### **Code Q3**
```cpp
class Solution {
public:
    long long minimumPerimeter(long long t) {
        long long sum = 0;
        for (int i = 1; ;i++) {
            sum += 12 * (long long)i * i;
            if (sum >= t) return 8 * (long long)i;
        }
        return sum;
    }
};
```
#### [统计特殊子序列的数目](https://leetcode-cn.com/problems/count-number-of-special-subsequences/)
设`f[i][j]`为从`0~i`中取，`j`型序列的数量。`j`的取值为`0,1,2`，分别表示：
1. 全都是0；
2. 连续0 + 连续1；
3. 连续0 + 连续1 + 连续2 （特殊序列）。

考虑状态转移：
 - 类型0：用第`i`个数是否属于之前子序列作为划分，可以将0放到之前的全0序列后面，这样又多了`f[i - 1][0]`个新的子序列；同时，也可以把自己当成一个序列（不属于前面任何序列），这样多了1个新序列。
`f[i][0] = 2 * f[i - 1][0] + 1`

 - 类型1：同理。用第`i`个数是否属于之前子序列作为划分，可以将1放到之前的类型1序列后面，这样又多了`f[i - 1][1]`个新的子序列；同时，也可以放到全0序列后面，这样多了`f[i - 1][0]`个新序列。
`f[i][1] = 2 * f[i - 1][1] + f[i - 1][0]`

 - 类型2：同理。用第`i`个数是否属于之前子序列作为划分，可以将1放到之前的类型2序列后面，这样又多了`f[i - 1][2]`个新的子序列；同时，也可以放到类型1序列后面，这样多了`f[i - 1][1]`个新序列。
`f[i][2] = 2 * f[i - 1][2] + f[i - 1][1]`

##### **Code Q4**

```cpp
class Solution {
public:
    int countSpecialSubsequences(vector<int>& nums) {
        int n = nums.size();
        const int MOD = 1e9 + 7;
        vector<vector<int>> f(n + 1, vector<int>(3, 0));
        for (int i = 1; i <= n; i++) {
            int x = nums[i - 1];
            if (x == 0) {
                f[i][0] += (2 * f[i - 1][0] % MOD + 1) % MOD;
                f[i][1] = f[i - 1][1];
                f[i][2] = f[i - 1][2];
            } else if (x == 1) {
                f[i][1] = (2 * f[i - 1][1] % MOD + f[i - 1][0]) % MOD;
                f[i][0] = f[i - 1][0];
                f[i][2] = f[i - 1][2];
            } else {
                f[i][2] = (2 * f[i - 1][2] % MOD + f[i - 1][1]) % MOD;
                f[i][1] = f[i - 1][1];
                f[i][0] = f[i - 1][0];
            }
        }
        return f[n][2];
    }
};
```