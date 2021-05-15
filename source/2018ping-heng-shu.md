---
title: '题解 [CCC 2018]平衡树'
date: 2021-05-15 09:05:01
tags: [题解,数论分块,哈希]
published: true
hideInList: false
feature: 
isTop: false
---
第一次发现哈希表这么好用。`^_^`

其实不用哈希表和 STL 的做法思想还是非常巧妙的。

<!-- more -->

简化后题意：

> 输入的 $n$ 每次除以一个数下取整，到 $1$ 停止，两个方案有一次除以的数不同就算作不同，方案有多少种？

直接转移的方程很好想，如果做过一些数论分块的题的话也应该很容易想到这是数论分块。

但是数论分块做复杂度大约是 $\log n \sqrt n$ 的，再记忆化的话用 `map` 就会 T 掉。那么 ~~考虑 unordered_map~~ 就自己实现一个哈希表吧。

我用了链式的方法处理冲突，本以为会被卡，结果一发就过了，没想到哈希表那么好用，~~以后就别用 `map` 了，直接啥都哈希表吧~~ 该用 `map` 的时候还是 `map` 好用。

代码。（不知道那种直接赋值的应该如何重载，所以写了 `twt.insert`）

```cpp
#include <cstdio>
#include <vector>
#include <utility>
#include <algorithm>
#define int long long
int n;
const int p = 951161;
struct twt {
	std::vector<std::pair<int, int> > g[1000000];
	bool count(int x) {
		int t = x % p;
		for(int i = 0; i < (signed)g[t].size(); i++)
			if(g[t][i].first == x) return true;
		return false;
	}
	int operator [] (int x) {
		int t = x % p;
		for(int i = 0; i < (signed)g[t].size(); i++)
			if(g[t][i].first == x) return g[t][i].second;
	}
	void insert(int x, int y) {
		int t = x % p;
		g[t].push_back(std::make_pair(x, y));
	}
} hash;
int solve(int x) {
	if(hash.count(x)) return hash[x];
	int an = 0;
	for(int l = 2; l <= x; l++) {
		int r = x / (x / l);
		an += solve(x/l) * (r-l+1);
		l = r;
	}
	hash.insert(x, an);
	return an;
}
signed main() {
	scanf("%lld", &n);
	hash.insert(1, 1);
	printf("%lld", solve(n));
	return 0;
}
```

这里用哈希似乎有点无脑了，那么不用哈希怎么做呢？

题解中有一个巧妙的想法：小范围内先处理出来一串。然后经过一堆看不懂的时间复杂度分析就可以过，以后如果实在没有办法可以试试这样的方式。

还有个大佬直接就建立一对一的映射了：

```cpp
b = sqrt(n);
int pos(int x) { return (x*x <= n) ? x : (b+n/x); }
```

原理大概是 `x*x <= n` 时比较密，然后后面就以 $\frac{n}{x}$ 为一段才出现一个了，所以直接加上去。

妙啊！以后遇上这种问题不妨多模拟几个小样例试试这样的一对一映射是否可行。

