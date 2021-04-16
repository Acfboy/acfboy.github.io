---
title: '题解 [AGC037E] Reversing and Concatenating'
date: 2021-04-16 14:14:11
tags: [题解]
published: true
hideInList: false
feature: 
isTop: false
---
总算是自己做了一道 AGC，没有浪费 AGC 的妙妙题了。以前看题解把这么好的题目都浪费掉了。

<!-- more -->


> 操作 $k$ 次，每次将 $s$ 复制一份反转一遍拼在其后面，从新字符串中取一个子串作为 $s$。求操作 $k$ 次后字典序最小的字符串。

首先想到只进行一步这样的操作，显然是要把最小的那个在一起最长的取走，若有相同长度，那么后面的字典序要尽可能的小。

那么要进行更多的步骤怎么做呢，显然每次把最小的放在头上是不合理的，因为只有尾部的会被复制，而一次操作可以把任意一段扔到队尾或放到开头，所以除了开头可结尾，让最小的字母组成的一段放在最末尾肯定是最优的。

但如果存在同样的几组的怎么办呢，模拟几个小的数据观察就会发现，复制后前面的一段就会反过来拼在最小的那段复制完的后面，而这一段是永远不会变的。

比如 $\textsf{twtwa}\rightarrow  \textsf{t\underline{wtwaa}wtwt}\rightarrow \textsf{wt\underline{waaaa}wtw} \rightarrow \textsf{aaaaw}$

虽然最后结果中前面的 `a` 的数量增加了，但最后一直是 $\textsf{wtwt}$ 后面减少掉了一些，所以若开始时最小的字母组成的一块 **前面一段倒过来** 的字典序是最小的，变换后肯定也是最小的。

所以我们就有了以下做法：

1. 使用一步将最小的字母组成的最长的块（相同的取前面一段倒过来字典序最小的）移到最后。
2. 按题目要求进行变换，同时保证最小的字母组成的一块在最后。
3. 使用最后一步将最小的字母组成的移动到字符串最前面。

需要注意的是，开始时取的前面一段字典序最小是要在复制后的意义下复制出来的部分前面一段字典序最小，因为那才是我们实际上取的，由于复制意义下才是我们真正取的，所以还要判断最后的几个字符复制以后成为最长的最小字母组成的一块的情况（因为这个调了很久）。

在第二步模拟时如果最小字母组成的那块长度已经大于 $n$, 就不用继续了。这条性质保证了时间复杂度是 $O(n \log_2 n)$ 的。（但第一步处理需要 $n^2$）

$k=1$ 的时候要进行特判，因为没有机会让我们执行前两步了。

代码

```cpp
#include <iostream>
#include <string>
#include <algorithm>
#include <cstdio>
int n, k;
std::string s;
int main() {
	std::cin >> n >> k;
	std::cin >> s;
	char min = 'z';
	for(int i = 0; i < n; i++) min = std::min(min, s[i]);
	int maxj = 0, maxs = 0;
	std::string minpre;
	for(int i = 0; i < n; i++) minpre += 'z'+1;
	for(int i = n-1; i >= 0; i--) 
		if(s[i] == min) {
			int cnt = 0, j = i;
			for( ; j >= 0 && s[j] == min; cnt++, j--) ;
			std::string tmp1 = s.substr(j+1, n-1-j-1+1);
			std::string tmp2 = s.substr(n-j-1, n-n+j+1+1);
			std::reverse(tmp2.begin(), tmp2.end());
			std::string twt = tmp1 + tmp2;
			if(cnt > maxs || cnt == maxs && twt < minpre) 
				maxs = cnt, maxj = j, minpre = twt;
		}	
	int j = n-1, cnt = 0;
	for( ; j >= 0 && s[j] == min; j--, cnt++) ;
	std::string tm = s.substr(n-j-1, j+1) + s.substr(j+1, n-j-1);
	std::reverse(tm.begin(), tm.end());
	if(cnt*2 > maxs || cnt*2 == maxs && tm < minpre) maxs = cnt, maxj = j, minpre = tm;
	maxj ++;
	std::string t;
	t = s;
	std::reverse(t.begin(), t.end());
	s = s + t;
	if(k == 1) {
		s = s.substr(maxj, n);
		std::cout << s;
		return 0;
	}
	if(maxj + maxs != n) {
		s = s.substr(n-maxj, n);
		k -= 1;
	}
	for(int i = 1; i < k; i++) {
		t = s;
		std::reverse(t.begin(), t.end());
		s = s + t;
		s = s.substr(maxs, n);
		maxs <<= 1;
		if(maxs >= n) break; 
	}
	if(maxs > n) maxs = n;
	t = s;
	std::reverse(t.begin(), t.end());
	s = s + t;
	s = s.substr(n-maxs, n);
	std::cout << s;
	return 0;
}
```