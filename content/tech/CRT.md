---
title: <科技点> 中国剩余定理 CRT
date: 2019-11-14 11:02:43
tags: 
    - 科技点
    - OI
    - CRT
    - 中国剩余定理
categories: 
    - OI
    - 科技点
---
## Chinese remainder theorem(CRT)

前置科技：$exgcd$

设 $m_1,m_2, \cdots m_n$ 是两两互质的整数，$m=\prod_{i = 1}^{n}m_i, M_i=m/m_i, t_i$ 是线性方程 $M_i t_i \equiv 1 \ (mod \ m_i) $ 的一个解，（用人话讲就是 $M_i$ 在模 $m_i$ 意义下的乘法逆元）。对于任意的 $n$ 个整数 $a_1, a_2, \cdots ,a_n$ ，方程组：

$$\begin{cases} x \equiv a_1 (mod \ m_1) \\ x \equiv a_2 (mod \ m_2) \\
\ \ \ \ \ \ \ \ \ \ \ \ \ \ \vdots \\ x \equiv  a_n (mod \ m_n)\end{cases}$$
有整数解，解为 $x = \sum_{i=1}^{n}a_i M_i t_i$。

解方程组过程就是两个两个同余先解出一个解，贡献到答案里，然后合并成为一个方程。

对于两个方程：

$x = a_1 + k_1 * m_1 $

$x = a_2 + k_2 * m_2 $

消去 $x$ 并移项

$ k_1 * m_1 - k_2 * m_2 = a_2 - a_1 $

令 $ C = a_2 - a_1 $ , $ g = gcd(m_1, m_2) $

通过exgcd解得 $k$ 的一组解：$x_1, y_1$

则: 

$ k_1 = x_1 * C / g $


$ k_2 = x_2 * C / g $ 


通解为：

$ k_1 = k_1 + m_2 / g * T $ 

$ k_2 = k_2 + m_1 / g * T $

易得最小正整数解为：

$ k_1 = (k_1 \% (m_2 / g) + m_2 / g ) \% g $

再带入 $ x = a_1 + k_1 * n_1 $ 得到 $ x $ 
 
$ \begin{cases} x \equiv a_1 \ (mod \ m_1) \\ x \equiv a_2 \ (mod \ m_2) \end{cases} \Rightarrow x \equiv a_1 + x * m_1 \ (mod \ LCM(p_1, p_2))$

然后一个一个做下去就得到了最终的解。

[P4777 【模板】扩展中国剩余定理（EXCRT）](https://www.luogu.org/problem/P4777)

```cpp
#include <bits/stdc++.h>
using namespace std;

inline int read() {
	int x = 0, f = 0; char c = getchar();
	for (; c < '0' || c > '9'; c = getchar()) if (c == '-') f = 1;
	for (; c >= '0' && c <= '9'; c = getchar()) x = (x << 3) + (x << 1) + (c ^ 48);
	return f ? -x : x;
}

const int MAXN = 1e5 + 1;
int n;
int a[MAXN], ni[MAXN];

int exgcd(int a, int b, int &x, int &y) {
	if (!b) { x = 1, y = 0; return a; }
	int d = exgcd(b, a % b, x, y);
	int z = x; x = y; y = z - y * (a / b);
	return d;
}

int EXCRT() {
	int a1 = a[1], n1 = ni[1];
	for (int i = 2; i <= n; ++i) {
		int a2 = a[i], n2 = ni[i];
		int g, A, B, C = a2 - a1;
		g = exgcd(n1, n2, A, B);
		A = ((A * C / g) % (n2 / g) + (n2 / g)) % (n2 / g);
		a1 = A * n1 + a1;
		n1 = n1 * n2 / g;
	}
	return a1;
}

int main() {
	n = read();
	for (int i = 1; i <= n; ++i)
		ni[i] = read(), a[i] = read();
	printf("%lld\n", (long long)EXCRT());
	return 0;
}
```