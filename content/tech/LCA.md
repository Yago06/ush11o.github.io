---
title: <科技点> LCA
date: 2019-11-15 09:51:09
tags: 
    - 科技点
    - OI
    - LCA
    - 最近公共祖先
categories: 
    - OI
    - 科技点
---
1.倍增

(远古代码，丑了点)

```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 500000 + 10;
int n, m, s;
struct Edge {
	int t;
	int next;
} e[maxn * 2];
int head[maxn];
int ecnt = 0;
int d[maxn];
int f[maxn][22];
int lg[maxn];

inline void read(int& x) {
	x = 0;
	int f = 1;
	char c = getchar();
	while (c < '0' || c > '9') {
		if (c == '-') f = -1;
		c = getchar();
	}
	while (c >= '0' && c <= '9') {
		x = x * 10 + c - '0';
		c = getchar();
	}
	x *= f;
}

inline void add(int u, int v) {
	ecnt++;
	e[ecnt].t = v;
	e[ecnt].next = head[u];
	head[u] = ecnt;
}

void dfs(int u, int fa) {
	d[u] = d[fa] + 1;
	f[u][0] = fa;
	for (int i = 1; (1 << i) <= d[u]; ++i) {
		f[u][i] = f[f[u][i - 1]][i - 1];
	}
	for (int i = head[u]; i; i = e[i].next) {
		if (e[i].t != fa) dfs(e[i].t, u);
	}
}

int lca(int a, int b) {
	if (d[a] < d[b]) swap(a, b);
	while (d[a] > d[b]) a = f[a][lg[d[a] - d[b]] - 1];
	if (a == b) return a;
	for (int k = lg[d[a]] - 1; k >= 0; k--) {
		if (f[a][k] != f[b][k]) {
			a = f[a][k];
			b = f[b][k];
		}
	}
	return f[a][0];
}

int a, b;
int main() {
	read(n); read(m); read(s);
	for (int i = 1; i < n; ++i) {
		read(a); read(b);
		add(a, b);
		add(b, a);
	}
	memset(d, 0, sizeof(d));
	dfs(s, 0);
	for(int i = 1; i <= n; i++) {
    	lg[i] = lg[i-1] + (1 << lg[i-1] == i);
	}
	for (int i = 0; i < m; ++i) {
		read(a); read(b);
		printf("%d\n", lca(a, b));
	}
	return 0;
}
```
2.树剖

```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 500000 + 7;
int n, m, s;
int stsz[maxn];
int hson[maxn];
int dep[maxn];
int tfa[maxn];
int tpfa[maxn];

struct Edge {
	int v;
	Edge *next;
} edge[maxn << 1];
Edge *head[maxn];
int ecnt, cnt;

inline void add(int u, int v) {
	ecnt++;
	edge[ecnt].v = v;
	edge[ecnt].next = head[u];
	head[u] = &edge[ecnt];
}

void dfs0(int u, int fa, int deep) {
	dep[u] = deep;
	tfa[u] = fa;
	stsz[u] = 1;
	int maxson = -1;
	for (Edge *i = head[u]; i; i = i->next) {
		int t = i->v;
		if (t == fa) continue;
		dfs0(t, u, deep + 1);
		stsz[u] += stsz[t];
		if (stsz[t] > maxson) {
			maxson = stsz[t];
			hson[u] = t;
		}
	}
}

void dfs1(int u, int topfa) {
	tpfa[u] = topfa;
	if (hson[u]) dfs1(hson[u], topfa);
	for (Edge *i = head[u]; i; i = i->next) {
		int t = i->v;
		if (t == hson[u] || t == tfa[u]) continue;
		dfs1(t, t);
	}
}

void lca(int x, int y) {
	while (tpfa[x] != tpfa[y]) {
		if (dep[tpfa[x]] >= dep[tpfa[y]]) x = tfa[tpfa[x]];
		else y = tfa[tpfa[y]];
	}
	printf("%d\n", dep[x] < dep[y] ? x : y);
}


int main() {
	scanf("%d%d%d", &n, &m, &s);
	int u, v;
	for (int i = 1; i < n; ++i) {
		scanf("%d%d", &u, &v);
		add(u, v);
		add(v, u);
	}
	dfs0(s, -1, 1);
	dfs1(s, s);

	for (int i = 0; i < m; ++i) {
		scanf("%d%d", &u, &v);
		lca(u, v);
	}
	return 0;
}
```

3.RMQ

```cpp
#include <bits/stdc++.h>
using namespace std;
inline int read() {
	int x = 0, f = 0; char c = getchar();
	for (; c < '0' || c > '9'; c = getchar()) if (c == '-') f = 1;
	for (; c >= '0' && c <= '9'; c = getchar()) x = (x << 3) + (x << 1) + (c ^ 48);
	return f ? -x : x;
}

const int MAXN = 500000 + 7;
int st[MAXN << 2][23];
int dep[MAXN], dfn[MAXN];
int idx, n, m, rt;
struct edge {
	int v, next;
} e[MAXN << 1];
int ec, head[MAXN];

void add(int u, int v) {
	e[++ec] = {v, head[u]};
	head[u] = ec;
}

int cmp(int x, int y) {
	return dep[x] < dep[y] ? x : y;
}

void dfs(int u, int fa) {
	st[++idx][0] = u;
	dfn[u] = idx;
	for (int i = head[u]; i; i = e[i].next) {
		if (e[i].v == fa) continue;
		dep[e[i].v] = dep[u] + 1;
		dfs(e[i].v, u);
		st[++idx][0] = u;
	}
}

int lg2[MAXN << 1];
void RMQ() {
	for (int i = 2; i <= (n << 1); ++i)
		lg2[i] = lg2[i >> 1] + 1;
	for (int i = 1; i <= 20; ++i) 
		for (int j = 1; j + (1 << i) - 1 <= idx; ++j) 
			st[j][i] = cmp(st[j][i - 1], st[j + (1 << (i - 1))][i - 1]);
}

int LCA(int x, int y) {
	x = dfn[x], y = dfn[y];
	if (x > y) swap(x, y);
	int k = lg2[y - x + 1];
	return cmp(st[x][k], st[y - (1 << k) + 1][k]);
}

int main() {
	n = read(); m = read(); rt = read();
	for (int i = 1, u, v; i < n; ++i) {
		u = read(), v = read();
		add(u, v); add(v, u);
	}
	dep[rt] = 1;
	dfs(rt, 0); RMQ();
	int u, v;
	while (m--) {
		u = read(); v = read();
		printf("%d\n", LCA(u, v));
	}
	return 0;
}
```