---
title: '题解 逛庙会'
date: 2021-04-17 08:07:58
tags: [题解,状态压缩,动态规划]
published: true
hideInList: false
feature: 
isTop: false
---
[传送门](https://www.luogu.com.cn/problem/P2238)

状压 dp 的好题目，巧妙利用状压传递了有后效性的信息从而消除了后效性。

<!-- more -->


这题如果没有取周围的那么很容易想到一个 dp, 但由于会取周围的而且还要选出 $r-1$ 个，所以有后效性，尝试通过改变 dp 的顺序来消除后效性，但是很遗憾，没有用。

~~所以上 老板dp/快速谭炜谭变换 吧。~~

然后我就没辙了，去看了 kkk 的题解，不得不说 kkk 的题解写得 ~~糟透了~~ **真的好**，非常考验读者的思维能力，不自己想根本就看不懂。

另一个解决后效性的方式：将需要记录的信息加入到状态中。

但知道了这一点要做这一题还是非常有难度，这题的状态还是很妙的。

考虑什么情况下是这题的难点，比如说要走如下的路径：

$$
\begin{array}{c}
\rightarrow & \rightarrow \\
\textsf{twt} & \downarrow
\end{array}
$$

那么 `twt` 的价值在两次中都可能被算进去，应该如何把取/没取的信息传递下去呢？这里有一个非常妙的做法，就是对于一个点，我们状压它 左下/下/右/右上 的状态。

这样子有什么的好处？那就是我们可以传递这样的一个状态了！

向右转移的时候必须保证其左下和原来的下是一样的，向下转移时保证新的右和原来的右下是一样的，这样子就可以把转角的信息（就是 `twt`）给不断传递了。

记得需要判断的是当前经过位置是一定要取的，然后还要取超过一个。

转移的时候只要加上相应的东西就可以了，因为剩下的都在其它的状态中判断过取不取的问题了，由于取了 $\min$, 可以保证其正确性。

代码：

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
const int N = 1005;
int n, m, sh[N][N], f[N][N][16];
int cnt[16] = {0, 1, 1, 2, 1, 2, 2, 3, 1, 2, 2, 3, 2, 3, 3, 4}; 
// 二进制中 1 的个数
char s[N];
int main() {
	scanf("%d%d", &n, &m);
	memset(f, 0x3f, sizeof f);
	for(int i = 1; i <= n; i++) {
		scanf("%s", s+1);
		for(int j = 1; j <= m; j++) sh[i][j] = (s[j] == '.' ? 0 : (s[j] - '0'));
	}
	f[1][1][15] = 0;
	for(int i = 1; i <= n; i++) 
		for(int j = 1; j <= m; j++)
			for(int k = 0; k < 16; k++) {
				for(int k2 = 0; k2 < 16; k2++) {
					if(((k&4)==0) != ((k2&8)==0) || !(k&2) || cnt[k&1] + cnt[k2&6] <= 1) continue;
					int cost = f[i][j][k];
					if(k2 & 1) cost += sh[i+2][j-1];
					if(k2 & 2) cost += sh[i+2][j];
					if(k2 & 4) cost += sh[i+1][j+1];
					f[i+1][j][k2] = std::min(f[i+1][j][k2], cost);
				}
				for(int k2 = 0; k2 < 16; k2++) {
					if(((k&2)==0) != ((k2&1)==0) || !(k&4) || cnt[k&8] + cnt[k2&6] <= 1) continue;
					int cost = f[i][j][k];
					if(k2 & 8) cost += sh[i-1][j+2];
					if(k2 & 4) cost += sh[i][j+2];
					if(k2 & 2) cost += sh[i+1][j+1];
					f[i][j+1][k2] = std::min(f[i][j+1][k2], cost);
				}
			}
	printf("%d", f[n][m][15]);
	return 0;
}
```