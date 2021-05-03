---
title: '题解  [APIO2013]道路费用'
date: 2021-05-04 07:01:11
tags: [题解,生成树,缩点]
published: true
hideInList: false
feature: 
isTop: false
---
涨见识了。神奇的缩点和最小生成树的套路。

<!-- more -->

拿到题目首先得发现并不是一定要把 $k$ 条全部放进最小生成树里才是最优的，然后一看 $k$ 的范围只有 $20$, 那么久可以枚举哪些放进去了。

可这样的复杂度就已经一百多万了，再跑最小生成树的复杂度肯定无法接受。发现最小生成树的边中有大量是重复的。于是可以先把 $k$ 条边都加入最小生成树里面，然后再跑一遍 Kruscal, 就可以得到一定在最小生成树里面的边，把这些边进行缩点，然后就得到了一张点数仅 $20$ 的图。

但边可能由很多，所以我们再对缩点后的原图跑一个最小生成树，就可以得到可能在最小生成树里的边了。这样边数也被限制在了 $k$ 的附近。

然后我们就可以枚举取的边了，但是边权限制最大是多少呢？可以用最小生成树的性质来优化边权，即加入一条边所构成的环里面不能有不在最小生成树上的边比在里面的更小。这样就可以约束边权了。因为范围小，不用倍增，直接爬树就行了。

代码。

```cpp
#include <cstdio>
#include <algorithm>
#include <vector>
#define int long long
const int N = 100005, M = 400005, INF = 0x3f3f3f3f;
int n, m, k, max[N], fae[N], p[N], sum[N], tot, ans, st, id[N], fa[N],
	d[N];
bool in[M];
std::vector<int> g[N], nods;
struct twt {
	int u, v, w;
	bool operator < (twt b) const {
		return w < b.w;
	}
} e[M], T[M];
int find(int x) {
	if(x != id[x]) id[x] = find(id[x]);
	return id[x];
}
void merge(int x, int y) { id[find(x)] = find(y); }
void add(int u, int v) {
	g[u].push_back(v);
	g[v].push_back(u);
}
void Kruscal() {
	for(int i = 1; i <= m; i++) in[i] = 0;
	std::sort(e+1, e+m+1);
	for(int i = 1; i <= m; i++) {
		int fx = find(e[i].u), fy = find(e[i].v);
		in[i] = (fx != fy), merge(fx, fy);
	}
}
void dfs(int u, int f) {
	sum[u] = p[u], fa[u] = f, d[u] = d[f] + 1;
	for(int i = 0; i < (signed)g[u].size(); i++) {
		int v = g[u][i];
		if(v == f) continue;
		dfs(v, u);
		sum[u] += sum[v];
	}
}
void upd(int u, int v, int w) {
	if(d[u] < d[v]) std::swap(u, v);
	while(d[u] > d[v]) max[fae[u]] = std::min(max[fae[u]], w), u = fa[u];
	while(u != v) max[fae[u]] = std::min(max[fae[u]], w), max[fae[v]] = std::min(max[fae[v]], w),
				  u = fa[u], v = fa[v];
}
signed main() {
	scanf("%lld%lld%lld", &n, &m, &k);
	for(int i = 1; i <= m; i++) scanf("%lld%lld%lld", &e[i].u, &e[i].v, &e[i].w);
	for(int i = 1; i <= k; i++) scanf("%lld%lld", &e[m+i].u, &e[m+i].v);
	for(int i = 1; i <= n; i++) scanf("%lld", &p[i]);
	for(int i = 1; i <= n; i++) id[i] = i;
	for(int i = m+1; i <= m+k; i++) merge(e[i].u, e[i].v);
	Kruscal();
	for(int i = 1; i <= n; i++) id[i] = i;
	for(int i = 1; i <= m; i++)
		if(in[i]) merge(e[i].u, e[i].v);
	for(int i = 1; i <= n; i++) 
		if(find(i) == i) nods.push_back(i);
		else p[id[i]] += p[i];
	st = find(1);
	for(int i = 1; i <= m+k; i++) e[i].u = find(e[i].u), e[i].v = find(e[i].v);
	Kruscal();
	for(int i = 1; i <= m; i++) 
		if(in[i]) T[++tot] = (twt){e[i].u, e[i].v, e[i].w};
	for(int i = 1; i <= tot; i++) e[i] = T[i];
	for(int i = 1; i <= k; i++) e[i+tot] = e[i+m];
	m = tot;
	// printf("%lld %lld\n", m, k);
		// for(int i = 1; i <= m+k; i++) printf("%lld %lld\n", e[i].u, e[i].v); //puts("");
	for(int S = 0; S < (1 << k); S++) {
		for(int i = 0; i < (signed)nods.size(); i++) id[nods[i]] = nods[i], g[nods[i]].clear();
		for(int i = 1; i <= k; i++)
			if((1 << (i-1)) & S) merge(e[i+m].u, e[i+m].v);
		Kruscal();
		for(int i = 1; i <= k; i++) 
			if((1 << (i-1)) & S) add(e[i+m].u, e[i+m].v), in[i+m] = 1;
			else in[i+m] = 0;
		int ecnt = 0;
		for(int i = 1; i <= m+k; i++) ecnt += in[i];
		if(ecnt > (signed)nods.size()-1) continue;
		for(int i = 1; i <= m; i++) if(in[i]) add(e[i].u, e[i].v);
		dfs(st, 0);
		for(int i = 1; i <= m+k; i++) 
			if(in[i]) {
				int u = e[i].u, v = e[i].v;
				if(fa[u] == v) fae[u] = i;
				else fae[v] = i;
				max[i] = (i <= m) ? 0 : INF;	
			}
		int an = 0;
		for(int i = 1; i <= m; i++)
			if(!in[i]) upd(e[i].u, e[i].v, e[i].w);
		for(int i = m+1; i <= m+k; i++)
			if(in[i]) an += sum[(fa[e[i].u] == e[i].v) ? e[i].u : e[i].v] * max[i];
		ans = std::max(ans, an);
		// printf("%lld\n", S);
	}
	printf("%lld", ans);
	return 0;
}
```

去磕这样的题感觉没有什么提高而又很大程度降低了做题效率，所以今天开始改一下 SOP 吧。