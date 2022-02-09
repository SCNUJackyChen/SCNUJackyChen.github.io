---
title: LC weekly contest 279
date: 2022-02-09 23:13:33
tags: [Leetcode, 模拟, 贪心, DP]
categories: Leetcode周赛
katex: true
description:  LC 周赛第279场题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **Summary**

手速场 + 运气好没卡题，勉强挤进top500，上了一波分，希望能继续保持吧~ヾ(•ω•`)o


###  **Solution**

#### [对奇偶下标分别排序](https://leetcode-cn.com/problems/sort-even-and-odd-indices-independently/)

##### **Code Q1**
```cpp
class Solution {
public:
    vector<int> sortEvenOdd(vector<int>& nums) {
        vector<int> a, b;
        for (int i = 0; i < nums.size(); i++) {
            if (i % 2) a.push_back(nums[i]);
            else b.push_back(nums[i]);
        }
        sort(a.begin(), a.end(), greater<int>());
        sort(b.begin(), b.end());
        vector<int> res;
        for (int i = 0, j = 0, k = 0; i < nums.size(); i++) {
            if (i % 2) res.push_back(a[j]), j++;
            else res.push_back(b[k]), k++;
        }
        return res;
    }
};
```

#### [重排数字的最小值](https://leetcode-cn.com/problems/smallest-value-of-the-rearranged-number/)

We can sort all digits in ascending order, pick the first non-zero number as the first digit of result, following with zero, if any, and then the remaining numbers.

For negative numbers, sort the digits in descending order and the following steps remain the same.

##### **Code Q2**
```cpp
class Solution {
public:
    long long smallestNumber(long long num) {
        if (num == 0) return 0;
        if (num > 0) {  // positive number
            vector<int> b;
            long long x = num;
            while (x) {
                b.push_back(x % 10);
                x /= 10;
            }
            sort(b.begin(), b.end());
            long long res = 0, head = -1;
            for (int i = 0; i < b.size(); i++) {
                if (b[i] == 0 && head == -1) continue;
                else {
                    head = i;
                    break;
                }
            }
            res = b[head];
            for (int i = 0; i < b.size(); i++) {
                if (i != head)
                    res = res * 10 + b[i];
            }
            
            return res;
        } else {  // negative number
            long long x = -num; 
            vector<int> b;
            while (x) {
                b.push_back(x % 10);
                x /= 10;
            }
            sort(b.begin(), b.end(), greater<int>());
            long long res = 0, head = -1;
            for (int i = 0; i < b.size(); i++) {
                if (b[i] == 0 && head == -1) continue;
                else {
                    head = i;
                    break;
                }
            }
            res = b[head];
            for (int i = 0; i < b.size(); i++) {
                if (i != head)
                    res = res * 10 + b[i];
            }
            
            return -res;
        }
        return 0;
    }
};
```

#### [设计位集](https://leetcode-cn.com/problems/design-bitset/)

The most hard function in this question should be `flip()`, since the operation number is up to $10^5$, `flip()` should finish in constant time.

The idea is that we can define a auxiliary string, which maintains the opposite bit state of the original one, e.g for original string `1010` , its auxiliary string would be `0101`. Each time we call the `flip()`, just swap this two string (only change pointer, thus it is $O(1)$).

##### **Code Q3**

```cpp
class Bitset {
public:
    
    int zero_cnt, one_cnt, tot;
    string str, str_flip;
    
    Bitset(int size) {
        for (int i = 0; i < size; i++) str.push_back('0'), str_flip.push_back('1');
        tot = size;
        zero_cnt = size;
        one_cnt = 0;        
    }
    
    void fix(int idx) {
        if (str[idx] == '1') return;
        str[idx] = '1';
        str_flip[idx] = '0';
        zero_cnt--;
        one_cnt++;
    }
    
    void unfix(int idx) {
        if (str[idx] == '0') return;
        str[idx] = '0';
        str_flip[idx] = '1';
        zero_cnt++;
        one_cnt--;
    }
    
    void flip() {
        swap(str, str_flip);
        swap(one_cnt, zero_cnt);
    }
    
    bool all() {
        return one_cnt == tot;
    }
    
    bool one() {
        return one_cnt > 0;
    }
    
    int count() {
        return one_cnt;
    }
    
    string toString() {
        return str;
    }
};

/**
 * Your Bitset object will be instantiated and called as such:
 * Bitset* obj = new Bitset(size);
 * obj->fix(idx);
 * obj->unfix(idx);
 * obj->flip();
 * bool param_4 = obj->all();
 * bool param_5 = obj->one();
 * int param_6 = obj->count();
 * string param_7 = obj->toString();
 */
```

#### [移除所有载有违禁货物车厢所需的最少时间](https://leetcode-cn.com/problems/minimum-time-to-remove-all-cars-containing-illegal-goods/)


Let us begin with considering the question from left to right. 

Define $A[i]$ as the minimum cost to remove all `"1"`s in $S_0$ ~ $S_i$. For the $i_{th}$ `"1"` from left, we have totally two ways to remove it:

1. directly pick it out, the cost will be $A[i - 1] + 2$;
2. remove all `"0"` and `"1"`, the cost will be $i + 1$, where $i \in [1,n)$

So we can update $A[i]$ by $A[i] = min(A[i - 1] + 2, i + 1)$

The same idea for right to left to get a $B$ array, whose elements represent the minimum cost to remove all `"1"` in $S_i$ ~ $S_n$.

Finally, in order to find a break point which leads to a global minimum, we can just run through $S$ and check each position and update the answer.

$$answer = min(A[i] + B[i + 1]), i \in [1, n)$$

##### **Code Q4**

```cpp
class Solution {
public:
    int minimumTime(string s) {
        int n = s.size();
        vector<int> a(n + 1), b(n + 1);

        for (int i = 1; i <= n; i++) {
            if (s[i - 1] == '1') {
                a[i] = min(a[i - 1] + 2, i);
            } else {
                a[i] = a[i - 1];
            }
        }

        for (int i = n - 1; i >= 0; i--) {
            if (s[i] == '1') {
                b[i] = min(b[i + 1] + 2, n - i);
            } else {
                b[i] = b[i + 1];
            }
        }

        int res = INT_MAX;
        for (int i = 1; i <= n; i++) {
            res = min(res, a[i] + b[i]);
        }

        return res;
    }
};
```