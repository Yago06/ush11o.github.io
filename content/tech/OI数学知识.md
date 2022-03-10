---
title: <科技点> 数学知识
date: 2019-11-07 09:06:59
tags: 
    - 数学
    - OI
    - 科技点
categories: 
    - OI
    - 科技点
---


## 1.质数

**Eratosthenes 筛法**

```cpp
bool v[maxn];
void primes(int n) {
    memset(v, 0, sizeof(v));
    for (int i = 2; i <= n; ++i) {
        if (v[i]) continue;
        cout << i << endl;
        for (int j = i; j <= n / i; ++i) v[i * j] = 1;
    }
}
```

Eratosthenes 筛法的时间复杂度为
$O(\sum _{p \leq N }\frac {N}{p} ) = O(N \log\log N) $
该算法实现简单，效率已经非常接近于线性，是算法竞赛中最常用的质数筛法。

**线性筛法**

>即使在优化后（从$x ^ {2}$开始），Eratosthenes 筛法仍然会重复标记合数。

线性筛法通过“从小到大累计质因子”的方式标记每个合数，实现每个合数只会被它的最小质因子$p$筛一次，时间复杂度为$O(N)$

```cpp
int v[maxn], prime[maxn], m;
void Euler(int n) {
    memset(v, 0, sizeof(v));
    m = 0;
    for (int i = 2; i <= n; ++i) {
        if (v[i] == 0) { v[i] = i; prime[++m] = i; }
        for (int j = 1; j <= m; ++j) {
            if (prime[j] > v[i] || prime[j] > n / i) break;
            // i 有比 prime[j] 更小的质因子, 停止循环
            v[i * prime[j]] = prime[j];
        }
    }
}
```
## 2.质因数的分解

**试除法**

时间复杂度$O(\sqrt {N})$

```cpp
int m, p[maxn], c[maxn];
void divide(int x) {
    m = 0;
    for (int i = 2; i <= sqrt(x); ++i) {
        if (x % i == 0) {
            p[++m] = i, c[m] = 0;
            while (x % i == 0) x /= i, c[m]++;
        }
    }
    if (x > 1) p[++m] = x, c[m] = 1;
}
```

最后一句曾经困扰着黑猫酱，不明白是为什么，最近才明白对于 $x$ ，大于$\sqrt{x}$ 的质因数最多只有一个，易知如果有两个或以上的大于$\sqrt{x}$的质因数，乘起来就会比 $x$ 大，所以在 $i > \sqrt{x}$ 时可以保证最多只剩下一个质因子，我们不用再去试除它，如果 $x$ 出了循环还是大于 $1$ 剩下的就是那唯一一个大于 $\sqrt{x}$ 的质因子，记下来就好。

## 3.约数

**定义**

若整数 $n$ 除以整数 $d$ 的余数为 $0$，即 $d$ 能整除 $n$，则称 $d$ 是 $n$ 的约数，$n$ 是 $d$ 的倍数，记为 $d \mid n$ 。(废话）

**算术基本定理的推论**

在算术基本定理中，若正整数 $N$ 被唯一分解为 $N = p_1^{c_1}p_2^{c_2} \cdots p_m^{c_m}$ ，其中 $c_i$ 都是正整数，$p_i$ 都是质数，而且满足 $p_1 < p_2 < \cdots < p_m$ ，则 $N$ 的正约数集合可写作：

$$\lbrace p_1^{b_1}p_2^{b_2} \cdots p_m^{b_m} \rbrace \quad (0 \leq b_i \leq c_i)$$ 

$N$ 的正约数个数为：

$$(1 + p_1 + p_1^ 2 + \cdots + p1^{c_1})* \cdot \cdot\prod_{i = 1}^{m} \left( \sum_{j=0}^{c_i} (p_i)^{j} \right)$$

(黑猫酱也搞不明白，反正记下来就对了）

**求 $N$ 的正约数集合 —— 试除法**

若 $d \geq \sqrt{N}$ 是 $N$ 的约数，则 $N / d \leq \sqrt{N} $
 也是 $N$ 的约数。换言之，约数总是成对出现的（对于完全平方数, $\sqrt{N}$ 单独出现）。

因此，只需要扫描 $d = 1 \sim \sqrt{N}$ 尝试 $d$ 能否整除 $N$，若能整除，则 $N/d$ 也是 $N$ 的约数。时间复杂度为 $O(\sqrt{N})$ 。

```cpp
int p[maxn], m = 0;
void get(int n) {
    for (int i = 1; i * i <= n; ++i) 
        if (n % i == 0) {
            p[++m] = i;
            if (i != n / i) p[++m] = n / i;
        }
}
```

**试除法的推论**

一个整数 $N$ 的约数个数上界为 $2\sqrt{N}$。

**求 $1 \sim N$ 每个数的正约数集合——倍数法**


若用“试除法”分别求出 $1 \sim N$ 每个数的正约数集合，时间复杂度为 $O(N\sqrt{N})$ 。可以反过来考虑，用“倍数法”求出  $1 \sim N$ 每个数的正约数集合：

```cpp
vector<int> p[maxn];
void get(int n) {
    for (int i = 1; i <= n; ++i) 
        for (int j = 1; j <= n / i; ++j)
            p[i * j].push_back(i);
}
```

时间复杂度为 $O(N + N/2 + N/3 + \cdots +  N/N ) = O(N log N)$
**倍数法推论**

$1 \sim N$ 每个数的约数个数总和大约为 $NlogN$。

***整除分块**

$\forall i \in [1 , N]$，$\lfloor N / i \rfloor$ 由不超过  $2\sqrt{N}$ 段组成，每一段 $i \in [x, \lfloor N / \lfloor N / x \rfloor \rfloor ]$ 中 $\lfloor N / i \rfloor $ 都等于 $\lfloor N / x \rfloor$ 。

```cpp
for (int l = 1, r; l <= n; l = r + 1) {
    r = n / (n / l);
    //...
}
```

虽然复杂度看起来很玄学，不过是 $O(\sqrt{N})$ 的。

**最大公约数**

* 欧几里得算法

```cpp
int gcd(int a, int b) {
    return b ? gcd(b, a % b) : a;
}
```

据说递归写法没有下面的那一种快，不过好记。

```cpp
int gcd(int a, int b) {
    int c;
    while (b) {
    c = b;
    b = a % b;
    a = c;
    }
    return a;
}

```

其实还有一种牛逼的写法（或者说调用方法）

```cpp
#include <algorithm>
std::__gcd(x, y);
```

* 欧拉函数

$1 \sim N$ 中与 $N$ 互质的数的个数被称为欧拉函数，记为 $\varphi (N)$ ，（读作phi）。

若在算术基本定理中，$N = p_1^{c_1}p_2^{c_2} \cdots p_m^{c_m}$ ，则：

$$\varphi (N) = N * \frac{p_1 - 1}{p_1}* \frac{p_2 - 1}{p_2} * \cdots * \frac{p_m - 1}{p_m} = N * \prod_{p \mid N} \left( 1 - \frac{1}{p} \right)$$

证明略（不会）

根据欧拉函数的计算式，我们只需要分解质因数，即可顺便求出欧拉函数.

```cpp

int phi(int n) {
    int ans = n;
    for (int i = 2; i <= sqrt(n); ++i) 
        if (n % i == 0) {
            ans = ans / i * (i - 1);
            while (n % i == 0) n /= i;
        }
    if (n > 1) ans = ans / n * (n - 1);
    return ans;
}

```

**性质**

1. $1 \sim n$ 中与 $n$ 互质的数的和为 $n * \varphi (n) / 2$ 。

2. 若 $a, b$ 互质，则 $\varphi (ab) = \varphi (a) \varphi (b) $ 。

证明略（还是不会）

**积性函数**

如果当 $a, b$ 互质时，有 $f(ab) = f(a)*f(b)$ 那么称函数 $f$ 为积性函数。

**性质**

3.若 $f$ 是积性函数，且在算术基本定理中， $n = \prod_{i = 1}^{m} p_i^{c_i}$ ，则 $f(n)=\prod_{i = 1}^{m} f(p_i^{c_i})$。

4.设 $p$ 为质数，若 $p|n$ 且 $p^2 \mid n$，则 $\varphi(n) = \varphi (n / p) * p $ 。

5.设 $p$ 为质数，若 $p \mid n$ 但 $p^2 \nmid n$，则 $\varphi (n) = \varphi (n / p) * (p - 1) $。

6. $\sum_{d \mid n} \varphi (d) = n$

**代码实现**

1.Eratosthenes 筛法，时间复杂度 $O(N log N)$ 。

```cpp
int phi[maxn];
void euler(int n) {
    for (int i = 2; i <= n; ++i) phi[i] = i;
    for (int i = 2; i <= n; ++i)
        if (phi[i] == i)
            for (int j = i; j <= n; j += i)
                phi[j] = phi[j] / i * (i - 1);
}
```
2.线性筛，利用性质 $4$ 和性质 $5$, 可以从 $\varphi (n / p) $ 递推到 $\varphi (n)$， 时间复杂度 $O(N)$。


```cpp
int v[maxn], prime[maxn], phi[maxn], m;
void euler(int n) {
    memset(v, 0, sizeof(v));
    m = 0;
    for (int i = 2; i <= n; ++i) {
        if (v[i] == 0) {
            v[i] = i; prime[++m] = i;
            phi[i] = i - 1;
        }
        for (int j = 1; j <= m; ++j) {
            if (prime[j] > v[i] || prime[j] > n / i) break;
            v[i * prime[j]] = prime[j];
            phi[i * prime[j]] = phi[i] * (i % prime[j] ? prime[j] - 1 : prime[j]);
        }
    }
}
```

能背下来就背吧。

## 4.同余
**费马小定理**

若 $p$ 是质数，则对于任意整数 $a$ 有 $a^{p} \equiv a \ (mod \ p)$。

**欧拉定理**

若正整数 $a,n$ 互质，则 $a^{\varphi (n)} \equiv 1 \ (mod \ p)$。

**拓展欧拉定理**

若正整数 $a,n$ 互质，则对于任意正整数 $b$，有 $a^b \equiv a^{b \ mod \ \varphi(n) } \ (mod \ n)$。

当 $a,n$ 不一定互质且 $b > \varphi (n)$ 时，有 $a^b \equiv a^{b \ mod \ \varphi(n) +\varphi (n)} \ (mod \ n)$。

* 拓展欧几里得

**Bezout 裴蜀定理**

对于任意整数 $a,b$，存在一对正整数 $x,y$, 满足 $ax + by = gcd(a,b) $。

计算：

```cpp
int exgcd(int a, int b, int &x, int &y) {
    if (b == 0) { x = 1, y = 0; return a; }
    int d = exgcd(b, a % b, x, y);
    int z = x; x = y; y = z - y * (a / b);
    return d;
}
```

推导：

对于 $gcd(a, b) = gcd(b, a \ mod \ b)$

有 $ax + by = bx'+ (a \  mod \  b)y'$
$= bx' + (a - \lfloor a / b\rfloor * b) y'$

$=ay' + b(x '- \lfloor a/b \rfloor * y')$

对齐 $a, b$ 系数可得 $exgcd$ 的算法

方程 $ax+by=c$ 有解当且仅当 $d \mid c $ 。
先求出 $ax+by=d$ 的一组特解 $x_0, y_0$ ，同时乘上 $c/d$ ,就得到了 $ax+by=c$ 的一组特解 $(c/d)x_0, (c/d)y_0 $。

方程的通解可以表示为

$$ x = \frac{c}{d}x_0 + k \frac{b}{d}, \ \ \ \ y = \frac{c}{d}y_0 - k \frac{a}{d} \ \ (d = gcd(a, b),\  k \in \mathbb Z)$$


**乘法逆元**

若整数 $b,m$ 互质，且 $b \mid a$ ，则存在一个整数 $x$，使得 $a/b \equiv a*x \ (mod \ m)$

当模数为质数时，根据费马小定理，有 $a^{p - 2} \equiv 1/a \ (mod \ p) $。


* 线性同余方程

给定整数 $a, b, m$ 求一个整数 $x$ 满足 $a*x \equiv b \ (mod \ m) $， 或给出无解。

解：设 $a*x-b=-ym$，则方程改写成 $a*x+m*y=b$，根据裴楚定理求解，过程略。

**中国剩余定理**

设 $m_1,m_2, \cdots m_n$ 是两两互质的整数，$m=\prod_{i = 1}^{n}m_i, M_i=m/m_i, t_i$ 是线性方程 $M_i t_i \equiv 1 \ (mod \ m_i) $ 的一个解，（用人话讲就是 $M_i$ 在模 $m_i$ 意义下的乘法逆元）。对于任意的 $n$ 个整数 $a_1, a_2, \cdots ,a_n$ ，方程组：

$$\begin{cases} x \equiv a_1 (mod \ m_1) \\ x \equiv a_2 (mod \ m_2) \\
\ \ \ \ \ \ \ \ \ \ \ \ \ \ \vdots \\ x \equiv  a_n (mod \ m_n)\end{cases}$$
有整数解，解为 $x = \sum_{i=1}^{n}a_i M_i t_i$。

## 5.矩阵乘法

$$C_{i,j}=\sum_{k = 1}^{m} A_{i,k} * B_{k,j}$$

可以用矩阵快速幂加速递推，此类题目特点：

1.可以抽象出一个长度为 $n$ 的以为向量，该向量在每个单位时间发生一次变化

2.变化的形式是一个线性递推

3.该递推式在每个时间可能作用于不同的数据上，但本身保持不变

4.向量变化时间（即递推的轮数）很长，但向量长度 $n$ 不大。

## 6.高斯消元

[传送门](https://www.luogu.org/blog/yuno/knowledge-gauss)

## 7.组合计数

**二项式定理**

$$(a + b) ^ n = \sum_{k = 0} ^ {n} C_n^k a^k b^ {n - k} $$

**Lucas 定理**

 若 $p$ 是质数，则对于任意整数 $1 \leq m \leq n $，有：
 
 $$C_n^m \equiv C_{n \ mod \ p} ^ {m \ mod \ p} * C_{n / p} ^ {m / p} (mod \ p) $$
 
 ```cpp
 int C(int a, int b) {
	if (a > b) return 0;
	return 1ll * fac[b] * inv(fac[a]) % p * inv(fac[b - a]) % p;
}

int Lucas(int a, int b) {
	if (b == 0) return 1;
	return 1ll * C(a % p, b % p) * Lucas(a / p, b / p) % p;
}
```
 
 **Catalan 数列**
 
 $$Cat_n = \frac {C_{2n} ^ n} {n + 1}$$
 
 ## 8.容斥原理与Mobius函数
 
 容斥原理略，现推就行
 
 **Mobius函数**
 
 设正整数 $N$ 按照算术基本定理分解质因数为 $N = p_1^{c_1} p_2^{c_2} \cdots  p_m^{c_m} $，定义函数
 
 $$ \mu (N) = \begin{cases} 0 \quad \quad \quad \quad  \quad \exists i \in [1, m], c_i > 1 \\ 1 \quad \ \ \ m \equiv 0 \ (mod \ 2), \forall i \in [1,m], c_i = 1 \\ -1 \quad m \equiv 1 \ (mod \ 2), \forall i \in [1,m], c_i = 1 \end{cases} $$
 
 称 $\mu(N)$ 为 $Mobius$ 函数。
 
 当 $N$ 包含相等的质因子时， $\mu(N) = 0$。当 $N$ 所有的质因子各不相等时，若 $N$ 有偶数个质因子，$\mu(N) = 1$，若 $N$ 有奇数个质因子，$\mu(N) = -1$。
 
利用埃筛，把所有 $\mu$ 值初始化为 $1$。接下来，对于晒出的每个质数 $p$，令 $\mu(p) = -1$，并扫描 $p$ 的倍数 $x = 2p, 3p, \cdots, \lfloor n / p \rfloor * p$，检查 $x$ 能否被 $p^2$ 整除。若能，则令 $\mu(x) = 0$，否则令 $\mu (x) = - \mu (x)$。

```cpp

for (int i = 1; i <= n; ++i) miu[i] = 1, v[i] = 0;
for (int i = 2; i <= n; ++i) {
	if (v[i]) continue;
	miu[i] = -1;
	for (int j = 2 * i; j <= n; j += i) {
		v[j] = 1;
		if ((j / i) % i == 0) miu[j] = 0;
		else miu[j] *= -1;
	}
}

```

## 9.概率与数学期望

略，这点看脸了，难的不会，简单的没有复习的意义

## 10.0/1分数规划

二分答案，验证式子

$$\sum_{i = 1}^{n}(a_i - mid * b) * x_i \geq 0 \ (or \leq 0)$$

相应的改变 $mid$ 

## 11.博弈论

**NIM 游戏**

