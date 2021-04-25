---
title: '题解 [JOI 2021 Final] 雪玉'
date: 2021-03-11 20:06:21
tags: [题解]
published: true
hideInList: false
feature: 
isTop: false
---
这题似乎比前两年的 JOI Final T2 要更有思维难度一些，让我想了比以往长得多的时间，而且开始时觉得不可做，后来调试都没调一下就过了，感觉这题十分有趣，就写题解来记录一下。

<!-- more -->

先考虑一下 $O(NQ)$ 的暴力做法怎么做，值域很大，暴力标记是不现实的，而一块地方的雪清空以后就不会再有了，所以可以对于每一个点标记周围已经到过了哪里。由于所有的雪球都是同时运动的，所以左边的一个雪球到了右边的一个向左曾经到过的地方的话，这个雪球（左边那个）向右就不可能得到新的雪了。

于是我们有了一个暴力的算法：对于每次操作，更新所有雪球相对于原始位置的最左和最右的位置，如果左边的最右位置比右边的最左位置靠后，那么左边 的这个能滚到的最右雪和右边那个能滚到的最左雪就确定了。最后答案就是雪球能滚到的左右边界之间的雪质量。

现在想想哪里有重复和不必要。

仔细想想就可发现，若相邻雪球的最大地盘没有相交的话，能滚的就是所有雪球相对于原来位置的左右范围，对于每个这样的雪球都一样，所以没有必要去更新每一个（所有都是一样的啊！），对于每一次操作，只需要把相交的相邻两个滚雪范围的 右/左 边界更新一下即可。

具体地：

1. 对两两间距进行一次排序，$dif_i$ 表示 $a_{i+1} - a_i$
1. 对于每次操作
	1. 更新所有没有碰上的 最左(记为 $L$ )/最右( $R$ ) 移动范围。
   2. 若当前最小的 $dif$ 满足 $L + R \ge dif$， 则更新这一个，直到不满足这一条
1. 对于所有没有被更新左右边界的，其左右边界就是 $a_i - L$ 和 $a_i + R$。
1. 第 $i$ 个雪球的答案是左右范围之差。

分析一下时间复杂度，由于 $dif$ 不会变化，所以指向最小的 $dif$ 的指针只会向后移，虽然有两重循环，但时间复杂度是线性的。

代码：

```cpp
#include <cstdio>
#include <cstring>
#include <algorithm>
#define int long long
const int N = 200005; 
int n, q, a[N], limitL[N], limitR[N], L, R, ans, now, x, w;
char opt;
struct twt {
	int id, val;
	bool operator < (twt b) const {
		return val < b.val;
	}
} dif[N];
signed main() {
	scanf("%lld%lld", &n, &q);
	for(int i = 1; i <= n; i++) scanf("%lld", &a[i]);
	for(int i = 1; i < n; i++) dif[i].val = a[i+1] - a[i], dif[i].id = i;
	std::sort(dif+1, dif+n);
	now = 1;
	memset(limitL, -1, sizeof limitL);
	memset(limitR, -1, sizeof limitR);
   // limitL 和 limit R 用于记录左右边界限制
	while(q--) {
		scanf("%lld", &x);
		w += x;
		if(w > 0) R = std::max(R, w), opt = 'R';
		else L = std::max(L, -w), opt = 'L';
     // opt 记录当前操作是向左还是向右扩张了
		while(L + R >= dif[now].val && now < n) {
			int x = dif[now].id, y = dif[now].id+1;
			limitL[y] = (opt == 'L') ? (a[x] + R) : (a[y] - L);
			limitR[x] = limitL[y];
			now++;
        // 不断更新满足条件的
		}
	}
	for(int i = 1; i <= n; i++)	 {
		if(limitL[i] == -1) limitL[i] = a[i] - L;
		if(limitR[i] == -1) limitR[i] = a[i] + R;
	}
   // 处理未被更新的
	for(int i = 1; i <= n; i++) printf("%lld\n", limitR[i] - limitL[i]);
	return 0;
}
```