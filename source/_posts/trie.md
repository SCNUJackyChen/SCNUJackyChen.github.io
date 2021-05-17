---
title: Trie树
date: 2021-05-17 20:35:33
tags: [trie]
categories: 模板题
katex: true
---

![trie](/images/template/trie.png)

<!--more-->

[pic source](https://diffnest.github.io/2019/09/22/%E6%9C%AD%E8%AE%B0%E4%B9%8BPHP%E5%AE%9E%E7%8E%B0%E5%AD%97%E5%85%B8%E6%A0%91-Trie/)

###  **前言**

Trie树又称为字典树，是一种多叉树。树的节点存储字符。这种数据结构能够存储单词，并且能够**高效查询**给定单词。和暴力方法——即逐个枚举字典中的单词（时间复杂度为$O(nL)$, $n$为单词总数，$L$为所有词的平均长度）所不同的是，trie树事先将具有相同前缀的单词聚合起来（所以又称为前缀树）。这里我使用*聚合*一词比较抽象，下面加以举例说明：

比如本文封面图片中存储了"JAVA"和"JS"这两个单词，它们有共同的前缀“J”。所谓聚合，就是将相同的部分（即“J”字符）**共享**一个节点。而沿着该节点往下延伸时，两个单词出现了不同，因此产生分叉。

为什么trie树支持高效查询呢？我们从树的根节点开始查询，每次需要判断每一个孩子节点所存储的字符是否等于给定词的对应字符，一个节点最多有$C$个孩子（$C$为字符集大小），而平均树高等于单词的平均长度$L$，因此时间复杂度为$CL$。所以，当$C < n$时，使用trie树更佳。实际上，一般应用场景都是满足$C < n$的，比如英文单词无非就是26个字母组成，这个数远小于一篇文章的字数，通常就是成千上万个单词。另外可以注意到，当$n$足够大时，$n$与$C$呈对数关系（因为每个字符有$C$种可能，一共$C^{nL}$种组合），所以时间复杂度也可以写成$O(Llogn)$。

### **经典应用**

#### **最大异或对**

> 给一个数组，求其中异或值最大的一对数。

暴力解是$O(n^2)$的。
利用trie树可以优化到$O(nlogn)$。

- **思路**
这里把数组中每个整数看成01比特串，我们可以构建出一棵01-trie树。找到最大的异或值，就是沿着trie树寻找与查询的数尽可能相反的数。

例题[最大异或对](https://www.acwing.com/problem/content/145/)

```cpp
#include<iostream>
#include<cmath>

using namespace std;

const int N = 100010, M = 31 * N; // 有符号整数最高有31位
int trie[M][2], a[N], idx; // 用数组模拟树

void insert(int x)
{
	int p = 0;
	for(int i = 30; i >= 0; i--){
		int u = x >> i & 1; // 分别取出每一位
		if(!trie[p][u]) trie[p][u] = ++idx; // 如果p指针所指向的节点还没有孩子节点u，则新建一个节点，idx从1开始计数
		p = trie[p][u]; // p往下走
	}
}

int query(int x)
{
	int p = 0, res = 0;
	for(int i = 30; i>=0; i--){
		int u = x >> i & 1;
		if(trie[p][!u]){ // 尽量往相反方向走
			p = trie[p][!u]; 
			res = res * 2 + !u;
		}else{  // 反方向没节点，只能走相同方向
			p = trie[p][u];
			res = res * 2 + u;
		}
	} 
	return res;
}

int main()
{
	int n, res = 0;
	cin>>n;
	for(int i = 0; i<n; i++) cin>>a[i];
	for(int i = 0; i<n; i++){
		insert(a[i]);
		int t = query(a[i]);
		res = max(res, a[i] ^ t);
		
	}
	cout<<res<<endl;
	return 0;
}
```

#### **最大异或和区间**

> 给一个数组，求其中异或值最大的一段区间。

例题[牛异或](https://www.acwing.com/problem/content/description/1416/)

这是对上一题的延伸，先利用前缀和预处理一下原数组，可以将问题化归为上一题。

 - **思路**
1. 求前缀异或和；
2. 从左到右枚举所有前缀和，将每个前缀和分别插入Trie树并查询跟它异或值最大的配对，记录最大的异或值

- **为什么求前缀和可以work？**
假设我们求的是普通加法和，不是异或和，那求完前缀和数组后可以在$O(1)$的时间内求一段区间$(i,j)$的和: $sum=prefix[j]−prefix[i−1]$。

现在把 $+$ 换成 $\oplus$, 由于异或运算的逆运算还是异或本身，也就是$a \oplus b = c \rightarrow c \oplus b = a$, 那么区间$(i,j)$的和: $sum = prefix[j] \oplus prefix[i - 1]$。

- **贪心性质**

注意到题目要求输出**长度最小**的区间，并且**右端点也应该最小**。这2点在代码中如何保证呢？

1. 首先区间长度最小。我们每次向Trie树中插入一个数后，应当把它在原数组中的坐标也保存下来，而且如果该数重复出现，应当覆盖掉原来的坐标。比如求出前缀和数组为: [0,1,1,2]，当向Trie树插入第二个1时，对应的1的坐标应该更新为3（假设下标从1开始）。这样做保证了每次查询返回的最大异或配对的坐标都是离当前遍历坐标最近的，因此保证了最小区间。可以用反证法证明这么做一定保证了能取到长度最小的区间（这里就不给出了）。

2. 接着是右端点最小。这个很简单，由于我们是从左向右遍历，只需要让异或值严格大于当前最大值时再更新最大值就可以保证了。

```cpp
#include<iostream>

using namespace std;

const int N = 100010;
int a[N], trie[N * 21][2], pos[N * 21], idx, l, r;  
int mm = -1;

void insert(int id)
{
    int p = 0, x = a[id];
    for(int i = 20; i >= 0; i--){
        int u = x >> i & 1;
        if(!trie[p][u]) trie[p][u] = ++idx;
        p = trie[p][u];
    }
    pos[p] = id;       // 记录一下插入的数的下标
}

pair<int, int> query(int id)  // 这里需要返回2个值，一个是最大的异或配对，一个是这个配对的下标
{
    int p = 0, res = 0, x = a[id];
    for(int i = 20; i >= 0; i--){
        int u = x >> i & 1;
        if(trie[p][!u]){
          p = trie[p][!u];  
          res = res * 2 + !u;
        }else{
            p = trie[p][u];
            res = res * 2 + u;
        }
    }
    return {res, pos[p]};
}

int main()
{
    int n;
    pair<int, int> t;
    cin>>n;

    insert(0);  // 预先插入一个0
    for(int i = 1; i <= n; i++) {
       cin>>a[i];
       a[i] ^= a[i - 1];
       t = query(i);
       if((t.first ^ a[i]) > mm){  // 如果异或值严格大于当前最大值，更新一下最大值，还有区间端点
          mm = (t.first ^ a[i]); 
          l = t.second + 1;
          r = i;
       }       
       insert(i);
    }
    cout<<mm<<' '<<l<<' '<<r<<endl;
    return 0;
}

```

#### **限制最大长度的最大异或和区间**

> 给一个数组，求其中异或值最大的一段区间，区间有最大长度限制。

例题[最大异或和](https://www.acwing.com/problem/content/3488/)

在上一题的基础上，维护一个滑动窗口，并且及时删除不在窗口内的数。

在实现删除上，采用打标记的方式，并不是真的物理删除。

```cpp
#include<iostream>
#include<algorithm>

using namespace std;

const int N = 100010 * 31, M = 100010;
int son[N][2], s[M], cnt[N], idx; // cnt数组记录当前节点有无孩子，如果插入重复的数，cnt的元素会大于1

void insert(int x, int v) 
{
    int p = 0;
    for(int i = 30; i >= 0; i--) {
        int u = x >> i & 1;
        if (!son[p][u]) son[p][u] = ++idx;
        p = son[p][u];
        cnt[p] += v;  
    }
}

int query(int x) {
    int p = 0, res = 0;
    for(int i = 30; i >= 0; i--) {
        int u = x >> i & 1;
        if (cnt[son[p][!u]]) p = son[p][!u], res = res * 2 + 1;
        else p = son[p][u], res = res * 2;
    }
    return res;
}

int main()
{
    int n, m;
    cin >> n >> m;
    for(int i = 1; i <= n; i++) {
        cin >> s[i];
        s[i] ^= s[i - 1];
    }
    
    int res = 0;
    insert(s[0], 1);
    
    for(int i = 1; i <= n; i++) {
        if (i > m) insert(s[i - m - 1], -1);  // -1 表示删除这个数
        res = max(res, query(s[i]));
        insert(s[i], 1);
    }
    cout << res << endl;
    return 0;
}
```
