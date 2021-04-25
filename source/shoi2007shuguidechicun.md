---
title: '题解 [SHOI2007]书柜的尺寸'
date: 2021-04-25 20:30:05
tags: [题解,动态规划]
published: true
hideInList: false
feature: 
isTop: false
---
被我想歪掉的 dp。

<!-- more -->

> 给定 $n$ 个元素，每一个有 $h_i$ 和 $t_i$, 把它们分成三组，使 $h$ 最大值的和 与 $t$ 和的最大值的乘积最大。

一看数据范围只有 $n \le 70$ 胆子就大了起来，直接就 $C_n^3$ 取出当做最大值的三个，然后先当作按照顺序从这三个断点划分，这样唯一能进行的调整就是把后面的换到前面。然后再来一个 dp, $f[i][j][k]$ 表示能变的前 $i$ 个变动后第一组 $t$ 值为 $j$, 第二组 $t$ 值为 $k$ 是否可行，然后转移一波再验证就好了。顺利成章，马上要开始写了——不对，$\mathcal O(C_n^3n s^2)$ 的复杂度没法接受啊……

等等！既然我排了序再搞这些名堂，为什么不直接排了序以后 dp? 这向上调换不就等于后面的能取到前面去吗？至于 $h$, 当作 dp 的值然后转移时若是从 $0$ 转移出去的就加上 $h$ 不就完了吗？

滚动一下，控制好空间。时限 $5s$, 可以通过此题。

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
#define int long long
const int N = 75, S = 2105, INF = 0x3f3f3f3fll;
struct twt {
	int h, t;
	bool operator < (twt b) const {
		return h > b.h;
	}
} a[N];
int n, sum[N], f[2][S][S], ans;
void ckmin(int &x, int y) { x = std::min(x, y); }
int max(int x, int y, int z) { return std::max(x, std::max(y, z)); }
signed main() {
	scanf("%lld", &n);
	for(int i = 1; i <= n; i++) scanf("%lld%lld", &a[i].h, &a[i].t);
	std::sort(a+1, a+1+n);
	for(int i = 1; i <= n; i++) sum[i] = sum[i-1] + a[i].t;
	for(int i = 0; i < S; i++)
		for(int j = 0; j < S; j++) f[0][i][j] = f[1][i][j] = 0x3f3f3f3f;
	f[1][0][0] = f[0][0][0] = 0;
	int now, pre;
	for(int i = 1; i <= n; i++) {
		now = i & 1, pre = now ^ 1;
		for(int j = 0; j < S; j++)
			for(int k = 0; k < S; k++) f[now][j][k] = 0x3f3f3f3f;
		for(int j = 0; j <= sum[i]; j++)
			for(int k = 0; k <= sum[i]; k++) {
				if(f[pre][j][k] == INF) continue;
				if(j == 0) ckmin(f[now][j+a[i].t][k], f[pre][j][k] + a[i].h);
				else ckmin(f[now][j+a[i].t][k], f[pre][j][k]);
				if(k == 0) ckmin(f[now][j][k+a[i].t], f[pre][j][k] + a[i].h);
				else ckmin(f[now][j][k+a[i].t], f[pre][j][k]);
				if(sum[i-1]-j-k == 0)ckmin(f[now][j][k], f[pre][j][k] + a[i].h);
				else ckmin(f[now][j][k], f[pre][j][k]);
			}
	}
	int ans = INF;
	for(int i = 1; i <= sum[n]; i++)
		for(int j = 1; j <= sum[n]; j++)
			if(sum[n]-i-j != 0) ckmin(ans, max(i, j, sum[n]-i-j)*f[now][i][j]);
	printf("%lld", ans);
	return 0;
}
```