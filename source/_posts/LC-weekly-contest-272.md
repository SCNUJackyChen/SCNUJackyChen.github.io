---
title: LC weekly contest 272
date: 2021-12-19 12:40:23
tags: [Leetcode, 模拟, 思维题, LIS]
categories: Leetcode周赛
katex: true
description:  LC 周赛第272场题解
---


![LC](/images/Leetcode.jpg)

<!--more-->

### **Summary**

近两周都是手速场，今天迟了10分钟才开始，T4没看出来考LIS，3题签到后梦游了1小时😪 总结：GG，经典模型不熟练，继续回炉重造

###  **Solution**

#### [找出数组中的第一个回文字符串](https://leetcode-cn.com/problems/find-first-palindromic-string-in-the-array/)

##### **Code Q1**
```cpp
class Solution {
public:
    
    bool check(string s) {
        int i = 0, j = s.size() - 1;
        while (i < j) {
            if (s[i] == s[j]) {
                i++, j--;
            } else {
                return false;
            }
        }
        return true;
    }
    string firstPalindrome(vector<string>& words) {
        for (string word : words) {
            if (check(word)) return word;
        }
        return "";
    }
};
```

#### [向字符串添加空格](https://leetcode-cn.com/problems/adding-spaces-to-a-string/)

##### **Code Q2**
```cpp
class Solution {
public:
    string addSpaces(string s, vector<int>& spaces) {
        string res;
        int last = 0;
        for (int x : spaces) {
            res += s.substr(last, x - last) + ' ';
            last = x;
        }
        
        res += s.substr(last);
        return res;
    }
};
```
#### [股票平滑下跌阶段的数目](https://leetcode-cn.com/problems/number-of-smooth-descent-periods-of-a-stock/)

We can seperate the array into different groups, which are all in *smooth descent period*. After the division, we only need to consider each sub-array and calculate the number of *smooth descent periods* via $(1 + 2 + 3 + ...+ n) = \frac{(1+n)n}{2}$. 

我们可以将数组进行分组，使得每组都是*平滑下降阶段*。分组后，只需要考虑每一个子数组并且计算相应的*平滑下降阶段*数量即可，计算公式为：$(1 + 2 + 3 + ...+ n) = \frac{(1+n)n}{2}$。
##### **Code Q3**
```cpp
class Solution {
public:
    long long getDescentPeriods(vector<int>& prices) {
        long long res = 0;
        int n = prices.size();
        if (n == 1) return 1;
        long long cnt = 1;
        for (int i = 0; i < n; i++) {
            if (i) {
                if (prices[i - 1] - prices[i] == 1) cnt ++;
                else res += (1 + cnt) * cnt / 2, cnt = 1;
            }
        }
        res +=  (1 + cnt) * cnt / 2;
        
        return res;
    }
};
```

#### [使数组 K 递增的最少操作次数](https://leetcode-cn.com/problems/minimum-operations-to-make-the-array-k-increasing/)

This question is based on [LC-300-Longest-increasing-subsequence](https://leetcode-cn.com/problems/longest-increasing-subsequence). To minimize the operation number, we can firstly calculate the longest increasing subsequence, which is no need to be modified; then for the rest number, we can simply change those numbers to their nearest numbers (left or right). Therefore, the number of operation will be $length(a) - LIS(a)$, where $a$ is the one of the $k$ increasing array.

这道题基于[LC-300-最长上升子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence)。为了最小化操作次数，我们可以先求出最长上升子序列，这一序列内的数字是不需要改动的；然后对于剩下的数字，我们只需要把它们修改成最近的数字即可（左边或者右边）。因此，操作次数可以表示为： $length(a) - LIS(a)$，这里$a$是$k$递增数组中的一个数组。

##### **Code Q4**

```cpp
class Solution {
public:
     int LIS(vector<int>& nums) {
        vector<int> q;
        q.push_back(-2e9);
        int n = nums.size();
        for (int i = 0; i < n; i++) {
            if (q.empty() || nums[i] >= q.back()) q.push_back(nums[i]);
            else {
                int l = 0, r = q.size() - 1;
                while (l < r) {
                    int mid = l + r + 1 >> 1;
                    if (q[mid] <= nums[i]) l = mid; // <= because our goal is to get a non-decreasing subarray
                    else r = mid - 1;
                }
                q[l + 1] = nums[i];
            }
        }   
        return q.size() - 1;
    }
    
    int kIncreasing(vector<int>& arr, int k) {
        
        int n = arr.size();
        
        vector<vector<int>> s(k + 1);
        int cnt = 0;
        for (int i = 0; i < n; i++) {
            s[i % k].push_back(arr[i]); // divided into several subarrays
        }
        
        for (int i = 0; i < k; i++) { // for each subarray, calculate the operation number
            cnt += s[i].size() - LIS(s[i]);
        }
        
        return cnt;
    }
};
```