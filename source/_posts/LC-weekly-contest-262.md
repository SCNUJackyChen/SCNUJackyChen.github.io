---
title: LC weekly contest 262
date: 2021-10-11 12:17:48
tags: [Leetcode, 贪心, 二分]
categories: Leetcode周赛
katex: true
description:  review and solution for LC weekly contest 262
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **Recent Situation**

These days I am burried in heavy course projects and courses, and that is the reason why I have not updated the LC contest for a month😭.

Although the frequency of participating the contest will be reduced, like twice a week, I hope I can keep my passion for programming contest (PC). I use the word "passion" here because I do not want it to be a utilitarian stuff, like preparing for a job interview. I would prefer to regard it as a *Thinking Gymnastics*. This  statement is not from me, but from a PC master and  I quite agree with him. For me, participating PC can keep my thinking active and acute; furthermore, converting my idea into codes is a quite satisfying things; lastly, each time when I review the contest and restate my solution, it is a good chance to practice my ability of expression.

Start with this blog, I will change my language from Chinese to English. I know my English writing is not good, but I will try my best🤣


###  **Solution**

#### [至少在两个数组中出现的值](https://leetcode-cn.com/problems/two-out-of-three/)

Since the length of `nums` is quite small, we can directly do the brute force.

More specifically, we can count the frequency of each number in all three arrays and put them together as the result.

##### **Code Q1**

```cpp
class Solution {
public:
    vector<int> twoOutOfThree(vector<int>& nums1, vector<int>& nums2, vector<int>& nums3) {
        unordered_map<int, int> hash;
        
        unordered_map<int, int> t;
        for (auto x: nums1) {
            if (!t.count(x)) {
                hash[x] ++;
                t[x] ++;   
            }
        }
        t.clear();
        for (auto x: nums2) {
            if (!t.count(x)) {
                hash[x] ++;
                t[x] ++;   
            }
        }
        t.clear();
        for (auto x: nums3) {
            if (!t.count(x)) {
                hash[x] ++;
                t[x] ++;   
            }
        }
        
        vector<int> res;
        for (auto [c, s] : hash) {
            if (s >= 2) {
                res.push_back(c);
            }
        }
        
        return res;
        
    }
};
```

#### [获取单值网格的最小操作数](https://leetcode-cn.com/problems/minimum-operations-to-make-a-uni-value-grid/)

Greedy algorithm. Flatten the 2D matrix and sort, then find the middle element as our pivot.

- Why the greedy algorithm works?
Obviously the pivot (the number that in the outcome matrix) can be any element in the matrix. Supposed we choose $y$ as our pivot, and there are $p$ elements less than $y$, $q$ elements greater than $y$. $x$ stands for the step length.
1. If $p < q$, when $y$ plus $x$, the total operation number can be reduced by $q - p$;
2. If $p > q$, when $y$ minus $x$, the total operation number can be reduced by $p - q$.

To conclusion, the optimized point must be $p = q$.


##### **Code Q2**
```cpp
class Solution {
public:
    int minOperations(vector<vector<int>>& grid, int x) {
        int n = grid.size(), m = grid[0].size();
        vector<int> f(n * m);
        for (int i = 0; i < n;i ++)
            for (int j = 0; j < m; j++) 
                f[i * m + j] = grid[i][j];
        sort(f.begin(), f.end());
        int mid = f.size() / 2;
        int sum = 0;

        bool flag = true;
        for (int i = 0; i < f.size(); i++) {
            if (abs(f[i] - f[mid]) % x) {
                flag = false;
                break;
            }
            sum += abs(f[i] - f[mid]);
        }
        if (flag) return sum / x;
        return -1;
    }
};
```
#### [股票价格波动](https://leetcode-cn.com/problems/stock-price-fluctuation/)

This problem is about advanced data structure. 

According to problem statement, we should dynamically maintain a sorted data structure for both time and price. A C++ `map` can be used to store `<time, price>` pair, while C++ `set` can be used for `price`.

##### **Code Q3**
```cpp
class StockPrice {
public:
    
    map<int, int> mp; // t->p
    multiset<int> price;
    
    StockPrice() {
        mp.clear();
        price.clear();
    }
    
    void update(int t, int p) {
        bool f = false;
        int tmp = -1;
        if (mp.count(t)) f = true, tmp = mp[t];
        mp[t] = p;
        if (f) {
            auto it = price.find(tmp);
            if (it != price.end()) {
                price.erase(it); 
            } 
        }
        price.insert(p);
    }
    
    int current() {
        return (*(--mp.end())).second;
    }
    
    int maximum() {
        return *(--price.end());
    }
    
    int minimum() {
        return *(price.begin());
    }
};

/**
 * Your StockPrice object will be instantiated and called as such:
 * StockPrice* obj = new StockPrice();
 * obj->update(timestamp,price);
 * int param_2 = obj->current();
 * int param_3 = obj->maximum();
 * int param_4 = obj->minimum();
 */
```

#### [将数组分成两个数组并最小化数组和的差](https://leetcode-cn.com/problems/partition-array-into-two-arrays-to-minimize-sum-difference/)

Notice that the data range is $[0,30]$, so brute-force is not feasible.

We can treat the $2n$ array as 2 seperate sub-arrays with length $n$. Then we can pick $[0,n]$ number of elements from each sub-array. Next, we reallocate each element in each sub-array to form 2 new arrays. If the element belong to array_1, then we add it to the sum; otherwise, we subtract it from the sum (mean to say it belongs to array_2).

Now, our question is equal to, find 2 valid (the total length is $n$) combinaton from 2 sub-arrays, make their sum as close as possible.

Since the original array is equally divided into 2 parts, we now can do the brute-force. Then we preprocess the array of sum by sorting. Finally we can apply binary search to find the closest sum.

*Similar question*
[1755. 最接近目标值的子序列和](https://leetcode-cn.com/problems/closest-subsequence-sum/)

*When the range of array element is small (e.g less than $10^5$), this problem can be solved by treating it as a 0/1 knapsack problem, the complexity will be $O(n^2k)$, where $k$ is the range.*
[1049. 最后一块石头的重量 II](https://leetcode-cn.com/problems/last-stone-weight-ii/)



##### **Code Q4**
```cpp
class Solution {
public:
    int minimumDifference(vector<int>& nums) {
        int n = nums.size() / 2;
        vector<vector<int>> s(n + 1);
        for (int i = 0; i < 1 << n; i++) {
            int cnt = 0, sum = 0;
            for (int j = 0; j < n; j++) {
                if (i >> j & 1) {
                    cnt++;
                    sum += nums[j];
                } else {
                    sum -= nums[j];
                }
            }
            s[cnt].push_back(sum);
        }

        for (auto& x : s) sort(x.begin(), x.end());

        int res = INT_MAX;
        for (int i = 0; i < 1 << n; i++) {
            int cnt = 0, sum = 0;
            for (int j = 0; j < n; j++) {
                if (i >> j & 1) {
                    sum -= nums[j + n];
                } else {
                    cnt ++;
                    sum += nums[j + n];
                }
            }

            int l = 0, r = s[cnt].size() - 1;
            while (l < r) {
                int mid = l + r + 1 >> 1;
                if (s[cnt][mid] <= sum) l = mid;
                else r = mid - 1;
            }
            res = min(res, abs(s[cnt][l] - sum));
            if (r < s[cnt].size() - 1) res = min(res, abs(s[cnt][l + 1] - sum));

        }
        return res;

    }
};
```