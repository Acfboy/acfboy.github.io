---
title: '题解 上帝造题的七分钟2'
date: 2021-04-15 07:33:23
tags: [题解,并查集,树状数组]
published: true
hideInList: false
feature: 
isTop: false
---
这题和上次有一场 CF 的 C 题有那么一些异曲同工之妙。

还巧妙运用了并查集。

<!-- more -->

> 输入一个序列，维护两种操作，查询一段区间内数的和 以及 对一段区间内的所有数开方。

虽然不会支持开方的神奇数据结构，但开方是一个很好的运算，因为开方没几次就会到 $1$ 以下，这样就没有继续开方的必要了。

所以考虑怎么跳过这样的一个过程。

大概是要使用一种类似链表的东西，如果变成 $1$ 或 $0$ 了就直接链接到下一个，并且能够把这种下一个的都合起来到最后，那么并查集就非常的合适了，路径压缩的过程就是合起来的过程。

这里还有一个小 trick，那就是如果一个变成 $1$, 把它之前的连到后面的可能会有更多细节问题要处理，比如说这样的连接交叉的时候。那么其实跳过不用这么严格，头尾都改一下也就两次，所以只需要把这个连接到下一个就可以了。

代码。

```cpp
#include <cstdio>
#include <algorithm>
#include <cmath>
#define int long long
const int N = 100005;
int a[N], fa[N], n, q, opt, l, r, t[N];
int Query(int x) {
	int an = 0;
	while(x) {
		an += t[x];
		x -= x & -x;
	}
	return an;
}
void add(int p, int x) {
	while(p <= n) {
		t[p] += x;
		p += p & -p;
	}
}
int find(int x) {
	if(x != fa[x]) fa[x] = find(fa[x]);
	return fa[x];
}
signed main() {
	scanf("%lld", &n);
	for(int i = 1; i <= n; i++) scanf("%lld", &a[i]), fa[i] = i, add(i, a[i]);
	scanf("%lld", &q);
	while(q--) {
		scanf("%lld%lld%lld", &opt, &l, &r);
		if(l > r) std::swap(l, r);
		if(opt == 1) printf("%lld\n", Query(r) - Query(l-1));
		else {
			int i = l;
			while(i <= r) {
				add(i, (int)sqrt(a[i]) - a[i]);
				a[i] = (int)sqrt(a[i]);
				fa[i] = (a[i] <= 1) ? (i+1) : i;
				i = (fa[i] == i) ? (i+1) : find(fa[i]);
			}
		}
	}
	return 0;
}
```