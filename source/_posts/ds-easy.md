---
title: ds 小练
date: 2025-6-14
categories: 

---

搬自我的 cnblogs 上一篇设密码的文章。

选了一些比较典的题。

其实这个题单不如叫多维数据结构习题。

## [A](https://www.luogu.com.cn/problem/P2305)

题意：给一棵以 $1$ 为根的有根树，每条边有长度，每个点有 $p(x),q(x),l(x)$ 三个权值。定义 $\mathrm{dis}(x,y)$ 表示表示树上 $x$ 到 $y$ 的距离。从 $x$ 可以到 $y$ 当且仅当 $\mathrm{dis}(x,y)\leq l(x)$ 且 $y$ 是 $x$ 的祖先，花费为 $\mathrm{dis}(x,y)\times p(x)+q(x)$。求从每一个点出发，到达根的最小费用。$n\leq 2\times 10^5,3\mathrm s,512\mathrm{MB}$。

sol：设 $d(i)$ 为到达根的距离，容易写出 dp 式：$f(i)=\min(f(j)+(d(i)-d(j))p(i)+q(i))$。变形成 $f(i)-d(i)p(i)-q(i)=\min(f(j)-d(j)p(i))$，可以斜率优化，不妨用李超树维护。对于当前节点 $u$，设 $v$ 为满足 $\mathrm{dis}(u,v)\leq l(u)$ 的最远祖先，我们需要一棵维护 $u$ 到 $v$ 链上的所有点对应的线段的李超树。由于我们只需要进行一次 dfs 来处理 dp 值，可以直接利用树的出栈序来转化成区间询问，于是可以直接上线段树套李超树。时间复杂度 $O(n\log^2 n)$，空间复杂度 $O(n\log n)$。

<details> <summary> Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;
#define rep(i, x, y) for (int i = (x); i <= (y); i++)
#define per(i, x, y) for (int i = (x); i >= (y); i--) 
#define int long long
using ll = long long; using ull = unsigned long long;
inline int read() {
    char ch = getchar(); int s = 0, f = 1;
    while (!isdigit(ch)) {if (ch == '-') f = -1; ch = getchar();}
    while (isdigit(ch)) s = (s << 1) + (s << 3) + (ch ^ 48), ch = getchar();
    return s * f;
}
constexpr int N = 2e5 + 5, V = 1e6;
int n, cnt, dp[N], p[N], q[N], l[N], dfn[N], d[N];
struct edge {int v, w;}; vector<edge> e[N];
vector<int> pad;
int tot, rt[N << 2];
struct seg {int k, b;}; seg lct[N << 5]; int ls[N << 5], rs[N << 5];
inline int calc(seg x, int p) {
    return x.k * p + x.b;
} 
void ins(seg x, int &id, int l = 0, int r = V) {
    if (!id) return (void)(lct[id = ++tot] = x);
    int mid = (l + r) >> 1; if (calc(x, mid) < calc(lct[id], mid)) swap(x, lct[id]);
    if (calc(x, l) < calc(lct[id], l)) ins(x, ls[id], l, mid);
    else if (calc(x, r) < calc(lct[id], r)) ins(x, rs[id], mid + 1, r);
} int rslct = 1e18;
void ask(int p, int id, int l = 0, int r = V) {
    if (!id) return ; rslct = min(rslct, calc(lct[id], p));
    int mid = (l + r) >> 1; p <= mid ? ask(p, ls[id], l, mid) : ask(p, rs[id], mid + 1, r);
}
void upd(int p, seg x, int id = 1, int l = 1, int r = n) {
    ins(x, rt[id]); if (l == r) return ;
    int mid = (l + r) >> 1; p <= mid ? upd(p, x, id << 1, l, mid) : upd(p, x, id << 1 | 1, mid + 1, r); 
}
int qry(int ql, int qr, int p, int id = 1, int l = 1, int r = n) {
    if (ql > qr) return 1e18;
    if (ql <= l && qr >= r) return rslct = 1e18, ask(p, rt[id]), rslct;
    int mid = (l + r) >> 1, res = 1e18; 
    if (ql <= mid) res = qry(ql, qr, p, id << 1, l, mid); if (qr > mid) res = min(res, qry(ql, qr, p, id << 1 | 1, mid + 1, r)); 
    return res;
}
void dfs1(int u) {
    for (auto [v, w] : e[u]) d[v] = d[u] + w, dfs1(v);
    dfn[u] = ++cnt;
}
inline int lower(int x) {
    int L = 0, R = (int)pad.size() - 1, res = 0;
    while (L <= R) {int mid = (L + R) >> 1; if (d[pad[mid]] >= x) R = mid - 1, res = pad[mid]; else L = mid + 1;} return res;
}
void dfs(int u) {
    for (auto [v, w] : e[u]) {
        int L = dfn[v], R = dfn[lower(d[v] - l[v])];
        dp[v] = qry(L, R, p[v]) + d[v] * p[v] + q[v];
        upd(L, {-d[v], dp[v]});
        pad.push_back(v); dfs(v); pad.pop_back();
    }
}
signed main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    n = read(); int kowen = read();
    rep (v, 2, n) {
        int u = read(), w = read();
        p[v] = read(), q[v] = read(), l[v] = read();
        e[u].push_back({v, w});
    } dfs1(1); pad.push_back(1), upd(dfn[1], {0, 0}); dfs(1);
    rep (i, 2, n) cout << dp[i] << '\n';
    return 0;
}
```

</details>

## [D](https://www.luogu.com.cn/problem/P5471)

题意：二维平面上 $n$ 个点，$m$ 次操作，每次选一个点 $u$ 向一个矩形连边。求 $1$ 号点到其他点的最短路。$n\leq 7\times 10^4,m\leq 1.5\times 10^5,2\mathrm s,128\mathrm{MB}$。

sol：空间有限制，可以尝试线段树套 Treap 优化建图，或者写 K-D Tree 优化建图，有更为优秀的 $O(n)$ 空间。注意这里直接连边代价太大，可以边松弛点边查询区间。

## [E](https://codeforces.com/problemset/problem/543/E)

题意：给你一个长度为 $n$ 的序列 $a$，和一个常数 $m$，定义一个函数 $f(l,x)$ 为 $[l,l+m)$ 中小于 $x$ 的数的个数，有 $q$ 个询问，每次给定 $l,r,x$ 查询 $\min_{i=l}^rf(i,x)$。强制在线。$n,q\leq 2\times 10^5,7\mathrm s,64\mathrm{MB}$。 

sol：考虑 $a_i<x$ 时的贡献，会对左端点区间 $(i-m,i]$ 整体 $+1$，也就是说我们要做的就是进行一车区间加操作，然后求 $[l,r]$ 的区间 $\min$。不妨对序列从小到大排序，每次询问二分找到一个最远的 $p$ 使得 $a_p<x$，然后执行 $[1,p]$ 的区间加操作，查询区间 $\min$。

使用位域主席树即可将空间卡过，时空复杂度均为 $O(n\log n)$。std 做法为操作分块再套一层序列分块，也是解决这类问题的一个经典做法。（如 [P10081](https://www.luogu.com.cn/problem/P10081)）

<details> <summary> Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;
#define rep(i, x, y) for (int i = (x); i <= (y); i++)
#define per(i, x, y) for (int i = (x); i >= (y); i--) 
// #define int long long
using ll = long long; using ull = unsigned long long;
inline int read() {
    char ch = getchar(); int s = 0, f = 1;
    while (!isdigit(ch)) {if (ch == '-') f = -1; ch = getchar();}
    while (isdigit(ch)) s = (s << 1) + (s << 3) + (ch ^ 48), ch = getchar();
    return s * f;
}
constexpr int N = 2e5 + 5;
int n, m, Q, ans, tot, rt[N]; array<int, 2> a[N];
struct segt { unsigned long long mx : 18, lsn : 23, rsn : 23; } t[N * 40];
#define ls t[id].lsn
#define rs t[id].rsn
#define lmx (l == mid ? ls : t[ls].mx)
#define rmx (r == mid + 1 ? rs : t[rs].mx)
int upd(int o, int ql, int qr, int l = 1, int r = n - m + 1) {
    if (l == r) return o + 1;
    int id = ++tot; t[id] = t[o];
    if (ql <= l && qr >= r) return ++t[id].mx, id;
    int mid = (l + r) >> 1, tag = t[id].mx - max(lmx, rmx);
    if (ql <= mid) ls = upd(t[o].lsn, ql, qr, l, mid);
    if (qr > mid) rs = upd(t[o].rsn, ql, qr, mid + 1, r);
    return t[id].mx = max(lmx, rmx) + tag, id; 
}
int qry(int id, int ql, int qr, int l = 1, int r = n - m + 1) {
    if (l == r) return id;
    if (ql <= l && qr >= r) return t[id].mx;
    int mid = (l + r) >> 1, res = 0, tag = t[id].mx - max(lmx, rmx);
    if (ql <= mid) res = qry(ls, ql, qr, l, mid);
    if (qr > mid) res = max(res, qry(rs, ql, qr, mid + 1, r));
    return res + tag; 
}
signed main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    n = read(), m = read();
    rep (i, 1, n) a[i] = {-read(), i};
    sort(a + 1, a + n + 1);
    rep (i, 1, n) {
        int R = a[i][1], L = max(R - m + 1, 1);
        rt[i] = upd(rt[i - 1], L, R); 
    } Q = read();
    while (Q--) {
        int l = read(), r = read(), x = read() ^ ans, p;
        p = lower_bound(a + 1, a + n + 1, (array<int, 2>){-x, (int)1e9}) - a - 1;
        cout << (ans = m - qry(rt[p], l, r)) << '\n';
    }
    return 0;
}
```

</details>

## [F](https://codeforces.com/problemset/problem/603/E)

题意：给定一张 $n$ 个点的无向图，初始没有边。依次加入 $m$ 条带权的边，每次加入后询问是否存在一个边集，满足 $n$ 个点的度数均为奇数。若存在，则还需要最小化边集中的最大边权。$n \le 10^5,m \le 3 \times 10^5,4\mathrm s,256\mathrm{MB}$。

sol：手玩几张图过后应该能够发现一个强力的性质：$n$ 个点的度数均为奇数当且仅当图中只存在大小为偶数的连通块，充要性证明略。于是我们要维护一棵动态加边的最小瓶颈生成树。可以线段树分治处理。复杂度 $\text{2log}$，详见代码。

<details> <summary> Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;
#define rep(i, x, y) for (int i = (x); i <= (y); i++)
#define per(i, x, y) for (int i = (x); i >= (y); i--) 
// #define int long long
using ll = long long; using ull = unsigned long long;
inline int read() {
    char ch = getchar(); int s = 0, f = 1;
    while (!isdigit(ch)) {if (ch == '-') f = -1; ch = getchar();}
    while (isdigit(ch)) s = (s << 1) + (s << 3) + (ch ^ 48), ch = getchar();
    return s * f;
}
constexpr int N = 3e5 + 5, NN = 1e5 + 5;
int n, m, top, p, odd, f[NN], sz[NN], ans[N];
struct edge {int u, v, w, id;} e[N]; vector<int> v[N << 2];
struct node {int x, y, v;} st[N];
int find(int x) {return f[x] == x ? x : find(f[x]);}
inline void mg(int x, int y) {
    x = find(x), y = find(y);
    if (x == y) return ; if (sz[x] < sz[y]) swap(x, y);
    st[++top] = {x, y, 0};
    if ((sz[x] & 1) && (sz[y] & 1)) odd -= 2, st[top].v += 2;
    f[y] = x, sz[x] += sz[y];
}
#define ls id << 1
#define rs id << 1 | 1
void upd(int ql, int qr, int x, int id, int l, int r) {
    if (ql <= l && qr >= r) return (void)(v[id].push_back(x));
    int M = (l + r) >> 1; if (ql <= M) upd(ql, qr, x, ls, l, M);
    if (qr > M) upd(ql, qr, x, rs, M + 1, r);
}
void dfs(int id, int l, int r) {
    int pt = top; 
    for (auto to : v[id]) mg(e[to].u, e[to].v);
    int M = (l + r) >> 1; if (l == r) {
        while (odd && p < m) if (e[++p].id <= l) {
            mg(e[p].u, e[p].v);
            if (e[p].id < l) upd(e[p].id, l - 1, p, 1, 1, m);
        }
        if (!odd) ans[l] = e[p].w; else ans[l] = -1;
    } else dfs(rs, M + 1, r), dfs(ls, l, M);
    while (top ^ pt) {
        int x = st[top].x, y = st[top].y;
        f[y] = y, sz[x] -= sz[y], odd += st[top--].v; 
    }
}
#undef ls
#undef rs 
signed main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    odd = n = read(), m = read();
    rep (i, 1, n) f[i] = i, sz[i] = 1;
    rep (i, 1, m) {
        e[i].u = read(), e[i].v = read(), e[i].w = read(), e[i].id = i;
    } sort(e + 1, e + m + 1, [](edge x, edge y) {return x.w < y.w;}); 
    dfs(1, 1, m); rep (i, 1, m) cout << ans[i] << '\n';
    return 0;
}
```

</details>

## [I](https://www.luogu.com.cn/problem/P5445)

题意：给一条 $n$ 个点的链，每条边上有布尔值 $0/1$ 表示当前时刻该边可通过/不可通过。$q$ 次操作，操作有两种：

- 切换第 $p$ 条边的状态。

- 从 $0$ 时刻到当前时刻，$a$ 到 $b$ 相通的时刻数。

$n,q\leq 3\times 10^5,5\mathrm s,512\mathrm {MB}$。

sol：由题面可以看出这是一个 二维数点 + 历史版本和 的问题。不妨用树套树维护一个矩阵 $G$，其中 $G(i,j)$ 表示**在假设后续没有修改操作的情况下** $i$ 点到 $j$ 点相通的时刻数。每次对一个单点修改影响到的为 $G(i,j)$ 里的两个矩形。二维差分后问题就变成单点修改矩形查询，这是树套树擅长的。时空复杂度 $O(n\log^2 n)$，但是开 $O(n\log n)$ 大小过了。。。

<details> <summary> 22 年的码风，鉴赏一下 </summary>

```cpp
#include<bits/stdc++.h>
using namespace std;
//#define int long long
#define mk make_pair
typedef pair<int, int> pii;
namespace IO
{
    inline int read()
    {
        int x = 0, f = 1;
        char ch = getchar();
        while(!isdigit(ch)) {if(ch == '-') f = -1; ch = getchar();}
        while(isdigit(ch)) x = (x << 3) + (x << 1) + ch - '0', ch = getchar();
        return x * f;
    }
    template <typename T> inline void print(T x)
    {
        if(x < 0) putchar('-'), x = -x;
        if(x > 9) print(x / 10);
        putchar(x % 10 + '0');
    }
}
using namespace IO;
const int N = 3e5 + 5;
int n, q, tot, rt[N];
char c[N];
set<pii> s;
#define ls t[id].lon
#define rs t[id].ron
#define fi first
#define se second
struct segt {int sum, lon, ron;}t[N * 80];
void pushup(int id) {t[id].sum = t[ls].sum + t[rs].sum;}
void update(int &id, int l, int r, int p, int x)
{
    if (!id) id = ++tot;
    if (l == r) {t[id].sum += x; return ;}
    int mid = (l + r) >> 1;
    if (p <= mid) update(ls, l, mid, p, x);
    else update(rs, mid + 1, r, p, x);
    pushup(id);
}
int query(int id, int l, int r, int a, int b)
{
    if (!id) return 0;
    if (a <= l && b >= r) return t[id].sum;
    int mid = (l + r) >> 1, res = 0;
    if (a <= mid) res += query(ls, l, mid, a, b);
    if (b > mid) res += query(rs, mid + 1, r, a, b);
    return res;
}
int lowbit(int x) {return x & -x;}
void add(int x, int y, int k) 
{
    for (int i = x; i <= n + 1; i += lowbit(i)) 
        update(rt[i], 1, n + 1, y, k);
}
int ask(int x, int y)
{
    int res = 0;
    for (int i = x; i; i -= lowbit(i)) 
        res += query(rt[i], 1, n + 1, 1, y);
    return res;
}
pii get(int x)
{
    auto to = s.upper_bound(mk(x, n + 1));
    return *(--to);
}
signed main()
{
    n = read(), q = read();
    int lst = 1;
    for (int i = 1; i <= n; i++)
    {
        cin >> c[i];
        if (c[i] == '0') 
            s.insert(mk(lst, i)), lst = i + 1;
    }
    s.insert(mk(lst, n + 1));
    for (auto to : s)
    {
        int l = to.fi, r = to.se;
        add(l, l, q);
        if (r <= n) add(r + 1, r + 1, q), add(r + 1, l, -q), add(l, r + 1, -q);
    }
    for (int i = 1; i <= q; i++)
    {
        string tmp;
        cin >> tmp;
        if (tmp[0] == 't')
        {
            int x = read();
            int l = get(x).fi, r = get(x + 1).se;
            if (c[x] == '0')
            {
                add(x + 1, x + 1, i - q), add(l, x + 1, q - i);
                if (r <= n) add(x + 1, r + 1, q - i), add(l, r + 1, i - q);
                s.erase(mk(l, x)), s.erase(mk(x + 1, r));
                s.insert(mk(l, r));
            }
            else
            {
                add(x + 1, x + 1, q - i), add(l, x + 1, i - q);            
                if (r <= n) add(x + 1, r + 1, i - q), add(l, r + 1, q - i);
                s.insert(mk(l, x)), s.insert(mk(x + 1, r));
                s.erase(mk(l, r));
            }
            c[x] = 97 - c[x];
        }
        else 
        {
            int x = read(), y = read();
            pii a = get(x), b = get(y);
            print(ask(x, y) - (a == b) * (q - i)), puts("");
        }
    }
    return 0;
} 
```

</details>

## [J](https://www.luogu.com.cn/problem/P3688)

题意：[题面有图](https://www.luogu.com.cn/problem/P3688)。

sol：

tag：树套树、矩阵。

其实有些题解是过不了 hack 的，这里给一个正常的线段树套线段树做法。

手摸一下不难发现题面的错误代码中的 $\mathrm{find}(x)$ 是在求 $x$ 的后缀和。所以 $\mathrm{query}(x+1,y)$ 实际上返回的值为 $[x,y-1]$ 的和。转化一下，我们要回答 $a_{x}=a_y$ 的概率。

考虑递推：设 $f(i,0/1)$ 表示当前推到第 $i$ 个操作，$a_x$ 等于/不等于 $a_y$ 的概率。我们可以把它们放进一个 $1\times 2$ 的矩阵里：$[f(i,0),f(i,1)]$，然后分类讨论一下转移矩阵怎么构造。下面设 $p=\dfrac{1}{r-l+1}$。

$$\begin{cases} 
\begin{bmatrix} 1-p & p \\ p & 1-p \end{bmatrix}\quad(x<l\leq y\leq r)\\
\begin{bmatrix} 1-2p & 2p \\ 2p & 1-2p \end{bmatrix}\quad(l\leq x\leq y\leq r)\\
\begin{bmatrix} 1-p & p \\ p & 1-p \end{bmatrix}\quad(l\leq x\leq r<y)\\
\end{cases}$$

不同于其他矩阵，此矩阵满足交换律。于是我们用线段树套线段树维护矩阵，支持矩形乘和单点查，标记永久化即可。时空复杂度均为 $O(n\log^2 n)$。

由于带了个 $2^3$ 的常数，交上去会被 hack 数据卡成 MLE。但是我们维护的矩阵是对称的且两个不同的元素的和为 $1$。所以实际需要维护的元素只有一个。稍微改改就过了。

同时这道题有个坑点：$x=0$ 时 $\mathrm{find}(x)$ 会直接返回 $0$。我们还需要新开一棵线段树维护每个点的前缀与后缀相等概率。

<details> <summary> Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;
#define rep(i, x, y) for (int i = (x); i <= (y); i++)
#define per(i, x, y) for (int i = (x); i >= (y); i--) 
// #define int long long
using ll = long long; using ull = unsigned long long;
inline int read() {
    char ch = getchar(); int s = 0, f = 1;
    while (!isdigit(ch)) {if (ch == '-') f = -1; ch = getchar();}
    while (isdigit(ch)) s = (s << 1) + (s << 3) + (ch ^ 48), ch = getchar();
    return s * f;
}
constexpr int N = 1e5 + 5, mod = 998244353;

#define debug(x) cerr << x.c[0][0] << ' ' << x.c[0][1] << ' ' << x.c[1][0] << ' ' << x.c[1][1] << '\n'

int n, Q;

inline ll qpow(ll x, int y) {
    ll res = 1;
    while (y) {
        if (y & 1) (res *= x) %= mod;
        (x *= x) %= mod, y >>= 1;  
    } return res;
}

struct matrix {
    int c;
    friend matrix operator * (const matrix &x, const matrix &y) {
        return {(1ll * x.c * y.c + 1ll * (1 - x.c) * (1 - y.c)) % mod};
    } 
} base = {1};

namespace tbt {
    #define ls t[id].lsn
    #define rs t[id].rsn
    int tot, rt[N << 2];
    struct segt {matrix x; int lsn, rsn;} t[N * 400];
    matrix qry(int id, int p, int l = 1, int r = n) {
        if (!id) return base; if (l == r) return t[id].x;
        int mid = (l + r) >> 1; return t[id].x * (p <= mid ? qry(ls, p, l, mid) : qry(rs, p, mid + 1, r));
    }
    void upd(int &id, int ql, int qr, matrix x, int l = 1, int r = n) {
        if (!id) t[id = ++tot] = {base, 0, 0};
        if (ql <= l && qr >= r) return (void)(t[id].x = t[id].x * x);
        int mid = (l + r) >> 1; if (ql <= mid) upd(ls, ql, qr, x, l, mid);
        if (qr > mid) upd(rs, ql, qr, x, mid + 1, r);
    }
    void add(int sx, int sy, int ex, int ey, matrix x, int id = 1, int l = 1, int r = n) {
        if (sx <= l && ex >= r) return upd(rt[id], sy, ey, x);
        int mid = (l + r) >> 1; if (sx <= mid) add(sx, sy, ex, ey, x, id << 1, l, mid);
        if (ex > mid) add(sx, sy, ex, ey, x, id << 1 | 1, mid + 1, r);
    }
    matrix ask(int x, int y, int id = 1, int l = 1, int r = n) {
        if (l == r) return qry(rt[id], y); int mid = (l + r) >> 1;
        return qry(rt[id], y) * (x <= mid ? ask(x, y, id << 1, l, mid) : ask(x, y, id << 1 | 1, mid + 1, r)); 
    }
}

namespace sgt {
    #define ls id << 1
    #define rs id << 1 | 1
    matrix t[N << 2];
    matrix qry(int p, int id = 1, int l = 1, int r = n) {
        if (l == r) return t[id]; int mid = (l + r) >> 1;
        return t[id] * (p <= mid ? qry(p, ls, l, mid) : qry(p, rs, mid + 1, r));
    }
    void upd(int ql, int qr, matrix x, int id = 1, int l = 1, int r = n) {
        if (ql <= l && qr >= r) return (void)(t[id] = t[id] * x);
        int mid = (l + r) >> 1; if (ql <= mid) upd(ql, qr, x, ls, l, mid);
        if (qr > mid) upd(ql, qr, x, rs, mid + 1, r);
    }
}

inline void add(int sx, int sy, int ex, int ey, const matrix &x) {
    if (sx > ex || sy > ey) return ; tbt::add(sx, sy, ex, ey, x);
}

signed main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    n = read(), Q = read(); tbt::t[0].x = base; fill(sgt::t, sgt::t + (N << 2), base);
    while (Q--) { 
        int op = read(), l = read(), r = read();
        if (op == 1) {
            int p = qpow(r - l + 1, mod - 2);
            add(1, l, l - 1, r, {1 - p});
            add(l, l, r, r, {1 - 2 * p});
            add(l, r + 1, r, n, {1 - p});
            if (l > 1) sgt::upd(1, l - 1, {0});
            if (r < n) sgt::upd(r + 1, n, {0});
            sgt::upd(l, r, {p}); 
        }
        else {
            if (l != 1) cout << (tbt::ask(l - 1, r).c + mod) % mod << '\n';
            else cout << (sgt::qry(r).c + mod) % mod << '\n';
        }
    }
    return 0;
}
```

</details>

## [L](https://codeforces.com/problemset/problem/1550/F)

题意：有 $n$ 个点 $a_1<a_2<\cdots<a_n$。初始给定 $s,d$，你初始站在 $a_s$ 上，步长为 $d$。$q$ 次询问，每次询问给 $e,k$，当 $k-d\le |a_i-a_j|\le k+d$ 时可以从 $a_i$ 跳到 $a_j$，问是否可以从 $s$ 跳到 $e$。$n,q\le 2\times 10^5,V\le 10^6,5\mathrm s,256\mathrm{MB}$。

sol：首先这个 $k$ 是满足单调性的，即 $a_s\to a_e$ 在 $k$ 时满足，在 $k+1$ 时也满足。可以对于每个 $e$ 预处理满足条件的最小 $k$。具体思路：实际上是跑一个原点为 $s$ 的 dijkstra 最短路，原式转一下就是 $|d-|a_i-a_j||\le k$，类似“隧道”一题，这个式子可以分成四条斜线放进李超树维护全局 $\min$，点松弛完了可以直接在李超树里删掉。注意到所有斜线的斜率绝对值都为 $1$，可以用正常线段树维护。复杂度 $O(V\log V)$。

<details> <summary> Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;
#define rep(i, x, y) for (int i = (x); i <= (y); i++)
#define per(i, x, y) for (int i = (x); i >= (y); i--) 
// #define int long long
using ll = long long; using ull = unsigned long long;
inline int read() {
    char ch = getchar(); int s = 0, f = 1;
    while (!isdigit(ch)) {if (ch == '-') f = -1; ch = getchar();}
    while (isdigit(ch)) s = (s << 1) + (s << 3) + (ch ^ 48), ch = getchar();
    return s * f;
}
constexpr int N = 2e5 + 5, V = 1e6;
int n, q, S, d, a[N], ans[V + 5];
#define ls id << 1
#define rs id << 1 | 1
struct segt { int b1, b2, mn, p, l, r; vector<bool> mp; } t[V * 4 + 5];
void add(int ql, int qr, int x, bool op, int id = 1, int l = 1, int r = V) {
    if (ql <= l && qr >= r) { x += op ? l - ql : qr - r;
        if (op) t[id].b1 = min(t[id].b1, x); else t[id].b2 = min(t[id].b2, x);
        if (t[id].l != r - l + 1) { int X = t[id].b1 + t[id].l, Y = t[id].b2 + r - l - t[id].r;
            if (t[id].mn > min(X, Y)) { if (X < Y) t[id].p = t[id].l + l, t[id].mn = X; else t[id].p = t[id].r + l, t[id].mn = Y; }
        } return ;
    } int m = (l + r) >> 1; if (ql <= m) add(ql, qr, x, op, ls, l, m); if (qr > m) add(ql, qr, x, op, rs, m + 1, r); 
    int X = min(t[id].mn, min(t[ls].mn, t[rs].mn)); if (t[ls].mn == X) t[id].mn = X, t[id].p = t[ls].p; if (t[rs].mn == X) t[id].mn = X, t[id].p = t[rs].p; 
}
void ins(int p, int id = 1, int l = 1, int r = V) {
    t[id].mp[p - l] = 1; if (l == r) return ;
    int m = (l + r) >> 1; p <= m ? ins(p, ls, l, m) : ins(p, rs, m + 1, r);
}
void upd(segt &x, int l, int r) {
    if (x.l == r - l + 1) return ;
    while (x.l < r - l + 1 && !x.mp[x.l]) x.l++;
    while (x.r >= 0 && !x.mp[x.r]) x.r--;
}
void ers(int p, int id = 1, int l = 1, int r = V) {
    t[id].mp[p - l] = 0, upd(t[id], l, r), t[id].mn = 1e9, t[id].p = 0; if (l == r) return ;
    if (t[id].l != r - l + 1) { int X = t[id].b1 + t[id].l, Y = t[id].b2 + r - l - t[id].r;
        if (X < Y) t[id].p = t[id].l + l, t[id].mn = X; else t[id].p = t[id].r + l, t[id].mn = Y;
    } int m = (l + r) >> 1; p <= m ? ers(p, ls, l, m) : ers(p, rs, m + 1, r); int X = min(t[id].mn, min(t[ls].mn, t[rs].mn));
    if (t[ls].mn == X) t[id].mn = X, t[id].p = t[ls].p; if (t[rs].mn == X) t[id].mn = X, t[id].p = t[rs].p; 
}
void bd(int id, int l, int r) { t[id].mp.resize(r - l + 1, 0);
    t[id].b1 = t[id].b2 = t[id].mn = 1e9; if (l == r) return ;
    int m = (l + r) >> 1; bd(ls, l, m), bd(rs, m + 1, r);
}
void bd1(int id, int l, int r) { 
    t[id].l = 0, t[id].r = r - l, upd(t[id], l, r); if (l == r) return ;
    int m = (l + r) >> 1; bd1(ls, l, m), bd1(rs, m + 1, r);
}
#undef ls
#undef rs
signed main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    n = read(), q = read(), S = read(), d = read(); bd(1, 1, V);
    rep (i, 1, n) a[i] = read(), ins(a[i]); bd1(1, 1, V);
    int K = 0, u = a[S]; ers(a[S]);
    rep (i, 1, n - 1) {
        if (u > 1) add(max(u - d, 1), u - 1, u - d < 1 ? d - u + 1 : 0, 1);
        if (u < V) add(u + 1, min(u + d, V), u + d > V ? u + d - V : 0, 0);
        if (u - d - 1 > 0) add(1, u - d - 1, 1, 0);
        if (u + d + 1 <= V) add(u + d + 1, V, 1, 1);
        ans[t[1].p] = (K = max(K, t[1].mn)); u = t[1].p; ers(t[1].p);
    } 
    while (q--) {
        int i = read(), k = read();
        if (ans[a[i]] <= k) cout << "Yes\n";
        else cout << "No\n";
    }
    return 0;
}
```

</details>

## [P](https://loj.ac/p/2838)

题意：给你一张 $n$ 个点 $m$ 条边的拓扑图，$Q$ 次询问，每次给出一个集合 $S$ 和一个点 $T$，问最长的以不在 $S$ 内的点开头、以 $T$ 结尾的链长度。$n,Q,\sum|S|\le 10^5, m\le 2\times 10^5,2\mathrm s,512\mathrm{MB}$。

sol：注意到 $|S|$ 总和一定，考虑对 $|S|$ 根号分治。对于 $|S|<B$，对每个点预处理出距离其长度前 $B$ 长的结点，合并信息的时候归并一下就可以做到不带 $\log$。对于 $|S|\ge B$，总询问次数不会超过 $Q/B$ 次，直接暴力即可。取 $B=\sqrt n$，时空复杂度均为 $O(n\sqrt n)$。笔者写了一坨 $O(n\sqrt n\log n)$ 的答辩，飘过去了。

<details> <summary> Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;
#define rep(i, x, y) for (int i = (x); i <= (y); i++)
#define per(i, x, y) for (int i = (x); i >= (y); i--) 
// #define int long long
using ll = long long; using ull = unsigned long long;
inline int read() {
    char ch = getchar(); int s = 0, f = 1;
    while (!isdigit(ch)) {if (ch == '-') f = -1; ch = getchar();}
    while (isdigit(ch)) s = (s << 1) + (s << 3) + (ch ^ 48), ch = getchar();
    return s * f;
}
using aii = array<int, 2>;
constexpr int N = 1e5 + 5, B = 316;
int n, m, Q, f[N], ss[N]; vector<int> e1[N], e2[N], p, buc[N]; 
aii dp[N][B + 5]; bool mp[N]; 
signed main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    n = read(), m = read(), Q = read();
    rep (i, 1, m) {
        int u = read(), v = read();
        e1[v].push_back(u); e2[u].push_back(v);
    }
    rep (u, 1, n) {
        vector<aii> ve; vector<int> vv;
        for (auto v : e1[u]) {
            rep (i, 1, B) {
                if (dp[v][i] == aii{0, 0}) break; int xx;
                buc[xx = dp[v][i][0] + 1].push_back(dp[v][i][1]);
                if (!mp[xx]) mp[xx] = 1, vv.push_back(xx);
            }
        } sort(vv.begin(), vv.end(), greater<int>{});
        for (auto to : vv) {
            mp[to] = 0;
            for (auto o : buc[to]) ve.push_back({to, o}); 
            buc[to].clear();
        } vv.clear();
        ve.push_back({0, u}); int c = 0, i = 0;
        while (vv.size() < B && c < ve.size() && ve[c][1]) {
            if (mp[ve[c][1]]) {c++; continue;}
            dp[u][++i] = ve[c]; mp[ve[c][1]] = 1, vv.push_back(ve[c][1]); c++;
        }
        for (auto to : vv) mp[to] = 0;
    }
    rep (jj, 1, Q) {
        int t = read(), y = read(), ans = -1; p.resize(y + 1);
        rep (i, 1, y) p[i] = read(), mp[p[i]] = 1;
        if (y < B) {
            rep (i, 1, B) if (dp[t][i][1] && !mp[dp[t][i][1]]) {
                ans = dp[t][i][0]; break;
            }
        } else { f[t] = 0;
            per (u, t, 1) {
                if (u ^ t) f[u] = -1e9;
                for (auto v : e2[u]) {
                    if (v > t) continue;
                    f[u] = max(f[u], f[v] + 1);
                } 
                if (!mp[u]) ans = max(ans, f[u]);
            }
        }
        cout << ans << '\n';
        rep (i, 1, y) mp[p[i]] = 0;
    }
    return 0;
}
```

</details>

## [Q](https://codeforces.com/problemset/problem/833/E)

题意：[题面](https://codeforces.com/problemset/problem/833/E)。

sol：记一个 $S=\{l_1,r_1,l_2,r_2,\dots,l_n,r_n\}$，考虑对 $\forall x\in S$ 求出 $[0,x]$ 中最大能长全的 $k$。考虑设 $f(i,j,1/2)$ 表示当前删掉了第 $i$ 个区间，已经删了 $1/2$ 个区间，在 $[0,j]$ 中最大能长全的 $k$，乱做一下就搞定了。由于个数上限为 $2$，当删掉的两个区间有交的情况可以特判一下。时间复杂度 $O(n\log n)$。

<details> <summary> Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;
#define rep(i, x, y) for (int i = (x); i <= (y); i++)
#define per(i, x, y) for (int i = (x); i >= (y); i--) 
// #define int long long
using ll = long long; using ull = unsigned long long;
inline int read() {
    char ch = getchar(); int s = 0, f = 1;
    while (!isdigit(ch)) {if (ch == '-') f = -1; ch = getchar();}
    while (isdigit(ch)) s = (s << 1) + (s << 3) + (ch ^ 48), ch = getchar();
    return s * f;
}
constexpr int N = 3e5 + 5;
int n, C, tag, f[N][2], tg[N];
array<int, 2> ans[N << 1];
struct seg {int l, r, c;} a[N];
struct node {int id, op;}; map<int, vector<node>> buc;
set<int> st; map<int, int> mp[N];
inline int lower(int c) {
    int L = 1, R = n, res = 0;
    while (L <= R) {
        int mid = (L + R) >> 1;
        if (a[mid].c <= C - c) L = mid + 1, res = mid;
        else R = mid - 1;
    } return res;
}
#define ls id << 1
#define rs id << 1 | 1
int t[N << 2];
void bd(int id, int l, int r) {
    if (l == r) return (void)(t[id] = f[l][0]);
    int m = (l + r) >> 1; bd(ls, l, m), bd(rs, m + 1, r); 
    t[id] = max(t[ls], t[rs]);
}
void upd(int p, int x, int id = 1, int l = 1, int r = n) {
    if (l == r) return (void)(t[id] = x);
    int m = (l + r) >> 1; p <= m ? upd(p, x, ls, l, m) : upd(p, x, rs, m + 1, r);
    t[id] = max(t[ls], t[rs]);
}
int qry(int ql, int qr, int id = 1, int l = 1, int r = n) {
    if (ql > qr) return -1e9; if (ql <= l && qr >= r) return t[id];
    int m = (l + r) >> 1, R = -1e9; if (ql <= m) R = qry(ql, qr, ls, l, m);
    if (qr > m) R = max(R, qry(ql, qr, rs, m + 1, r)); return R; 
}
#undef ls
#undef rs
signed main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    n = read(), C = read();
    if (!n) {
        int Q = read();
        while (Q--) {
            int k = read(); cout << k << '\n';
        } return 0;
    }
    rep (i, 1, n) {
        a[i].l = read(), a[i].r = read(), a[i].c = read();
    } sort(a + 1, a + n + 1, [](seg x, seg y) {return x.c < y.c;});
    rep (i, 1, n) {
        buc[a[i].l].push_back({i, 1});
        buc[a[i].r].push_back({i, -1});
        if (a[i].c > C) f[i][0] = f[i][1] = tg[i] = -1e9;
    } int l = 0, ii = 0; bd(1, 1, n);
    for (auto [r, to] : buc) { ii++;
        ans[ii] = {ans[ii - 1][0], r};
        if (!st.size()) {
            tag += r - l; ans[ii][0] += r - l;
        } else if (st.size() == 1) {
            int i = *st.begin(); if (a[i].c <= C) { 
            f[i][0] = f[i][0] + r - l; upd(i, f[i][0]);
            f[i][1] = f[i][1] + r - l;
            int p = lower(a[i].c), mx = -1e9;
            if (i <= p) mx = max(qry(1, i - 1), qry(i + 1, p)); 
            else mx = qry(1, p); mx = max(mx, tg[i]);
            f[i][1] = max(f[i][1], mx + r - l);
            ans[ii][0] = max(ans[ii][0], max(f[i][0], f[i][1]) + tag);
        }} else if (st.size() == 2) {
            int x = *st.begin(), y = *st.rbegin();
            if (a[x].c + a[y].c <= C) {
                mp[x][y] += r - l, tg[x] = max(tg[x], mp[x][y] + f[y][0]), tg[y] = max(tg[y], mp[x][y] + f[x][0]);
                tg[x] = max(tg[x], mp[x][y] + f[x][0]), tg[y] = max(tg[y], mp[x][y] + f[y][0]);
                ans[ii][0] = max(ans[ii][0], mp[x][y] + tag);
                ans[ii][0] = max(ans[ii][0], max(tg[x], tg[y]) + tag);
            } 
        }// cerr << ii << ' ' << ans[ii][0] << ' ' << ans[ii][1] << ' ' << st.size() << endl;
        for (auto o : to) 
            if (o.op == -1) st.erase(o.id);
            else st.insert(o.id); 
        l = r; 
    } int Q = read();
    while (Q--) {
        int k = read();
        int p = lower_bound(ans + 1, ans + ii + 1, (array<int, 2>){k, 0}) - ans; 
        if (k > ans[ii][0]) cout << ans[ii][1] + (k - ans[ii][0]) << '\n';
        else cout << ans[p][1] - (ans[p][0] - k) << '\n';
    }
    return 0;
}
```

</details>

## [R](https://www.luogu.com.cn/problem/P8528)

题意：[题面](https://www.luogu.com.cn/problem/P8528)。

sol：

tag：dsu on tree、历史版本和。

考虑维护一个答案矩阵，即 $G_{i,j}=0/1$ 表示当前二元组 $(i,j)$ 是否满足条件。初始时全为 $1$，考虑怎样会变 $0$。

对于两个点 $x,y$，设他们的 LCA 为 $z$，同时令 $a_x<a_y$，分类讨论它们与 LCA 的权值大小关系：

- $a_x<a_z<a_y$，无事发生。

- $a_x<a_y<a_z$，第一维在 $[1,a_x]$，第二维在 $[a_y,a_z)$ 的点会被推平为 $0$。

- $a_z<a_x<a_y$，第一维在 $(a_z,a_x]$，第二维在 $[a_y,n]$ 的点会被推平为 $0$。

考虑 dsu on tree，假设当前钦定了 $x,z$，$a_y$ 为 $a_x$ 的前驱/后继时才能覆盖最大，所以有用的 $(x,y,z)$ 只会有 $O(n\log n)$ 个，直接 set 维护即可。

一个询问相当于是对左上角 $(L,L)$，右下角 $(R,R)$ 的矩形求 $1$ 的个数。不妨把所有推平操作和询问离线下来。将推平操作转化成矩形 $+1$，同时初始矩阵全为 $0$，询问转化为矩形求 $0$ 的个数。对第一维扫描线。询问转化为扫描到 $R$ 时的 $[L,R]$ 历史 $0$ 个数 $-$ 扫描到 $L-1$ 时的 $[L,R]$ 历史 $0$ 个数。

由于最小值不可能 $<0$，所以使用线段树维护：区间最小值、最小值个数、区间历史 $0$ 个数。时间复杂度 $O(n\log^2 n)$。

代码很好写，只改了一个地方就过了。

<details> <summary> Code </summary>

```cpp
#include <bits/stdc++.h>
using namespace std;
#define rep(i, x, y) for (int i = (x); i <= (y); i++)
#define per(i, x, y) for (int i = (x); i >= (y); i--) 
#define int long long
using ll = long long; using ull = unsigned long long;
inline int read() {
    char ch = getchar(); int s = 0, f = 1;
    while (!isdigit(ch)) {if (ch == '-') f = -1; ch = getchar();}
    while (isdigit(ch)) s = (s << 1) + (s << 3) + (ch ^ 48), ch = getchar();
    return s * f;
}
constexpr int N = 2e5 + 5;
int n, m, tot, a[N], da[N], sz[N], mxson[N], p[N], dfn[N]; vector<int> e[N], cur[N];
set<int> s[N];
struct node {int l, r, op;}; vector<node> Q[N];
void dfs1(int u) {
    sz[u] = 1, p[u] = u, dfn[u] = ++tot;
    for (auto v : e[u]) {
        dfs1(v), sz[u] += sz[v];
        if (sz[v] > sz[mxson[u]]) mxson[u] = v;
    } if (mxson[u]) p[u] = p[mxson[u]];
}
void dfs(int u, bool b) {
    for (auto v : e[u]) dfs(v, v == mxson[u]);
    int z = a[u];
    for (auto v : e[u]) if (v ^ mxson[u]) {
        int L = dfn[v], R = sz[v] + dfn[v] - 1;
        rep (i, L, R) {
            int x = da[i]; auto it = s[p[u]].lower_bound(x);
            if (it != s[p[u]].end()) {
                if (x > z) {
                    Q[z + 1].push_back({*it, n, 1});
                    Q[x + 1].push_back({*it, n, -1});
                } else if (x < z && *it < z) {
                    Q[1].push_back({*it, z - 1, 1});
                    Q[x + 1].push_back({*it, z - 1, -1});
                }
            } 
            if (it != s[p[u]].begin()) { it = prev(it);
                if (x > z && *it > z) {
                    Q[z + 1].push_back({x, n, 1});
                    Q[*it + 1].push_back({x, n, -1});
                } else if (x < z) {
                    Q[1].push_back({x, z - 1, 1});
                    Q[*it + 1].push_back({x, z - 1, -1});
                }
            }
        } rep (i, L, R) s[p[u]].insert(da[i]);
    }
    if (b) s[p[u]].insert(z);
} 
int ans[N]; 
struct Node {int l, r, op, id;}; vector<Node> q[N];
struct segt {int mn, mnct, smtg, hism, hstg;} t[N << 2];
#define ls id << 1
#define rs id << 1 | 1
inline void pushup(int id) {
    t[id].mn = min(t[ls].mn, t[rs].mn);
    if (t[ls].mn == t[rs].mn) t[id].mnct = t[ls].mnct + t[rs].mnct;
    else if (t[ls].mn > t[rs].mn) t[id].mnct = t[rs].mnct; else t[id].mnct = t[ls].mnct;
    t[id].hism = t[ls].hism + t[rs].hism; 
}
inline void maketag(int id, int x) {
    t[id].mn += x, t[id].smtg += x; 
}
inline void pushdown(int id) {
    maketag(ls, t[id].smtg), maketag(rs, t[id].smtg);
    if (t[ls].mn == t[id].mn) {
        t[ls].hstg += t[id].hstg;
        t[ls].hism += t[ls].mnct * t[id].hstg;
    } 
    if (t[rs].mn == t[id].mn) {
        t[rs].hstg += t[id].hstg;
        t[rs].hism += t[rs].mnct * t[id].hstg;
    } t[id].hstg = t[id].smtg = 0;
}
void upd(int ql, int qr, int x, int id = 1, int l = 1, int r = n) {
    if (ql <= l && qr >= r) return maketag(id, x);
    int M = (l + r) >> 1; pushdown(id); if (ql <= M) upd(ql, qr, x, ls, l, M); 
    if (qr > M) upd(ql, qr, x, rs, M + 1, r); pushup(id);
}
int qry(int ql, int qr, int id = 1, int l = 1, int r = n) {
    if (ql <= l && qr >= r) return t[id].hism;
    int M = (l + r) >> 1, R = 0; pushdown(id); if (ql <= M) R = qry(ql, qr, ls, l, M);
    if (qr > M) R += qry(ql, qr, rs, M + 1, r); return R;
}
void bd(int id, int l, int r) {
    if (l == r) return (void)(t[id].mnct = 1); 
    int M = (l + r) >> 1; bd(ls, l, M), bd(rs, M + 1, r);
    pushup(id);
}
#undef ls
#undef rs
signed main() {
    ios::sync_with_stdio(false), cin.tie(0), cout.tie(0);
    n = read(), m = read();
    rep (i, 1, n) a[i] = read();
    rep (v, 2, n) {
        int u = read();
        e[u].push_back(v);
    } dfs1(1); 
    rep (i, 1, n) da[dfn[i]] = a[i]; dfs(1, 0);
    rep (i, 1, m) {
        int l = read(), r = read();
        q[l - 1].push_back({l, r, -1, i});
        q[r].push_back({l, r, 1, i});
        ans[i] -= (r - l) * (r - l + 1) >> 1; 
    } bd(1, 1, n);
    rep (i, 1, n) {
        for (auto [l, r, op] : Q[i]) upd(l, r, op);
        if (!t[1].mn) t[1].hstg++, t[1].hism += t[1].mnct;
        for (auto [l, r, op, id] : q[i]) {
            ans[id] += op * qry(l, r);
        }
    }
    rep (i, 1, m) cout << ans[i] << '\n';
    return 0;
}
```

</details>

## [U](https://codeforces.com/problemset/problem/1017/G)

题意：给定一棵树，维护以下 $3$ 种操作：

- 如果节点 $x$ 为白色，则将其染黑。否则对这个节点的所有儿子递归进行相同操作。

- 将以节点 $x$ 为根的子树染白。

- 查询节点 $x$ 的颜色。

$n,q\leq 10^5,3\mathrm s,256\mathrm {MB}$。

sol：设 $f(x)$ 表示 $x$ 到根的链上有多少个白点。将所有点编号按 dfs 序重构，操作可以转化为：

- $[x,x+\mathrm{sz}(x)-1]$ 区间修改为 $\max(f(i)-1,f(fa(x)))$。

- $[x,x+\mathrm{sz}(x)-1]$ 区间修改为 $d(i)-(d(fa(x))-f(fa(x)))$。

- 查询 $f(x)-f(fa(x))$。

显然可以上势能线段树维护，时间复杂度 $O(n\log^2 n)$。

<details> <summary> Code </summary>

```cpp
#pragma GCC optimize("Ofast")
#pragma GCC optimize("unroll-loops")
#pragma GCC target("sse,sse2,sse3,ssse3,sse4,popcnt,abm,mmx,avx,avx2,tune=native")
#include<bits/stdc++.h>
using namespace std;
// #define int long long
#define pb emplace_back
#define IOS ios::sync_with_stdio(false), cin.tie(0), cout.tie(0)
const int N = 1e5 + 5, B = 4e2, inf = 1e9 + 10;
int n, m, cnt, d[N], sz[N], dfn[N], a[N], fa[N];
vector<int> e[N];
void dfs(int u = 1, int Fa = 0)
{
    dfn[u] = ++cnt, sz[u] = 1;
    for (auto v : e[u])
    {
        if (v == Fa) continue;
        d[v] = d[u] + 1, fa[v] = u;
        dfs(v, u), sz[u] += sz[v];
    }
}
struct segt {int mn, se, tag1, tag2, tag3;} t[N << 2];
struct segtd {int mn, se;} td[N << 2];
#define ls (id << 1)
#define rs (id << 1 | 1)
void pushup(int id)
{
    t[id].mn = min(t[ls].mn, t[rs].mn);
    if (t[ls].mn == t[rs].mn) t[id].se = min(t[ls].se, t[rs].se);
    else if (t[ls].mn > t[rs].mn) t[id].se = min(t[ls].mn, t[rs].se);
    else t[id].se = min(t[rs].mn, t[ls].se);
}
void maketag(int id, int l, int r, int tag1, int tag2, int tag3)
{
    if (tag2) t[id].mn = td[id].mn, t[id].se = td[id].se, t[id].tag2 = tag2, t[id].tag1 = 0, t[id].tag3 = -inf;
    if (tag1) t[id].mn += tag1, t[id].se += tag1, t[id].tag1 += tag1, t[id].tag3 += tag1;
    if (tag3 > -(inf >> 1) && t[id].mn < tag3 && t[id].se > tag3) t[id].mn = tag3, t[id].tag3 = tag3;
}
void pushdown(int id, int l, int r)
{
    int mid = (l + r) >> 1;
    maketag(ls, l, mid, t[id].tag1, t[id].tag2, t[id].tag3);
    maketag(rs, mid + 1, r, t[id].tag1, t[id].tag2, t[id].tag3);
    t[id].tag1 = 0, t[id].tag2 = 0, t[id].tag3 = -inf;
}
void build(int id = 1, int l = 1, int r = n)
{
    t[id].tag3 = -inf, t[id].se = td[id].se = inf;
    if (l == r) {t[id].mn = td[id].mn = d[l]; return ;}
    int mid = (l + r) >> 1; build(ls, l, mid), build(rs, mid + 1, r);
    pushup(id); td[id].mn = t[id].mn, td[id].se = t[id].se;
}
void upd1(int ql, int qr, int x, int id = 1, int l = 1, int r = n)
{
    if (ql <= l && qr >= r) 
    {
        t[id].mn += x; t[id].se += x;
        t[id].tag1 += x, t[id].tag3 += x; return ;
    }
    int mid = (l + r) >> 1; pushdown(id, l, r);
    if (ql <= mid) upd1(ql, qr, x, ls, l, mid);
    if (qr > mid) upd1(ql, qr, x, rs, mid + 1, r);
    pushup(id);
}
void upd2(int ql, int qr, int id = 1, int l = 1, int r = n)
{
    if (ql <= l && qr >= r) 
    {
        t[id].mn = td[id].mn, t[id].se = td[id].se;
        t[id].tag1 = 0, t[id].tag3 = -inf, t[id].tag2 = 1;
        return ;
    }
    int mid = (l + r) >> 1; pushdown(id, l, r);
    if (ql <= mid) upd2(ql, qr, ls, l, mid);
    if (qr > mid) upd2(ql, qr, rs, mid + 1, r);
    pushup(id);
}
void upd3(int ql, int qr, int x, int id = 1, int l = 1, int r = n)
{
    if (t[id].mn >= x) return ;
    if (ql <= l && qr >= r && t[id].se > x) 
    {
        t[id].mn = x; t[id].tag3 = x;
        return ;
    }
    int mid = (l + r) >> 1; pushdown(id, l, r);
    if (ql <= mid) upd3(ql, qr, x, ls, l, mid);
    if (qr > mid) upd3(ql, qr, x, rs, mid + 1, r);
    pushup(id);
}
int qry(int p, int id = 1, int l = 1, int r = n)
{
    if (!p) return 0;
    if (l == r) return t[id].mn;
    int mid = (l + r) >> 1; pushdown(id, l, r);
    if (p <= mid) return qry(p, ls, l, mid);
    else return qry(p, rs, mid + 1, r);
}
signed main()
{
    IOS; 
    cin >> n >> m;
    for (int u = 2; u <= n; u++) 
    {
        int v; cin >> v;
        e[u].pb(v), e[v].pb(u);
    } d[1] = 1, dfs(); vector<int> tmp(n + 1), tfa(n + 1);
    for (int i = 1; i <= n; i++) tmp[dfn[i]] = d[i];
    for (int i = 1; i <= n; i++) d[i] = tmp[i];
    for (int i = 1; i <= n; i++) tmp[dfn[i]] = sz[i], tfa[dfn[i]] = dfn[fa[i]];
    for (int i = 1; i <= n; i++) sz[i] = tmp[i], fa[i] = tfa[i]; build();
    for (int i = 1; i <= m; i++)
    {
        int op, x; cin >> op >> x; int l = dfn[x], r = l + sz[l] - 1;
        if (op == 1) upd1(l, r, -1), upd3(l, r, qry(fa[l]));
        else if (op == 2) upd2(l, r), upd1(l, r, qry(fa[l]) - d[fa[l]]);
    	else cout << (qry(l) - qry(fa[l]) ? "white" : "black") << '\n';
    }
    return 0;
}
```

</details>
