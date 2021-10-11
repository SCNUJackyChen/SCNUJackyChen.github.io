---
title: LC 双周赛第60场
date: 2021-09-06 23:34:28
tags: [Leetcode, 前缀和, DFS, 最大独立集]
categories: Leetcode周赛
katex: true
description:  LC双周赛第60场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

###  **题解**

#### [找到数组的中间位置](https://leetcode-cn.com/problems/find-the-middle-index-in-array/)

可以看成两段区间和的比较问题。涉及到区间和，可以考虑利用前缀和来计算。

##### **Code Q1**
```cpp
class Solution {
public:
    int findMiddleIndex(vector<int>& nums) {
        int n = nums.size();
        int s = 0;
        for (auto x : nums) s += x;
        int sum1 = 0, sum2 = s - nums[0];
        for (int i = 1; i < n; i++) {
            if (sum1 == sum2) return i - 1;
            sum1 += nums[i - 1];
            sum2 -= nums[i];
        }
        return sum1 == sum2 ? n - 1 : -1;
    }
};
```

#### [找到所有的农场组](https://leetcode-cn.com/problems/find-all-groups-of-farmland/)
##### **Code Q2**

一眼看上去像是一个深搜题。细看发现其实根本不需要，因为每个农场都是不相邻的，那么意味着，从左往右遍历到的第一个点必然是整个农场的左上角，此时只需要向右下方找到对角位置的坐标即可。

```cpp
class Solution {
public:
    vector<vector<int>> findFarmland(vector<vector<int>>& land) {
        int n = land.size(), m = land[0].size();
        vector<vector<bool>> st = vector<vector<bool>> (n + 1, vector<bool>(m + 1, false));
       
        vector<vector<int>> res;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (land[i][j] == 1 && !st[i][j]) {
                    int k = i, q = j;
                    while(k < n && land[k][j] == 1) k++;
                    k--;
                    while(q < m && land[k][q] == 1) q++;
                    q--;
                    res.push_back({i, j, k, q});
                    for (int a = i; a <= k; a++)
                        for (int b = j; b <= q; b++)
                            st[a][b] = true;
                }
            }
        }
        return res;
    }
};
```

#### [树上的操作](https://leetcode-cn.com/problems/operations-on-tree/)

考察了树的深度遍历。

##### **Code Q3**
```cpp
#define null nullptr

struct Node {
    Node* parent;
    vector<Node*> children;
    int data;
    Node(Node* p = null, int d = 0) {parent = p, data = d;} 
};

class LockingTree {
public:
    Node* root;
    unordered_map<int, Node*> hash;
    unordered_map<int, int> is_lock;
    
    LockingTree(vector<int>& parent) {
        int n = parent.size();
        for (int i = 0; i < n; i++) is_lock[i] = -1;
        root = new Node(null, 0);
        hash[0] = root;
        for (int i = 1; i < parent.size(); i++) {
            if (!hash.count(i)) hash[i] = new Node(null, i);
            Node* t = hash[i];
            if (!hash.count(parent[i])) hash[parent[i]] = new Node(null, parent[i]);
            t->parent = hash[parent[i]];
            hash[parent[i]]->children.push_back(t);
        }

    }
    
    void visit(Node* u) {
        cout << u->data << endl;
        for (auto& c : u->children) 
            visit(c);
    }
    
    bool lock(int num, int user) {
        if (is_lock[num] == -1) {
            is_lock[num] = user;
            return true;
        }
        return false;
        
    }
    
    bool unlock(int num, int user) {
        if (is_lock[num] == user) {
            is_lock[num] = -1;
            return true;
        }
        return false;
    }
    
    int dfs(Node* u) {
        if (u == null) return 0;
        int res = 0;

        for (auto& c : u->children) {
            if (is_lock[c->data] != -1) {
                res++;
                is_lock[c->data] = -1;   
            }
            res += dfs(c);
        }
        return res;
    }
    
    bool upgrade(int num, int user) {
        if (is_lock[num] != -1) return false;
        Node* rt = hash[num];
        Node* t = rt;
        while(t->parent) {
            if (is_lock[t->parent->data] != -1) return false;
            t = t->parent;
        }
        if (dfs(rt) == 0) return false;
        is_lock[num] = user;
        return true;
        
    }
};

/**
 * Your LockingTree object will be instantiated and called as such:
 * LockingTree* obj = new LockingTree(parent);
 * bool param_1 = obj->lock(num,user);
 * bool param_2 = obj->unlock(num,user);
 * bool param_3 = obj->upgrade(num,user);
 */

```

#### [好子集的数目](https://leetcode-cn.com/problems/the-number-of-good-subsets/)

本题等价于最大独立集问题，而该问题是NP hard的，因此最好的算法也是暴搜，这也是为啥用的是DFS来做。

为什么等价呢？不妨把每个数看成一个结点，如果两个数不互质，那么它们之间连一条边。我们要在所有的数和边构成的图中，求解一个最大独立集问题（即有多少个两两互不相邻的集合）。

那么如何利用暴搜来解决此题呢？首先注意到一些数是肯定不可能出现在答案中的，当一个数包含2个相同因子时，它不能被选，因为题目要求需要**互不相同**的因子。筛掉一部分数字之后，在剩下的数字中暴搜。

分别枚举每个数选或者不选，分别做DFS。如果选，要先保证这个数和之前选的数都互质。总体方案数应该乘以这个数出现的次数（乘法原理）。如果不选，直接进入下一层递归即可。

单独讨论一下1的情况，1乘以任何数都是原数。因此可以加任意多的1到一个集合中。如果有$k$个1，那么可以有$2^k$种不同的选法。

##### **Code Q4**

```cpp
using LL = long long;
const int MOD = 1e9 + 7;

class Solution {
public:
    int s[31] = {0};
    bool st[31] = {0};
    int g[31][31] = {0};
    int C = 1;
    vector<int> path;
    
    
    int gcd(int a, int b) {
        return b ? gcd(b, a % b) : a;
    }
    
    int dfs(int u, int sum) {
        if (!sum) return 0;
        if (u > 30) {
            if (path.empty()) return 0;
            return sum * (LL)C % MOD;
        }
        
        int res = dfs(u + 1, sum);
        if (!st[u]) {
            bool flag = true;
            for (int x : path) 
                if (g[x][u]) {
                    flag = false;
                    break;
                }
            if (flag) {
                path.push_back(u);
                res = (res + dfs(u + 1, sum * (LL)s[u] % MOD)) % MOD;
                path.pop_back();
            }
        }
        return res;
    }
    
    
    int numberOfGoodSubsets(vector<int>& nums) {
        for (auto x: nums) s[x] ++ ;
        for (int i = 0; i < s[1]; i ++ ) C = C * 2 % MOD;

        for (int i = 2; i * i <= 30; i ++ )
            for (int j = 1; j * i * i <= 30; j ++ )
                st[j * i * i] = 1;

        for (int i = 1; i <= 30; i ++ )
            for (int j = 1; j <= 30; j ++ )
                if (gcd(i, j) > 1)
                    g[i][j] = 1;

        return dfs(2, 1);

    }
};
```