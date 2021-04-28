---
title: '题解 [NOI Online #1 入门组] 魔法'
date: 2021-04-27 19:55:32
tags: [题解,动态规划,矩阵]
published: true
hideInList: false
feature: 
isTop: false
---
是一道考察对矩阵理解的好题。

自己做了很久都没有往矩阵上去想。

<!-- more -->

[传送门](https://www.luogu.com.cn/problem/P6190)

DAG 和 $k=0$ 的情况都很简单，由于 DAG 有做法而往回走了最后还得到终点，所以想到了缩点。

但是发现缩点并没有什么用，因为可以重复绕，而变成负数又有限制，所以强连通分量没有什么用。最多是环有点用处，可以处理出当前点绕环中使用 $i$ 次魔法的最优情况，但谁说圈一定要绕满？所以也没啥用。

数据范围似乎有些误导性，看到 $90$ 分那一档就认为是分层图了，一直想着怎么去优化，但最终未果。

其实正解是 dp 的矩阵优化，首先可以想到一个类似弗洛伊德的 dp， $f[k][i][j]$ 表示 $k$ 次魔法，从 $i$ 到 $j$ 的最优答案。至于转移，随便 $f[k-1][i][l]$ 和 $f[1][l][j]$ 取一个 $\min$ 就可以了。时间复杂度是 $n^4$ 的，不能接受。

这个 dp 看上去没有什么优化的空间，但实际上，可以矩阵加速。

矩阵的本质是什么？看过 3b1b 的会说， **矩阵是一种变换**， 而我们的矩阵乘法快速幂的要求就是要满足结合律，所以我们其实可以把加法和乘法用另外的运算替换掉，描述一种变换，然后可以同样是可以搞快速幂的。

在此题中，可以重载成这样的一种运算：

```cpp
twt operator * (twt b) {
	twt c;
	for(int k = 1; k <= n; k++)
		for(int i = 1; i <= n; i++)
			for(int j = 1; j <= n; j++)
				c[i][j] = std::min(c[i][j], t[i][k] + b[k][j]);
	return c;
}
```

看上去这似乎是一个弗洛伊德，但它的答案不是来源于同一个数组中的，而是来源于运算符两边的 `twt`。我们回顾那个无法接受的转移，其实不一定必须从 $k-1$ 及 $1$ 转移过来，任意一个 $k-c$ 和 $c$ 都是可以的。

所以就有了上面这个运算，它的路径剖开，一边用的是经过了若干次魔法的，另一边用的是经过一些魔法的，合起来就成了使用两边原来使用的魔法次数的和的，而且这个样子是由结合律的（感性理解：用 $3,2,1$ 次魔法的合并， 最后不管怎么并都是使用了 $6$ 次魔法），所以我们就可以使用快速幂。

单位矩阵当然变了，由定义可知，单位矩阵就是不使用魔法的，那么就是直接用弗洛伊德跑一遍的结果了。

完整代码。

```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
#define int long long
const int N = 105, M = 2505;
int n, m, k, x[M], y[M], w[M];
struct twt {
	int t[N][N];
	twt() { memset(t, 0x3f, sizeof t); }
	twt operator * (twt b) {
		twt c;
		for(int k = 1; k <= n; k++)
			for(int i = 1; i <= n; i++)
				for(int j = 1; j <= n; j++)
					c[i][j] = std::min(c[i][j], t[i][k] + b[k][j]);
		return c;
	}
	int* operator [] (int x) { return t[x]; };
} a, f;
twt Pow(twt a, int b) {
	twt an = f;
	while(b) {
		if(b & 1) an = an * a;
		a = a * a;
		b >>= 1;
	}
	return an;
}
signed main() {
	scanf("%lld%lld%lld", &n, &m, &k);
	for(int i = 1; i <= n; i++) f[i][i] = 0;
	for(int i = 1; i <= m; i++) {
		scanf("%lld%lld%lld", &x[i], &y[i], &w[i]);
		f[x[i]][y[i]] = w[i];
	}
	for(int k = 1; k <= n; k++)
		for(int i = 1;  i <= n; i++)
			for(int j = 1; j <= n; j++)
				f[i][j] = std::min(f[i][j], f[i][k] + f[k][j]);
	for(int k = 1; k <= m; k++) {
		int u = x[k], v = y[k], c = w[k];
		for(int i = 1; i <= n; i++)
			for(int j = 1; j <= n; j++)
				a[i][j] = std::min(a[i][j], std::min(f[i][j], f[i][u]+f[v][j]-c));	
	}
	if(k == 0) printf("%lld", f[1][n]);
	else printf("%lld", Pow(a, k)[1][n]);
	return 0;
}
```