---
title: '题解 [APIO2010]巡逻'
date: 2021-04-15 08:01:38
tags: [题解,树]
published: true
hideInList: false
feature: 
isTop: false
---
一个有些妙的想法，刷新了我对树的直径的一些认知。

<!-- more -->

> 从 $1$ 号店出发遍历每一条路，再回到起点， 可以添加 $K$ 条边，问最小的经过边的次数。必须要经过加上的边。 $K \in[1, 2]$

考虑加上一条边改变了什么。显然，本来要一个个退回去的现在直达就好了，中间那些边都减掉了，那么减掉的越多越好，把直径俩端点一连完事儿。

重点在如何连第二条。

1. 若环不重合，那么直接再来一条新直径就可以了。
2. 若重合，因为新建的是必需要跑的，环上重合的也就必需跑两次，等于对于那些东西，第一条就白连了。

怎么解决？

很妙：把第一次直径上的边权改成负的然后直接求直径就可以了。

很妙，想不到，但告诉你了就很好理解了。

注意有负权的直径就不能使用 dfs 了，只能 dp，因为证明中那个交点就不能断开重组了，因为可以能一边负的而另一边有很多正的。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
const int N = 100005, M = 2*N;
int n, k, x, y, S, T, len[M], vet[M], next[M], head[N],
	dis[N], f[N], tmax, num, dep[N];
void add(int x, int y, int c) {
	num++;
	vet[num] = y, len[num] = c;
	next[num] = head[x];
	head[x] = num;
}
void dfs(int u, int fa, int &x) {
	if(dis[u] > dis[x]) x = u;
	f[u] = fa;
	for(int i = head[u]; i; i = next[i]) {
		int v = vet[i];
		if(v == fa) continue;
		dis[v] = dis[u] + 1;
		dfs(v, u, x);
	}
}
void change(int u, int v) {
	for(int i = head[u]; i; i = next[i]) {
		int vv = vet[i];
		if(vv == v) { len[i] = -1; break; }
	}
}
void dp(int u, int fa, int &x) {
	int max = -N, sax = -N;
	for(int i = head[u]; i; i = next[i]) {
		int v = vet[i];
		if(v == fa) continue;
		dp(v, u, x);
		dep[u] = std::max(dep[u], dep[v] + len[i]);
		if(dep[v]+len[i] > max) sax = max, max = dep[v]+len[i];
		else if(dep[v]+len[i] > sax) sax = dep[v]+len[i];
	}
	// if(u == 5) printf("*%d %d\n", max, sax);
	if(sax == -N) sax = 0;
	x = std::max(x, max + sax);
}
int main() {
	scanf("%d%d", &n, &k);
	for(int i = 1; i < n; i++) {
		scanf("%d%d", &x, &y);
		add(x, y, 1), add(y, x, 1);
	}
	if(k == 1) {
		dfs(1, 0, S);
		memset(dis, 0, sizeof dis);
		dfs(S, 0, T);
		printf("%d\n", 2*n - dis[T] - 1);
	}
	else {
		dfs(1, 0, S);
		memset(dis, 0, sizeof dis);
		dfs(S, 0, T);
		int L1 = dis[T], L2 = 0;
		for(int now = T; now != S; now = f[now]) change(now, f[now]), change(f[now], now);
		dp(1, 0, L2);
		// printf("*%d\n", L2);
		printf("%d\n", n*2 - L1 - L2);
	}
	return 0;
}
```