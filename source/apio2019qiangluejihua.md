---
title: '题解 [APIO2009]抢掠计划'
date: 2021-04-15 07:32:18
tags: [题解,缩点]
published: true
hideInList: false
feature: 
isTop: false
---
很久很久以前就想学缩点，现在因为这题终于学会了 tarjan 求强联通分量和缩点。

<!-- more -->

> 给定一个有向图，每个节点都可以通过无限次，但上面的权值只能加一次，求从 $1$ 开始到一些点结束经过的节点权值和最大。

如果这是一张有向无环图，那么我们肯定可以利用拓扑排序来愉快地 dp 求出这个答案。

但这不是一张有向无环图，缩点就是这样一种算法，可以将图中互相联通的一坨点缩成一个，使图变成 DAG，让你可以愉快地 dp。

现在来讲一讲 tarjan 算法。

这个算法的主要思想是这样的：

1. dfs 一遍，记录 dfs 序，记作 `dfn`, 记录当前一个点可以连到最小 dfs 序，记作 `low`
2. 在 dfs 的过程中，先标记当前点被访问，然后将其加入栈中，更新时若连到的点没有 dfs 过，那么就 dfs 下去，然后更新 `low`, 不然只更新 `low`。
3. 若 `dfn == low` 那么现在栈中的点就是一个强联通分量中的点了，把它们弹出做你想要的操作，然后都标记成未访问就可以了。

这样的做法为什么是对的？

首先，如果连到了当前 dfs 到这点的路径上的点，那么肯定可以回去再来就是互相到达了，所以我们把点都塞进栈中，并且用 `low` 来做到识别是否连回去到更早的。

如果不是在 dfs 树上最早被抵达的强联通分量的点肯定 `dfn` 不和 `low` 相等，反之肯定相等，我们可以用这个性质来缩点，把其它点的性质都加入到这个最早被访问到的点上。栈就是为了记录这样的一些点。

然后为什么要把强联通分块中的标记成未访问呢？因为搜索树上不是向自己的祖先，而是横叉出去的边肯定不会和其它边构成环，所以把它们都取消可以成为强联通分量的资格，从候选的栈中弹出。

会缩点了一个就是一个拓扑排序和 dp 就可以了。

代码。

```cpp
#include <cstdio>
#include <queue>
#include <stack>
const int N = 500005, M = N;
int vet[M], uet[M], next[M], head[N], vet2[M], next2[M], head2[N], num, num2,
	low[N], dfn[N], vis[N], c[N], scc[N], n, m, x, y, in[N], sum[N], tim, s, p;
std::queue<int> q;
std::stack<int> st;
void add(int u, int v) {
	vet[++num] = v; uet[num] = u;
	next[num] = head[u];
	head[u] = num;
}
void add2(int u, int v) {
	in[v] ++;
	vet2[++num2] = v; 
	next2[num2] = head2[u];
	head2[u] = num2;
}
void dfs(int u) {
	low[u] = dfn[u] = ++tim;
	st.push(u);
	vis[u] = 1;
	for(int i = head[u]; i; i = next[i]) {
		int v = vet[i];
		if(!dfn[v]) {
			dfs(v);
			low[u] = std::min(low[u], low[v]);
		}
		else if(vis[v]) low[u] = std::min(low[u], low[v]);
	}
	if(dfn[u] == low[u]) {
		int now;
		while(!st.empty()) {
			now = st.top();
			st.pop();
			scc[now] = u;
			vis[now] = 0;
			if(now == u) break;
			c[u] += c[now];
		}
	}
}
int topo() {
	q.push(scc[s]);
	sum[scc[s]] = c[scc[s]];
	while(!q.empty()) {
		int u = q.front(); q.pop();
		for(int i = head2[u]; i; i = next2[i]) {
			int v = vet2[i];
			sum[v] = std::max(sum[v], sum[u] + c[v]);
			in[v] --;
			if(in[v] == 0) q.push(v);
		}
	}
	int ans = 0;
	for(int i = 1; i <= p; i++) {
		scanf("%d", &x);
		ans = std::max(ans, sum[scc[x]]);
	}
	return ans;
}
int main() {
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= m; i++) {
		scanf("%d%d", &x, &y);
		add(x, y);
	}
	for(int i = 1; i <= n; i++) scanf("%d", &c[i]);
	scanf("%d%d", &s, &p);
	dfs(s);
	for(int i = 1; i <= n; i++) {
		if(scc[i] == 0) continue;
		for(int j = head[i]; j; j = next[j]) 
			if(scc[vet[j]] && scc[vet[j]] != scc[i]) add2(scc[i], scc[vet[j]]);
	}
	printf("%d", topo());
	return 0;
}
```