---
title: '题解 ABC149E Handshake'
date: 2021-03-24 15:43:37
tags: [题解,二分]
published: true
hideInList: false
feature: https://z3.ax1x.com/2021/03/24/6bcGqJ.jpg
isTop: false
---
模拟赛……一个悲伤的故事，切了俩 ABC 的 F, 结果被这一个绿色的 E 给卡死了，必须写题解记录一下。

<!-- more -->

> 给定一个数组 $A$, 求所有 $A_i + A_j(1 \le i,j \le n)$ 的前 $m$ 大的和

$m$ 可以取到 $n^2$ 直接模拟显然不可做。

比赛的时候一直在模拟那种两个优先队列求两个数组前 $k$ 大的做法，希望发现一线性质，然而未果。比赛最后几分钟搞了一个假的线性做法和一个假的证明，然后调了一中午显然错的做法。

错的做法是排个序，把和变成一张表，然后从右下角开始一层层向左上做，理由是右下一定比左上好。但样例就是反例，可以平着多伸出去一点的。

**来讲正确的做法。**

如果确定了一个数，使两个数的和要比它大，那么可以枚举一个，另一个显然具有单调性，可以二分。利用前缀和可以确定所有满足这个的和。满足要求的数对数量也容易确定。

所以可以二分出恰是 $m$ 大的数对和，然后用 $n \lg n$ 的时间记录和判断。

需要处理的细节：可能不是恰好取到，因为同样大小的可能会有多个，所以最后要减去多余的。

代码如下。

```cpp
#include <bits/stdc++.h>
#define int long long
const int N = 200005;
int n, m, a[N], sum[N], res, l, r, ans, cnt;
bool check(int x) {
    cnt = 0, res = 0;
    for (int i = 1; i <= n; i++) {
        int pos = std::lower_bound(a + 1, a + n + 1, x - a[i]) - a;
        cnt += n - pos + 1;
        res += sum[n] - sum[pos - 1] + a[i] * (n - pos + 1);
    }
    return cnt >= m;
}
signed main() {
    scanf("%lld%lld", &n, &m);
    for (int i = 1; i <= n; i++) scanf("%lld", &a[i]);
    std::sort(a + 1, a + n + 1);
    for (int i = 1; i <= n; i++) sum[i] = sum[i - 1] + a[i];
    l = 0, r = 200005;
    while (l <= r) {
        int mid = (l + r) >> 1;
        if (check(mid)) ans = mid, l = mid + 1;
        else r = mid - 1;
    }
    printf("%lld\n", res - (cnt - m) * ans);
    return 0;
}

```