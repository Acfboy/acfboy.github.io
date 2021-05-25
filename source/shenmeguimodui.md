---
title: '题解 由乃喜欢二次离线根号黑科技'
date: 2021-05-25 20:10:41
tags: [莫队,分块]
published: true
hideInList: false
feature: 
isTop: false
---
题目 *Yuno loves sqrt technology II* 中的二是不是有双关的意思？

这是一道二次离线的黑科技版题。

<!-- more -->

感觉现在去写这种黑科技意义不大，因为在做（卡）这题的时候不能通过自己的思考体会到算法的精妙，不过体验一下黑题也是不错的。~~这句话中的“卡”其实也有双关的意思~~

另外，这是 Ynoi 模拟赛，所以老板其实没有完成第一道 Ynoi（

## 二次离线基础思想

有的时候用离线后用莫队做移动一个端点消耗的时间不是 $O(1)$ 的，所以总复杂度除了一个根号还得乘上那个操作的复杂度，没有办法接受。

所以可以把“移动”这一个操作再离线一遍，放到后面再一起做，使移动的复杂度均摊 $O(1)$

## 具体实现

> 求区间逆序对数。

首先有个简单的莫队做法，将询问离线，然后每次莫队移动的时候用个树状数组统计 比它大/比它小 的即变化量。时间复杂度大概是 $O(n \sqrt n \log_2n)$。

分开仔细考虑一下左右指针移动时的变化量。

1. 左指针向左移动：答案会增加当前的 $[l, r]$ 区间中比 $a_{l-1}$ 小的的个数。
2. 左指针向右移动：答案会减少第 $[l+1, r]$ 个中比 $a_l$ 小的的个数。
3. 右指针向左： 减少第 $[l, r-1]$ 个比 $a_r$ 大的。
4. 右指针向右：增加第 $[l, r]$ 中比 个$a_{r+1}$ 大的。

那么就记 $f(x, k)$ 是第 $[1,k]$ 个中比 $a_x$ 大的， $g(x, k)$ 是前 $k$ 个中比 $a_x$ 小的。如果能快速处理这些东西，就可以快速计算上面的内容了。

把四种操作的增量重新写一遍。

1. $+g(l-1, r) - g(l-1, l-1)$
2. $-g(l, r) + g(l, l)$
3. $-f(r, r-1) + f(r, l-1)$
4. $+f(r+1, r) - f(r+1, l-1)$

发现其中的 $g(x, x)$ 和 $f(x, x-1)$ 可以方便地用树状数组求出来，然后考虑把剩下的离线下来处理。

可以从头到尾循环一遍，每次将 $a_i$ 加入到某个数据结构中。可以发现照这个顺序，处理 $r$ 移动的会在前面 $l$ 个都处理出来的时候就得到答案，同理 $l$ 移动要等到 $r$ 出来的时候才得到答案，所以可以用 `vector` 将这些询问挂到对应的端点上，然后在循环的时候加（减）上相应的答案就可以了。

然后想想这个数据结构得是个啥。它需要支持 $O(1)$ 查询一个值域内数的个数（不然你白离线了）， 但修改可以在 $\log_2$ 或者根号的时间内。

那么就上值域分块吧。

最后要注意我们记录的只是变化量，要做一个前缀和才是答案。

## 最后

代码有点长，点击展开看。

<details>
<summary> Code </summary>

```cpp
#include <cstdio>
#include <algorithm>
#include <vector>
#include <cmath>
#define re register
const int N = 100005, M = 350;
int a[N], c[N], l[M], r[M], s[M], sv[M][M], bv[N], blo[N], t[N], 
	n, m, size, B, ls[N], gr[N];
long long ans[N];
struct twt { int l, r, c, op, id; };
std::vector<twt> v[N];
struct dwd {
	int l, r, id; long long ans;
	bool operator < (dwd b) const {
		return (blo[l] == blo[b.l]) ? ((blo[l]&1) ? (r < b.r) : (r > b.r)) : (blo[l] < blo[b.l]);
	}
} q[N];
inline int ask(int x) { re int an = 0; for(re int i = x; i >= 1; i -= i&-i) an += t[i]; return an; }
inline void add(int x, int y) { for(re int i = x; i <= n; i += i &-i) t[i] += y; }
inline void update(int x) {
	for(re int i = x-l[bv[x]], *k = sv[bv[x]]; i < size; i++) k[i] ++;
	for(re int i = bv[x]; i <= bv[n]; i++) ++s[i];
}
inline int query(int x) { return s[bv[x]-1] + sv[bv[x]][x-l[bv[x]]];}
int main() {
	cin >> n >> m;
	for(re int i = 1; i <= n; i++) cin >> a[i], ls[i] = a[i];
	std::sort(ls+1, ls+n+1);
	for(re int i = 1; i <= n; i++) a[i] = std::lower_bound(ls+1, ls+n+1, a[i]) - ls;
	for(re int i = 1; i <= n; i++) {
		ls[i] = ask(a[i] - 1);
		gr[i] = ask(n) - ask(a[i]);
		add(a[i], 1);
	}
	
	size = sqrt(n);
	for(re int i = 1; i <= n; i++) bv[i] = (i-1) / size + 1;
	for(re int i = 0, j = 1; i < n; i += size, j++) l[j] = i+1, r[j-1] = i;
	r[bv[n]] = n;
	
	B = n / sqrt(m);
	for(re int i = 1; i <= n; i++) blo[i] = (i-1) / B + 1;
	for(re int i = 1; i <= m; i++) cin >> q[i].l >> q[i].r, q[i].id = i;
	std::sort(q+1, q+m+1);
	for(re int i = 1, l = 1, r = 0; i <= m; i++) {
		if(l > q[i].l) v[r].push_back((twt){q[i].l, l-1, 1, 1, i});
		while(l > q[i].l) q[i].ans -= ls[--l];
		if(r < q[i].r) v[l-1].push_back((twt){r+1, q[i].r, 0, -1, i});
		while(r < q[i].r) q[i].ans += gr[++r];
		if(l < q[i].l) v[r].push_back((twt){l, q[i].l-1, 1, -1, i});
		while(l < q[i].l) q[i].ans += ls[l++];
		if(r > q[i].r) v[l-1].push_back((twt){q[i].r+1, r, 0, 1, i});
		while(r > q[i].r) q[i].ans -= gr[r--];
	}
	
	for(re int i = 1; i <= n; i++) {
		update(a[i]);
		for(re int j = 0; j < (signed)v[i].size(); j++) {
			twt x = v[i][j];
			int l = x.l, r = x.r, c = x.c, op = x.op, id = x.id;
			for(int k = l; k <= r; k++)
				if(c == 1) q[id].ans += op * query(a[k]-1);
				else q[id].ans += op * (query(n) - query(a[k])); 
		}
	}
	
	for(re int i = 2; i <= m; i++) q[i].ans += q[i-1].ans;
	for(re int i = 1; i <= m; i++) ans[q[i].id] = q[i].ans;
	for(re int i = 1; i <= m; i++) cout << ans[i] << '\n';
	return 0;
}
```
</details>
  

这题卡常卡了好久，发现奇偶性优化还是挺管用的。

放个重载好的快读快写吧。
  
<details>
  <summary>输入输出优化</summary>

```cpp
namespace IO {
#define BUFSIZE 10000000
struct read {
    char buf[BUFSIZE], *p1, *p2, c, f;
    read() : p1(buf), p2(buf) {}
    char gc(void) {
        if (p1 == p2) p2 = buf + fread(p1 = buf, 1, BUFSIZE, stdin);
        if (p1 == p2)
            return EOF;
        else
            return *p1++;
    }
    read& operator>>(int& x) {
        c = gc(), f = 1, x = 0;
        for (; c < '0' || c > '9'; c = gc())
            if (c == '-') f = -1;
        for (; c >= '0' && c <= '9'; c = gc()) x = x * 10 + c - '0';
        x *= f;
        return *this;
    }
};
struct write {
    char buf[BUFSIZE], *p1, *p2, s[50];
    int tp;
    write() : p1(buf), p2(buf + BUFSIZE) {}
    ~write() { flush(); }
    void flush(void) {
        fwrite(buf, 1, p1 - buf, stdout);
        p1 = buf;
    }
    void pc(char c) {
        if (p1 == p2) flush();
        *p1++ = c;
    }
    write& operator<<(long long x) {
        if (x < 0) x = -x, pc('-');
        do {
            s[tp++] = x % 10 + '0', x /= 10;
        } while (x);
        while (tp) pc(s[--tp]);
        return *this;
    }
    write& operator<<(char x) {
        pc(x);
        return *this;
    }
};
read cin;
write cout;
} 
using IO::cin;
using IO::cout;
```
  
</details>