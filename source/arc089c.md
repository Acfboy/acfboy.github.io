---
title: '题解  [ARC089C] GraphXY'
date: 2021-05-12 08:24:44
tags: [题解,构造]
published: true
hideInList: false
feature: 
isTop: false
---
构造题似乎就是考验思维了，那些所谓套路不怎么管用啊。。

<!-- more -->

> 构造一个图，不超过 $300$ 节点，确定的边权不超过 $100$, 还有些边权填上 $x$ 和 $y$。要求满足 $x = i, y = j$ 时从 $S$ 到 $T$ 的最短路恰好是 $d[i][j]$。

想着构造题大概得先确定一下无解的情况，很容易想到若 $x,y$ 变大了最短路还变小肯定不科学，所以从上到下（或从左到右）有递增了则显然无解。

接着想着要从易到难，于是开始考虑只有 $x$ 的情况，想到可以前面连一根，然后后面看差值再用相邻的差的个数的 $x$ 堆起来。

可是这样的话又发现了很多无解的情况，比如等差着等差着公差越来越小了肯定不行，不等差的那个间隔又比前面 $x$ 的个数与 $x$ 的积小了又不行……但大致的想法还是比较清晰，就是连一条过去，然后再连一些边使原来的一些路径“局部短路”。

拓展到二维大概也是这样的情况，就是要建两条边，然后中间七七八八的连很多。可是这样无解的情况就越来越复杂而难以判断，所以我没辙了。

看了题解发现思路挺巧妙的（~~废话，不然怎么叫思维题~~），令 $f[i][j]$ 表示这条路上取 $i$ 个 $x$ 和 $j$ 个 $y$ 的合适的其它路径长度。有了 $f$ 数组，我们就可以连两条长长的链，再把靠近起点的第 $i$ 个和靠近终点的第 $j$ 个就可以了。

因为每一中间的连边都会产生一条 $f[i][j] + i \times x + j \times y$ 的路，所以显然有 $d[x][y] = \min\{f[i][j]+i\times x + j \times y\}$， 所以我们知道有 $f[i][j]+i\times x + j \times y \ge d[x][y]$， 将这个不等式移项可得 $f[i][j] \ge d[x][y] - i \times x - j \times y$, 然后又可以将这个不等式化成有 $\max$ 的形式，即 $f[i][j] = \max\{ d[x][y] - i \times x - j \times y\}$。

感觉这似乎是本题最难想到的部分，刚看题解时我也不理解，所以解释一下。开始的那个式子所有的 $x, y$ 都是确定的，然后枚举了 $i$ 和 $j$, 而第二个式子的 $i, j$ 才是确定的，所以**若存在等号**，那么一定是里面的那些东西取到最大值的时候才取到等号。对，是若存在等号，若不存在，根本就没有解。所以反过来验证一下就可以了。

代码。

```cpp
#include <cstdio>
#include <algorithm>
const int INF = 0x3f3f3f3f;
int n, m, f[105][105], d[15][15];
int main() {
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= n; i++)
		for(int j = 1; j <= m; j++)
			scanf("%d", &d[i][j]);
	for(int i = 0; i < 100; i++)
		for(int j = 0; j < 100; j++)
			for(int x = 1; x <= n; x++)
				for(int y = 1; y <= m; y++)
					f[i][j] = std::max(f[i][j], d[x][y] - i*x - j*y);
	for(int x = 1; x <= n; x++) 
		for(int y = 1; y <= m; y++) {
			int now = INF;
			for(int i = 0; i <= 100; i++)
				for(int j = 0; j <= 100; j++)
					now = std::min(now, f[i][j] + i*x + j*y);
			if(now != d[x][y]) return printf("Impossible"), 0;
		}
	puts("Possible\n202 10401");
	for(int i = 1; i <= 100; i++) printf("%d %d X\n", i, i+1);
	for(int i = 102; i < 202; i++) printf("%d %d Y\n", i, i+1);
	for(int i = 0; i <= 100; i++)
		for(int j = 0; j <= 100; j++) printf("%d %d %d\n", i+1, 202-j, f[i][j]);
	puts("1 202"); 
	return 0;
}
```