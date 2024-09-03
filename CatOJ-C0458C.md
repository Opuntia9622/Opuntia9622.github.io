---
title: 【题解】CatOJ C0458C baka's trick
date: 2024-09-03
---

标题 trick 的名字我也不知道是什么，就这样吧。

**upd on 2024.7.18：该题 trick 名字为 baka's trick。**

首先有显然的 dp 式子：$f(i)=\min \{f(j) \times \max\{a_{j+1},\dots,a_i\}\}$。考虑怎么去优化它。

有显然的 $\mathcal O(n\log n)$：考虑线段树优化 dp。用增的单调栈维护 $a$，若每次弹出顶部一个下标 $p$，则 $[p+1,i]$ 的 $\max$ 都被推平成 $a_i$，栈维护一下 $\max$ 连续段，于是问题变成区间加。分析一下连续段个数是 $\mathcal O(n)$ 的。

<details><summary>Code（这份代码 5e5 的包 WA 了，但大致思路可以参考一下）</summary>

```cpp

#include<bits/stdc++.h>
using namespace std;
#define fi first
#define se second
#define pb emplace_back
#define rep(_, __, ___) for (int _ = (__); _ <= (___); _++)
#define per(_, __, ___) for (int _ = (__); _ >= (___); _--)
// #define int long long
using ll = long long; using pii = pair<int, int>;
inline int read() {
    char ch = getchar(); int s = 0, f = 1;
    while (!isdigit(ch)) f = (ch == '-' ? -1 : 1), ch = getchar();
    while (isdigit(ch)) s = (s << 1) + (s << 3) + (ch ^ 48), ch = getchar();
    return s * f;
}
constexpr int N = 5e5 + 5, mod = 1e9 + 7;
int n, k, a[N], s1[N], t1, t2; ll pw[N]; tuple<int, int, int> s2[N];
#define ls id << 1
#define rs id << 1 | 1
ll mn[N << 2], tag[N << 2];
inline void maketag(int id, int x) {
    tag[id] += x, mn[id] += x;
}
void upd(int a, int b, int x, int id = 1, int l = 1, int r = n) {
    if (a <= l && b >= r) return maketag(id, x);
    int mid = (l + r) >> 1; maketag(ls, tag[id]), maketag(rs, tag[id]), tag[id] = 0;
    if (a <= mid) upd(a, b, x, ls, l, mid);
    if (b > mid) upd(a, b, x, rs, mid + 1, r);
    mn[id] = min(mn[ls], mn[rs]);
}   
ll qry(int a, int b, int id = 1, int l = 1, int r = n) {
    if (a <= l && b >= r) return mn[id];
    int mid = (l + r) >> 1; ll res = 1e18; maketag(ls, tag[id]), maketag(rs, tag[id]), tag[id] = 0;
    if (a <= mid) res = min(res, qry(a, b, ls, l, mid));
    if (b > mid) res = min(res, qry(a, b, rs, mid + 1, r));
    return res;
}
signed main() {    
//    freopen("knight.in", "r", stdin);
//    freopen("knight.out", "w", stdout);
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    n = read(), k = read();
    pw[0] = 1; rep (i, 1, n) a[i] = read(), pw[i] = pw[i - 1] * 23 % mod;
    ll ans = 0, dp;
    rep (i, 1, n) {
        while (t1 && a[s1[t1]] < a[i]) t1--;
        int x = s1[t1] + 1; s2[t2 + 1] = {get<1>(s2[t2]) + 1, i, 0}, t2++;
        while (t2 && get<0>(s2[t2]) >= x) {
            upd(get<0>(s2[t2]), get<1>(s2[t2]), a[i] - get<2>(s2[t2]));
            t2--;
        } 
        s2[++t2] = {x, i, a[i]}, s1[++t1] = i; dp = qry(max(i - k + 1, 1), i);
        if (i < n) upd(i + 1, i + 1, dp);
        ans += pw[n - i] * (dp % mod) % mod;
    }
    cout << ans % mod;
    return 0;
}

```

</details>

进阶一下，上面的线段树优化 dp 可以每次修改的都是一个后缀，可以用可删堆维护 $\min$，定期弹出过期元素即可。

~~于是我们得到了乱搞做法：把上面的可删堆换成压位 Trie，时间复杂度是 $\mathcal O(n\log V/\log w)$ 的。~~

观察复杂度瓶颈在于可删堆的求 $\min$。注意到可删堆内的元素其实是在一段窗口里，所以有人类智慧的思考：考虑基于一个点 $p$，每次重构时预处理出它到两边的前缀 / 后缀 $\min$，每次查询的时候是可以将两段拼起来。当窗口内不包含 $p$ 时，就取 $p$ 为当前窗口中点重构即可。

分析一下复杂度：如果右端点不扩，势能是不断减小的，即 $\mathcal O(len+\dfrac{len}{2}+\dfrac{len}{4}+...)=\mathcal O(len)$，由于右端点往右扩是均摊 $\mathcal O(n)$ 的，所以复杂度即为 $\mathcal O(\sum len)=\mathcal O(n)$。

<details><summary>Code</summary>

```cpp

#include<bits/stdc++.h>
using namespace std;
#define fi first
#define se second
#define pb emplace_back
#define rep(_, __, ___) for (int _ = (__); _ <= (___); _++)
#define per(_, __, ___) for (int _ = (__); _ >= (___); _--)
#define int long long
using ll = long long; using pii = pair<int, int>;
inline int read() {
    char ch = getchar(); int s = 0, f = 1;
    while (!isdigit(ch)) f = (ch == '-' ? -1 : 1), ch = getchar();
    while (isdigit(ch)) s = (s << 1) + (s << 3) + (ch ^ 48), ch = getchar();
    return s * f;
}
constexpr int N = 1e7 + 5, mod = 1e9 + 7;
int n, k, st = 1, ed, mid = 1, dp[N], mn[N], pw[N];
struct node {signed l, r; int tag;} s[N];
inline int get(int i) {
    return s[i].tag + dp[s[i].l - 1];
}
inline void rebuild() {
    if (st > ed) return (void)(mid = st); mid = (st + ed) >> 1;
    rep (i, st, ed) mn[i] = get(i);
    per (i, mid - 1, st) mn[i] = min(mn[i], mn[i + 1]);
    rep (i, mid + 2, ed) mn[i] = min(mn[i], mn[i - 1]); 
}
signed main() {    
    // freopen("knight.in", "r", stdin);
    // freopen("knight.out", "w", stdout);
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    n = read(), k = read(); int ans = 0;
    pw[0] = 1; rep (i, 1, n) pw[i] = pw[i - 1] * 23 % mod;
    rep (i, 1, n) {
        int x = read();
        while (st <= ed && s[st].l < i - k + 1) {
            if (s[st].r < i - k + 1) {
                if (++st > mid) rebuild();
            }
            else {
                s[st].l = i - k + 1; mn[st] = get(st);
                if (st < mid) mn[st] = min(mn[st], mn[st + 1]);
            }
        }
        while (st <= ed && s[ed].tag <= x) if (--ed <= mid) rebuild();
        if (st > ed) s[++ed] = (node){max(1ll, i - k + 1), i, x};
        else s[ed + 1] = (node){s[ed].r + 1, i, x}, ed++;
        mn[ed] = get(ed); if (ed - 1 > mid) mn[ed] = min(mn[ed], mn[ed - 1]);
        dp[i] = min(mn[ed], mn[st]); ans += pw[n - i] * (dp[i] % mod) % mod;
    }
    cout << ans % mod;
    return 0;
}
/*
3 4 4 8 10 12
*/

```

</details>
