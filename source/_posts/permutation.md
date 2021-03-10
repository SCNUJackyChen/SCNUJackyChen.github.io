---
title: 排列专题( LC-31, 46, 47, 60)
date: 2021-03-09 23:49:05
tags: [Leetcode, 排列, DFS, 搜索]
description: 总结排列相关的问题
categories: 算法
katex: true
---

![LC](/images/Leetcode.jpg)

<!--more-->

###  排列专题

总结了4道相关题目：
1. [全排列](https://leetcode-cn.com/problems/permutations/)
2. [带重复元素的全排列](https://leetcode-cn.com/problems/permutations-ii/)
3. [下一个排列](https://leetcode-cn.com/problems/permutations-ii/)
4. [第k个排列](https://leetcode-cn.com/problems/permutation-sequence/)

#### 全排列

一般指不带重复元素的全排列。解决方法是DFS。涉及到搜索一般要考虑搜索的顺序，这里有两种实现方式：
1. 将第一个元素分别与后面的元素进行交换，然后分别做DFS（不保证字典序，但代码较短）；
2. 对每个位置从小到大进行枚举（保证字典序，代码较长）。

##### swap方式
```cpp
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;

    vector<vector<int>> permute(vector<int>& nums) {
        int n = nums.size();
        dfs(nums, 0, n);
        return ans;
    }

    void dfs(vector<int>& nums, int u, int n) {
        if(u == n) {
            ans.push_back(path);
            return;
        }

        for(int i = u; i < n; i++) {
            swap(nums[u], nums[i]);
            path.push_back(nums[u]);
            dfs(nums, u + 1, n);
            path.pop_back();
            swap(nums[u], nums[i]);
        }
    }
};
```

##### 按位枚举方式

```cpp
class Solution {
public:

    vector<vector<int>> ans;
    vector<int> path;
    vector<bool> st;

    vector<vector<int>> permute(vector<int>& nums) {
        path = vector<int>(nums.size());
        st = vector<bool>(nums.size());

        dfs(nums, 0);

        return ans;
    }

    void dfs(vector<int>& nums, int u) {
        if (u == nums.size()) {
            ans.push_back(path);
            return;
        }

        for (int i = 0; i < nums.size(); i ++ ) {
            if (st[i] == false) {
                path[u] = nums[i];
                st[i] = true;
                dfs(nums, u + 1);
                st[i] = false;
            }
        }
    }
};
```

#### 带重复元素的全排列

1的进阶版，需要对结果去重。解决思路是让重复的元素顺序填入，对于不按顺序的答案不作记录。

```cpp
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;
    vector<bool> st;

    vector<vector<int>> permuteUnique(vector<int>& nums) {
        int n = nums.size();
        sort(nums.begin(), nums.end());
        path = vector<int>(n);
        st = vector<bool>(n);
        dfs(nums, 0);
        return ans;
    }

    void dfs(vector<int>& nums, int u) {
        if(u == nums.size()) {
            ans.push_back(path);
            return;
        }

        for(int i = 0; i < nums.size(); i++) {
            if(!st[i]) {
                if(i && nums[i - 1] == nums[i] && !st[i - 1]) continue; // 前一个相同元素还没被用，所以当前元素不能用（我大哥还没出场，小弟我自然不能出场，不然让大哥多没牌面）
                st[i] = true;
                path[u] = nums[i];
                dfs(nums, u + 1);
                st[i] = false;
            }
        }
    }
};
```

#### 下一个排列

抓住一个排列的性质：末尾若干位一定按照降序排列。

排列从小到大的变化也是有规律的，后面的元素先变，前面的后变。下一个排列遵循是前面的尽量不改变，而改变后面的元素排列。根据性质，有以下算法：

1. 从后往前遍历，找到第一个不降序的元素，记为$k$；
2. 遍历降序序列，找到第一个比$k$大的数，记为$t$；
3. 交换$k$, $t$；
4. 把降序序列翻转，变成升序。

```cpp
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int k = nums.size() - 1;
        while(k > 0 && nums[k - 1] >= nums[k]) k--;
        if(k <= 0) {
            reverse(nums.begin(), nums.end());
        } else {
            int t = k;
            while(t < nums.size() && nums[t] > nums[k - 1]) t++;
            swap(nums[t - 1], nums[k - 1]);
            reverse(nums.begin() + k, nums.end());
        }
    }
};
```

#### 第k个排列

举个例子比较好理解，求`[1,2,3,4]`的第8个排列。

假如以1开头，那么一共有$3! = 6$种排列，$6 < 8$，所以开头不是1.

开头为2的话，也有6种排列，加上前面1的6种，一共12种，大于8，所以2是第一个元素。

接着枚举第二位，原理类似。

```cpp
class Solution {
public:
    string getPermutation(int n, int k) {
        string res;
        vector<bool> st = vector<bool>(n + 1);
        for(int i = 0; i < n; i++) {
            int fact = 1;
            for(int j = 1; j <= n - i - 1; j++) fact *= j;

            for(int j = 1; j <= n; j++) {
                if(!st[j]) {
                    if(fact < k) k -= fact;
                    else{
                        res += j + '0';
                        st[j] = true;
                        break;
                    }
                }
            }
        }
        return res;
    }
};
```
