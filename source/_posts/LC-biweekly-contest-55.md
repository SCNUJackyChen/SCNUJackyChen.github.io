---
title: LC 双周赛第55场
date: 2021-07-01 11:04:56
tags: [Leetcode, 模拟, 状态机DP, 数据结构]
categories: Leetcode周赛
katex: true
description:  LC双周赛第55场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **题解**

#### [删除一个元素使数组严格递增](https://leetcode-cn.com/problems/remove-one-element-to-make-the-array-strictly-increasing/)

暴力模拟即可，枚举删除每个数，然后判断删除该数之后数组是否严格递增。

##### **Code Q1**
```cpp
class Solution {
public:
    bool canBeIncreasing(vector<int>& nums) {
        int n = nums.size();
        for(int i = 0; i < n; i++) {
            bool flag = true;
            int last = 0;
            for(int j = 0; j < n; j++) {
                if (j == i) continue;
                if (nums[j] <= last) {
                    flag = false;
                    break;
                }
                last = nums[j];
            }
            if (flag) return true;
        }
        return false;
    }
};
```

#### [删除一个字符串中所有出现的给定子字符串](https://leetcode-cn.com/problems/remove-all-occurrences-of-a-substring/)

##### **Code Q2**
```cpp
class Solution {
public:
    string removeOccurrences(string s, string part) {
        while (1) {
            int k = s.find(part);
            if (k == -1) break;
            s.erase(k, part.size());
        }
        return s;
    }
};
```

#### [最大子序列交替和](https://leetcode-cn.com/problems/maximum-alternating-subsequence-sum/)

分奇偶两种情形讨论。
1. 如果当前是第奇数个数，那么前面可以不选或者选偶数个数然后加上当前数；
2. 如果当前是第偶数个数，那么前面可以选奇数个数然后减掉当前数。


##### **Code Q3**
```cpp
typedef long long LL;
const int N = 100010;
LL f[N][2];

class Solution {
public:
    long long maxAlternatingSum(vector<int>& nums) {
        int n = nums.size();
        LL INF = 1e15;
        memset(f, -INF, sizeof f);
        f[0][0] = 0ll;
        
        for(int i = 1; i <= n; i++) {
            f[i][0] = max(f[i - 1][0], f[i - 1][1] + nums[i - 1]);
            f[i][1] = max(f[i - 1][1], f[i - 1][0] - nums[i - 1]);
        }
        return max(f[n][0], f[n][1]);
    }
};
```

DP考试的时候没有想到，吭哧吭哧写了个贪心解法也过了。可以观察到最优解一定为奇数，并且一定是从遵循：最高点-->最低点-->最高点--> ... -->最高点/最低点 往复。

###### **Code Q3 贪心解法**
```cpp
class Solution {
public:
    
    long long maxAlternatingSum(vector<int>& nums) {
        int n = nums.size();
        if (nums.size() == 1) return nums[0];
        int s = 1;
         
        long long res1 = 0;
        if (nums[0] > nums[1]) res1 += nums[0];
        for(int i = 0; i < n; i++) {
            int j = i + 1; 
            int last = i;
            if (s == 1) {
                while (j < n && nums[j] <= nums[last]) last++, j++;
                s = -1;
            } else {
                while (j < n && nums[j] >= nums[last]) last++, j++;
                s = 1;
            }
            if (s == -1 && j == n) break;
            if (last != i) res1 += s * nums[last];
            i = last - 1;
        }
        long long res2 = 0;
        s = -1;
        if (nums[0] > nums[1]) res2 += nums[0];
        for(int i = 0; i < n; i++) {
            int j = i + 1; 
            int last = i;
            if (s == 1) {
                while (j < n && nums[j] <= nums[last]) last++, j++;
                s = -1;
            } else {
                while (j < n && nums[j] >= nums[last]) last++, j++;
                s = 1;
            }
            if (s == -1 && j == n) break;
            if (last != i) res2 += s * nums[last];
            i = last - 1;
        }
        return max(res1, res2);
    }
};
```

#### [设计电影租借系统](https://leetcode-cn.com/problems/design-movie-rental-system/)

一道数据结构题。可以用红黑树实现的set作为容器来维护所有的(shop, movie, price)对。

##### **Code Q4**
```cpp
const int N = 10010;

struct Node {
    int shop, movie, price;
    bool operator< (const Node& t) const {
        if (t.price != price) return price < t.price;
        if (t.shop != shop) return shop < t.shop;
        return movie < t.movie;
    }
};

set<Node> mv[N];
set<Node> rented;
map<pair<int, int>, int> pr;

class MovieRentingSystem {
public:
    MovieRentingSystem(int n, vector<vector<int>>& entries) {
        for(int i = 0; i < N; i ++) mv[i].clear();
        rented.clear();
        pr.clear();

        for(auto& e : entries) {
            int shop = e[0], movie = e[1], price = e[2];
            mv[movie].insert({shop, movie, price});
            pr.insert({{shop, movie}, price});
        }
    }
    
    vector<int> search(int movie) {
        vector<int> res;
        for(auto& e : mv[movie]) {
            res.push_back(e.shop);
            if (res.size() >= 5) break;
        }
        return res;
    }
    
    void rent(int shop, int movie) {
        auto it = pr.find({shop, movie});
        rented.insert({shop, movie, it->second});
        mv[movie].erase({shop, movie, it->second});
    }
    
    void drop(int shop, int movie) {
        auto it = rented.find({shop, movie, pr.find({shop, movie})->second});
        mv[movie].insert(*it);
        rented.erase(it);
    }
    
    vector<vector<int>> report() {
        vector<vector<int>> res;
        for(auto& e : rented) {
            res.push_back({e.shop, e.movie});
            if (res.size() >= 5) break;
        }
        return res;
    }
};

/**
 * Your MovieRentingSystem object will be instantiated and called as such:
 * MovieRentingSystem* obj = new MovieRentingSystem(n, entries);
 * vector<int> param_1 = obj->search(movie);
 * obj->rent(shop,movie);
 * obj->drop(shop,movie);
 * vector<vector<int>> param_4 = obj->report();
 */
```