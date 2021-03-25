---
title: '题解 [洛谷 4314]CPU监控'
date: 2021-03-25 18:42:39
tags: [题解,线段树]
published: true
hideInList: false
feature: https://z3.ax1x.com/2021/03/25/6XPfz9.jpg
isTop: false
---
又是好不容易才 ~~疑似~~ 明白的神仙作业题，考验对懒惰标记的理解，好题目。

<!-- more -->

> 给定一个序列，要求维护如下操作
>
>1. 查询区间最大值
>2. 查询区间历史最大值
>3. 将一段加上一个数
>4. 将一段数都变成一个

先考虑如何用 lazy tag 处理加上一个数和变成一个数的问题。

可以发现，使用了一次覆盖操作以后所有加的操作都可以用覆盖来代替了，这样就可以记录两个 tag 来分别描述加和覆盖而不会出现问题。

然后考虑如何记录区间最大值和区间历史最大值。

如果直接处理，这个点的子子孙孙都没有办法得到及时的更新，无法获得正确的区间最大值答案。为了解决这个问题，我们可以再开两个 lazy tag， 来记录区间历史最大覆盖和区间历史最大加，像其它的 lazy tag 一样下放，这样确保被更新区间的子子孙孙都可以得到正确的区间历史最大值。

难点主要在于下放标记的操作。

将往一个点上加上加标记定义为函数 `plus`, 加上覆盖标记定义为 `cover`, 这两个函数都需要两个参数，一个是要加上的现在标记，一个是要用来更新历史的标记。

注意，是更新历史的标记，而不是历史标记。

如用来更新的 `Change` 函数应该是这么写的。

```cpp
void Change(int p, int l, int r, int x, int opt) {
	if(l == t[p].l && r == t[p].r) {
		if(opt == ADD) t[p].plus(x, x);
		else t[p].cover(x, x);
		return;
	}
	pushDown(p);
	int mid =  t[p].l + (t[p].r - t[p].l) / 2;
	if(l > mid) Change(p+p+1, l, r, x, opt);
	else if(r <= mid) Change(p+p, l, r, x, opt);
	else Change(p+p, l, mid, x, opt), Change(p+p+1, mid+1, r, x, opt);
	pushUp(p);
}
```

注意到 

```cpp
if(opt == ADD) t[p].plus(x, x);
else t[p].cover(x, x);
```
如果写成 `t[p].plus(x, t[p].hadd)` 就会 WA 成 $40$ 分，因为没有更新历史最值的标记。我开始一直不理解这个问题。

现在来写 `pushDown`

```cpp
void pushDown(int p) {
	t[p+p].plus(t[p].add, t[p].hadd);
	t[p+p+1].plus(t[p].add, t[p].hadd);
	t[p].add = t[p].hadd = 0;
	if(t[p].isCov) {
		t[p+p].cover(t[p].cov, t[p].hcov);
		t[p+p+1].cover(t[p].cov, t[p].hcov);
		t[p].isCov = t[p].cov = t[p].hcov = 0;
	}
}
```

那这里为什么写的是 `t[p].hadd` 呢？因为这里是用父亲的标记取更新孩子的标记，相当于把这个 lazy tag 下放，注意记得清空标记。

理解了上面的内容，就比较容易写出这个结构体了。不理解的话看了这个结构体的实现也就理解了。

```cpp
struct twt { 
	int cov, l, r, add, max, hmax, hcov, hadd;
	bool isCov;
	twt() {
		add = cov = hcov = isCov = 0;
		max = hmax = -INF;
	}
	void cover(int d, int hd) {
		if(isCov) hcov = std::max(hcov, hd);
		else {
			isCov = true;
			hcov = hd;
		}
		hmax = std::max(hmax, hd);
		cov = max = d;
		add = 0;
	}
	void addd(int d, int hd) {
		hadd = std::max(hadd, add+hd);
		hmax = std::max(hmax, max+hd);
		add += d, max += d;
	}
	void plus(int d, int hd) {
		if(isCov) cover(cov+d, cov+hd);
		else addd(d, hd);
	}
} t[N<<2];
```

`plus` 中是判断是直接加上还是用覆盖来代替加法，`addd` 是真正的加入加标记，注意我们更新 `hadd`、`hmax`、`hcov` 用的都是 `hd`, 而更新 `cov`, `max` 和 `add` 用的都是 `d` 因为一个是在加入历史最值的标记，另一个是维护现在的标记。

~~说句闲话， `twt` 指的不是 [谭炜谭信息科技](https://twt-tec.github.io/)， 而是我的名字之一 $\sout \textsf{\color{red}T\color{black}heman\color{red}W\color{black}ithgoldwhoalwayshoutsandshousasasaien\color{red}T}$。~~

实现了这几个函数以后，线段树的其它部分就比较常规了，自行实现不难。

开始的时候一直不理解将区间的历史值当做标记来下放，现在想想，还是觉得这个考验了对懒惰标记的真正理解，非常妙，是一道好题目。我开始疑惑的为什么不直接打区间最大的标记在理解以后就很好想通了，算不出啊，得有了这两个标记才能算出哇！