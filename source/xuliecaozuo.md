---
title: '题解 [清华集训2012]序列操作'
date: 2021-03-25 08:09:46
tags: [题解,线段树,组合计数]
published: true
hideInList: false
feature: 
isTop: false
---
此题继续展现线段树的神奇力量——这都能维护！

[传送门](https://www.luogu.com.cn/problem/P4247)
<!-- more -->


对于前面两个操作都可以直接适用 lazy_tag。

考虑询问怎么完成。因为 $c$ 非常的小，仅有 $20$, 可以开一个长 $20$ 的数组 $f$ 记录对于每一个节点每一个问题的答案。根据 $f$ 的定义，可以容易地知道对于单个节点的初始化值，所以考虑如何合并。

合并有一个妙极了的式子。$f_{l, i}$ 表示左边的 $c=i$ 的答案， $f_{r, i}$ 表示右边的， $f_{p, i}$ 表示当前的。

$$
f_{p, i} = \sum_{k=0}^i f_{l, k} \times f_{r, i-k}
$$

其中 $k$ 枚举的是左边选了几个，左边所有的取 $k$ 个的和右边取 $i-k$ 的一一搭配，可以不重不漏的覆盖所有情况。其实这个分界点可以选在任何地方，只是因为我们写的是线段树，所以认为它选在中间。

可以写出 `merge` 函数来合并节点

```cpp
node merge(node x, node y) {
	node an;
	an. l = x.l, an.r = y.r;
	for(int i = 1; i <= std::min(an.r - an.l + 1, 20ll); i++) {
		an.f[i] = 0;
		for(int k = 0; k <= i; k++) an.f[i] = (an.f[i] + x.f[k] * y.f[i-k] %P) % P;
	}
	return an;
}
```

对于取反的 tag 直接把乘了奇数次的取反就可以了，那么如何处理加的情况呢。

可以来观察一下若干个数相乘和若干个数加上一个数再相乘后的变化。

如 五个里面取三个

$$
a_1 a_2 a_3 + a_1 a_2 a_4 + a_1 a_2 a_5  + a_2 a_3 a_4 + a_2 a_3 a_5 +... + a_3 a_4 a_5
$$

把其变成

$$
(a_1+x)(a_2+x)(a_3+x) + (a_1+x)(a_2+x)(a_3+x) +...+(a_3+x)(a_4+x)(a_5+x)
$$

展开后考虑对 $x$ 的每一个次幂分别计算，比如对于 $a_1a_2x$ 会出现几次呢？

在式子中，保持 $(a_1+x)(a_2+x)$ 不变，剩下的一个可以选 $(a_3+x)$、$(a_4+x)$ 和 $(a_5+x)$, 即在计算 $x^k$ 的答案时，每一个前面确定的都可以在剩下的 $n-(c-k)$ 中取出 $k$ 个，而已经确定的部分就是 $f_{i-k}$

所以可以得到如下的式子用以更新相加的操作。

$$
f_i = \sum_{k=0}^i C_{n-i+k}^k f_{i-k}x^k
$$

加操作的代码

```cpp
void doAdd(int p, int v) {
	v = (v%P + P) % P;
	for(int i = std::min(t[p].r - t[p].l +1, 20ll); i >= 1; i--) 
		for(int k = 1, pow = v; k <= i; k++, pow = pow * v % P) 
			t[p].f[i] = (t[p].f[i] + C[t[p].r-t[p].l+1-(i-k)][k] * t[p].f[i-k] % P * pow % P) % P;
	t[p].add = (t[p].add + v) % P;		
}
```

然后就可以根据前面所说的来下方懒惰标记。

```cpp
void doRev(int p) {
	for(int i = 1; i <= std::min(t[p].r-t[p].l+1, 20ll); i += 2) 
		t[p].f[i] = (-t[p].f[i]%P + P) % P;
	t[p].add = (-t[p].add%P + P) % P;
	t[p].rev *= -1;
}
void pushDown(int p) {
	if(t[p].rev != 1) doRev(p+p), doRev(p+p+1), t[p].rev = 1;
	if(t[p].add != 0) doAdd(p+p, t[p].add), doAdd(p+p+1, t[p].add), t[p].add = 0;
}
```

其它的操作都比较常规，用 `merge` 代替合并就可以了，别忘记建树时要给点的 `rev` 标记赋值初值 $1$。