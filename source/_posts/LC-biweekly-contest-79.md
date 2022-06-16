---
title: LC biweekly contest 79
date: 2022-06-16 11:13:15
tags: [Leetcode, 线段树, 二分]
categories: Leetcode周赛
katex: true
description:  LC 双周赛第79场题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

###  **Solution**

#### [以组为单位订音乐会的门票](https://leetcode.cn/problems/booking-concert-tickets-in-groups/solution/)

分析一下需求，我们有3个任务：

1. 查询下标最小的可以坐连续$k$个人的行（无论是`gather`还是`scatter`都适用，下文会进一步说明）；
2. 修改某一行的上座数；
3. 查询某一区间的上座数。


由于区间长度为$5\times10^4$，且查询次数为$5\times10^4$，直接暴力不满足时间限制的要求。我们的修改和查询应该控制在$O(nlogn)$的时间以内，这里利用线段树可以动态支持单点修改和区间查询，分别对应需求2和3。对于需求1，可以在线段树上维护区间最小值，进而通过二分在$O(logn)$时间内找出满足要求的最小下标。

对于线段树的每一个结点，除了常规的区间左右端点，我们还需要维护一个区间和（需求3）以及区间最小值（需求1）。对于`gather`操作，我们需要找到一段连续的空位，使得空位数量大于需求数量，并且下标最小。由于我们维护的是区间最小值，问题等价于：找到一个最小下标，使得该下标对应的行的上座数$<$(列数$-$需求数)。这个问题是具有二段性的：因为我们要找最小下标，所以保证了全局至多只有一个解（也可能无解），我们每次查询优先查找左半区间的最小值（因为要满足下标最小），只有当左半区间的最小值不满足要求时，才会考虑右半区间。因此这个问题可以通过在线段树上进行二分搜索来解决。

对于`scatter`操作，方法和上文的线段树二分一致，只需要将需求数降为0即可，即找到满足上座数$<$列数的最小行号。

最后一个问题是，如何给观众上座呢？第一反应是需要实现一个区间修改的函数，而且复杂度需要$O(nlogn)$。但是，本题有一个性质，每个座位只会坐一次，也就是一旦坐上去之后就不能修改了。所以我们完全可以直接逐行遍历并将值修改为列数即可（也就是修改为满座），时间复杂度是线性的。

##### **Code Q4**

```cpp
const int N = 50010;
struct Node{
    int l, r;
    int mi;
    long long sum;
}tr[N * 4];

class BookMyShow {
public:

    int n, m;

    void pushup(int u) {
        tr[u].mi = min(tr[u << 1].mi, tr[u << 1 | 1].mi);
        tr[u].sum = tr[u << 1].sum + tr[u << 1 | 1].sum;
   
    }

    void build(int u, int l, int r) {
        tr[u] = {l ,r};
        if (l == r) {
            tr[u].mi = tr[u].sum = 0;
            return;
        }
        int mid = l + r >> 1;
        build(u << 1, l, mid);
        build(u << 1 | 1, mid + 1, r);
        pushup(u);
    }

    long long query(int u, int l, int r) {
        if (l <= tr[u].l && tr[u].r <= r) return tr[u].sum;

        int mid = tr[u].l + tr[u].r >> 1;
        
        long long sum = 0;
        
        if (l <= mid) {
            sum = query(u << 1, l, r);
            
        }
        if (r > mid) {
            sum += query(u << 1 | 1, l, r);
            
        }
        return sum;
    }

    int query_index(int u, int l, int r, int val) {
        if (tr[u].mi > val) return 0;
        if (tr[u].l == tr[u].r) return tr[u].l;
        int mid = tr[u].l + tr[u].r >> 1;
        int pos = 0;
        if (l <= mid) pos = query_index(u << 1, l, r, val);
        if (pos) return pos;
        if (r > mid) pos = query_index(u << 1 | 1, l, r, val);
        return pos;
    }


    void modify(int u, int x, int v) {
        if (tr[u].l == x && tr[u].r == x) {
            tr[u].mi = tr[u].sum = v;
            return;
        }

        int mid = tr[u].l + tr[u].r >> 1;
        if (x <= mid) modify(u << 1, x, v);
        else modify(u << 1 | 1, x, v);
        pushup(u);
    }
 
    BookMyShow(int _n, int _m) {
        n = _n, m = _m;
        build(1, 1, n);
    }
    
    vector<int> gather(int k, int maxRow) {
        int pos = query_index(1, 1, maxRow + 1, m - k);
        if (!pos) return {};
        int seats = query(1, pos, pos);
        modify(1, pos, seats + k);
        return {pos - 1, seats};
    }
    
    bool scatter(int k, int maxRow) {
        long long sum = query(1, 1, maxRow + 1);
        if (((long long)maxRow + 1) * m - sum < k) return false;
        
        int st = query_index(1, 1, maxRow + 1, m);
        int seats = query(1, st, st);
        
        while (k > m - seats) {
            modify(1, st, m);
            k -= m - seats;
            st ++;
            seats = query(1, st, st);
        }
        modify(1, st, seats + k);
        return true;
    }
};

/**
 * Your BookMyShow object will be instantiated and called as such:
 * BookMyShow* obj = new BookMyShow(n, m);
 * vector<int> param_1 = obj->gather(k,maxRow);
 * bool param_2 = obj->scatter(k,maxRow);
 */
```