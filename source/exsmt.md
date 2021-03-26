---
title: '题解  【模板】进阶线段树'
date: 2021-03-26 08:03:31
tags: [线段树]
published: true
hideInList: false
feature: 
isTop: false
---
其实是这道 [线段树分裂](https://www.luogu.com.cn/problem/P5494) 的模板，但是涉及了很多进阶线段树，所以把它称为进阶线段树，包含的操作有动态开点、内存回收、线段树合并、线段树分裂、线段树上二分等。

<!-- more -->

### 动态开点 与 内存回收

动态开点只需要将每个节点的儿子用一个数组存起来，然后每次若要访问儿子但儿子是空就给一个新的编号就可以了。

涉及删除操作的，删除了的节点若没有用了，就可以放到一个垃圾桶中，若要开新节点，先从垃圾桶中取，再开新节点。

上面两个的代码

```cpp
int newNode() {
	if(sizeBin != 0) return bin[sizeBin--];
	else return ++tot;
}
void del(int p) {
	bin[++sizeBin] = p;
	t[p][0] = t[p][1] = val[p] = 0;
	return;
}
```

动态开点的加操作和查询操作和原来没有什么区别，只是用表示左儿子的 `t[p][0]` 代替了 `p+p`, 用 `t[p][1]` 代替了 `p+p+1` 而已，记得如果这个点不存在就要新开出来。

```cpp
void add(int &p, int l, int r, int x, int y) {
	if(p == 0) p = newNode();
	if(l == r) {
		val[p] += y;
		return;
	}
	int mid = l + (r-l)/2;
	if(x <= mid) add(t[p][0], l, mid, x, y);
	else add(t[p][1], mid+1, r, x, y);
	pushUp(p);
	return;
}
int Query(int p, int l, int r, int x, int y) {
	if(l == x && r == y) return val[p];
	int mid = l + (r-l) / 2;
	if(y <= mid) return Query(t[p][0], l, mid, x, y);
	else if(x > mid) return Query(t[p][1], mid+1, r, x, y);
	else return Query(t[p][0], l, mid, x, mid) + Query(t[p][1], mid+1, r, mid+1, y);
}
```

### 线段树的合并与分裂

合并的边界情况很好考虑，若一个没有了，那么就直接接上另一个，先把 `y` 的值合并到 `x`, 然后递归左右就好了，函数可以返回新的根节点编号。

```cpp
int merge(int x, int y) {
	if(x == 0 || y == 0) return x^y;
	val[x] += val[y];
	t[x][0] = merge(t[x][0], t[y][0]);
	t[x][1] = merge(t[x][1], t[y][1]);
	del(y);
	return x;
}
```

分裂一般以 $k$ 大之后分裂出去来描述的，分三种情况讨论，如果 $k$ 比左边的个数大，那么左边保持不变，向右边递归；如果恰好相等，则把当前点的右儿子和新节点的右儿子互换；不然就先互换再向左递归。记得更新点的权值。

```cpp
void split(int x, int &y, int k) {
	if(x == 0) return;
	y = newNode();
	if(k > val[t[x][0]]) split(t[x][1], t[y][1], k - val[t[x][0]]);
	else if(k == val[t[x][0]]) std::swap(t[x][1], t[y][1]);
	else {
		std::swap(t[x][1], t[y][1]);
		split(t[x][0], t[y][0], k);
	}
	val[y] = val[x] - k;
	val[x] = k;
	return;
}
```

### 时间复杂度的正确性

除了合并与分裂，其它操作和原来的线段树没有区别，都是 $\log_2 n$ 的，只需要考虑分裂的时间。

直接考虑一次分裂/合并那么时间显然是 $\mathcal O (n)$ 的，但这样没有意义，要从均摊的角度来考虑时间，即所有点进行合并/分裂的总时间是多少。

不大了解，大概是只有部分包含的需要建新的节点，全部包含的就直接移动了，所以总复杂度是 $\mathcal O(n \log_2 n)$ 的, 均摊一下一次就是 $\log_2 n$ 的。
