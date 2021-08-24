---
title: LC 周赛第255场
date: 2021-08-24 16:37:24
tags: [Leetcode, GCD, 状压DP, 数学]
categories: Leetcode周赛
katex: true
description:  LC周赛第255场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

###  **题解**

#### [找出数组的最大公约数](https://leetcode-cn.com/problems/find-greatest-common-divisor-of-array/)
考察了gcd算法。

##### **Code Q1**
```cpp
class Solution {
public:
    int gcd(int x, int y) {
        int r;
        while(y) {
            r = x % y;
            x = y;
            y = r;
        }
        return x;
    }
    int findGCD(vector<int>& nums) {
        int ma = INT_MIN, mi = INT_MAX;
        for (auto c : nums) {
            ma = max(c, ma);
            mi = min(c, mi);
        }
        return gcd(ma, mi);
    }
};
```

#### [找出不同的二进制字符串](https://leetcode-cn.com/problems/find-unique-binary-string/)
从小到大枚举二进制，判断是否在数组中出现，注意前导0。

##### **Code Q2**

```cpp
class Solution {
public:
    string findDifferentBinaryString(vector<string>& nums) {
        int n = nums.size();
        for (int i = 0; i < 1 << n; i++) {
            string t;
            int x = i;
            while (x) {
                t += (x & 1) + '0';
                x >>= 1;
            }
            if (t.empty()) {
                for (int j = 0; j < n; j++)
                    t += '0';
            }
            if (t.size() < n) {
                    reverse(t.begin(), t.end());
                    while (t.size() < n) {
                        t.push_back('0');
                    }
                    reverse(t.begin(), t.end());
                }
            bool f = false;
            for (auto c : nums) {
                // cout << t << ' ' << c << endl;
                if (t == c) { 
                    f = true;
                    break;
                }
            }
            if (!f) {
                return t;
            }
        } 
        return "0";
    }
};

```
#### [最小化目标值与所选元素的差](https://leetcode-cn.com/problems/minimize-the-difference-between-target-and-chosen-elements/)
思路类似于背包问题，因为数据限制在每个数最大为70，所以总和最大只有$70^2 = 4900$。把每一个总和是否可达记为$f(num)$，逐行递推即可，最后遍历一遍找出所有可达的和中，与target绝对值最接近的一个。

在实现时可以借助bitset优雅地进行转移（学到了学到了orz

##### **Code Q3**

```cpp
class Solution {
public:
    int minimizeTheDifference(vector<vector<int>>& f, int target) {
        bitset<4900> S;
        S.set(0);
        int n = f.size();
        for (auto &v : f) {
            bitset<4900> t;
            for (int x : v) {
                t |= S << x;  // 十分优雅的转移
            }
            S = t;
        }
        int res = INT_MAX;
        
        for (int j = 0; j <= 4900; j++) {
            if (S[j])
                res = min(res, abs(target - j));
        }
        return res;
    }
};
```
#### [从子集的和还原数组](https://leetcode-cn.com/problems/find-array-given-subset-sums/)
如果所有的数都是非负数，本题是一道经典的递归题：每次取出`sums`中的最小值即为原集合的一个元素。

现在考虑更一般化的形式，将非负数改为正数。整体的思路还是向非负数版本靠拢，我们可以将`sums`中所有的数都加上一个偏差（这个偏差就是`sums`数组的最小值），使得`sums`中的数组全部变成非负数，这是我们可以求解的。

假设求出的集合为$S'$，如何得到原来的集合$S$？

注意到，所有的负数之和必然等于`sums`中的最小值。利用这一规律，我们可以在$S'$中寻找一系列数，使得它们的和恰好等于这个最小值的绝对值。然后把它们取相反数即可。

为什么这种操作必然可行？因为加了偏差前后2个集合是一一对应的，而求解出来的$S$和$S'$也是一一对应的，因此$S$中构成最小值的数一一对应于$S'$中的数。

##### **Code Q4**
```cpp
class Solution {
public:
    
    bool dfs(vector<int>& A, int target, int step) {
        if (step == A.size()) return target == 0;
        int x = A[step];
        if (dfs(A, target, step + 1)) return true;
        A[step] = -x;
        if (dfs(A, target - x, step + 1)) return true;
        A[step] = x;
        return false;
    }
    
    vector<int> recoverArray(int n, vector<int>& sums) {
        int mi = *min_element(begin(sums), end(sums));
        for (int &c : sums) c += -mi;
        multiset<int> S(begin(sums), end(sums)), t;
        
        vector<int> res;
        for (int i = 0; i < n; i++) {
            t.clear();
            int num = *next(S.begin());
            res.push_back(num);
            while(S.size()) {
                int first = *S.begin();
                t.insert(first);
                S.erase(S.begin());
                S.erase(S.find(first + num));  // 筛掉和num相关的和
            }
            swap(S, t);
        }
        
        dfs(res, -mi, 0);
        return res;
        
    }
};
```