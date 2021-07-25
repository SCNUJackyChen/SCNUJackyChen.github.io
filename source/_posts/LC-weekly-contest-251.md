---
title: LC 周赛第251场
date: 2021-07-25 13:01:58
tags: [Leetcode, 模拟, 贪心, DFS, Trie, 哈希, 序列化]
categories: Leetcode周赛
katex: true
description:  LC周赛第251场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

难得的手速场（T4除外）


### **题解**

##### [字符串转化后的各位数字之和](https://leetcode-cn.com/problems/sum-of-digits-of-string-after-convert/)
模拟即可。但是稍微麻烦了些，写得比较慢。

##### **Code Q1**
```cpp
class Solution {
public:
    int getLucky(string s, int k) {
        string str;
        for (int i = 0; i < s.size(); i++) {
            str += to_string(s[i] - 'a' + 1);
        }
        int sum = 0;
        while(k--) {
            sum = 0;
            for (auto c : str) {
                sum += c - '0';
            }
            str = to_string(sum);
        }
        return sum;
    }
};
```

##### [子字符串突变后可能得到的最大整数](https://leetcode-cn.com/problems/largest-number-after-mutating-substring/)
要使得突变后的串尽可能大，直观的想法就是让突变位置尽可能靠前（高位）。贪心性质也十分明显——高位的突变得到的结果一定比地位要优。所以从前往后遍历找到第一个可突变（带来收益）的子串，再进行突变即可。


##### **Code Q2**
```cpp
class Solution {
public:
    string maximumNumber(string num, vector<int>& change) {
        int n = num.size();
        for (int i = 0; i < n; i++) {
            if (change[num[i] - '0'] > num[i] - '0') {
                int j = i;
                while (j < n && change[num[j] - '0'] >= num[j] - '0') {
                    num[j] = change[num[j] - '0'] + '0';    
                    j++;
                }
                return num;          
            }                
        }
        return num;
    }
};
```
##### [最大兼容性评分和](https://leetcode-cn.com/problems/maximum-compatibility-score-sum/)
数据规模比较小，直接DFS都能过。。。


##### **Code Q3**

```cpp
class Solution {
public:
    vector<vector<int>> stu, men;
    vector<bool> vis;
    int res;
    
    void dfs(int u, int sum) {
        if (u >= stu.size()) {
            res = max(res, sum);
            return;
        }
        for (int i = 0; i < men.size(); i++) {
            if (vis[i]) continue;
            int t = 0;
            for (int j = 0; j < men[0].size(); j++) {
                t += stu[u][j] == men[i][j] ? 1 : 0;
            }
            vis[i] = true;
            dfs(u + 1, sum + t);
            vis[i] = false;
            
        }
    }
    
    int maxCompatibilitySum(vector<vector<int>>& students, vector<vector<int>>& mentors) {
        stu = students;
        men = mentors;
        vis = vector<bool>(stu.size(), false);
        res = 0;
        dfs(0, 0);
        return res;
    }
};
```
##### [删除系统中的重复文件夹](https://leetcode-cn.com/problems/delete-duplicate-folders-in-system/)
(to be continue...)
##### **Code Q4**