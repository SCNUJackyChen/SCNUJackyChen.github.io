---
title: LC weekly contest 292
date: 2022-05-08 15:46:21
tags: [Leetcode, DFS, DP, 记忆化搜索]
categories: Leetcode周赛
katex: true
description:  LC 周赛第292场题解
---

![LC](/images/Leetcode.jpg)

<!--more-->


### **Preface**

眨眼间Blog停更了3个月，这段时间主要在忙着春招找工作，现在春招也临近尾声，手头的面试也差不多告一段落了。这周刚好闲下来可以重回LC周赛（回归第一场就被锤爆），接下来计划更新我在国内春招和新加坡找工的笔试面试经验，以及每周会更新LC周赛题解。ヾ(•ω•`)o

### **Summary**

个人觉得这周的题质量还蛮高的，T2, T3, T4都比较贴近面试水平。

###  **Solution**

#### [字符串中最大的 3 位相同数字](https://leetcode.cn/problems/largest-3-same-digit-number-in-string/)

按照题意把连续相同的3位数字抠出来然后比较即可。

Based on the description, we can slice out all the 3-digit substring with the same number and compare it with the maximum.

##### **Code Q1**

```cpp
class Solution {
public:
    string largestGoodInteger(string s) {
        string res;
        for (int i = 0, j = 0; i < s.size(); i++) {
            if (!i || s[i] == s[i - 1]) j++;
            else j = 1;
            if (j == 3) {
                res = max(res, s.substr(i - 2, 3));
            }
        }
        return res;
    }
};
```

#### [统计值等于子树平均值的节点数](https://leetcode.cn/problems/count-nodes-equal-to-average-of-subtree/)

通过深度优先搜索可以维护出每个节点的子孙节点的总数与值之和，对于每个点判断是否满足条件即可。

For each node in the tree, we can maintain the number of its children and grand-children nodes, and the sum of their values. Then we can check whether they satisfy the requirement. We can use depth-first-search to implement this process.

##### **Code Q2**

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
    int res;
    int averageOfSubtree(TreeNode* root) {
        res = 0;
        dfs(root);
        return res;
    }
    
    vector<int> dfs(TreeNode* root) {
        if (!root) return {0, 0};
        vector<int> left = dfs(root->left);
        vector<int> right = dfs(root->right);
        if ( (left[0] + right[0] + root->val) / (left[1] + right[1] + 1) == root->val) res++;
        return {left[0] + right[0] + root->val, left[1] + right[1] + 1};
    }
    
};
```

#### [统计打字方案数](https://leetcode.cn/problems/count-number-of-texts/)

本题的关键在于挖掘出递推关系，本质上和爬楼梯问题相同，但不容易看出递推关系。

The key to solving this problem is finding the recursion relation. In essence, it is the same as the problem of climbing stairs, but a bit difficult to find the relation.


可以将输入字符串拆解成若干段，这些子串由连续相同的字母组成。由于按键不可能“跨键”组合，所以每一段是独立的，我们只需要算每一段的方案数，然后根据乘法原理累乘即可得出最终结果。

We can split the string into several segments, which are made of continuous same characters. Given that different characters can not be merged, each segment can be regarded as a independent sub-problem. What we need to do is to calculate the number of cases from each segment, then get the final result by multiplying them up.

不妨设$f(i)$为具有连续$i$个相同字母的字符串的方案数。对于非7，9数字来说，可以将末尾的1个字母固定，剩下的$i - 1$个字母构成$f(i - 1)$的子问题。同理，可以将末尾2个字母合成一个（比如2个a可以构成b），剩下的$i - 2$个字母构成$f(i - 2)$。3个字母类似不再赘述。因此，$f(i)$可以由以下式子递推得到：

$$f(i) = f(i-1)+f(i-2)+f(i-3)$$

对于7和9，由于它们可以将4个相同字母合成一个，所以递推式应该增加一项：

$$g(i) = g(i-1)+g(i-2)+g(i-3)+g(i-4)$$

初始化时，$f(1)=1,f(2)=2,f(3)=4$; 特别地，我们定义$g(0)=1$，这样可以正确推出$g(4)$。

Let $f(i)$ be the number of case(s) of a segment with continuous same $i$ characters. For those number except for 7 and 9, we can fix the last character, the remaining characters form a sub-problem $f(i - 1)$. Similarly, we can fix the last two characters and merge them, the remaining characters form a sub-problem $f(i - 2)$. The same for 3 characters. Therefore, $f(i)$ can be derived through the following relation:

$$f(i) = f(i-1)+f(i-2)+f(i-3)$$

For 7 and 9, since they can merge 4 characters into 1, an extra item should be added to the recursion formula:

$$g(i) = g(i-1)+g(i-2)+g(i-3)+g(i-4)$$

For the initial step, $f(1)=1,f(2)=2,f(3)=4$; specially, we let $g(0)=1$ in order to correctly derive $g(4)$.

##### **Code Q3**

```cpp
class Solution {
public:
    const int MOD = 1e9 + 7, N = 1e5;
    using LL = long long;

    int countTexts(string s) {
        int n = s.size();
        vector<LL> f(N + 1), g(N + 1);
        g[0] = 1;
        f[1] = g[1] = 1;
        f[2] = g[2] = 2;
        f[3] = g[3] = 4;

        for (int i = 4; i <= N; i++) {
            f[i] = (f[i - 1] + f[i - 2] + f[i - 3]) % MOD;
            g[i] = (g[i - 1] + g[i - 2] + g[i - 3] + g[i - 4]) % MOD;
        }
        LL res = 1;
        for (int i = 0; i < n; i++) {
            int j = i + 1;
            while (j < n && s[j - 1] == s[j]) j++;
            if (s[i] == '7' || s[i] == '9') res = (res * g[j - i]) % MOD;
            else res = (res * f[j - i]) % MOD; 
            i = j - 1;           
        }
        return res;
    }
};
```

#### [检查是否有合法括号字符串路径](https://leetcode.cn/problems/check-if-there-is-a-valid-parentheses-string-path/)

本题的一维版本十分简单，直接遍历括号序列并判断合法性即可。

二维版本由于每个位置可以有2种选择，注意到$n$的值还是比较大的，直接暴搜会超时。考虑在暴搜的同时加入必要的剪枝操作：

1. 如果左括号数量超过了$\frac{(n+m)}{2}$并且已经走过$\frac{(n+m)}{2}$的路程，那么剩下的即使全是右括号也无法把左括号匹配掉；
2. 记忆化搜索。定义状态$f(x,y,c)$表示从$(0,0)$到$(x,y)$有$c$个左括号的所有路径的集合。因为我们最终只需判断是否可达，所以只要有一条路径能达到状态$f(x,y,c)$即可。剩下的路径不需要搜索。

The 1-D version of this question is simple, we can iterate through the parenthese sequence and check the validity.

For 2-D version, since each position we have 2 choices, it is likely to TLE due to $n=100$, which is big. Therefore, we can consider to add pruning in the searching tree:

1. If we have moved more than $\frac{(n+m)}{2}$ steps with more than $\frac{(n+m)}{2}$ left parentheses in hand, we can just stop searching because even though the remaining parentheses are all right parentheses, we still cannot match all of the left parentheses.
2. Memorized searching. Define $f(x,y,c)$ as a set of paths which are from $(0,0)$ to $(x,y)$ with $c$ left parentheses. Given that our goal is to check whether we can reach the target legally, once the state $f(x,y,c)$ is reach, we don't need further duplicate search for this state, thus reducing the time complexity.


##### **Code Q4**

```cpp
class Solution {
public:
    vector<vector<char>> g;
    int n, m;
    vector<vector<vector<bool>>> vis; 

    bool hasValidPath(vector<vector<char>>& grid) {
        g = grid;
        n = g.size(), m = g[0].size();
        vis = vector<vector<vector<bool>>>(n, vector<vector<bool>>(m, vector<bool>(m + n)));
        return dfs(0, 0, 0);    
    }

    bool dfs(int x, int y, int u) {
        if (x >= n || y >= m) return false;
        if (u > m - x + n - y - 1) return false;
        if (vis[x][y][u]) return false;
        vis[x][y][u] = true;
        if (g[x][y] == '(') u++;
        else u--;
        

        if (x == g.size() - 1 && y == g[0].size() - 1) {
            if (!u) return true;
            return false;
        }
        return u >= 0 && (dfs(x + 1, y, u) || dfs(x, y + 1, u));
    }


};
```