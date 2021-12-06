---
title: LC weekly contest 270
date: 2021-12-06 10:47:12
tags: [Leetcode, 链表,  DFS, 欧拉路径]
categories: Leetcode周赛
katex: true
description:  LC 周赛第270场题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

###  **Solution**

#### [找出 3 位偶数](https://leetcode-cn.com/problems/finding-3-digit-even-numbers/)

Since the length of array `digits` is small, we can simply brute-force all the combinations of three digits.

因为`digit`数组的长度比较小，我们可以简单地暴力枚举所有数字位的组合。

##### **Code Q1**
```cpp
class Solution {
public:
    vector<int> findEvenNumbers(vector<int>& digits) {
        int n = digits.size();
        set<int> S;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                for (int k = 0; k < n; k++) {
                    if (i == j || i == k || j == k || digits[k] % 2 || !digits[i]) continue;
                    S.insert(digits[i]*100 + digits[j]*10 + digits[k]);
                }
            }
        }
        vector<int> res;
        for (int x: S) res.push_back(x);
        return res;
    }
};
```


#### [删除链表的中间节点](https://leetcode-cn.com/problems/delete-the-middle-node-of-a-linked-list/)

What a classic interview code question 🤣

经典面试题...🤣

##### **Code Q2**
```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode() : val(0), next(nullptr) {}
 *     ListNode(int x) : val(x), next(nullptr) {}
 *     ListNode(int x, ListNode *next) : val(x), next(next) {}
 * };
 */
class Solution {
public:
    ListNode* deleteMiddle(ListNode* head) {
        int n = 0;
        for (auto p = head; p; p = p->next) n++;
        if (n == 1) return nullptr; // [1] -> []
        n /= 2;
        auto p = head;
        auto q = p;
        for (int i = 0; i < n; i++) {
            q = p;
            p = p->next;
        }
        q->next = p->next;
        return head;
    }
};
```

#### [从二叉树一个节点到另一个节点每一步的方向](https://leetcode-cn.com/problems/step-by-step-directions-from-a-binary-tree-node-to-another/)

Given two numbers, we can find their positions in the tree by depth first searching. At the same time, we can get the paths from root to these two nodes. With these two paths, we can subtract the common prefix from them, and for the rest, we can  invert the order of one of the path (i.e 'LR' -> 'U'), so the result would be like "UUU...ULRLRLR".

给定2个数，我们可以找到它们在树中的位置。同时，我们可以得到从根节点到这两个点的路径。基于这两条路径，我们可以删掉它们的共同前缀，对于剩下的字符串，我们可以翻转其中一条路径的顺序（即“LR”->U)，因此结果会长这样子 "UUU...ULRLRLR"。

##### **Code Q3**
```cpp
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
class Solution {
public:
    
    string a, b;
    
    void get(TreeNode* root, int x, int y, string& t) {
        if (root == nullptr) return;
        if (root->val == x) {
            a = t;
        }
        if (root->val == y) {
            b = t;
        }
        t += 'L';
        get(root->left, x, y, t);
        t.pop_back();
        t += 'R';
        get(root->right, x, y, t);
        t.pop_back();
        return;
    }
    
    string getDirections(TreeNode* root, int start, int dest) {
        string t = "";
        get(root, start, dest, t);
    
        
        int i = 0, j = 0;
        string s1 = a, s2 = b;
        while (s1[i] == s2[j]) {
            s1[i] = 'U';
            i++; j++;
        }
        for (char& c : s1) c = 'U';
        s1 = s1.substr(i);
        s2 = s2.substr(j);
        
        
        return s1 + s2;
    }
};
```


#### [合法重新排列数对](https://leetcode-cn.com/problems/valid-arrangement-of-pairs/)

Each pair can be regarded as an edge between two nodes. The question would be converted into find an Euler path in the graph. 

每个对都可以看作是一条边与两个点。那么问题就变成了求图上的欧拉路径。

> Extension
> [LC332. 重新安排行程](https://leetcode-cn.com/problems/reconstruct-itinerary/) 
> [LC753. 破解保险箱](https://leetcode-cn.com/problems/cracking-the-safe/)
> 本题和332的对比：如果返回的是路径，那么是在枚举每一条临边之后立马加入答案；如果返回的是点，那么是在枚举完所有临边之后加入答案（对比res.push_back()的位置)


##### **Code Q4**
```cpp
class Solution {
public:
    unordered_map<int, vector<int>> mp;
    unordered_map<int, int> deg;
    vector<vector<int>> ans;
    
    void dfs(int x) {
        int ver = -1;
        while (mp[x].size()) {
            int ver = mp[x].back();
            mp[x].pop_back();
            dfs(ver);
            
        }
        if (ver >)
        ans.push_back({x, ver});
            
    }
    
    
    vector<vector<int>> validArrangement(vector<vector<int>>& pairs) {
        for (auto p : pairs) {
            mp[p[0]].push_back(p[1]);
            deg[p[0]]--, deg[p[1]]++;
        }
        for (auto [c, s] : deg) {
            if (s == -1) dfs(c);
        }
        if (ans.empty()) dfs(pairs[0][0]);
        reverse(ans.begin(), ans.end());
        return ans;
    }
};
```