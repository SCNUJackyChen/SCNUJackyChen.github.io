---
title: LC 周赛第243场
date: 2021-05-30 18:37:19
tags: [Leetcode, 贪心, 堆, DP]
categories: Leetcode周赛
katex: true
description:  LC周赛第243场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **题目**

[检查某单词是否等于两单词之和](https://leetcode-cn.com/problems/check-if-word-equals-summation-of-two-words/)
[插入后的最大值](https://leetcode-cn.com/problems/maximum-value-after-insertion/)
[使用服务器处理任务](https://leetcode-cn.com/problems/process-tasks-using-servers/)
[准时抵达会议现场的最小跳过休息次数](https://leetcode-cn.com/problems/minimum-skips-to-arrive-at-meeting-on-time/)


### **题解**

第一题模拟题。


#### **Code Q1**
```cpp
class Solution {
public:
    int cal(string s) {
        int res = 0;
        for(int i = 0; i < s.size(); i++) {
            res = res * 10 + s[i] - 'a';
        }
        return res;
    }
    bool isSumEqual(string a, string b, string c) {
        return cal(a) + cal(b) == cal(c);
    }
};
```

第二题是一个简单的贪心。分正负数讨论，如果是正数，那么要使得整个数最大，就是找到左边第一个比当前数小的数，插入到该位置即可；负数的情况相反。

#### **Code Q2**
```cpp
class Solution {
public:
    string maxValue(string s, int x) {
        if (s[0] != '-') {
            for(int i = 0; i < s.size(); i++) {
                string str = to_string(x);
                if (s[i] - '0' < x) {
                    s.insert(i, str);
                    return s;
                }
                if (i == s.size() - 1) {
                    s += str;
                    return s;
                }
            }
        } else {
            for(int i = 1; i < s.size(); i++) {
                string str = to_string(x);
                if (s[i] - '0' > x) {   
                    s.insert(i, str);
                    return s;
                }
                if (i == s.size() - 1) {
                    s += str;
                    return s;
                }
            }
        }
        return "";
    }
};
```

第三题是一个比较复杂的数据结构题。考虑到服务器被调度完之后是可以继续使用的，所以被调度的服务器需要暂时存放在另外一个容器中。由题意，需要每次都找到权重最小的服务器，所以考虑维护一个优先队列（最小堆）；在服务器调度完归还时，需要找到最近的调度完的服务器，所以这里也需要维护一个优先队列（最小堆）。

在模拟调度算法时，需要记录一下时间戳，正常情况下每一个时间戳都会由一个新任务进来，但是可能没有服务器可供调度，所以这些任务会滞后，直到最早结束的服务器空闲出来。因此在维护时间戳的时候需要分情况讨论。

#### **Code Q3**
```cpp
class Solution {
public:
    vector<int> res;
    priority_queue<pair<int,int>,vector<pair<int,int> >,greater<pair<int,int> >> A;
    priority_queue<pair<int,int>,vector<pair<int,int> >,greater<pair<int,int> >> B;
    
    
    vector<int> assignTasks(vector<int>& servers, vector<int>& tasks) {
        int n = servers.size();
        int m = tasks.size();
        int timer = 0, id = 0;
        
        for(int i = 0; i < n; i++) {
            A.push({servers[i], i});
        }
        
        while(res.size() < m) {
            while(B.size()) {
                auto p = B.top();
                if (p.first <= timer) {
                    B.pop();
                    A.push({servers[p.second], p.second});
                } else {
                    break;
                }
            }
            
            if (A.size()) {
                while(A.size() && id <= timer) {
                    auto s = A.top();
                    A.pop();
                    res.push_back(s.second);
                    if (res.size() == m) return res;
                    B.push({timer + tasks[id], s.second});
                    id ++;
                }
                timer++;
            } else {
                timer = B.top().first; // 没有服务器可以调度，则时间戳更新为最早的服务器空闲时间
            }
            
        }
        return res;
        
    }
};
```

第四题是一个DP题。

用`f[i,j]`表示前`i`辆车操作`j`次所用的最小时间。它可以按是否操作第`i`辆车分为2类，在这两类中取min即可。
$$f[i,j] = min(\lceil f[i - 1,j] + time \rceil, f[i - 1, j - 1] + time)$$


#### **Code Q4**
```cpp
class Solution {
public:
    int minSkips(vector<int>& dist, int speed, int m) {
        int n = dist.size();
        vector<vector<double>> f(n + 1, vector<double>(n + 1, 1e9));
        f[0][0] = 0;
        f[0][1] = 0;
        f[1][0] = dist[0] * 1.0 / speed;
        
        for(int i = 1; i <= n; i++) {
            f[i][0] = ceil(f[i - 1][0] + dist[i - 1] * 1.0 / speed);
        }
        for(int i = 1; i <= n; i++) {
            double t = dist[i - 1] * 1.0 / speed;
            for(int j = 0; j <= n; j++) {
                if (j)
                    f[i][j] = min(f[i - 1][j - 1] + t - 1e-8, ceil(f[i - 1][j] + t));
            }
        }
        for(int i = 0; i <= n; i++) {
            if (f[n][i] <= m) return i;
        }
        return -1;
    }
};

```