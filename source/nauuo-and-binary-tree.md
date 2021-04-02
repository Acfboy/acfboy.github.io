---
title: '题解 [CF -1172D]Nauuo and Binary Tree'
date: 2021-04-02 07:21:17
tags: [树,树链剖分 ,题解]
published: true
hideInList: false
feature: 
isTop: false
---
清新的好题，考验了对树剖的总和理解和运用。

只可惜 [赛前四天撞题](https://ouuan.github.io/post/bad-round-%E4%B8%8E%E5%87%BA%E9%A2%98%E4%BA%BA%E7%9A%84%E5%9D%9A%E5%AE%88/)，这道题没能出现在真正的 CF 里。

<!-- more -->

> **交互题**。$n$ 个节点的二叉树，每次可以询问两个点的距离，求这个二叉树的结构。树的大小不超过 $3000$, 可以询问 $30000$ 次。

看到这个题面我是没有一点想法，甚至连暴力做法都不太清楚。

看了题解直呼好题。

1. 首先可以询问出每一个点的深度，这样可以按照深度确定点，从而确保去确定一个点的时候它的祖先都已经确定了。

2. 每次确定一个点时
	1. 先用 dfs 维护已知树上的轻重链。注意，因为是二叉树，所以一个点若有两个儿子，那么必定是一条轻边一条重边。如果只有一个儿子，我们规定连向它的是重边。这个性质非常有用。
	2. 在 $u$ 的子树中找 $k$ 时，若 $u$ 已经没有儿子了，肯定 $u$ 是 $k$ 的父亲。
	3. 否则询问当前的 $k$ 和 $u$ 所在重链底部的距离记作 $d$, 通过类似通过 $\texttt{LCA}$ 求两点距离的那个式子可以求出 $\texttt{LCA}$ 的深度，进而求出 $\texttt{LCA}$, 然后就可以确定 $k$ 所在的新子树，递归求解。
  	4. 如果 $u$ 只有一个儿子，那它就是 $k$ 了，因为只有一个儿子那么肯定是重边，那么 $u$ 所在重链的最底端也是它自己。
    
具体可以见下图。

![](https://oi-wiki.org/graph/images/hld2.png)

时间复杂度是 $\mathcal O(n^2)$ 的，询问由于每次都会确定一个子树，所以是 $\log$ 级别的，具体分析可以去看 OI-wiki

代码。

```cpp
#include <iostream>
#include <algorithm>
const int N = 3005;
struct twt {
	int dep, id;
	bool operator < (twt b) const {
		return dep < b.dep;
	}
} a[N];
int n, ch[N][2], son[N], fa[N], bot[N], deep[N], size[N];
void Print() {
	std::cout << "! ";
	for(int i = 2; i <= n; i++) std::cout << fa[i] << " ";
	std::cout << std::endl;
}
int ask(int u, int v) {
	std::cout << "? " << u << " " << v << std::endl;
	int d;
	std::cin >> d;
	return d;
}
void add(int u, int v) {
	fa[v] = u;
	if(ch[u][0] != 0) ch[u][1] = v;
	else ch[u][0] = v;
}
void dfs(int u) {
	if(ch[u][0]) dfs(ch[u][0]);
	if(ch[u][1]) dfs(ch[u][1]);
	size[u] = size[ch[u][0]] + size[ch[u][1]] + 1;
	if(ch[u][1] != 0) son[u] = size[ch[u][1]] > size[ch[u][0]];
	else son[u] = 0;
	if(ch[u][son[u]] != 0) bot[u] = bot[ch[u][son[u]]];
	else bot[u] = u;
}
void solve(int u, int k) {
	if(ch[u][0] == 0) {
		add(u, k);
		return;
	}
	int d = ask(k, bot[u]), v = bot[u];
	while(deep[v] > (deep[k] + deep[bot[u]] - d) / 2) v = fa[v];
	int w = ch[v][son[v] ^ 1];
	if(w != 0) solve(w, k);
	else add(v, k);
}
int main() {
	std::cin >> n;
	for(int i = 2; i <= n; i++) {
		a[i].id = i;
		a[i].dep = ask(1, i);
		deep[i] = a[i].dep;
	}
	std::sort(a+2, a+1+n);
	for(int i = 2; i <= n; i++) {
		dfs(1);
		solve(1, a[i].id);
	}
	Print();
	return 0;
}
```