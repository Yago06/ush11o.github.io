---
title: <科技点> LinkCutTree
date: 2019-11-14 16:26:03
tags: 
    - 科技点
    - OI
    - Link Cut Tree
    - LCT
    - 动态树
categories: 
    - OI
    - 科技点
---
前置科技点：树链剖分，$Splay$

$LinkCutTree$ 维护的是实链剖分下的 Splay 森林，通过虚实交替的树边，支持一堆牛逼的操作：

* 修改、查询链上信息

* 指定原树的根（换根）

* 动态连边、删边

* 合并两棵树，分离一棵树（~~跟上面不是一样的吗~~）

* 动态维护连通性

* 更多牛逼操作

[华华盗图的地点](https://www.cnblogs.com/flashhu/p/8324551.html)

## LCT 的主要性质

1.每一个 Splay 维护的是一条从上到下按在原树中深度严格递增的路径，且中序遍历Splay得到的每个点的深度序列严格递增。

2.每个节点仅包含在一个 $Splay$ 中

3.边分为实边和虚边，实边包含在Splay中，而虚边总是由一棵Splay指向另一个节点（指向该Splay中中序遍历最靠前的点在原树中的父亲）。 因为性质2，当某点在原树中有多个儿子时，只能向其中一个儿子拉一条实链（只认一个儿子），而其它儿子是不能在这个Splay中的。 那么为了保持树的形状，我们要让到其它儿子的边变为虚边，由对应儿子所属的Splay的根节点的父亲指向该点，而从该点并不能直接访问该儿子（认父不认子）。

## LCT 操作

### access(u)

LCT 的核心操作，由于虚边的存在，所以想要操作的两个节点不一定在同一个 Splay 中，不能进行维护，$access()$ 操作打通一条从指定节点到根的链，强行把两个点放到一个 Splay 里。

~~蒟蒻深知没图的痛苦qwq~~

有一棵树，假设一开始实边和虚边是这样划分的

![](https://cdn.luogu.com.cn/upload/image_hosting/lqwozang.png)

那么所构成的LCT可能会长这样

![](https://cdn.luogu.com.cn/upload/image_hosting/povo1e22.png)

每一个小框框表示一个 Splay ，其中每一个 Splay 都满足中序遍历在原图中深度递增

$access(N)$ 把 N 到根打通，拉进一个 Splay 里，由于 Splay 不能记住其他节点，所以这一条链之外的节点都得给这条链让路。所以我们需要对这些边进行操作，相应的变轻变重。

![](https://cdn.luogu.com.cn/upload/image_hosting/gr561fe6.png)

具体怎么操作呢？

要一步一步从下往上操作，先 Splay(N) 让 N 变成自己这个 Splay 的根，然后因为只要根到 N 的节点，所以现在 N 的右子节点，也就是在原树中比 N 的深度要深的节点，全部都要抛弃掉，直接单方面把 N 的右儿子置0。(认父不认子）

变成酱紫：

![](https://cdn.luogu.com.cn/upload/image_hosting/h3mtj80a.png)

接着把 N 的父亲 I 也转到自己所在 Splay 的根，同样的要抛弃在 I 的 Splay 中原深度比 I 深的点，然后链 N 到 I 的右儿子，这样 N, I 就在一个 Splay 里了。

![](https://cdn.luogu.com.cn/upload/image_hosting/zawlnp1q.png)

接着 Splay(H)，丢掉右子树，链上 I：

![](https://cdn.luogu.com.cn/upload/image_hosting/jbk2iuyq.png)

最后 Splay（A)，丢掉右子树，链上 H：

![](https://cdn.luogu.com.cn/upload/image_hosting/2t0c9uvm.png)

然后就大功告成了，现在 N 和根 A 就在一个 Splay 中了。

虽然现在在 Splay 中 A 和 N 看起来不是在一条链上，实际上按照中序遍历，在原树中已经链上一条 A - C - G - H - I - L - N 的链，可以进行操作了。

splay 操作也就是循环进行下面的操作：

* 转到根

* 换儿子

* 更新信息

* 换节点

```cpp
void access(int u) {
	for (int t = 0; u; t = u, u = fa[u]) {
		splay(u);
		ch[u][1] = t;
		pushup(u);
	}
}
```

### evert(u)

也有许多许多人都把这叫做 makeroot()，这个操作是建立在 access() 上，很多时候我们不止需要进行点到根这一条路径的操作，而是要进行两个点之间的路径操作，这时候就要通过换根，然后 access() 另一个节点来打通一条两个点的路径。所以 access(u), play(u)，把 u 转到自己所在 splay 的根节点，此时 u 是没有右子树的，也就是在原树中是深度最深的点，我们水平翻转整个 Splay ，由于中序遍历是深度递增的性质，水平翻转之后 Splay 的中序遍历就倒过来了，此时没有左子树的 u 就成了深度最小的点，也就是根节点。（注意翻转要标记下传，我们不只是翻转一个节点的左右儿子，而是所有 Splay 内节点的左右儿子）

```cpp
void evert(int u) {
	access(u);
	splay(u);
	tag[u] ^= 1;
}
```

### find(u)

寻找 u 所在 Splay 的根，一般用来判断连通性 find(x) == find(y) ，因为一个 Splay 只有一个根。

```cpp
int find(int u) {
	access(u); 
	splay(u);
	while (ch[u][0])
		u = ch[u][0];
	splay(u);
	return u;
}
```

### split(x, y)

通过上面的几个操作我们已经可以做到在任意两点间拉出一条链了。

```cpp
void split(x, y) {
	evert(x); access(y);
}
```

### link(x, y)

按照 LCT 性质，只要把一个节点转到自己 Splay 的根，然后父亲指向另一个节点。

```cpp
void link(int u, int v) {
	evert(u);
	if (find(v) == u) 
		return ;
	fa[u] = v;
}
```

### cut(x, y)

和 link 其实差不多，如果原来两点有连边，在 Splay 完成后一定有一条直接连的边，断掉就行，如果两个节点中间还有夹着别的节点，说明原图上是没有直接连边的。

```cpp
void cut(int u, int v) {
	evert(u);
	if (find(v) != u || fa[v] != u || ch[v][0])
		return ;
	ch[u][1] = fa[v] = 0;
	pushup(u);
}
```

~~没了~~

其实 LCT 还有不少牛逼的操作，不过黑猫酱暂时还不会，所以也只能先咕咕咕了。


[P3690 【模板】Link Cut Tree （动态树）](https://www.luogu.org/problem/P3690)

```cpp
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 300007;

int n, m;
int fa[MAXN], ch[MAXN][2];
int val[MAXN], sum[MAXN];
int tag[MAXN];
int sta[MAXN], top;

int id(int u) {
	return ch[fa[u]][1] == u;
}

int isroot(int u) {
	return ch[fa[u]][0] != u && ch[fa[u]][1] != u;
}

void pushup(int u) {
	sum[u] = val[u] ^ sum[ch[u][0]] ^ sum[ch[u][1]];
}

void pushdown(int u) {
	if (tag[u]) {
		swap(ch[u][0], ch[u][1]);
		tag[u] ^= 1;
		tag[ch[u][0]] ^= 1;
		tag[ch[u][1]] ^= 1;
	}
}

void rotate(int u) {
	int v = fa[u], w = fa[v], s = id(u);
	if (!isroot(v)) 
		ch[w][id(v)] = u;
	fa[u] = w;
	ch[v][s] = ch[u][s ^ 1]; 
	fa[ch[u][s ^ 1]] = v;
	ch[u][s ^ 1] = v;
	fa[v] = u;
	pushup(v);
	pushup(u);
}

void splay(int u) {
	sta[top = 1] = u;
	for (int i = u; !isroot(i); i = fa[i])
		sta[++top] = fa[i];
	while (top)
		pushdown(sta[top--]);
	for (int f = fa[u]; !isroot(u); rotate(u), f = fa[u])
		if (!isroot(f))	rotate(id(f) ^ id(u) ? u : f);
}

void access(int u) {
	for (int t = 0; u; t = u, u = fa[u]) {
		splay(u);
		ch[u][1] = t;
		pushup(u);
	}
}

void evert(int u) {
	access(u);
	splay(u);
	tag[u] ^= 1;
}

int find(int u) {
	access(u); 
	splay(u);
	while (ch[u][0])
		u = ch[u][0];
	splay(u);
	return u;
}

void link(int u, int v) {
	evert(u);
	if (find(v) == u) 
		return ;
	fa[u] = v;
}

void cut(int u, int v) {
	evert(u);
	if (find(v) != u || fa[v] != u || ch[v][0])
		return ;
	ch[u][1] = fa[v] = 0;
	pushup(u);
}

void change(int u, int w) {
	val[u] = w;
	access(u);
	splay(u);
}

int query(int u, int v) {
	evert(u);
	access(v);
	splay(v);
	return sum[v];
}

int main() {
	scanf("%d%d", &n, &m);
	for (int i = 1; i <= n; ++i)
		scanf("%d", &val[i]);
	while (m--) {
		int op, u, v;
		scanf("%d%d%d", &op, &u, &v);
		if (op == 0) printf("%d\n", query(u, v));
		if (op == 1) link(u, v);
		if (op == 2) cut(u, v);
		if (op == 3) change(u, v);
	}
}
```
传送门：

[[题解] P2387 [NOI2014]魔法森林](https://www.luogu.org/blog/yuno/solution-p2387)
