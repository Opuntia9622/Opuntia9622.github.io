---
title: AtCoder 蓝黄橙
date: 2025-12-6
categories: 

---

上古文章，补个档。

绿为独立做出来，蓝为比较会，讨论或看了题解做出来，红为完全不会。

### $\color{blue}{\text{ARC165D}}$ $\text{*2589}$

可以跑 $n$ 次 tarjan，同时拿并查集维护两个点值是否相同，每次 tarjan 完了之后同一个强连通分量里的就合并。tarjan 连边就是需要字典序小的往大的连边。

[提交记录](https://atcoder.jp/contests/arc165/submissions/45767682)。

### $\color{green}{\text{ARC166C}}$ $\text{*1813}$

可以按着斜着的轮廓拆出 $\min(n,m)$ 条阶梯状的链。发现每条链计数互不影响，将方案乘起来即可。每条链跑一个形如 $n$ 个数中不能取相邻的数的方案数的 dp 状物，即为这条链的方案数。发现 $n$ 条链各个链的长度是有规律可循的，于是可以做前缀积和快速幂来做到 $\mathcal O(Tn\log n)$。

[提交记录](https://atcoder.jp/contests/arc166/submissions/48966439)。

### $\color{green}{\text{ARC166D}}$ $\text{*2192}$

可以从前往后贪心，拿一个队列维护一下当前没有固定出右端点的左端点，动态维护一下即可。

[提交记录](https://atcoder.jp/contests/arc166/submissions/48968424)。

### $\color{green}{\text{ARC169C}}$ $\text{*2005}$

和前面几道相比，这题比较简单。

考虑设一个 dp 状态 $f(i,j)$ 表示填了前 $i$ 个位置，$a_i=j$ 且 $a_{i+1}\ne j$ 的方案数。暴力转移是四方的，前缀和优化一下即可。

[提交记录](https://atcoder.jp/contests/arc169/submissions/48988048)。

### $\color{blue}{\text{ARC168C}}$ $\text{*2071}$

有难度的。

由于字符集特别小，可以考虑枚举怎么换的。对于一个原串上的字符，有 $6$ 种换法，分别为 $a\to b,b\to a,a\to c,c\to a,b\to c,c\to b$。然后发现根据这 $6$ 种换法会出现 $5$ 种置换环，分别为 $a\to b\to a,a\to c\to a,b\to c\to b,a\to b\to c\to a,a\to c\to b\to a$。

发现其实最后两种置换环是相互独立的，所以只需要枚举前三种情况，再枚举长度为 $3$ 的置换环，分类讨论一下取哪种，并用组合数计算一下方案数。时间复杂度四方。

[提交记录](https://atcoder.jp/contests/arc168/submissions/48994536)。

### $\color{green}{\text{ARC167D}}$ $\text{*2313}$

又是简单题。

考虑 $i\to a_i$ 连边。根据排列性质，我们会得到一堆环，然后 swap 两个环内的元素就可以合并两个环。要求字典序最小，可以从前往后贪心地钦定 $a_i$ 的值，尽量 swap 过来一个更小的值。

[提交记录](https://atcoder.jp/contests/arc167/submissions/48995199)。

### $\color{green}{\text{ARC168D}}$ $\text{*2440}$

考虑区间 dp。设 $f(i,j)$ 表示序列中 $[i,j]$ 填满，其他地方不填的最大操作次数。可以通过前缀和优化做到三方。

[提交记录](https://atcoder.jp/contests/arc168/submissions/48996449)。

### $\color{blue}{\text{ARC167C}}$ $\text{*2338}$

计数仍需加训。

考虑对于每种边的权值单独计数。先将 $a$ 按 $p$ 重排。分类讨论两种情况：

- 第 $i$ 个点向前面 $k$ 个点连一条边，连到了 $a_i$ 的边，那么这前面 $k$ 个点至少有一个点权值 $\leq a_i$。

- 第 $i$ 个点向后面 $k$ 个点连一条边，连到了 $a_i$ 的边，那么这个与其相连的点 $j$ 前面的 $k$ 个点的权值都 $> a_j$，且 $a_i$ 是最小的。

上面两种情况组合计数一下即可。

[提交记录](https://atcoder.jp/contests/arc167/submissions/49000322)。
