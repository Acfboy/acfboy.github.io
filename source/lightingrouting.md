---
title: '题解 [2019 ICPC 上海网络赛] Lighting Routing I'
date: 2021-03-31 08:26:37
tags: [题解 ,树,线段树]
published: true
hideInList: false
feature: https://tse4-mm.cn.bing.net/th/id/OIP.y-5OPQgq9ZOhTuRuFwHIcgHaHD?pid=ImgDet&rs=1
isTop: false
---
~~封面图是随便找的真实的 Lighting Routing~~

据说这题有神奇的 LCT 和 树套树 的做法，很可惜，我都不会。

于是搞了一个用欧拉序来维护直径的做法，搭配线段树和倍增 LCA 来解决这题。

<!-- more -->

> 支持边的修改操作，每次询问树上离一个点最远的点和它的距离。

我们知道从树上任意一个点出发，能到达的最远的点一定是树的直径的一个端点，所以这题也就是要动态维护树的直径，然后求一下距离就可以了。

那么怎么维护树的直径呢？这里介绍一下一个神奇的做法，欧拉序。

欧拉序是一个类似 DFS 序的东西，不同的是，欧拉序在遍历完一个点的子树的时候还会把这个点加入欧拉序中，相当于一个人沿 DFS 的顺序走完树上所有点的路径。欧拉序的长度是 $2n-1$, 因为每一条边两个端点都要算一次，最后退出的时候根节点过算了一次。

求得欧拉序的代码, 其中 `pos[(i+1)/2] = v` 是将输入边的编号和指向的点对应起来，因为是双向建边的，所以要除以二上取整。而且这样子给这条边定的指向的点是以我们定的根为根的，这个处理其实挺妙。

`tin` 和 `tout` 用来记录一个子树在欧拉序中的起点和终点。

```cpp
void dfs(int u, int fa) {
	dfn[++tot] = u; tin[u] = tot;
	for(int i = head[u]; i; i = next[i]) {
		int v = vet[i];
		if(v == fa) continue;
		pos[(i+1)/2] = v, dep[v] = dep[u] + len[i];
		dfs(v, u);
		dfn[++tot] = u;
	}
	tout[u] = tot;
}
```

那么使用欧拉序有什么好处呢？

在两个点的欧拉序之间，深度最小的一定就是它们的 LCA 了，这很好理解，因为欧拉序就是走过每一个点的路径嘛，你要走过两个点，肯定是要先到一个，然后退出来，再接着到第二个。这样就把 LCA 的问题转换成了 RMQ 问题。因为欧拉序也具有 dfs 序子树都在一起的性质，所以给一条边的边权加上一个数就是给一段区间的 `dep` 都加上一个数。

然后考虑在欧拉序上维护直径。

我们在树形 DP 求直径的时候是到每一个点把它们最深的两个拼在了一起，链的长度是两个点的距离，即 $dep_x + dep_y - 2dep_{lca}$ ，经过每一个点时，就是要让这个东西最大，或者直接采用下面的答案。

可以用线段树来维护这样几个东西。

- 最大的 $dep$, 记作 `maxd`
- 最大的 $-2dep_{lca}$, 记作 `vlca`
- 最大的左边和中间拼起来的，即最大的 $dep_x - 2dep_{lca}$, 记作 `lm`
- 同理 `mr`
- 还有当前子树的直径 `lmr`

因为一段欧拉序代表的都是树的一部分，所以可以这样来维护，也可以合并。
`pushup` 的时候 `maxd` 和 `vlca` 显然都是子树取 $\max$ 就可以了，`lm` 要考虑左边的 `maxd` 和 `vlca` 拼成或直接用下面的答案，`mr` 同理， `lmr` 要考虑直接用下面的答案或左边的 `lm` 拼右边的 `maxd` 或者右边的 `mr` 配左边的 `maxd`。

但这样怎么记录方案呢？我们可以参考 dp 的时候记录方案的做法，从哪边转移过来就继承哪边的结果。然后我们就可以得到以下的 `pushup` 代码。

```cpp
void pushUp(int p) {
	maxd[p] = std::max(maxd[p+p], maxd[p+p+1]);
	vlca[p] = std::max(vlca[p+p], vlca[p+p+1]);
	lm[p] = std::max(std::max(lm[p+p], lm[p+p+1]), maxd[p+p] + vlca[p+p+1]);
	mr[p] = std::max(std::max(mr[p+p], mr[p+p+1]), vlca[p+p] + maxd[p+p+1]);
	lmr[p] = std::max(std::max(lmr[p+p], lmr[p+p+1]), std::max(lm[p+p]+maxd[p+p+1], maxd[p+p]+mr[p+p+1]));
	
	if(maxd[p] == maxd[p+p]) vd[p] = vd[p+p];
	else vd[p] = vd[p+p+1];
	
	if(lm[p] == lm[p+p]) vl[p] = vl[p+p];
	else if(lm[p] == lm[p+p+1]) vl[p] = vl[p+p+1];
	else vl[p] = vd[p+p];
	
	if(mr[p] == mr[p+p]) vr[p] = vr[p+p];
	else if(mr[p] == mr[p+p+1]) vr[p] = vr[p+p+1];
	else vr[p] = vd[p+p+1];
	
	if(lmr[p] == lmr[p+p]) s[p] = s[p+p], t[p] = t[p+p];
	else if(lmr[p] == lmr[p+p+1]) s[p] = s[p+p+1], t[p] = t[p+p+1];
	else if(lmr[p] == lm[p+p] + maxd[p+p+1]) s[p] = vl[p+p], t[p] = vd[p+p+1];
	else s[p] = vd[p+p], t[p] = vr[p+p+1];
}
```

这是整道题的核心。

剩下的部分就是普通的线段树以及求两点间距离了(线段树中单个点的 `maxd` 其实就是 `dep`)， 不难实现， 要注意区分欧拉序编号和原始编号，我为这个调了很久。

[完整代码链接](https://vjudge.net/solution/30295703)