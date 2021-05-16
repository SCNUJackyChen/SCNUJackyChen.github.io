---
title: LC 双周赛第48场
date: 2021-03-21 12:32:25
tags: [Leetcode, 哈希, 贪心, 状压DP]
categories: Leetcode周赛
katex: true
description:  LC双周赛第48场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

###  **题目**
[字符串中第二大的数字](https://leetcode-cn.com/problems/second-largest-digit-in-a-string/)
[设计一个验证系统](https://leetcode-cn.com/problems/design-authentication-manager/)
[你能构造出连续值的最大数目](https://leetcode-cn.com/problems/maximum-number-of-consecutive-values-you-can-make/)
[N 次操作后的最大分数和](https://leetcode-cn.com/problems/maximize-score-after-n-operations/)

### 赛后总结（吐槽）

😭😭😭 裂开的一场周赛...晚了20分钟开始（晚上出去玩了），然后第一题上来就WA了2发，第二题题干巨长（但其实不难），不想看，然后跳到第三题苦想了大概半小时，思路对头但是还是差一点点想到正确答案（贪心的思路真是具有跳跃性）。于是不开心地回来读第二题，好在最后还是理解了题意。心心念念的第三题最后还是没能做出来555。以后做得不好的周赛都要回来写一篇题解来惩罚一下自己😶

### **题解**

第一第二题属于简单题，直接暴力模拟即可。

#### **Code Q1**
```cpp
class Solution {
public:
    int secondHighest(string s) {
        int first = INT_MIN, second = INT_MIN;
        for(int i = 0; i < s.size(); i++) {
            if(s[i] >= '0' && s[i] <= '9') {
                int c = s[i] - '0';
                if(c > first) {
                    second = first;
                    first = c;
                } else if (c > second && c != first) {
                    second = c;
                }
            }    
        }
        if(second == INT_MIN) return -1;
        return second;
    }
};
```

#### **Code Q2**
```cpp
class AuthenticationManager {
public:
    int t;
    unordered_map<string, int> hash;
    
    
    AuthenticationManager(int timeToLive) {
        t = timeToLive;
    }
    
    void generate(string tokenId, int currentTime) {
        hash[tokenId] = currentTime;
    }
    
    void renew(string tokenId, int currentTime) {
        if(hash.count(tokenId)) {
            if(currentTime - hash[tokenId] < t)
                hash[tokenId] = currentTime;
        }
    }
    
    int countUnexpiredTokens(int currentTime) {
        int cnt = 0;
        for(auto [i, c]: hash) {
            if(currentTime - c < t) cnt++;
        }
        return cnt;
    }
};

/**
 * Your AuthenticationManager object will be instantiated and called as such:
 * AuthenticationManager* obj = new AuthenticationManager(timeToLive);
 * obj->generate(tokenId,currentTime);
 * obj->renew(tokenId,currentTime);
 * int param_3 = obj->countUnexpiredTokens(currentTime);
 */
```

#### **你能构造出连续值的最大数目（贪心）**

先排序。

令前缀和$S_i = \sum_i a_i$。

不妨设$a_1$ - $a_{i-1}$能够构造出 $[0, S_{i-1}]$，讨论$a_i$和$S_{i-1} + 1$的大小关系:

① $a_i > S_{i-1} + 1$, 那么$S_{i-1}+1$这个数无法构成，答案即为$S_{i-1}+1$；
② $a_i \le S_{i-1} + 1$, 那么$a_1$ - $a_i$能构成$[0 + a_i, S_{i-1} + a_i]$。$[0, a_i]$这个区间呢？因为：$a_i \le S_{i-1} + 1$且所有的数都是整数，那么可以这个不等式可以分解成：$a_i = S_{i-1} + 1$ 和 $a_i \le S_{i-1}$。分别讨论：
1. $a_i = S_{i-1} + 1$ ，显然$a_1$ - $a_{i}$能构成 $[0, S_{i}]$；
2. $a_i \le S_{i-1}$，因为$[0, S_{i-1}]$可以由$a_1$ - $a_{i-1}$构成，而$a_i \le S_{i-1}$，所以$[0, a_i]$亦可以由$a_1$ - $a_{i-1}$构成。因此，能构成 $[0, S_{i}]$。

因此在②情况中，我们继续看下一个数即可。

#### **Code Q3**
贪心的代码很简单，但思维比较难。
```cpp
class Solution {
public:
    int getMaximumConsecutive(vector<int>& coins) {
        sort(coins.begin(), coins.end());
        int res = 1;
        for(auto &x : coins) {
            if(x > res) break;
            res += x;
        }
        return res;
    }
};
```

#### **N 次操作后的最大分数和（状压DP）**

数组最多只有14个数，比较小，可以考虑用一个14位长度的二进制数来表示每一个数是否被选（1表示被选，0表示没被选）。

DP：
集合表示： `f[i]`表示在状态为`i`时(`i`看成若干个1和0)继续游戏，所有操作的方案，存的值为方案中得分最大值 

集合计算： 枚举状态`i`中1的2个位置(假设为j ,k)，然后计算这个选法的得分。得分的计算由3部分组成：
1. gcd(nums[j], nums[k]);
2. cnt， 这个可以通过计算状态中含有几个0来求得，因为操作了几次，就会少掉操作次数的2倍那么多个0；
3. f[i - (1<<j) - (1<<k)]，这里表示从i状态中去掉这2个选中位置的最大得分。

因为1，2都是定值，3是max值，所以求得的`f[i]`也是这种选法的最大值。

**时间复杂度分析：**

状态数：$O(2^{14})$
计算每个状态：$O(C_{14}^2)$
总时间：$O(2^{14} C_{14}^2)$

#### Code Q4
```cpp
class Solution {
public:
    int gcd(int a, int b) {
        return b ? gcd(b, a % b) : a;
    }

    int maxScore(vector<int>& nums) {
        int n = nums.size();
        vector<int> f(1 << n);
        for(int i = 0; i < 1 << n; i++) {
            int cnt = 0;
            for(int j = 0; j < n; j++) {
                if(!(i >> j & 1)) cnt ++;
            }
            cnt = cnt / 2 + 1;

            for(int j = 0; j < n; j++) {
                if(i >> j & 1)
                    for(int k = j + 1; k < n; k++) {
                        if(i >> k & 1)
                            f[i] = max(f[i], f[i - (1 << j) - (1 << k)] + cnt * gcd(nums[j], nums[k]));
                    }
            }
        }
        return f[(1 << n) - 1];
    }
};
```