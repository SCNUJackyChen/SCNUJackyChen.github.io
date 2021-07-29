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
本题实际上考察的是字典树 + 序列化。
1. 首先确定数据结构，文件系统实际上是多叉树结构。
2. 如何判断子树是否相同? 如果采用暴力递归的方式，复杂度是不可接受的。这里采用将子树先预处理成字符串的形式（即树的序列化过程），这样可以将一棵子树映射成一个字符串。只需要比较2个字符串是否一致即可判断子树结构是否一致。
3. 如何删除冗余子树？在序列化时顺便记录每个子树序列出现的次数，在第二遍深搜时，如果发现当前结点所对应的序列出现次数大于1，则不加入答案。

##### **Code Q4**
```cpp
struct Trie {
    string seq;
    unordered_map<string, Trie*> children;
};

class Solution {
public:
    unordered_map<string, int> cnt;
    vector<vector<string>> res;

    void build(Trie* root) {
        if (root->children.empty()) return;
        vector<string> s;
        for (auto& [k, v] : root->children) {
            build(v);
            s.push_back(k + "(" + v->seq + ")");
        }
        sort(s.begin(), s.end()); // 这里sort是因为可能出现镜像结构
        for (auto& e : s) {
            root->seq += e;
        }
        cnt[root->seq] ++;
    }

    void prune(Trie* root, vector<string>& t) {
        if (cnt[root->seq] > 1) return;
        if (!t.empty()) res.push_back(t);
        for (auto& [k, v] : root->children) {
            t.push_back(k);
            prune(v, t);
            t.pop_back();
        }
    }

    vector<vector<string>> deleteDuplicateFolder(vector<vector<string>>& paths) {
    	// 建树
        auto root = new Trie();
        for (auto& p : paths) {
            auto t = root;
            for (auto& c : p) {
                if (!t->children.count(c)) 
                    t->children[c] = new Trie();
                t = t->children[c];
            }
        }
		// 第一遍序列化
        build(root);
		
		// 第二遍删除子树
        vector<string> t;
        prune(root, t);

        return res;
    }
};
```