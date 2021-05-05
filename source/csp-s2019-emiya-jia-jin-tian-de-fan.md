---
title: '题解 [CSP-S2019] Emiya 家今天的饭'
date: 2021-05-05 07:59:46
tags: [题解,组合计数,动态规划]
published: true
hideInList: false
feature: 
isTop: false
---
很久以前就做过未果的一道题。现在终于给它过了。

<!-- more -->

直接做由于有一列选择的不能超过所有选择的一半的限制，所以不太好做。但直接计算所有的方案非常好做，直接乘法原理即可，因此可以考虑容斥。先计算出没有这一条限制的所有情况，再计算包含了超过一半的方案数。

超过一半的列显然只有一列，所以可以枚举这一列。记它为 $c$。

$f[i][j][k]$ 表示前 $i$ 行该列取了 $j$ 个，而其它列取了 $k$ 个。考虑最后一个放进了哪里就可以得出 $f[i][j][k] = f[i-1][j][k] + f[i-1][j-1][k] \times a[i][c] + f[i-1][j][k-1] \times (s_i - a[i][c])$

但是这样是过不了的，需要继续优化。

其实不需要记录 $j,k$ 的具体数值，只需要它们之间的差就可以了，这样就可以通过此题了。这样为什么是对的呢？它相当于把所有的状态揉在了一起，但揉在一起的原来要分别转移到的揉在一起仍然是这个状态要转移到的地方，所以它是对的。

代码。

```cpp
#include <cstdio>
#include <cstring>
#define int long long
const int N = 105, M = 2005, p = 998244353;
int n, m, a[N][M], s[N], g[N][N], f[N][N*2], d, ans;
signed main() {
	scanf("%lld%lld", &n, &m);
	for(int i = 1; i <= n; i++) 
		for(int j = 1; j <= m; j++) {
			scanf("%lld", &a[i][j]); a[i][j] %= p;
			s[i] += a[i][j], s[i] %= p;
		}
	g[0][0] = 1;
	for(int i = 1; i <= n; i++)
		for(int j = 0; j <= n; j++) 
			g[i][j] = (g[i-1][j] + s[i] * ((j > 0) ? g[i-1][j-1] : 0) % p) % p;
	for(int i = 1; i <= n; i++) ans = (ans + g[n][i]) % p;
	for(int c = 1; c <= m; c++) {
		memset(f, 0, sizeof f);
		f[0][n] = 1;
		for(int i = 1; i <= n; i++)
			for(int j = n-i; j <= n+i; j++)
				f[i][j] = (f[i-1][j] + a[i][c] * f[i-1][j-1] % p + (s[i]-a[i][c]+p) % p * f[i-1][j+1] % p) % p;		
		for(int i = 1; i <= n; i++) ans = (ans - f[n][n+i] + p) % p;
	}
	printf("%lld", ans);
	return 0;
}
```