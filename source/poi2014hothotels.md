---
title: '题解 [POI2014]HOT-Hotels'
date: 2021-04-28 20:39:10
tags: [题解,动态规划,组合计数,树]
published: true
hideInList: false
feature: 
isTop: false
---
看了题解发现我的思路并没有走偏，只是遇到重复这类的问题时没有想到解决的办法。大概要获得解决这样问题的能力只能靠多做题积累吧。

<!-- more -->

> 给定一棵树，在树上选 $3$ 个点，要求两两距离相等，求方案数。

首先有一个显然的性质，那就是只能是三个点从一个点直直的伸出去，即深度较深的两个点距离它们 LCA 的距离相同，也就是说深度要相同了。

那么考虑应该如何统计这样子深度相同的点。直接记录是不行的，因为要保证它们的 LCA 要在当前点，不能产生更深的公共祖先。

既然没法一蹴而就，那么久先来两个的。于是有了两种想法，一个是枚举两个点，再通过什么方式来找到第三个点的个数（倍增似乎可做，但时间上无法接受），另一个是通过神奇的 dp 来解决。~~然后我就没有想法了~~

平常我们的 dp 都是通过一个数组来解决，最多再来一个辅助计算。可是这里难以在一两个数组内解决问题，所以尝试记录更多的信息。可以枚举一个点，把它当做那两个较深的点的 LCA，再一个个枚举它的子树，用 $f[i]$ 记录前面的子树中有两个满足条件的深度为 $i$ 的点的数量，那么再乘上当前子树中深度为 $i$ 的节点数量就肯定满足要求了。

同样的分解还可以继续，令 $g[i]$ 是前面子树中深度为 $i$ 的点数量，$box[i]$ 是当前子树中深度为 $i$ 的点的数量，每次更新的时候让答案先加一个 $f[i] \times box[i]$, 再把 $f[i]$ 加上 $g[i] \times box[i],g[i]$ 加上 $box[i]$ 就可以了。

代码。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
#include <vector>
#define int long long
const int N = 5005;
int x, y, n, f[N], box[N], g[N], ans, lim;
std::vector<int> G[N];
void dfs(int u, int fa, int d) {
	lim = std::max(lim, d);
	box[d] ++;
	for(int i = 0; i < (signed)G[u].size(); i++) {
		int v = G[u][i];
		if(v == fa) continue;
		dfs(v, u, d+1);
	}
}
signed main() {
	scanf("%lld", &n);
	for(int i = 1; i < n; i++) {
		scanf("%lld%lld", &x, &y);
		G[x].push_back(y), G[y].push_back(x);
	}
	for(int i = 1; i <= n; i++) {
		memset(f, 0, sizeof f), memset(g, 0, sizeof g);
		for(int j = 0; j < (signed)G[i].size(); j++) {
			int v = G[i][j];
			lim = 0;
			memset(box, 0, sizeof box);
			dfs(v, i, 1);
			for(int k = 1; k <= lim; k++) {
				ans += f[k] * box[k];
				f[k] += g[k] * box[k];
				g[k] += box[k];
			}
		}
	}
	printf("%lld", ans);
	return 0;
}
```