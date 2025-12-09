---
title: 操作分块 bitset | P11831 [省选联考 2025] 追忆
date: 2025-06-05
categories: 

---

纪念题解。

谴责造原题数据的人。省选被这道题送走了，即使 Day2 T2 T3 没交反还是上不了线。

考虑没修改怎么做。然而我赛时连这玩意都没想明白。维护 $G_x$ 表示 $x$ 的可达点集，前缀和 bitset $A_i$ 维护 $a_x\in [1,i]$ 的所有 $x$ 的集合，**后缀和** bitset $B_i$ 同理维护 $b$ 序列。对于询问 $(x,l,r)$，先求 $X =(A_r\oplus A_{l-1}) \wedge G_x$，然后再二分第一个和 $X$ 做与运算结果不为空集的 $B_i$，于是就求得最大值了。

接下来考虑操作分块，块长 $S_1$。发现重构一次的复杂度有点爆炸。因为 $O(n/w)$ 和 $O(1)$ 的散块处理复杂度差距特别大，所以可以通过分块平衡复杂度。于是对 $A$ 和 $B$ 再做序列分块，块长 $S_2$。

取 $S_1=w$，$S_2=\dfrac{n}{w}$。

分析下复杂度（比较粗略，一些没啥影响的参数就不计算了）：

重构：$O(\dfrac{q}{S_1}\cdot\dfrac{n^2}{wS_2})=O(\dfrac{qn}{w})$。

$A$ 查询区间和：$O(\dfrac{qn}{w}+q{S_2})=O(\dfrac{qn}{w})$。

二分 $B$：$O(\dfrac{qn}{w}\log\dfrac{n}{S_2}+q{S_2})=O(\dfrac{qn}{w}\log w)$。

所以复杂度就是：$O(\dfrac{qn}{w}\log w)$。可过。

:::info[Code（超级无敌炫酷牛逼元神大王好写）]

```cpp
#include <bits/stdc++.h>
using namespace std;
#define rep(i, x, y) for (int i = (x); i <= (y); i++)
#define per(i, x, y) for (int i = (x); i >= (y); i--)
inline int read() {
    int s = 0, f = 1; char c = getchar();
    while (!isdigit(c)) { if (c == '-') f = -1; c = getchar(); }
    while (isdigit(c)) s = s * 10 + (c ^ 48), c = getchar();
    return s * f;
}
constexpr int N = 1e5 + 5, W = 64;
int n, m, q, a[N], b[N], ba[N], bb[N]; bool buc[N];
int S;
vector<int> e[N], cur;
bitset<N> G[N], A[W + 5], B[W + 5], ta;
void build() {
    rep (i, 1, W) {
        int l = (i - 1) * S + 1, r = min(i * S, n);
        A[i] = A[i - 1];
        rep (j, l, r) A[i][ba[j]] = 1;
    }
    per (i, W, 1) {
        int l = (i - 1) * S + 1, r = min(i * S, n);
        B[i] = B[i + 1];
        rep (j, l, r) B[i][bb[j]] = 1;
    }  
}
void add(int x) {
    if (!buc[x]) cur.push_back(x), buc[x] = 1;
}
void sol() {
    n = read(), m = read(), q = read();
    rep (i, 1, n) e[i].clear(), buc[i] = 0;
    cur.clear();
    rep (i, 1, m) {
        int u = read(), v = read();
        e[u].push_back(v);
    }
    per (i, n, 1) {
        G[i].reset(); G[i].set(i);
        for (auto v : e[i]) G[i] |= G[v];
    }
    rep (i, 1, n) a[i] = read(), ba[a[i]] = i;
    rep (i, 1, n) b[i] = read(), bb[b[i]] = i;
    if (n < 64) {
        while (q--) {
            int o = read(), x = read(), l, r, y;
            if (o == 1) {
                y = read();
                swap(a[x], a[y]);
                ba[a[x]] = x, ba[a[y]] = y;
            } else if (o == 2) {
                y = read();
                swap(b[x], b[y]);
            } else {
                l = read(), r = read(); int ans = 0;
                rep (i, l, r) ans = max(ans, G[x][ba[i]] * b[ba[i]]);
                cout << ans << '\n';
            }
        }
        return ;
    }
    S = (n + W - 1) / W;
    build();
    rep (CC, 1, q) {
        if (!(CC % W)) {
            build();
            for (auto to : cur) buc[to] = 0; cur.clear();
        }
        int o = read(), x = read(), l, r, y;
        if (o == 1) {
            y = read();
            swap(a[x], a[y]);
            ba[a[x]] = x, ba[a[y]] = y;
            add(x), add(y);
        } else if (o == 2) {
            y = read();
            swap(b[x], b[y]);
            bb[b[x]] = x, bb[b[y]] = y;
            add(x), add(y);
        } else {
            l = read(), r = read();
            int bl = (l - 1) / S + 1, br = (r - 1) / S + 1, ans = 0;
            if (bl + 1 < br) ta = A[br - 1] ^ A[bl]; else ta.reset();
            if (bl == br) {
                rep (i, l, r) ta[ba[i]] = 1;
            } else {
                rep (i, l, bl * S) ta[ba[i]] = 1;
                rep (i, (br - 1) * S + 1, r) ta[ba[i]] = 1;
            }
            for (auto to : cur) ta[to] = 0;
            ta &= G[x];
            int L = 1, R = 64, p = 0;
            while (L <= R) {
                int mid = (L + R) >> 1;
                if ((B[mid] & ta).any()) p = mid, L = mid + 1;
                else R = mid - 1;
            }
            if (p) {
                per (i, min(n, p * S), (p - 1) * S + 1) {
                    if (ta[bb[i]]) { ans = i; break; }
                }
            }
            for (auto to : cur) if (G[x][to] && a[to] >= l && a[to] <= r)
                ans = max(ans, b[to]);
            cout << ans << '\n';
        }
    }
}
signed main() {
    // freopen("recall.in", "r", stdin);
    // freopen("recall.out", "w", stdout);
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    int __ = read(), _ = read();
    while (_--) sol();
    return 0;
}
```
:::

已经没有写题解的耐心了。
