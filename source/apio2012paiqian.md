---
title: '题解 [APIO2012]派遣 & [JLOI2015]城池攻占'
date: 2021-04-02 19:51:33
tags: [堆,题解]
published: true
hideInList: false
feature: 
isTop: false
---
左偏树的基本运用加上一点点的树形 dp。

堆里也可以懒标记。
<!-- more -->

## [APIO2012]派遣
> 给定一棵数，每个点有 $l$ 值和 $c$, 选定一棵子树，在其中选一些根节点，记子树的根为 $w$, 求满足 $\sum c \le m$ 的情况瞎 $l_u \times$ 选中点个数 的最大值。

我们要选的点 $c$ 肯定要尽可能的小，因为这样才能选用更多的点，于是可以利用堆。

但是对于每个子树重新建堆太慢了，需要快速地合并两个堆，所以介绍左偏树(可并堆)。

左偏树是这样的一棵树:

1. 满足堆的性质
2. 任意一点左儿子到儿子不全的点的距离(简称“距离”)大于右儿子的

显然对于合并两棵左偏树(根为 $x,y$)，可以这样(假设小根)：

1. 若 $v_x > v_y$ 交换 $x, y$
2. 将 $y$ 并到 $x$ 的右儿子
3. 递归求解
4. 若不满足左偏性质就交换左右儿子。
5. 返回新的根 $x$

就是这样, `dis` 就是前文所说距离。

```cpp
int merge(int x, int y) {
	if(x == 0 || y == 0) return x+y;
	if(c[x] < c[y]) std::swap(x, y);
	rs[x] = merge(rs[x], y);
	if(dis[ls[x]] < dis[rs[x]]) std::swap(ls[x], rs[x]);
	dis[x] = dis[rs[x]] + 1;
	return x;
}
```
其实这样确保树 ”左偏“ 以及在右边合并都是为了确保个堆在整个过程中尽可能平衡，避免退化成链的情况，又用了动态的儿子分配方式，方便合并，又好写，感觉完胜原来的堆啊。

但这题还需要记录一个堆中的元素的和，怎么办，记在堆顶上是不行的，因为这个被删掉了就完了。

这里采用类似树形 dp 的方式，直接用 `size` 和 `sum` 标记在一个节点上来表示以其为根当前的堆大小和元素和。

这题由于保证了边一定从小连到大，所以 dfs 都不需要，直接从后往前扫描一遍来建树就可以了。

注意我们每次直接更新了父亲，所以只选叶子节点的答案没有被算进，需要先计算。

代码

```cpp
signed main() {
	dis[0] = -1; // 空节点
	scanf("%lld%lld", &n, &m);
	for(int i = 1; i <= n; i++) {
		scanf("%lld%lld%lld", &fa[i], &c[i], &l[i]);
		rt[i] = i, sum[i] = c[i], size[i] = 1;
		ans = std::max(ans, l[i]);
	}
	for(int i = n; i > 1; i--) {
		rt[fa[i]] = merge(rt[fa[i]], rt[i]);
		size[fa[i]] += size[i], sum[fa[i]] += sum[i];
		while(sum[fa[i]] > m) {
			sum[fa[i]] -= c[rt[fa[i]]];
			rt[fa[i]] = merge(ls[rt[fa[i]]], rs[rt[fa[i]]]);
			size[fa[i]] --;
		}
		ans = std::max(ans, l[fa[i]] * size[fa[i]]);
	}
	printf("%lld", ans);
}
```

## [JLOI2015]城池攻占

其实也是维护树上每一个点都开一个堆，然后进行合并，这里是小根堆，每次把不符合要求的就弹出。

但是对于攻击力的值怎么变化呢？更新所有会超时。每个堆的中数的变化方式是一样的，可如果记录下来变化方式的话，又记录在哪里呢？

这时就可以用到我们的老朋友，懒惰标记。对于堆，同样也是只要用到的进行处理就可以了啊，为什么每次要更新所有？只要在根节点打个懒标记，要删了时，直接把懒标记下传再合并两个就可以了。

注意，合并的时候也先给把懒标记下传， 因为合来合去的要依靠当前的根，而且树形会发生变化，所以处理之前得先把标记给儿子。

同时乘和加相同的数，不会改变元素间大小关系，所以这样子是对的。

带懒标记的代码。

```cpp
void cov(int x, int mu, int ad) {
    if(x == 0) return;
    s[x] *= mu, s[x] += ad;
    mul[x] *= mu, add[x] *= mu, add[x] += ad;
}
void pushdown(int x) {
    // printf("*%lld\n", x);
    cov(ls[x], mul[x], add[x]);
    cov(rs[x], mul[x], add[x]);
    mul[x] = 1, add[x] = 0;
}
int merge(int x, int y) {
    if(x == 0 || y == 0) return x + y;
    pushdown(x); pushdown(y);
    if(s[x] > s[y]) std::swap(x, y);
    rs[x] = merge(rs[x], y);
    if(dis[ls[x]] < dis[rs[x]]) std::swap(ls[x], rs[x]);
    dis[x] = dis[rs[x]] + 1;
    return x;
}
```

删除操作：

```cpp
pushdown(rt[u]);
rt[u] = merge(ls[rt[u]], rs[rt[u]]);
```