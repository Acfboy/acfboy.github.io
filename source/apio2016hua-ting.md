---
title: '题解 [APIO2016]划艇'
date: 2021-05-08 15:19:53
tags: [题解,动态规划,组合计数]
published: true
hideInList: false
feature: 
isTop: false
---
自己做的时候还是很懵的，觉得这个状态都开不下，完全不可行。最后的解决方式还是挺妙的。

<!-- more -->

> $n$ 组球，第 $i$ 组球可以取  $a_i$ 到 $b_i$ 个，也可以不取，但取出来的都要递增，每一组里的所有球都是一模一样的。求最后取球的方案数。

首先想到的当然是 $f[i][j]$ 表示当前取第 $i$ 个，上次取的是 $j$ 个的方案数。转移的时候要判断一下 $j$ 在不在 $a_i$ 到 $b_i$ 的范围内，另外因为可以不取，所以还得枚举上一个 $i$ 和 $j$。

然后别谈时间了，光这样一个状态就遇上了严重的问题，因为 $j$ 可以到 $10^9$, 根本开不下。而输入只有 $1000$ 个数，所以想到离散化，但可以取中间的啊，若要以一段为状态，那么就要一下子计算其中所有的答案。

有一个结论：

> 在 $[0,L]$ 中选出一个长度为 $n$ 的非零的严格递增的数列有多少种方案。

若没有 $0$ 直接 ${L \choose n}$ 就可以了，因为选出来一些数后答案就唯一了。那么有 $0$ 呢？可以构造一个序列，长度为 $L+n$, 前面有 $n$ 个 $0$, 这样取 $n$ 个若取到 $0$ 就当成第几个取 $0$, 否则就是填上取到的数，这样就相当于选出长度为 $n$ 的串且保证选出的非 $0$ 的严格递增了。

这个结论其实就是在计算前面选出哪些是非 $0$ 的， 因为下标是严格递增的。

既然是一段一段的，那我们就把状态用输入的这些数岔开，因为这些块肯定是可以一起记录和转移的，所以这样做是没事的。$f[i][j]$ 现在表示现在选到第 $i$ 个，且取的数量落在第 $j$ 个区间中的方案数，然后可以得到转移。

$$
f_{i,j} = \sum_{k=1}^{j-1} \sum_{t=0}^{i-1} {L+m-1 \choose m} f_{t,k}
$$

这里的 $m$ 是 $t+1 \to  i$ 中能选区间 $j$ 的数量，枚举的 $k$ 是上一个有选的区间， $t$ 是枚举上一个选的组。

然后这个东西可以用前缀和优化。

代码。

```cpp
#include <cstdio>
#include <algorithm>
#define int long long
const int N = 505, p = 1000000007;
int n, tot, inv[N], C[N], g[N], a[N], b[N], num[N*2], ans;
signed main() {
	scanf("%lld", &n);
	inv[1] = 1;
	for(int i = 2; i <= n; i++) inv[i] = (p - p/i) * inv[p % i] % p;
	for(int i = 1; i <= n; i++) {
		scanf("%lld%lld", &a[i], &b[i]);
		num[++tot] = a[i], num[++tot] = b[i] + 1;
	}
	std::sort(num+1, num+1+tot);
	tot = std::unique(num+1, num+1+tot) - num - 1;
	for(int i = 1; i <= n; i++) {
		a[i] = std::lower_bound(num+1, num+1+tot, a[i]) - num; 
		b[i] = std::lower_bound(num+1, num+1+tot, b[i]+1) - num;
	}
	C[0] = 1, g[0] = 1;
	for(int j = 1; j < tot; j++) {
		int L = num[j+1] - num[j];
		for(int i = 1; i <= n; i++) C[i] = C[i-1] * (L+i-1) % p * inv[i] % p;
		for(int i = n; i >= 1; i--)
			if(a[i] <= j && j+1 <= b[i]) {
				int f = 0, m = 1, c = L;
				for(int t = i-1; t >= 0; t--) {
					f = (f + c * g[t] % p) % p;
					if(a[t] <= j && j+1 <= b[t]) c = C[++m];
				}
				g[i] = (g[i] + f) % p;
			}
	}
	for(int i = 1; i <= n; i++) ans = (ans + g[i]) % p;
	printf("%lld", ans);
	return 0;
}
```