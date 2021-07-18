---
title: LC 双周赛第56场
date: 2021-07-18 00:14:57
tags: [Leetcode, 模拟, BFS, 博弈论, DP, 最短路]
categories: Leetcode周赛
katex: true
description:  LC双周赛第56场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **题解**

#### [统计平方和三元组的数目](https://leetcode-cn.com/problems/count-square-sum-triples/)
暴力枚举即可。

##### **Code Q1**
```cpp
class Solution {
public:
    int countTriples(int n) {
        int res = 0;
        for(int a = 1; a <= n; a++ ) {
            for(int b = 1; b <= n; b++) {
                for(int c = 1; c <= n; c++) {
                    if (a * a  + b * b == c * c) 
                        res ++;
                }
            }
        }
        return res;
    }
};
```

#### [迷宫中离入口最近的出口](https://leetcode-cn.com/problems/nearest-exit-from-entrance-in-maze/)
一道BFS模板题。考试的时候忘记判重卡了好久...

##### **Code Q2**
```cpp
class Solution {
public:
    int nearestExit(vector<vector<char>>& maze, vector<int>& e) {
        typedef pair<int, int> PII;
        int n = maze.size(), m = maze[0].size();
        vector<vector<int>> dist(n, vector<int>(m, 1e8));
        vector<vector<bool>> st(n, vector<bool>(m, false));
        queue<PII> q;      
        int dx[4] = {-1, 0, 1, 0}, dy[4] = {0, 1, 0, -1};
        q.push({e[0], e[1]});  
        dist[e[0]][e[1]] = 0;

        while (q.size()) {
            auto t = q.front();
            q.pop();
            
            int x = t.first, y = t.second;
            st[x][y] = true;
            for (int i = 0 ; i < 4; i++) {
                int a = x + dx[i], b = y + dy[i];
                if (a < n && a >= 0 && b < m && b >= 0 && maze[a][b] == '.' && dist[a][b] > dist[x][y] + 1) {
                    dist[a][b] = dist[x][y] + 1;
                    if (a == 0 || a == n - 1 || b == 0 || b == m - 1) return dist[a][b];
                    if (!st[a][b]) {
                        q.push({a, b});
                    }
                }
            }

        }
        return -1;
    }
};
```
#### [求和游戏](https://leetcode-cn.com/problems/sum-game/)
分类讨论左右半边的和与问号数量的关系。
1. 如果左边和 > 右边 并且左边的问号比右边多。Alice只要采取和Bob对称取值的策略可以获胜，反之同理；
2. 如果左边和 > 右边但是左边的问号比右边少。站在Bob的视角，为了达到平衡，Bob一开始只能和Alice对称取，直到一边的问号全部填完。剩下的问题变成：有$x$个问号，两人轮流填数，总和等于某数则Bob赢，否则Alice赢。（1）当$x$为奇数时，Alice必胜，因为最后一次填数是Alice，他无论如何都可以破坏平衡性。（2）当$x$为偶数时，我们可以两两分组，即每一次Alice+Bob填数视为一组。如果总和是9的倍数，假设Alice填的数是$a$，那么Bob总可以找到一个$9 - a$，使得剩下的局面又是Bob的必胜态（即总和为9的倍数），所以Bob必胜，反之，Alcie必胜。



##### **Code Q3**
```cpp
class Solution {
public:
    bool sumGame(string num) {
        int n = num.size();
        int sum1 = 0, sum2 = 0;
        int cnt1 = 0, cnt2 = 0;
        for(int i = 0; i < n; i++) {
            if (i < n / 2) {
                if (num[i] == '?') cnt1 ++;
                else sum1 += num[i] - '0';
            } else {
                if (num[i] == '?') cnt2 ++;
                else sum2 += num[i] - '0';
            }
        }
        if (cnt1 > cnt2 && sum1 > sum2) return true;
        if (cnt1 < cnt2 && sum1 < sum2) return true;
        int cnt = abs(cnt1 - cnt2);
        int sum = abs(sum1 - sum2);
        if (cnt == 0) return sum != 0;
        if (cnt % 2) return true;
        if ((cnt / 2) * 9 == sum) return false;
        return true;
    }
        
    
};
```
#### [规定时间内到达终点的最小花费](https://leetcode-cn.com/problems/minimum-cost-to-reach-destination-in-time/)
如果不考虑时间，本题是一个正边权的最短路问题，利用dijkstra或者SPFA等最短路算法可以解决。而如果加上时间，可以将原图的每一个点增加一个维度（拆点）。不妨记$ (t,u)$为点 $u$ 为在 $t$时刻的状态，每条有向边记为$((t,u),(t+w,v))$，表示点$u$到点$v$经过了$w$时间。

设$f(t, u)$表示从$(0,0)$到$(t,u)$的最少费用，初始状态$f(0,0) = passfee(0)$，其余设为无穷大。转移方程$f(t,v)=min(f(t,v),f(t−w,u)+passfee(v))$。最终答案为  $min(f(t,n−1)) $。

本题在实现时有一个技巧，注意到当$t_1 < t_2$时，一定是用$f_1$的状态去更新$f_2$，因此我们可以先枚举时间维度，然后再枚举所有边即可（边的枚举顺序不影响结果）。这种方法的好处是不需要单独建图。


##### **Code Q4**
```cpp
class Solution {
public:
    int minCost(int m, vector<vector<int>>& edges, vector<int>& pf) {
        const int INF = 1e8;
        int n = pf.size();
        vector<vector<int>> f(n, vector<int>(m + 1, INF));
        f[0][0] = pf[0];
        for (int i = 1; i <= m; i++) {
            for (const auto& e : edges) {
                int x = e[0], y = e[1], cost = e[2];
                if (i < cost) continue;
                f[x][i] = min(f[x][i], f[y][i - cost] + pf[x]);
                f[y][i] = min(f[y][i], f[x][i - cost] + pf[y]);
            }
        }
        int ans = INF;
        for (int i = 1; i <= m; i++)
            ans = min(ans, f[n - 1][i]);
        return ans == INF ? -1 : ans;
    }
};

```