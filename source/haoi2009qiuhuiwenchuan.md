---
title: '题解 [HAOI2009]求回文串'
date: 2021-04-26 10:50:00
tags: [题解,贪心]
published: true
hideInList: false
feature: 
isTop: false
---
贪心题。做的时候有些绝望觉得想不到，后来发现也不难。

<!-- more -->

> 每次可以把相邻两个进行交换，最少需要多少次将当前的字符串换成回文串。

首先有一个被我遗忘但挺重要的知识点：两两交换使一个序列有序的最小步数就是其逆序对数。因为每一次交换可以且仅可以消除一对逆序对。

所以我们只要思考要怎样构造一个原始的序列使那个次数最小然后再跑一遍逆序对就可以了。

显而易见的事实：

1. 如果最前和最后一段已经匹配就不用动它们了。
2. 每次肯定是把要到最外面的先移动，不然会把它们换到更里面从而产生更多步骤。
3. 同种字符向外靠的肯定是该字符中间最向外的。

然后就可以得到以下贪心策略：匹配上的直接跳过，然后认定左边的就应该在那里，把该同字符的最右边的换到对应的右边。

说起来简单，但自己做就是想不到这里，其实又三条事实可以得到这样是对的，因为这样扫到的一个一定是最向外的，而移动的另一个也是最向外的。

那么怎么实现？用 `vector` 记录一下每一个字母出现的位置就可以了。

判断无解很简单。

```cpp
#include <cstdio>
#include <vector>
#include <cstring>
#define int long long
const int N = 1000005;
char s[N];
std::vector<int> pos[26];
int tub[26], ans[N], an, t[N];
bool vis[N];
int query(int x) {
	int an = 0;
	for( ; x >= 1; an += t[x], x -= x&-x) ;
	return an;
}
void add(int x) {
	for( ; x <= 1000000; t[x]++, x += x&-x) ;
}
signed main() {
	scanf("%s", s+1);
	int n = strlen(s+1);
	for(int i = 1; i <= n; i++) {
		pos[s[i]-'A'].push_back(i);
		tub[s[i]-'A']++;
	}
	int sumOdd = 0;
	for(int i = 0; i < 26; i++) sumOdd += tub[i] & 1;
	if((n % 2 == 0 && sumOdd >= 1) || (n % 2 == 1 && sumOdd != 1)) return puts("-1"), 0;
	if(n % 2 == 1) {
		for(int i = 0; i < 26; i++) 
			if(tub[i] % 2 == 1) {
				vis[pos[i][pos[i].size()/2]] = 1;
				ans[n/2+1] = pos[i][pos[i].size()/2];
				break;
			}
	}
	int i = 1, l = 1, r = n;
	while(i <= n && l <= r) {
		if(vis[i]) { i++; continue; }
		ans[l] = i, vis[i] = 1, l++;
		ans[r] = pos[s[i]-'A'][pos[s[i]-'A'].size()-1], vis[ans[r]] = 1, r--;
		pos[s[i]-'A'].erase(--pos[s[i]-'A'].end());
		i++;
	}
	for(int i = 1; i <= n; i++)
		an += query(1000000) - query(ans[i]),
		add(ans[i]);
	printf("%lld", an);
	return 0;
}
```