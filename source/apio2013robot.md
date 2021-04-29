---
title: '题解 [APIO2013]机器人'
date: 2021-04-29 20:10:37
tags: [题解,动态规划,最短路]
published: true
hideInList: false
feature: 
isTop: false
---
这居然也是 dp? 很难想到啊！这个数据范围就只想着搜索了。

这居然还是斯坦纳树？刚做完斯坦纳树模板就不会……神题啊。

<!-- more -->

题目很长，放[传送门](https://www.luogu.com.cn/problem/P3638)。

看到 $9$ 的数据范围认为应该是道搜索题了，但是直接暴力的复杂度大得不得了，猜想应该预处理出些什么，或者有什么性质。但是什么神奇的性质都没有发现。

其实这题还是隐隐约约有一些图论模型在的，因为并不是在这个网格中随便的走，而是要撞上障碍物。所以容易想到可以处理出每一个点向每一个方向走最后会到达的位置。

然后就考验对斯坦纳树的算法的理解了。

在斯坦纳树中，我们定义了一个 $f[i][S]$ 表示 $S$ 状态的 $i$ 为根的最小的权值和，然后先进行了一下常规的转移，即可以用原来结果直接得到的转移，然后通过一次最短路让当前的一些状态去松弛其它的状态。

这道题中也可以用类似的思想在做。

因为题目中要求的是连续区间才能合并成一个新的区间，我们可以用区间 dp 的方式来实现这一点。 $f[i][j][k]$ 表示 $[i,j]$ 中的合成一块儿到 $k$ 号位置的最小代价，然后先直接合并原来的答案，再用 spfa 松弛一遍就好了。~~这 spfa 被卡需要用玄学优化才能过，但我们不管，直接吸氧~~

这不经让我思考：到底什么样的东西才可以用这样的方法来做的呢？

其实关键在于那个 spfa 的转移，因为前面的常规转移在一般的情况下是很容易被想到的。在这两题中，用 spfa 的转移都有一个性质，那就是可能会构成一个环形的转移，并且能搞那什么三角等式。其实这不就是图上最短路的一个特征吗，我们把 dis 看成是 dp 数组，它就没有办法直接转移，因为不知道顺序是怎样的，而又可以松弛，所以就用到最短路的算法了。

最后放上代码吧。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
const int N = 505;
int f[10][10][N*N], h, w, n, left[5], right[5], id[N][N], cnt, tran[N*N][4], p[N*N],
	ans, nowa, nowb, q[N*N], head, tail;
int dx[4] = {0, 0, 1, -1},
	dy[4] = {1, -1, 0, 0};
bool vis[N*N], vi[N*N][4];
char s[N][N];
bool cmp(int a, int b) { return f[nowa][nowb][a] < f[nowa][nowb][b]; }
bool pd(int i, int j) { return (i < 1 || j < 1 || i > h || j > w || s[i][j] == 'x'); }
void init() {
	left[0] = 3, left[3] = 1, left[1] = 2, left[2] = 0;
	right[0] = 2, right[2] = 1, right[1] = 3, right[3] = 0;
	memset(f, 0x3f, sizeof f);
	memset(tran, -1, sizeof tran);
}
void spfa(int x, int y, int s) {
	head = tail = 1;
	q[tail] = s;
	while(head <= tail) {
		int u = q[head];
		vis[u] = 0;
		head++;
		for(int i = 0; i < 4; i++) {
			int v = tran[u][i];
			if(v <= 0) continue;
			if(f[x][y][v] > f[x][y][u] + 1) {
				f[x][y][v] = f[x][y][u] + 1;
				if(!vis[v]) q[++tail] = v, vis[v] = 1;
			}
		}
	}
}
int dfs(int i, int j, int k) {
	if(tran[id[i][j]][k] != -1) return tran[id[i][j]][k];
	if(vi[id[i][j]][k]) return 0;
	vi[id[i][j]][k] = 1;
	int xx = i + dx[k], yy = j + dy[k];
	if(pd(xx, yy)) {
		vi[id[i][j]][k] = 0, tran[id[i][j]][k] = id[i][j];
		return id[i][j];
	}
	if(s[xx][yy] == 'A') {
		tran[id[i][j]][k] = dfs(xx, yy, left[k]);
		vi[id[i][j]][k] = 0;
		return tran[id[i][j]][k];
	}
	else if(s[xx][yy] == 'C') {
		tran[id[i][j]][k] = dfs(xx, yy, right[k]);
		vi[id[i][j]][k] = 0;
		return tran[id[i][j]][k];
	}
	tran[id[i][j]][k] = dfs(xx, yy, k);
	vi[id[i][j]][k] = 0;
	return tran[id[i][j]][k];
}
int main() {
	init();
	scanf("%d%d%d", &n, &w, &h);
	for(int i = 1; i <= h; i++) scanf("%s", s[i]+1);
	for(int i = 1; i <= h; i++)
		for(int j =  1; j <= w; j++)
			if(!pd(i, j)) {
				id[i][j] = ++cnt;
				if(s[i][j] >= '0' && s[i][j] <= '9') 
					f[s[i][j]-'0'][s[i][j]-'0'][cnt] = 0;
			}
	for(int i = 1; i <= h; i++)
		for(int j = 1; j <= w; j++) 
			if(!pd(i, j)) {
				for(int k = 0; k < 4; k++) tran[id[i][j]][k] = dfs(i, j, k);
			}
	for(int i = 1; i <= cnt; i++) p[i] = i;
	for(int len = 1; len <= n; len++)
		for(int i = 1; i+len-1 <= n; i++) {
			int j = i + len - 1;
			for(int k = 1; k <= cnt; k++)
				for(int l = i; l < j; l++)
					f[i][j][k] = std::min(f[i][j][k], f[i][l][k] + f[l+1][j][k]);
			nowa = i, nowb = j;
			std::sort(p+1, p+1+cnt, cmp);
			for(int k = 1; k <= cnt; k++) spfa(i, j, p[k]);
		}
	ans = 2000000000;
	for(int i = 1; i <= cnt; i++) ans = std::min(ans, f[1][n][i]);
	if(ans == 0x3f3f3f3f) puts("-1");
	else printf("%d", ans);
	return 0;
}
```