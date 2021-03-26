---
title: '妙哉！Second Sum！'
date: 2021-03-17 16:01:17
tags: [题解,单调栈,ST表,链表]
published: true
hideInList: false
feature: 
isTop: false
---
提供两种做法，一种是依赖于单调栈和 ST 表的 $\mathcal O(n \lg n)$ 做法，思维难度较低；还有一个绝妙的做法，根本想不到，但不需要任何现成的 算法/数据结构， 时间复杂度 $\mathcal O(n)$。

[传送门](https://www.luogu.com.cn/problem/AT5206)
<!-- more -->

### 做法一

分别考虑每一个数在多少个区间内成为了次大值。

最大值在前面时，能当最大值的只有离当前这一个（位置记为 $r$）最近的比他大的，当取到这个最大值（位置记为 $l$）时，区间的最大和次大已确定，只需要把区间 $[l, r]$ 取进去，$l$ 左边和 $r$ 右边比他俩都小的去不去都无所谓。设 $l$ 左边第一个比 $a_r$ 大的为 $l'$, $r$ 右边第一个比 $a_r$ 大的是 $r'$, 则 $a_r$ 对答案的贡献是 $(l - l') \times (r' - r)$ 。

实际做的时候还要再跑一遍最大值在后面的。

用单调栈可以方便地维护 $l$ ，那么如何维护 $l'$ 呢？ 可以使用 ST 表实现，类似倍增求 LCA 那样能跳的救跳，即若这一段最大值还是小于 $a_r$ 就跳。

代码：

```cpp
#include <cstdio>
#include <stack>
#include <algorithm>
#define int long long
const int N = 100005;
int n, pre[N], suc[N], stL[N][20], stR[N][20], a[N], ans;
std::stack<int> st;
int findL(int now, int m) {
	for(int i =  17; i >= 0; i--) 
		if(stL[now][i] < m && now - (1 << i) >= 1) now = now - (1 << i);
	return now;
}
int findR(int now, int m) {
	for(int i =  17; i >= 0; i--) 
		if(stR[now][i] < m && now + (1 << i) <= n) now = now + (1 << i);
	return now;
}
signed main() {	
	scanf("%lld", &n);
	for(int i = 1; i <= n; i++) scanf("%lld", &a[i]);
	for(int i = 1; i <= n; i++) stL[i][0] = a[i-1], stR[i][0] = a[i+1];
	for(int j = 1; j <= 17; j++)
		for(int i = 1; i <= n; i++)
			if(i - (1 << j) >= 1) stL[i][j] = std::max(stL[i][j-1], stL[i - (1 << (j-1))][j-1]);
	for(int j = 1; j <= 17; j++)
		for(int i = n; i >= 1; i--)
			if(i + (1 << j) <= n) stR[i][j] = std::max(stR[i][j-1], stR[i + (1 << (j-1))][j-1]);
	a[0] = n+1, a[n+1] = n+1;
	st.push(0);
	for(int i = 1; i <= n+1; i++) {
		while(!st.empty() && a[st.top()] < a[i]) {
			suc[st.top()] = i;
			st.pop();
		} 
		pre[i] = st.top();
		st.push(i);
	}
	for(int i = 1; i <= n; i++) {
		int lBer = pre[i], rBer = suc[i];
		int tmpl = findL(lBer, a[i]), tmpr = findR(rBer, a[i]);
		if(lBer >= 1) ans += (lBer - tmpl + 1) * (rBer - i) * a[i];
		if(rBer <= n) ans += (tmpr - rBer + 1) * (i - lBer) * a[i];
	}
	printf("%lld", ans);
	return 0;
}
```

[提交记录链接](https://www.luogu.com.cn/record/47982443)

### 做法二

来考虑如何优化做法一，或者你不会单调栈，也不会 ST 表怎么办？

做法一看上去似乎没有可以优化的地方了，但实际上线性的做法特别的简洁，让人体会到算法竞赛的魅力：

**换一种求解的顺序**

在原来的做法里，我们先统计第一个对答案的贡献，再统计第二个对答案的贡献，没有什么是现成的，一切都需要重新求解。

但如果我们从 $a_i$ 等于 $1$ 的位置开始求解呢？

这时候，其它所有的显然都比它大，不需要求解，而后面再求解的数显然都比这个要大，所以遇到这一个直接跳过就好了。


细品一下这一段话，我们其实是利用了原先求解的结果，去掉了不必要的求解。因为原先做法中，用 ST 表就是为了跳过所有比当前这个小的，但用从小到大的求解顺序，只需要把原来求到过的都跳过就行了，不需要重新求解，所以降低了时间复杂度。

每次更新时，比当前更小的都被跳过了，直接调用没跳过的就是大于当前的，所以这样是对的。

具体的实现使用双向链表 $L_i$ 表示这个去掉所有跳过的左边是什么，$R_i$ 表示去掉跳过的 $i$ 的右边是什么。最后更新一下，跳过当前这一个就可以了。

代码：

```cpp
#include <cstdio>
#define int long long
const int N = 100005;
int L[N], R[N], x, map[N], n;
signed main() {
    scanf("%lld", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%lld", &x);
        map[x] = i;
        L[i] = i - 1, R[i] = i + 1;
    }
    int ans = 0;
    for (int i = 1; i <= n; i++) {
        int l = L[map[i]], r = R[map[i]];
        if (l >= 1) ans += i * (l - L[l]) * (r - map[i]);
        if (r <= n) ans += i * (R[r] - r) * (map[i] - l);
        L[r] = l, R[l] = r;
    }
    printf("%lld", ans);
    return 0;
}
```

仅 $20$ 行，从各个方面吊打原来的 $60$ 行做法，妙不妙？

