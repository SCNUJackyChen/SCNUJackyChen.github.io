---
title: LC-4 寻找两个正序数组的中位数
date: 2021-03-02 10:31:01
tags: [Leetcode, 递归, 难题杂烩]
categories: 算法
katex: true
description: 给定两个严格递增序列，求合并后数组的中位数
---

 ![LC](/images/Leetcode.jpg)

<!--more-->

[题目](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

###  思路

时间复杂度$O(log(n + m))$

更一般地， 此题也可问：

> 在两个有序数组中，找出第$k$小的数。

因此找中位数就相当于找第$(n + m) / 2$小的数。

解法的核心点在于：利用$nums1[k / 2 - 1]$ 与 $nums2[k / 2 - 1]$的大小关系来删掉非答案区间，进而缩小问题的规模，最后规约到一个朴素的情况。

 - 如果$nums1[k / 2 - 1] > nums2[k / 2 - 1]$，考察$nums2[k / 2 - 1]$这个数，在$nums2$中，严格小于它的数有$k / 2$个；在$nums1$中，严格小于它的数**最多**有$k / 2 - 1$个。所以，$nums2[k / 2 - 1]$这个数最多排第$k - 1$小，我们要找的第$k$小不在这个区间。同时，$nums2$中排在这个数左边的数更不可能存在第$k$小。因此，可以排除$nums2$中的部分数。
 - 如果$nums1[k / 2 - 1] < nums2[k / 2 - 1]$, 同理，可以排除$nums1$中的部分数。

每一次问题的规模都将减少一半，因为求的是中位数。

### 代码

```cpp
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int total = nums1.size() + nums2.size();
        if(total % 2 == 0){
            int left = find(nums1, 0, nums2, 0, total / 2);
            int right = find(nums1, 0, nums2, 0, total / 2 + 1);
            return (left + right) / 2.0;
        }else{
            return find(nums1, 0, nums2, 0, total / 2 + 1);
        }
    }
    int find(vector<int>& nums1, int i, vector<int>& nums2, int j, int k) {
        if(nums1.size() - i > nums2.size() - j) return find(nums2, j, nums1, i, k);
        if(i == nums1.size()) return nums2[j + k - 1];
        if(k == 1) return min(nums1[i], nums2[j]);
        
        int si = min((int)nums1.size(), i + k / 2), sj = j + k - k / 2;
        if(nums1[si - 1] < nums2[sj - 1]) return find(nums1, si, nums2, j, k - (si - i));
        else return find(nums1, i, nums2, sj, k - (sj - j));
    }
};


```

