---
title: '题解 [AGC009B] Tournament'
date: 2021-04-03 17:23:55
tags: [题解,树]
published: true
hideInList: false
feature: 
isTop: false
---
我记得 CF 也有一题是这个名字，当时做到感觉从题意到做法都很难懂，有些阴影，所以又做了这题。

这题代码非常短，但一看题面还是觉得一副不可做的样子，~~所以叫这名字的没一个好东西~~。
<!-- more -->

> 有 $n$ 个选手进行淘汰赛，每场比赛后输的一方就会立刻被淘汰。现在比赛已经结束，我们已知 $1$ 号是最后的胜者，而第 $i(i>1)$ 号是被 $a_i$ 号淘汰的。求获得胜利需要比赛最多的人最少的比赛次数。

考虑建出题目中的那一棵树，那这棵树是什么意义呢？即父亲淘汰了它的一堆儿子。

由于淘汰以后和比赛无关，没有后效性，又显然有最优子结构性质，所以考虑进行树形 dp. 令 $f_u$ 表示 $u$ 这个点的答案，那么由于它被它父亲给淘汰，所以和父亲比试要么父亲的答案要增加一场，但如果儿子的答案更大的话，那么得看作儿子多比一场，所以只要确定一个顺序来比赛即可。

显然，儿子答案小的来最优。

```cpp
#include <cstdio>
#include <vector>
#include <algorithm>
const int N = 100005;
int n, x, f[N];
std::vector<int> g[N];
bool cmp(int x, int y) {return f[x] < f[y];}
void dfs(int u) {
    for(int i = 0; i < g[u].size(); i++) dfs(g[u][i]);
    std::sort(g[u].begin(), g[u].end(), cmp);
    for(int i = 0; i < g[u].size(); i++) f[u] = std::max(f[u], f[g[u][i]]) + 1;
}
int main() {
    scanf("%d", &n);
    for(int i = 2; i <= n; i++) {
        scanf("%d", &x);
        g[x].push_back(i);
    }
    dfs(1);
    printf("%d", f[1]);
    return 0;
}
```

是一道好题，讲起来简单，但思路有点难以想到，也不是很好理解这个转移，不愧是 AGC.