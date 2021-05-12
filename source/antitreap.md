---
title: '十分钟“平衡树”'
date: 2021-05-11 12:25:46
tags: [题解,块状链表]
published: true
hideInList: false
feature: 
isTop: false
---
“平衡树”速成。

<!-- more -->

既然打了引号，那么就肯定不是真的平衡树了（

众所周知， `vector`  插入删除的常数非常小，以至于你直接用就可以通过普通平衡树的模板。对于加强版，我们也做相应的加强：用 `vector`  维护块状链表。

而且块状链表也不需要真的块状链表，直接若干次操作后拍扁重建就可以了。

上代码。可以通过【模板】普通平衡树（数据加强版）。

```cpp
#include <cstdio>
#include <algorithm>
#include <vector>
const int N = 100005, M = 1000005, t = 10000, B = 10000; 
int n, m, tot, a[N+M], op, x, lst, ans, cnt, size;
std::vector<int> v[B];
void build(bool flag = 0) {
	if(!flag) tot = 0;
	cnt = 0;
	for(int i = 1; i <= size; i++) {
		for(int j = 0; j < (signed)v[i].size(); j++) 	
			a[++tot] = v[i][j];
		v[i].clear();
	}
	size = std::max(tot / B + (tot % B != 0), 1);
	for(int i = 1; i < size; i++)
		for(int j = 0; j < B; j++)
			v[i].push_back(a[++cnt]);
	while(cnt < tot) v[size].push_back(a[++cnt]);
}
int work(int x) {
	int i = 1;
	while(i <= size && v[i][0] < x) ++i;
	i--;
	return std::max(1, i);
}
void insert(int x) {
	int i = work(x);
	v[i].insert(std::lower_bound(v[i].begin(), v[i].end(), x), x);
}
void erase(int x) {
	int i = work(x);
	std::vector<int>::iterator tmp = std::lower_bound(v[i].begin(), v[i].end(), x);
	if(tmp != v[i].end()) v[i].erase(tmp);
	else v[i+1].erase(v[i+1].begin());
	if(v[size].size() == 0) size = std::max(size-1, 1);
}
int rank(int x) {
	int i = work(x), ans = 0;
	for(int j = 1; j < i; j++) ans += v[j].size();
	return ans + std::lower_bound(v[i].begin(), v[i].end(), x) - v[i].begin() + 1;
}
int kth(int x) {
	for(int i = 1; i <= size; i++) {
		if(x <= (signed)v[i].size()) return v[i][x-1];
		x -= v[i].size();
	}
}
int pre(int x) { return kth(rank(x)-1); }
int next(int x) { return kth(rank(x+1)); }
int main() {
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= n; i++) scanf("%d", &a[i]);
	std::sort(a+1, a+1+n);
	tot = n; build(1);
	while(m--) {
		if(m % t == 0) build();
		scanf("%d%d", &op, &x);
		x ^= lst;
		if(op == 1) insert(x);
		else if(op == 2) erase(x);
		else if(op == 3) lst = rank(x);
		else if(op == 4) lst = kth(x);
		else if(op == 5) lst = pre(x);
		else if(op == 6) lst = next(x);
		if(op >= 3) ans ^= lst;
	}
	printf("%d", ans);
	return 0;
}
```

不过参数要小心设置啊，不然就翻车了，要论保险，还是直接写普通块状链表比较好。

当年写过一个题 *Big String*，块状链表的诸多细节调了很久，所以比赛的时候来不及调试的话还是暴力重建版比较好，~~我愿称之为“替罪羊向量块状链表”~~。

既然要小心设置参数，那么我们就得理性分析一下复杂度，来确定什么参数是最好的。

1. 每块长 $B$， 有 $\lceil\frac{n+m}{B}\rceil$ 块。
2. 所以查询在第几块的复杂度是 $O(\frac{n+m}{B})$
3. 除了重建，操作都要待查找在第几块，所以时间都要带上它。
4. 查找排名需要二分，所以时间复杂度是 $O(\frac{n+m}{B} + \log_2 B)$。
5. 前驱后继就是查排名和数，所以时间复杂度同样是  $O(\frac{n+m}{B} + \log_2 B)$。
6. 插入删除要找在第几块，又要二分，又要在块中插入，所以复杂度 $O(\frac{n+m}{B} + \log_2 B + \frac{(B +t)}{w} )$, `vector` 小常数用 $\frac{1}{w}$ 表示，其实可以把 `vector` 的时间理解成是根号的。

综上，复杂度是 $O(\frac{m^2}{t} + m(\frac{m}{B} +\log_2 B +\frac{B+t}{w}))$。