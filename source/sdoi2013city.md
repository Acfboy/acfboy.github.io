---
title: '题解 [SDOI2013]城市规划'
date: 2021-03-24 08:53:52
tags: [题解]
published: true
hideInList: false
feature: 
isTop: false
---
[传送门](https://www.luogu.com.cn/problem/P3300)

因为 $m$ 非常的小，很容易维护，所以可以按行建立一颗线段树，维护其中联通块的数量。处理联通块的时候用并查集记录上下两个边界的联通状况以及是否有建筑物。

难点在于如何合并，即 `pushup`。

<!-- more -->

1. 将两边块的 `ans` 合起来。
2. 将新的左端点设为左边的左端点，右端点设为右边的右端点。
3. 现在合并并查集
	1. 左边的并查集复制一份，存在 $[1, 2m]$, 将右边复制一份，存在 $[2m+1, 4m]$。
	2. 枚举两块交界处的，若可以合并，则将其合并，若两边都有建筑物，则 `ans--`。
   1. 经过上面的操作，并查集被合并成了 $[1,4m]$ 的并查集，但是我们只要存上面的边界和下面的边界的联通性。
   2. 建立一个新数组，将上下边界的 `fa` 都连向它们。
   3. 然后将每一个的 `fa` 都改为它的 `fa` 所指向的。
	3. 经过这些操作，所有点集合的根都在上边界和下边界了，可以直接返回。
    
其实思路清晰的话也没有很多难写的细节，代码也不至于长达 $200$ 行。

代码如下。

```cpp
#include <cstdio>
#include <cstring>
const int N = 100005, M = 8;
int n, m, q, l, r, x, y, tmpfa[M*4], tmpflag[M*4], v[M*4];
char opt[5], z[5], map[N][M];
struct node {
	int l, r, ans;
	int fa[2*M], flag[2*M];
	node() {
		for(int i = 1; i <= m; i++) fa[i] = fa[i+i] = 0, flag[i] = 0; 
	}
	void init() {
		ans = 0;
		for(int i = 1; i <= m; i++) fa[i] = fa[i+m] = i;
		for(int i = 1; i <= m; i++) flag[i] = (map[l][i] == 'O') ? 1 : 0;
	}
	int find(int x) {
		if(fa[x] != x) fa[x] = find(fa[x]);
		return fa[x]; 
	}
	void merge(int x, int y) {
		int fx = find(x), fy = find(y);
		if(fx == fy) return;
		fa[fy] = fx;
		flag[fx] |= flag[fy];
	}
	void makeNode() { // 构造线段树的叶子
		init();
		for(int i = 1; i < m; i++) {
			char now = map[l][i], next = map[l][i+1];
			if(now != '|' && now != '.' && next != '|' && next != '.') merge(i, i+1);
		}
		for(int i = 1; i <= m; i++) find(i+m);
		for(int i = 1; i <= m; i++) 
			if(fa[i] == i && flag[i]) ans++;			
	}
} tree[N << 2];
int findTmp(int x) {
	if(x != tmpfa[x]) tmpfa[x] = findTmp(tmpfa[x]);
	return tmpfa[x];
}
node mergeAns(node x, node y) { // 合并过程上面写得很详细了
	node an;
	an.l = x.l, an.r = y.r;
	an.ans = x.ans + y.ans;
	for(int i = 1; i <= 2*m; i++) tmpfa[i] = x.fa[i], tmpflag[i] = x.flag[i];
	for(int i = 1; i <= 2*m; i++) tmpfa[i+2*m] = y.fa[i]+2*m, tmpflag[i+2*m] = y.flag[i];
	for(int i = 1; i <= m; i++) {
		char lval = map[x.r][i], rval = map[y.l][i];
		if(lval != '.' && lval != '-' && rval != '.' && rval != '-') {
			int fx = findTmp(i+m), fy = findTmp(i+2*m);
			if(fx == fy) continue;
			if(tmpflag[fx] && tmpflag[fy]) an.ans--;
			tmpfa[fy] = fx;
			tmpflag[fx] |= tmpflag[fy];
		}
	}
	memset(v, 0, sizeof v);
	for(int i = 1; i <= m; i++) {
		int fi = findTmp(i);
		if(!v[fi]) v[fi] = i, an.flag[i] = tmpflag[fi];
		fi = findTmp(i+3*m);
		if(!v[fi]) v[fi] = i+m, an.flag[i+m] = tmpflag[fi];
	}
	for(int i = 1; i <= m; i++) an.fa[i] = v[findTmp(i)], an.fa[i+m] = v[findTmp(i+3*m)];
	return an;
}
void pushUp(int p) { tree[p] = mergeAns(tree[p+p], tree[p+p+1]); }
void build(int p = 1, int l = 1, int r = n) {
	tree[p].l = l, tree[p].r = r;
	if(l == r) return tree[p].makeNode();
	int mid = l + (r-l) / 2;
	build(p+p, l, mid);
	build(p+p+1, mid+1, r);
	pushUp(p);
}
void Change(int p, int x, int y, char z) {
	if(tree[p].l == tree[p].r) {
		map[tree[p].l][y] = z;
		return tree[p].makeNode(); // 修改的时候重新构造即可
	}
	int mid = tree[p].l + (tree[p].r - tree[p].l) / 2;
	if(x <= mid) Change(p+p, x, y, z);
	else Change(p+p+1, x, y, z);
	pushUp(p);
}
node Query(int p, int l, int r) {
	if(l == tree[p].l && r == tree[p].r) return tree[p];
	int mid = tree[p].l + (tree[p].r - tree[p].l) / 2;
	if(l > mid) return Query(p+p+1, l, r);
	else if(r <= mid) return Query(p+p, l, r);
	else return mergeAns(Query(p+p, l, mid), Query(p+p+1, mid+1, r));
}
int main() {
	scanf("%d%d", &n, &m);
	for(int i = 1; i <= n; i++)
		scanf("%s", map[i]+1);
	build();
	scanf("%d", &q);
	while(q--) {
		scanf("%s", opt);
		if(opt[0] == 'Q') {
			scanf("%d%d", &l, &r);
			printf("%d\n", Query(1, l, r).ans);
		}
		else {
			scanf("%d%d%s", &x, &y, z);
			Change(1, x, y, z[0]);
		}
	}
}
```