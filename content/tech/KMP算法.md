---
title: <科技点> KMP算法
date: 2019-11-11 21:43:32
tags: 
    - 科技点
    - OI
    - KMP算法
categories: 
    - OI
    - 科技点
---
$KMP$ 的精髓在于，对于每次失配之后，我都不会从头重新开始枚举，而是根据我已经得知的数据，从某个特定的位置开始匹配；而对于模式串的每一位，都有唯一的“特定变化位置”，这个在失配之后的特定变化位置可以帮助我们利用已有的数据不用从头匹配，从而节约时间。

[P3375 【模板】KMP字符串匹配](https://www.luogu.org/problem/P3375)
```cpp
#include <bits/stdc++.h>
using namespace std;

const int maxn = 1000000 + 77;
char b[maxn];
char a[maxn];
int nxt[maxn];
int f[maxn];

int main() {
	scanf("%s", b + 1);
	scanf("%s", a + 1);
	int n = strlen(a + 1);
	int m = strlen(b + 1);
	nxt[1] = 0;
	for (int i = 2, j = 0; i <= n; ++i) {
		while (j > 0 && a[i] != a[j + 1]) j = nxt[j];
		if (a[i] == a[j + 1]) ++j;
		nxt[i] = j;
	}
	for (int i = 1, j = 0; i <= m; ++i) {
		while (j > 0 && (j == n || b[i] !=  a[j + 1])) j = nxt[j];
		if (b[i] == a[j + 1]) j++;
		f[i] = j;
		if (f[i] == n) printf("%d\n", i - n + 1);
	}
	for (int i = 1; i <= n; ++i)
		printf("%d ", nxt[i]);
	return 0;
}
```