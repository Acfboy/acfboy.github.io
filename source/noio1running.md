---
title: '题解 [NOI Online #1 入门组] 跑步'
date: 2021-04-26 12:22:07
tags: [题解,分块,动态规划]
published: true
hideInList: false
feature: 
isTop: false
---
这个解法太妙了，在一堆生成函数中简直是一股清流！

这题其实考察了对 dp 及分块的理解。

<!-- more -->

> 求将 $n$ 分解成若干个**非严格**递增的整数的和的方案数。

很容易想到一个完全背包的做法，时间复杂度是 $\mathcal O(n^2)$ 的。2020  年 3 月 8 日，我就写了这个做法，~~而且居然还是二维的。~~

然后有一个绝妙的想法：分块。

设 $m=\lceil \sqrt n \rceil$, 然后把 $n < m$ 和 $n \ge m$ 的分开进行处理，因为 $m$ 个 $m$ 一定可以达到 $n$（要上取整），所以两边拼出来一定可以有答案。

然后考虑怎么进行处理。

可以令 $g[i][j]$ 表示 $i$ 个大于等于 $m$ 的数拼出 $j$ 的方案数。那么这个东西如何进行转移呢？	这里需要考察对动规的理解了，我们推导出一个状态 $g[i][j]$ 只需要把它转换成一个不重不漏构造出这个状态的方式的前一步就可以了。

比如完全背包，构造方式一是这一种只取一个，二是这一种再取一个，$n$ 次这样的构造方式可以构造出所有的状态，所以我们就取这种构造方式的上一步转移过来。

这个 $g$ 有一种很妙的构造方式，那就是每次要么每一个都加上 $1$, 要么加入一个 $m$，因为题目要求是递增的，所以这样子能构造出所有的状态。那么我们只要从这种构造方式的上一个步骤转移过来就可以了。即 $g[i][j] = g[i][j-i] + g[i-1][j-m]$。

然后就把 $f$ 和 $g$ 一一搭配就可以了，同样因为递增的保证，不需要考虑怎么组合的问题，组合方式是唯一的。枚举小于 $m$ 的和是多少就可以了。即答案是 $\sum_{i=0}^n \left( f_{m-1,i} \times \sum_{j=0}^m g_{j,n-i} \right)$。

代码。

```cpp
#include <cstdio>
#include <cmath>
#define int long long
const int N = 100005, M = 400;
int n, p, g[M][N], f[N], ans;
signed main() {
	scanf("%lld%lld", &n, &p);
	int m = sqrt(n) + 1;
	f[0] = 1;
	for(int i = 1; i < m; i++) 
		for(int j = i; j <= n; j++) f[j] = (f[j] + f[j-i]) % p;
	g[0][0] = 1;
	for(int i = 1; i < m; i++)
		for(int j = i; j <= n; j++) {
			g[i][j] = g[i][j-i];
			if(j >= m) g[i][j] = (g[i][j] + g[i-1][j-m]) % p;
		}
	int ans = 0;
	for(int i = 0; i <= n; i++) {
		int sum = 0;
		for(int j = 0; j < m; j++) sum = (sum + g[j][n-i]) % p;
		ans = (ans + f[i] * sum) % p;
	}
	printf("%lld", ans);
	return 0;
}
```