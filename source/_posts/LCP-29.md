---
title: LCP-29 乐团站位
date: 2021-04-06 10:37:32
tags: [Leetcode, 找规律, 数学]
categories: Leetcode周赛
katex: true
description: 给定一个方阵，从左上角开始延顺时针螺旋向内填入数字1~9，循环往复。求给定坐标所对应的数字。
---

![LC](/images/LCP-2021/LOGO.png)

<!--more-->

[题目](https://leetcode-cn.com/problems/SNJvJP/)

### **思路**

我们可以先计算出目标点向外一共有多少圈，然后再分别讨论目标点在当前圈的上、右、下、左四种情况下的取值。


#### **1.计算圈数**

目标点有可能落在所在圈的上、右、下、左四个方向，如何根据其下标$(x,y)$来确定其在第几圈？

可以分别计算目标点到四个最外层边界的距离，最短的距离即为圈数。

#### **2.计算外圈的长度**

不难注意到，边长为$a$的圈长度为$4a - 4$，向内一圈的边长为$a - 2$，长度为$4(a-2) - 4 = 4a - 12$。所以是一个首项为$4a - 4$，公差为$-8$的等差数列。利用等差数列求和公式$S_n = a_1n + n(n-1)d/2$即可求出外圈总长度。

*注意：由于数据范围为$10^9$, 直接使用求和公式会爆  `long long`，需要进行等价变形*

#### **3.分类讨论目标点落在所在圈的哪个方向**

不妨令所在圈四个边界的坐标为`left, right, up, down`

1. 如果在上方，则只需要加1段，即从左上角出发，向右走$y - left + 1$的距离；
2. 如果在右方，则需要加2段，即从左上角出发，向右走$y - left$的距离，再向下走$x - up + 1$；
3. 如果在下方，则需要加3段，即从左上角出发，向右走到底，再向下走到底，一共走$2(y - left)$，再向左走$right - y + 1$;
4. 如果在左方，则需要加4段，即从左上角出发，向右走到底，再向下走到底，再向左走到底，一共走$3(y - left)$，再向上走$down - x + 1$.


### **参考代码**

```cpp
class Solution {
public:
    int orchestraLayout(int n, int x, int y) {
        long long layer = min(min(x, y), min(n - x - 1, n - y - 1));

        // 下面这2行是等差数列求和公式的变形，为了防爆long long
        long long len = layer * (n - 1) - (layer - 1) * layer;
        len <<= 2; 
        
        long long up = layer, down = n - layer - 1;
        long long left = layer, right = n - layer - 1;
       
        if (x == up) len += y - left + 1; 
        else if (y == right) len += (right - left) + (x - up + 1);
        else if (x == down) len += (right - left) * 2 + (right - y + 1);
        else if (y == left) len += (right - left) * 3 + (down - x + 1);
      
        
        return len % 9 == 0 ? 9 : len % 9;
    }
};
```
