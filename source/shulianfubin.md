---
title: '题解 “数链抙玢”'
date: 2021-04-27 11:19:55
tags: [题解,搜索]
published: true
hideInList: false
feature: 
isTop: false
---
让我做了一晚上的题，其它作业一直没做，都攻这道了。过了后觉得其实挺好想到的……
$\color{red}\text{Update:}$ 代码又被叉掉了，修正后更新了代码。

<!-- more -->

先普及一下两个生僻字：

![](https://www.hualigs.cn/image/6087486d91aa0.jpg)

同“捊”，用手捧的意思。

![](https://www.hualigs.cn/image/608748bb1b3fb.jpg)

玻璃纸又称“赛璐玢”。英文词是由“cellulose”（纤维素）和“diaphane”（透明的）二词合成的。

所以，题目意思就是数链用手捧着玻璃纸把它隔开！（要不还是叫“fu bin” 吧，捧玉挺好的）~~真是非常形象啊~~。

---
$$\\$$
> 在 $n$ 个数的相邻之间插入乘号或加号，求答案为 $p$ 的方案有多少种。
>
> $n \le 35, 0 \le p \le 10^9, 0 \le a_i \le 10^9$

看到这个数据范围，那么大概率就是折半搜索了。可是直接折半会遇上一个问题，那就是如果这个位置填乘号能构成答案，那么这些答案都不会被我们记下，因为把乘号拼在一起的话前后就有关联了，不能折半搜索。

然后我就想到了一个“歪门邪道”的做法，按照加号个数进行折半搜索，把中间的答案都记录下来，再反着跑一遍。噼里啪啦写完发现连样例都过不了。

两个显然的问题：

1. 会重复计算，每个答案在每个加号都会计算一次。
2. 时间复杂度是错的，因为把中间的都得记下来。

然后就没辙了，一下午就过去了。

后来晚上仔细一想发现其实很早就有的一个很自然的想法其实是对的，枚举向后的最靠中间的那个加号。听上去很没有道理吧，但它是对的，为什么呢？

1. 在枚举点在中间的时候会记录下每一个中间是加号的答案。
2. 然后定下中间是乘的，那么枚举右边第一个是加的到中间就一定是乘号，原来 $2^{\frac{n}{2}}$ 种还是那么多种，不会多出来，所以复杂度是对的。
3. 1 记下了中间是加号的所有答案， 2 记下了中间是乘号的所有答案，不重不漏。

然后考虑实现上的问题，感觉实现也挺难的。大体的思路就是第一次搜索到中间然后用 vector 记录下所有的**答案**和**该方案最后一段乘起来的**，然后每向后一个就把所有的答案再用乘号连上新的那个，并重新排序，顺便移除已经不可能的。

但是还有 $0$ 的情况需要考虑，问题就有点多了起来了，我的程序也屡次因为这个出错，甚至过了之后仍然被我做自己给 hack 了，~~现在说不定还是错的欢迎 hack~~。

出问题的关键在于我们为了避免高精度而进行判断如果当前的答案已经大于 $p$ 了就直接退出，而可能当前的一段可能乘上一个 $0$ 就又不超过 $p$ 了。所以处理的时候不能直接退出，而是打一个标记，如果后面没有 $0$ 了那么这个状态就只能舍弃，如果有 $0$ 再把最后一段的乘积变为 $0$ 然后重新加入到搜索之中。

同理，我们记录下来的答案也必须要都打上这样的一个标记，在每次乘上当前的数的时候也得进行相同的处理。注意排序的时候要把有标记的放到最后，因为它们是不构成答案的。

然后来理性的分析一下复杂度：

1. 正向 dfs 时间复杂度 $2^{\frac{n}{2}}$， 反向 dfs 复杂度还要乘上二分的复杂度，就是 $2^{\frac{n}{2}}\frac{n}{2}$
2. 往后循环的复杂度是 $\frac{n}{2}$ 的，然后里面需要跑一遍 dfs, 还需要用 $\frac{n}{2}$ 的时间更新每一个答案，然后排序需要的时间是 $n \log n$ 的，即  $2^{\frac{n}{2}} \frac{n}{2}$。
3. 所以总的复杂度是 $2^{\frac{n}{2}} \left(\frac{n}{2}\right)^2$。

算一下是八千多万，可以通过此题。

代码。

```cpp
#include <map>
#include <vector>
#include <cstdio>
#include <algorithm>
#define int long long
const int N = 40;
int n, p, a[N], an;
struct twt {
	int val, prod;
	bool over;
	bool operator < (twt b) const {
		return over < b.over || (over == b.over && val < b.val);
	}
};
std::vector<twt> ans;
typedef std::vector<twt>::iterator twtIT;
void dfs(int t, int sum, int prod, bool flag) {
	if(t > n/2) {
		if(!flag) ans.push_back((twt){sum + prod, prod, 0});
		else ans.push_back((twt){sum+prod, prod, 1});
		return;
	}
	if(sum > p) return;
	if(sum+prod > p) flag = 1;
	if(a[t] == 0 && flag) dfs(t+1, sum, 0, 0); 
	else if(flag) dfs(t+1, sum, prod, 1);
	else {
		dfs(t+1, sum, prod*a[t], 0);
		dfs(t+1, sum+prod, a[t], 0);
	}
}
void dfs2(int t, int lim,int sum, int prod, bool flag) {
	if(t < lim) {
		int l = std::lower_bound(ans.begin(), ans.end(), (twt){p-sum-prod, 0, 0}) - ans.begin(),
			r = std::upper_bound(ans.begin(), ans.end(), (twt){p-sum-prod, 0, 0}) - ans.begin();
		an += r-l;
		return;
	}
	if(sum > p) return;
	if(sum+prod > p) flag = 1;
	if(a[t] == 0 && flag) dfs2(t-1, lim, sum, 0, 0);
	else if(flag) dfs2(t-1, lim, sum, prod, 1);
	else {
		if(t != n) dfs2(t-1, lim, sum, prod*a[t], 0);
		dfs2(t-1, lim, sum+prod, a[t], 0);
	}
}
signed main() {
	scanf("%lld%lld", &n, &p);
	for(int i = 1; i <= n; i++)	scanf("%lld", &a[i]);
	dfs(2, 0, a[1], false);
	std::sort(ans.begin(), ans.end());	
	for(int i = n/2+1; i <= n; i++) {
		if(ans.empty()) break;
		dfs2(n, i, 0, 0, false);
		for(int j = 0; j < (signed)ans.size(); j++) 
			if(!ans[j].over) {
				ans[j].val -= ans[j].prod, ans[j].prod *= a[i], ans[j].val += ans[j].prod;
				if(ans[j].prod > p) ans[j].over = 1;
			}
			else if(a[i] == 0) ans[j].val -= ans[j].prod, ans[j].prod = 0, ans[j].over = 0;
		std::sort(ans.begin(), ans.end());
		for(twtIT j = ans.end()-1; j != ans.begin(); j--)
			if(!ans.empty() && j->val - j->prod > p) ans.erase(j);
		if(!ans.empty() && ans.begin()->val - ans.begin()->prod > p) ans.erase(ans.begin());
	}
	dfs2(n, n+1, 0, 0, 0);
	printf("%lld", an);
	return 0;
}
```

能为了一题不断研究，过了之后仍能找出并修正错误，这才是我心中的 OI !

另外建议加强数据，~~防止别人和我一样水过~~。