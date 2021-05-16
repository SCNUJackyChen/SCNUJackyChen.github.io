---
title: LC 周赛第241场
date: 2021-05-16 22:42:58
tags: [Leetcode, 模拟, 哈希表, 斯特林数]
categories: Leetcode周赛
katex: true
description:  LC周赛第241场复盘+题解
---

![LC](/images/Leetcode.jpg)

<!--more-->

### **题目**
[找出所有子集的异或和再求和](https://leetcode-cn.com/problems/sum-of-all-subset-xor-totals/)
[构成交替字符串需要的最小交换次数](https://leetcode-cn.com/problems/minimum-number-of-swaps-to-make-the-binary-string-alternating/)
[找出和为指定值的下标对](https://leetcode-cn.com/problems/finding-pairs-with-a-certain-sum/)
[恰有K根木棍可以看到的排列数目](https://leetcode-cn.com/problems/number-of-ways-to-rearrange-sticks-with-k-sticks-visible/)

### **题解**

第一题来源于求子集的经典问题，可以利用dfs或者位运算来求解决，考试的时候用位运算来做，代码比较简单。

#### **Code Q1**
```cpp
class Solution {
public:
    int subsetXORSum(vector<int>& nums) {
        int res = 0;
        int n = nums.size();
        for(int i = 0; i < 1 << n; i++) {
            int t = 0;
            for(int j = 0; j < n; j++) {      
                if (i >> j & 1) {
                    t ^=  nums[j];
                }
            }
            res += t;
        }
        return res;
    }
};
```
第二题不难找出规律，如果有解，必然1和0的个数相差不超过1。当1的个数比0多时，必然最后的字符串为`101010...`，反之，如果0的个数比1多，最后为`010101...`。如果一样多，则分别算出转化为上述2种字符串所需的移动次数，然后取较小者即可。

#### **Code Q2**
```cpp
class Solution {
public:
    int minSwaps(string s) {
        int one = 0, zero = 0;
        for (int i = 0; i < s.size(); i++) {
            if (s[i] == '1') one++;
            else if (s[i] == '0') zero++;
        }
        if (abs(one - zero) > 1) return -1;
        
        int res = 0;
        if (one > zero) {
            int diff = 0;
            for(int i = 0; i < s.size(); i++) {
                if (i % 2) {
                    if (s[i] != '0') diff++;
                } else {
                    if (s[i] != '1') diff++;
                }
            }
            res = diff;
        } else if (one < zero) {
            int diff = 0;
            for(int i = 0; i < s.size(); i++) {
                if (i % 2) {
                    if (s[i] != '1') diff++;
                } else {
                    if (s[i] != '0') diff++;
                }
            }
            res = diff;
        } else {
            int diff = 0;
            for(int i = 0; i < s.size(); i++) {
                if (i % 2) {
                    if (s[i] != '1') diff++;
                } else {
                    if (s[i] != '0') diff++;
                }
            }
            int t1 = diff;
            diff = 0;
            for(int i = 0; i < s.size(); i++) {
                if (i % 2) {
                    if (s[i] != '0') diff++;
                } else {
                    if (s[i] != '1') diff++;
                }
            }
            int t2 = diff;
            res = min(t1, t2);
        }
        return res / 2;
    }
};
```

第三题考试的时候真的裂呀😭脑子瓦特了... 一开始突发奇想用multimap，结果发现`add`和`count`只能有一个函数是$O(nlogn)$的（因为`add`的时候需要用下标作为key，而`count`则需要值作为key）然后一番思索后想出优化方案：`add`不用真的删除map里面的值，而单独开一个minus记录删去的值，在`count`的时候再减去就行了（俗称”假删“）。感觉这波稳了，但是！！TLE了！我当时就纳闷了，`add`我只是用了插入操作，复杂度在$O(klogn)$（k是操作次数）；难道`count`不是对数级别的吗？

在最后发觉根本不用multimap，直接哈希存一下值和**个数**就好了（脑子是真瓦特了

#### **Code Q3**
```cpp
class FindSumPairs {
public:
    map<long long, int> S2;
    unordered_map<int, long long> aux2;
    vector<int> n1, n2;
    
    FindSumPairs(vector<int>& nums1, vector<int>& nums2) {
        for(int i = 0; i < nums2.size(); i++) {
            S2[nums2[i]]++;
            aux2[i] = nums2[i];
        }
        n1 = nums1, n2 = nums2;
        sort(n1.begin(), n1.end());
    }
    
    void add(int index, int val) {
        long long old_value = aux2[index];
        aux2[index] += val;
        long long new_value = old_value + val;
        S2[old_value] --;
        S2[new_value] ++;
    }
    
    int count(int tot) {
        int res = 0;
        for(auto num : n1) {
            if (num > tot) break;
            if(S2.count(tot - num))
                res += S2[tot - num];
        }
        return res;
    }
};

/**
 * Your FindSumPairs object will be instantiated and called as such:
 * FindSumPairs* obj = new FindSumPairs(nums1, nums2);
 * obj->add(index,val);
 * int param_2 = obj->count(tot);
 */
```

但是还是没搞清楚为啥multimap就不行啊，这里贴一个当时TLE的代码

```cpp
class FindSumPairs {
public:
    multiset<long long> S2;
    unordered_map<int, long long> aux2;
    unordered_map<long long, int> minus;
    vector<int> n1, n2;
    
    FindSumPairs(vector<int>& nums1, vector<int>& nums2) {
        for(int i = 0; i < nums2.size(); i++) {
            S2.insert(nums2[i]);
            aux2[i] = nums2[i];
        }
        n1 = nums1, n2 = nums2;
        sort(n1.begin(), n1.end());
    }
    
    void add(int index, int val) {
        long long old_value = aux2[index];
        aux2[index] += val;
        long long new_value = old_value + val;
        S2.insert(new_value);
        minus[old_value] ++;
    }
    
    int count(int tot) {
        int res = 0;
        for(auto num : n1) {
            if (num > tot) break;
            res += S2.count({tot - num});
            res -= minus[tot - num];
        }
        return res;
    }
};

/**
 * Your FindSumPairs object will be instantiated and called as such:
 * FindSumPairs* obj = new FindSumPairs(nums1, nums2);
 * obj->add(index,val);
 * int param_2 = obj->count(tot);
 */
```

事后诸葛亮一下，其实问题就在multimap自带的`count`函数。
查阅STL源码可以看到`count`函数是multimap的父类`_Tree`提供的接口：

```cpp
_NODISCARD size_type count(const key_type& _Keyval) const {
        if _CONSTEXPR_IF (_Multi) {
            const auto _Ans = _Eqrange(_Keyval);
            return static_cast<size_type>(_STD distance(
                _Unchecked_const_iterator(_Ans.first, nullptr), _Unchecked_const_iterator(_Ans.second, nullptr)));
        } else {
            return _Lower_bound_duplicate(_Find_lower_bound(_Keyval)._Bound, _Keyval);
        }
    }
```
这里逻辑比较清楚，先用`_Eqrange`求出给定key在multimap中第一次出现的位置（迭代器）和最后出现的位置，因为`_Tree`的实现是一棵红黑树，亦是一棵排序二叉树，因此相同的键必然是逻辑上相邻的（这也就是为什么会返回一个range）。接下来是重点，有了range，自然要计算这个range中有多少个元素，所以进入`distance`一探究竟。

```cpp
_NODISCARD _CONSTEXPR17 _Iter_diff_t<_InIt> distance(_InIt _First, _InIt _Last) {
    if constexpr (_Is_random_iter_v<_InIt>) {
        return _Last - _First; // assume the iterator will do debug checking
    } else {
        _Adl_verify_range(_First, _Last);
        auto _UFirst             = _Get_unwrapped(_First);
        const auto _ULast        = _Get_unwrapped(_Last);
        _Iter_diff_t<_InIt> _Off = 0;
        for (; _UFirst != _ULast; ++_UFirst) {
            ++_Off;
        }

        return _Off;
    }
}
```
这里的逻辑是：如果迭代器是random_iter，那么直接就可以计算出距离了（因为随机访问迭代器支持相减操作），如果迭代器不是random_iter，那么只能乖乖地遍历。那么multimap的迭代器是不是random_iter呢？答案是：No！原因也很直观，因为底层实现是红黑树，这种数据结构的节点在内存中都是分散的，不是一段连续的内存（如vector、array）。

所以，multimap的`count`函数实际上是$O(n)$的。由题目数据范围可算：$1000 * 10^5 = 10^8$， 妥妥的TLE了。

---

第四题来自于一种经典问题——第一类斯特林数。懂得都懂，我就不懂。

可以把从左往右看到的每一根柱子及其右边被它遮挡的柱子们视为一个区块(假设大小为$s$，这个区块的排列数目恰好是一个圆排列($(s - 1)!$)，整体来看就是问长为$n$的排列划分为$k$个非空圆排列的方案数，也就是**第一类斯特林数**。

第一类斯特林数的递推公式：
$f(i,j)$表示$i$ 个数划分为 $j$ 个非空圆排列的方案数，可拆分成2种子问题：
1. 第$i$ 个数自己分一组，那么剩下 $i - 1$个数需要划分为$j - 1$ 个非空圆排列，即$f(i - 1, j - 1)$;
2. 第$i$ 个数分到之前的组，不妨让它分别插到每个组中每个数的后面，因为有 $i - 1$个数，所以有 $i - 1$ 种插法，每一种插法对应的方案数为 $f(i - 1, j)$;

所以，
$$f(i,j) = f(i - 1, j -1) + f(i - 1, j) * (i - 1)$$
#### **Code Q4**
```cpp
class Solution {
public:
    int rearrangeSticks(int n, int k) {
        const int MOD = 1e9 + 7;
        vector<vector<long long>> f(n + 1, vector<long long>(k + 1));
        f[0][0] = 1;
        for(int i = 1; i <= n; i++) {
            for(int j = 1; j <= k; j++) {
                f[i][j] = (f[i - 1][j - 1] + (i - 1) * f[i - 1][j]) % MOD;
            }
        }
        return f[n][k];
    }
};
```