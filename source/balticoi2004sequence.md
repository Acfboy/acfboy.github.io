---
title: '题解 [BalticOI 2004]Sequence 数字序列'
date: 2021-04-03 11:56:38
tags: [题解,堆]
published: true
hideInList: false
feature: 
isTop: false
---
感觉是很有意思的一道题，巧妙运用了可并堆来动态维护中位数，还有改严格递增为非严格的 trick, 感觉都挺妙的，不是那么容易想到。

<!-- more -->

> 给出序列 $A$ 求严格单调递增序列 $B$，使得  $\sum |A_i-B_i|$ 最小

考虑以下结论:

1. 若 $A$ 递增则令 $B = A$ 即可
2. 若 $A$ 递减则 $B$ 中所有数都相同且是 $A$ 的中位数最优。

对于 $2$, 就是 $B$ 递增了反而拉开更大距离，没用，所以每一个数都相同；然后就变成了初中数学题，显然取中位数最优。

可是题目要求严格，那么我们可以把每个 $A_i$ 都减去 $i$， 最后再补上去，这样差不会有变化而 $B$ 就可以不严格了，因为若相同，最后差还是会有 $1$, 而差不能为负，所以加上后肯定不会相同。感觉挺妙的。

然后我们可以把每个数都单独作为一份，取的都是它自己。合并时按照原来的原则，如果下降了就用新的中位数，上升了那直接 $B = A$ 就可以了。

怎么维护能合并的中位数呢？原来动态维护中位数用堆，我们这里可以使用可并堆，每次维护当前区间最大的几个就可以了。

最后计算答案并把每一块中的答案都输出就好了。

代码。

```cpp
#include <cstdio>
#include <algorithm>
#include <cmath>
#define int long long
const int N = 1000005;
int n, a[N], val[N], L[N], R[N], rt[N], size[N], cnt,
    ls[N], rs[N], dis[N], ans;
int merge(int x, int y) {
    if(x == 0 || y == 0) return x+y;
    if(val[x] < val[y]) std::swap(x, y);
    rs[x] = merge(rs[x], y);
    if(dis[ls[x]] < dis[rs[x]]) std::swap(ls[x], rs[x]);
    dis[x] = dis[rs[x]] + 1;
    return x;
}
void pop(int x) {
    size[x] --;
    rt[x] = merge(ls[rt[x]], rs[rt[x]]);
}
signed main() {
    dis[0] = -1;
    scanf("%lld", &n);
    for(int i = 1; i <= n; i++) scanf("%lld", &a[i]), a[i] -= i;
    for(int i = 1; i <= n; i++) {
        size[++cnt] = 1, val[i] = a[i];
        rt[cnt] = i;
        L[cnt] = R[cnt] = i;
        while(cnt > 1 && val[rt[cnt-1]] > val[rt[cnt]]) {
            cnt--;
            size[cnt] += size[cnt+1], R[cnt] = R[cnt+1];
            rt[cnt] = merge(rt[cnt], rt[cnt+1]);
            while(size[cnt] > (R[cnt] - L[cnt])/2+1) pop(cnt);
        }
    }
    for(int i = 1; i <= cnt; i++)
        for(int j = L[i]; j <= R[i]; j++)
            ans += abs(a[j] - val[rt[i]]);
    printf("%lld\n", ans);
    for(int i = 1; i <= cnt; i++)
        for(int j = L[i]; j <= R[i]; j++) printf("%lld ", val[rt[i]] + j);
}
```