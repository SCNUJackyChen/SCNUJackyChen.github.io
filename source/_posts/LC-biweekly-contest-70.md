---
title: LC biweekly contest 70
date: 2022-01-23 16:35:32
tags: [Leetcode, 贪心, 模拟, BFS]
categories: Leetcode周赛
katex: true
description:  新年快乐！ヾ(•ω•`)o  LC双周赛第70场复盘+题解
---

![LC](/images/new-year-wishes.jpg)

<!--more-->

### **Saying Before**

Happy New Year! I haven't updated my blogs for one month! The excuse would be... I am busy taking the courses and preparing for interviews. This is my first blog in 2022, but I don't wanna to write a 2021 summary (at least in this blog🤣, maybe before the Chinese New Year I will do that). A good news is that I passed all the interviews and got the offer, a good start for my 2022. For details about the job hunting experience, I will write a summary in later updates.

In 2022, people still need to live with threat from the COVID-19. In this special period I hope everyone can stay safe and keep healthy. 🧡

For 2022, I will keep updating my solutions for Leetcode weekly contest and my thinking for some ideas in papers or some techniques (but these kind of blogs may be updated in a low frequency since it really takes me time to write down, but I will try my best). In addition, 2022 will be my job-hunting year and I may take lots of interviews in the coming future, so I will share my experience for my interview to record my growth.

So, a new start, let's keep going!

### **Solution**

#### [打折购买糖果的最小开销](https://leetcode-cn.com/problems/minimum-cost-of-buying-candies-with-discount/)

Since only candy with lower price can be rewarded, a validate strategy is to pick candies in descending order of price. How to prove the correctness of this greedy strategy? We can assume that we do not pick the two candies with highest and second highest price, in this case, we will be rewarded with a candy (noted as $C$), and $Price_C < Price_A$, where $A$ is the candy with third highest price. However, since the rule that "we can only be rewarded with a candy with lower price", the two candies with highest and second highest price will finally be bought. Therefore, we can prove the correctness of this greedy algorithm.

##### **Code Q1**

```cpp
class Solution {
public:
    int minimumCost(vector<int>& cost) {
        sort(cost.begin(), cost.end());
        reverse(cost.begin(), cost.end());
        int sum = 0;
        
        for (int i = 0, k = 0; i < cost.size(); i++) {
            if (k == 2) k = 0;
            else {
                sum += cost[i];
                k++;
            }
        }
        return sum;
    }
};
```

#### [统计隐藏数组数目](https://leetcode-cn.com/problems/count-the-hidden-sequences/)

In order to check whether all the numbers in original array can fall within the interval, we can calculate the range of this array. If the range is smaller that given range of the interval, then it works. 

##### **Code Q2**

```cpp
class Solution {
public:
    int numberOfArrays(vector<int>& d, int lower, int upper) {
        long long c = 0;
        long long mi = 0, ma = 0;
        for (int i = 0; i < d.size(); i++) {
            c += d[i];
            mi = min(mi, c);
            ma = max(ma, c);
        }
        
        if (ma - mi > upper - lower) return 0;
        
        return upper - lower - (ma - mi) + 1;
    }
};
```

#### [价格范围内最高排名的 K 样物品](https://leetcode-cn.com/problems/k-highest-ranked-items-within-a-price-range/)

This is a good question and I think it is perfect for an interview.

Actually the main idea is BFS, but it adds more constraints and more tricks, like customizing sorting rules, recording levels of BFS etc.

##### **Code Q3**

```cpp
class Solution {
public:
    
    struct node {
        int l, v, x, y;
        node(int k, int a, int b, int c): l(k), v(a), x(b), y(c){}
        bool operator < (const node& e) const {
            if (l != e.l) return l < e.l;
            if (v != e.v) return v < e.v;
            if (x != e.x) return x < e.x;
            return y < e.y;
        }
    };
    
    vector<vector<int>> highestRankedKItems(vector<vector<int>>& grid, vector<int>& pricing, vector<int>& start, int k) {
        int n = grid.size(), m = grid[0].size();
        
        queue<pair<int, int>> q;
        q.push({start[0], start[1]});
        
        vector<vector<int>> st(n, vector<int>(m, 0));
        vector<node> res;
        
        int dx[4] = {-1, 0, 1, 0}, dy[4] = {0, 1, 0, -1};
        st[start[0]][start[1]] = 1;
        int level = 0;
        
        while (q.size()) {
            int s = q.size();
            level ++;
            if ((int)res.size() - 1 >= k) break;
            while (s--) {
                
                auto top = q.front(); q.pop(); 
                int x = top.first, y = top.second;
                if (grid[x][y] >= pricing[0] && grid[x][y] <= pricing[1])
                    res.push_back(node(level, grid[x][y], x, y));

                vector<node> near;
                for (int i = 0; i < 4; i++) {
                    int a = x + dx[i], b = y + dy[i];
                    if (a >= 0 && a < n && b >= 0 && b < m && grid[a][b] != 0 && !st[a][b]) {
                        
                        st[a][b] = 1;
                        q.push({a, b});
                    }
                }
                
            }
        }
        
        sort(res.begin(), res.end());

        vector<vector<int>> tt;
        for (auto p : res) tt.push_back({p.x, p.y});
        if (tt.size() <= k) return tt;
        vector<vector<int>> t;
        for (int i = 0 ; i < k; i++) t.push_back(tt[i]);
        return t;
    }
};

```

#### [分隔长廊的方案数](https://leetcode-cn.com/problems/number-of-ways-to-divide-a-long-corridor/)

T4 is even easier than T3😅...

We can iterate the whole string and look at each "S", each "P" between two "S" can be placed a corrider. We can count the result by multiplication principle.


##### **Code Q4**

```cpp
class Solution {
public:
    int numberOfWays(string s) {
        int res = 1, cnt = 0, pre = 0;
        int MOD = 1e9 + 7;
        for (int i = 0; i < s.size(); i++) {
            if (s[i] == 'S') {
                cnt ++;
                if (cnt >= 3 && cnt % 2) {
                    res = (long long)res * (i - pre) % MOD;
                }
                pre = i;
            }
        }
        if (!cnt || cnt % 2) return 0;
        return res;
    }
};
```