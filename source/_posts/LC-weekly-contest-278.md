---
title: LC weekly contest 278
date: 2022-02-02 12:43:35
tags: [Leetcode, 模拟, 前缀和, 滑动窗口, 字符串哈希, 并查集]
categories: Leetcode周赛
katex: true
description:  LC 周赛第278场题解
---

![LC](/images/Leetcode.jpg)

<!--more-->


### **Saying Before**

春节快乐！虎年大吉！

###  **Solution**

#### [将找到的值乘以 2](https://leetcode-cn.com/problems/keep-multiplying-found-values-by-two/)

##### **Code Q1**
```cpp
class Solution {
public:
    int findFinalValue(vector<int>& nums, int original) {
        while(1) {
            bool flag = 0;
            for (int x : nums) {
                if (x == original) {
                    original *= 2;
                    flag = 1;
                    break;
                }
            }
            if (!flag) break;
        }
        return original;
    }
};
```

#### [分组得分最高的所有下标](https://leetcode-cn.com/problems/all-divisions-with-the-highest-score-of-a-binary-array/)

We can compute a prefix summation array and a suffix summation array in advanced in $O(n)$, then iterate each position to get the maximum result.

提前预处理出前后缀数组，这一步是$O(n)$的，然后再枚举每个位置来更新答案。

##### **Code Q2**
```cpp
class Solution {
public:
    vector<int> maxScoreIndices(vector<int>& nums) {
        int n = nums.size();
        vector<int> s(n + 10), s2(n + 10);
        for (int i = 1; i <= n; i++) {
            if (!nums[i - 1]) s2[i] = s2[i - 1] + 1;
            else s2[i] = s2[i - 1];
        }
        for (int i = n; i > 0; i--) {
            if (nums[i - 1]) s[i] = s[i + 1] + 1;
            else s[i] = s[i + 1];
        }
        vector<int> res;
        int ma = -1;

        for (int i = 0; i <= n; i++) {
            ma = max(ma, s2[i] + s[i + 1]);
        }

        for (int i = 0; i <= n; i++) {
            if (s2[i] + s[i + 1] == ma) 
                res.push_back(i);
        }
        return res;
    }
};
```


#### [查找给定哈希值的子串](https://leetcode-cn.com/problems/find-substring-with-given-hash-value/)

The background of this question is based on string hashing algorithm, which is widely used in string comparation. The most tricky part in this problem is how to update the hash value from previous value in $O(n)$. Since the operation is run on modulus, we cannot directly use division operation. Given such condition, we can slide the window from back forward, avoiding to use division but multiplication. 

这道题的背景是基于字符串哈希，它经常被用在字符串的比较问题中。本题最棘手的地方在于如何基于旧的哈希值来更新新的哈希值，并且需要再在$O(1)$的时间内完成。因为这是取模运算，我们不能直接使用除法。基于这样的限制，我们可以从后往前来滑动窗口，从而避免使用除法，而是使用乘法。

##### **Code Q3**
```cpp
class Solution {
public:
    using LL = long long;
    
    string subStrHash(string str, int power, int mod, int k, int hashValue) {
        int s = 0, p = 1;
        int n = str.size();
        int i = n - k;
        
        for (int j = i; j < n; j++) {
            s = (s + (str[j] - 'a' + 1) * (LL)p) % mod;
            if (j < n - 1) p = (LL)p * power % mod;
        }
        
        int ans = 0;
        
        while (i >= 0) {
            
            if (s == hashValue) ans = i;
            
            s = (s - ((LL)p * (str[i + k - 1] - 'a' + 1) % mod) + mod) % mod;
            s = s * (LL)power % mod;
            if (i) s = (s + str[i - 1] - 'a' + 1) % mod;
            
            i--;
        }
        
        return str.substr(ans, k);
    }
};
```

#### [字符串分组](https://leetcode-cn.com/problems/groups-of-strings/)

We can use a $26$-bits state to represent a string. The $k_{th}$ bit is set to $1$ if and only if the corresponding character exists in the string. 

我们可以使用一个$26$比特的状态来表示一个串。第$k$个比特为$1$当且仅当对应位置的字符被出现在字符串中。

For each word, we can further enumerate all possible states that such word can be converted to, via the $3$ operations -- add, delete and modify. Because the length of each state of word is up to $26$, and each character can be converted up to $26$ characters, the time of this procedure is $O(26^2)$. Therefore, the total time complexity will be $O(26^2n)$.

对于每个单词，我们可以进一步枚举它所能被转移的所有状态，通过$3$种基本操作 -- 新增，删除和修改。因为每个单词的状态的长度最长为$26$，并且每个字符最多可以转移到$26$种字符，所以这个枚举操作的时间复杂度是$O(26^2)$。所以总的时间复杂度是$O(26^2n)$。

To handle the set operation -- which should support fast query and fast merge, we can use union-find set. We can also maintain the maximum size and group number when merging the union-find set.
为了处理需要支持快速查找和合并的集合操作，我们可以使用并查集。最大集合大小以及总的集合数量也可以在合并的过程中维护出来。


##### **Code Q4**
```cpp
class Solution {
public:

    unordered_map<int, int> fa, size;
    int cnt, ma;

    int find(int x) {
        return fa[x] == x ? x : fa[x] = find(fa[x]);
    }

    void merge(int a, int b) {
        if (!fa.count(b)) return;
        int x = find(a), y = find(b);
        if (x == y) return;
        fa[x] = y;     // x --> y
        size[y] += size[x];
        ma = max(ma, size[y]);
        cnt --;

    }

    vector<int> groupStrings(vector<string>& words) {
        cnt = words.size();
        ma = 0;
        for (auto& word : words) {
            int x = 0;
            for (char& c : word) {
                x |= (1 << (c - 'a'));
            }
            fa[x] = x;
            size[x] ++;
            ma = max(ma, size[x]);
            if (size[x] > 1) cnt--;
        }

        for (auto& [x, _] : fa) {
            for (int i = 0; i < 26; i++) {
                merge(x, x ^ (1 << i));   // add or remove
                if ((x >> i) & 1) {  
                    for (int j = 0; j < 26; j++) {  // change to other character
                        if ((x >> j & 1) == 0) {
                            merge(x, x ^ (1 << i) | (1 << j));
                        }
                    }
                }
            }
        }

        return {cnt, ma};
    }
};
```