---
title: LC biweekly contest 71
date: 2022-02-08 23:36:00
tags: [Leetcode, 贪心, 模拟, DP, 堆, 前后缀分解]
categories: Leetcode周赛
katex: true
description:  LC双周赛第71场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **Solution**

#### [拆分数位后四位数字的最小和](https://leetcode-cn.com/problems/minimum-sum-of-four-digit-number-after-splitting-digits/)


To get the smallest sum, we just need to ensure that the ten digits are smallest. So we can divide the number into four digits, sort them and pick up the first two smallest number as our ten digits, and the left two number for one digits. The result should be $10\times (s[0] + s[1]) + s[2] + s[3]$.
##### **Code Q1**

```cpp
class Solution {
public:
    int minimumSum(int num) {
        vector<int> s;
        while (num) {
            s.push_back(num % 10);
            num /= 10;
        }
        sort(s.begin(), s.end());
        return 10 * (s[0] + s[1]) + s[2] + s[3];
    }
};
```

#### [根据给定数字划分数组](https://leetcode-cn.com/problems/partition-array-according-to-given-pivot/)

##### **Code Q2**
```cpp
class Solution {
public:
    vector<int> pivotArray(vector<int>& nums, int pivot) {
        vector<int> a, b, c;
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] < pivot) a.push_back(nums[i]);
            else if (nums[i] > pivot) b.push_back(nums[i]);
            else c.push_back(nums[i]);
        }
        
        vector<int> res = a;
        for (int i = 0; i < c.size(); i++) res.push_back(c[i]);
        for (int i = 0; i < b.size(); i++) res.push_back(b[i]);
        return res;
    }
};
```

#### [设置时间的最少代价](https://leetcode-cn.com/problems/minimum-cost-to-set-cooking-time/)

Totally we have 2 cases:

1. $a$ minute(s) + $b$ second(s), where $b < 60$;
2. $(a - 1)$ minutes(s) + $(b + 60)$, where $b + 60 \le 99$

We can calculate the cost for each case by emulating the move and press process, the minimum of them will be the answer.

##### **Code Q3**
```cpp
class Solution {
public:
    int minCostSetTime(int startAt, int moveCost, int pushCost, int target) {
        int m = target / 60, s = target % 60;
        if (m > 99) {  // e.g target == 6000
            m--;
            s += 60;
        }

        // case 1
        string str;
        str.push_back(m / 10 + '0');
        str.push_back(m % 10 + '0');
        str.push_back(s / 10 + '0');
        str.push_back(s % 10 + '0');
        
        reverse(str.begin(), str.end());
        while (str.back() == '0') str.pop_back();
        reverse(str.begin(), str.end());
        
        int res = INT_MAX;
        int cost = 0, cur = startAt;
        
        for (int i = 0; i < str.size(); i++) {
            if (i == 0) {
                if (str[i] - '0' != startAt) cost += moveCost;
                cur = str[i] - '0';
                cost += pushCost;
            } else {
                if (str[i] - '0' != cur) cost += moveCost;
                cur = str[i] - '0';
                cost += pushCost;
            }
        }
        
        res = min(res, cost);

        // case 2
        cost = 0, cur = startAt;
        m = target / 60 - 1, s = target % 60 + 60;
        if (s > 99 || target < 60) return res;
        str = "";
        str.push_back(m / 10 + '0');
        str.push_back(m % 10 + '0');
        str.push_back(s / 10 + '0');
        str.push_back(s % 10 + '0');
        
        
        reverse(str.begin(), str.end());
        while (str.back() == '0') str.pop_back();
        reverse(str.begin(), str.end());
        for (int i = 0; i < str.size(); i++) {
            if (i == 0) {
                if (str[i] - '0' != startAt) cost += moveCost;
                cur = str[i] - '0';
                cost += pushCost;
            } else {
                if (str[i] - '0' != cur) cost += moveCost;
                cur = str[i] - '0';
                cost += pushCost;
            }
        }
        res = min(res, cost);
        
        return res;
    }
};

```
#### [删除元素后和的最小差值](https://leetcode-cn.com/problems/minimum-difference-in-sums-after-removal-of-elements/)

The question is equivalent to find a break point, which will divide the $3n$ array into two parts. In these $2$ parts with length longer than $n$, we will select $n$ numbers from each of them and sum them up respectively to get the smallest difference. 

So the question now will be how to select those $n$ numbers, in order to get the smallest $S_{first} - S_{second}$?

Since the calculation of $S_{first}$ and $S_{second}$ are independent, we may simply compute the minimum of $S_{first}$ and maximum of $S_{second}$.

To maintain a prefix/suffix minimum/maximum sum, with restriction of size $n$, the data structure -- heap is commonly used, which can maintain smallest or biggest number in  $O(logn)$.

Therefore, we may have a $O(nlogn)$ algorithm.

##### **Code Q4**

```cpp
class Solution {
public:
    long long minimumDifference(vector<int>& nums) {
        int n = nums.size(), m = n / 3;
        priority_queue<int, vector<int>, greater<int>> min_pq;
        long long sum = 0;
        for (int i = n - m; i < n; i++) {
            min_pq.push(nums[i]);
            sum += nums[i];
        }

        vector<long long> b(n);  // 从后i个数中选，包含第i个数的所有n个数的集合的最大和
        b[n - m] = sum;
        for (int i = n - m - 1; i >= m; i--) {
            min_pq.push(nums[i]);
            sum += nums[i] - min_pq.top();
            b[i] = sum;
            min_pq.pop();
        }

        sum = 0;
        priority_queue<int, vector<int>> max_pq;
        for (int i = 0; i < m; i++) {
            max_pq.push(nums[i]);
            sum += nums[i];
        }

        long long res = sum - b[m];  // 从前i个数中选，包含第i个数的所有n个数的集合的最小和
        for (int i = m; i < n - m; i++) {
            max_pq.push(nums[i]);
            sum += nums[i] - max_pq.top();
            res = min(res, sum - b[i + 1]);
            max_pq.pop();
        }

        return res;
    }
};
```