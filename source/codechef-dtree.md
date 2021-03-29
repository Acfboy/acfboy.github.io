---
title: '题解 CodeChef-DTREE'
date: 2021-03-29 19:54:26
tags: [题解,树]
published: true
hideInList: false
feature: https://img-blog.csdn.net/20180715124201842?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZvcmV2ZXJfZHJlYW1z/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70
isTop: false
---
考验对数的直径的 dp 做法变形处理的好题目。
<!-- more -->


> 给定一棵树，求去掉一个点构成的森林中树的直径的最大值，输出对于对于每一个点，去掉它的结果。

先想一想我们原来求树的直径的做法是怎么样的，能求出些什么。

原来就是找出一个点向下最深的和次深的将它们连在一起，找出最大的。所以我们用原来的做法，假设删掉 $u$, 可以很容易地求出它的所有儿子所在子树的直径，以及它们的最大值，所以现在只需要考虑 $u$ 向上的子树的直径怎么计算。

上面的直径没有办法直接计算，那么考虑它有哪些可能。参考原来的做法，可以想到从一个点的父亲出发一条向上的链再和经过这个点的父亲的某条向下的链可能可以拼成上面树的直径。所以先考虑向上的最长的链怎么求。

令 $up_u$ 表示以 $u$ 为起点向上最长的链的长度，可得一种情况是直接由原来向上的加一拼接而成，另一种情况是在这个点父亲那点转向了下方。如图，注意 $3$ 指的是要更新点的父亲。

[![cCz5dA.png](https://z3.ax1x.com/2021/03/29/cCz5dA.png)](https://imgtu.com/i/cCz5dA)

那么这条链当然是要和经过 $3$ 号点的另一条链拼成的，所以我们需要记录除了父亲到当前点所在子树以外的最长的链和它拼在一起。

当然，也有可能不向上延伸，就是当前点兄弟子树向下的最大直径，或上面本来就有的直径。

还有可能是经过当前点的父亲拼起来的两条向下的最大的链。

把它们画在一起就是这个样子的。

[![cP9MrD.png](https://z3.ax1x.com/2021/03/29/cP9MrD.png)](https://imgtu.com/i/cP9MrD)

然后再用下面的更新就可以了。

因为枚举的点是要空出来的，所以我们用当前点计算出来的一堆值其实是更新下面的点用的。

注意判断当前的这个子树是 最大/次大 的情况，选其它的最大和次大不能把当前选进去，所以还需要取出 次次大，在最大或次大被占用时作为次大使用。


代码。

注意代码中向下的深度 最大/次大/次次大 值初值都为 $-1$ 是因为下面都计算了它们的根到当前点的连边，找不到的时候一加上恰好为 $0$, 而下面原有的子树(即 `maxf`) 初值是 $0$ 因为它们不需要有加上的运算，而是直接使用。

```cpp
#include <cstdio>
#include <algorithm>
#include <cstring>
#include <vector>
const int N = 100005;
int T, n, x, y, ans[N], down[N], f[N], up[N];
std::vector<int> g[N];
void init() {
	memset(down, 0, sizeof down);
	memset(up, 0, sizeof up);
	memset(f, 0, sizeof f);
	memset(ans, 0, sizeof ans);
	for(int i = 1; i < N; i++) g[i].clear();
}
void update2(int x, int &max, int &sax) {
	if(x > max) sax = max, max = x;
	else if(x > sax) sax = x;
}
void update3(int x, int &max1, int &max2, int &max3) {
	if(x > max1) max3 = max2, max2 = max1, max1 = x;
	else if(x > max2) max3 = max2, max2 = x;
	else if(x > max3) max3 = x;
}
void dfs1(int u, int fa) {
	int max = 0, sax = -1;
	for(int i = 0; i < (signed)g[u].size(); i++) {
		int v = g[u][i];
		if(v == fa) continue;
		dfs1(v, u);
		update2(down[v]+1, max, sax);
		f[u] = std::max(f[u], f[v]);
	}
	down[u] = max;
	f[u] = std::max(f[u], max + sax);
}
void dfs2(int u, int fa) {
	int maxd1 = -1, maxd2 = -1, maxd3 = -1,
		maxf1 = 0, maxf2 = 0;
	int tmp = ans[u]; // 记下仅上面部分的答案，以便更新需要
	for(int i = 0; i < (signed)g[u].size(); i++) {
		int v = g[u][i];
		if(v == fa) continue;
		ans[u] = std::max(ans[u], f[v]);
		update3(down[v], maxd1, maxd2, maxd3);
		update2(f[v], maxf1, maxf2);
	}
	for(int i = 0; i < (signed)g[u].size(); i++) {
		int v = g[u][i];
		if(v == fa) continue;
		up[v] = std::max(up[u] + 1, (down[v] == maxd1) ? maxd2 + 2 : maxd1 + 2); // 求出向上的最长链
		ans[v] = tmp; // 上面本来就有的 
		ans[v] = std::max(ans[v], (f[v] == maxf1) ? maxf2 : maxf1); // 兄弟本来就有的情况
		ans[v] = std::max(ans[v], (down[v] == maxd1) ? (maxd2 + up[u] + 1) : (up[u] + maxd1 + 1)); // 上下拼起来的情况
		if(down[v] == maxd1) ans[v] = std::max(ans[v], maxd2 + maxd3 + 2); 
		else if(down[v] == maxd2) ans[v] = std::max(ans[v], maxd1 + maxd3 + 2);
		else ans[v] = std::max(ans[v], maxd1 + maxd2 + 2);
      // 经过这点下面的链拼起来的情况
		dfs2(v, u);
	}
}
int main() {
	scanf("%d", &T);
	while(T--) {
		init();
		scanf("%d", &n);
		for(int i = 1; i < n; i++) {
			scanf("%d%d", &x, &y);
			g[x].push_back(y), g[y].push_back(x);
		}
		dfs1(1, 0);
		dfs2(1, 0);
		for(int i = 1; i <= n; i++) printf("%d ", ans[i]);
		puts("");
	}
	return 0;
}
```