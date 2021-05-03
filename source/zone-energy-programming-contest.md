---
title: 'ZONe Energy Programming Contest 游记'
date: 2021-05-02 15:45:45
tags: [游记]
published: true
hideInList: false
feature: 
isTop: false
---
题目质量还是比较高的——就是数据有点水啊。

<!-- more -->

## A

字符串入门题。

## B 

初中数学之三角函数。

## C - 1

为什么要把这题难的放在前面？

开始做的时候觉得是贪心，但很容易找出反例发现不一定要选一位上最大的，所以就没有什么思路了，先去看看 D 再来。

## D

看上去高大上，其实仍然是水题，搞个 `deque` 处理反转和添加，然后用栈处理相邻的会消掉的问题。

C 耽误了一些时间，不然更快地过这题，可能排名会更高。

## C - 2

现在重新来思考 C 题。有了前面的经验可以知道大概率不会是 dp 了，因为局部最优没有办法得到全局最优，且这些数那么大也不能拿来做状态。

考虑了二分，但是由于没法直接选相似的原因，二分不能验证。

所以回到最初的思路，直接暴力枚举三个，这样的话时间复杂度是 $n^3$ 的，不能接受，但只要优化掉一维就可以通过了。

注意到题目中出现的常数 $5$ 和 $3$, 题目没有设置 $m$ 个肯定是因为这俩常数是有用的。有抽屉原理可以知道，选择的三行中，至少有一行包含了超过一个最大值。

但是这样还是没有办法做，因为既可能是 $3-1-1$ 的分布，也有可能是 $2-2-1$ 的分布。然后又卡了一会儿恍然大悟，同样抽屉原理可以得到两行肯定包含 $4$ 个以上的最大值啊！

所以只要枚举两行再枚举没有取到最大值的一列给它取上当前列的最大值就可以了。至于那一列的最大值可以直接读入的时候预处理出来。

时间复杂度 $\mathcal O(n^2)$。代码如下。

```cpp
#include <cstdio>
#include <algorithm>
int n, max[3005], a[3005][10], ans;
int main() {
    scanf("%d", &n);
    for(int i = 1; i <= n; i++) 
        for(int j = 1; j <= 5; j++) 
            scanf("%d", &a[i][j]), max[j] = std::max(max[j], a[i][j]);
    for(int i = 1; i <= n; i++)
        for(int j = i+1; j <= n; j++) {
            int tmax[6] = {0, 0, 0, 0, 0, 0};
            for(int k = 1; k <= 5; k++) tmax[k] = std::max(tmax[k], std::max(a[i][k], a[j][k]));
            for(int k = 1; k <= 5; k++) {
                int tmp = tmax[k], an = 2000000000;
                tmax[k] = max[k];
                for(int k = 1; k <= 5; k++) an = std::min(an, tmax[k]);
                ans = std::max(ans, an);
                tmax[k] = tmp;
            }
        }
    printf("%d", ans);
    return 0;
}
```

## E

乍一看还以为是 Dijkstra 的板子。

抱着暴力出奇迹的信仰 ~~没有分析复杂度~~ 我写了一发 Dijkstra 就直接交了，居然只 T 了一个点，然后卡了卡常，但也没能通过。

~~比赛结束有有人告诉我胡乱剪枝就过了。~~

然后开始尝试构造 hack 数据，不过构造得有些麻烦，因为要让点尽可能多的被重复松弛，最后成功卡掉我自己和其它直接 Dijkstra 的方法是随机一半的概率放 $0$，其它放随机数，这样生成的极限数据 Dijkstra 需要 2.5s 才过。

直接做当然 T, 因为边数可以达到 $500^3$ 条，点数也有 $500^2$ 个。而 Dijkstra 的复杂度是 $n^2$ 或 $(n+m) \log_2 n$ 的。~~能 hack 的话我就把他们全部叉掉！~~

那么正确做法是怎么样的呢？

如果没有最后一条限制，那么直接 Dijkstra 可以通过，因为边数和 $n^2$ 一个数量级。

那么考虑把最后一种操作转换成向前面一样的常规操作。如果向上的代价是 $i$，那么直接向上连长度为 $1$ 的边就可以了。但这里要 $i+1$, 所以用分层图来处理，第二层图向上连长度为 $1$ 的边，再用长度为 $0$ 的边连到第一层。然后一层到二层再连上长度为 $1$ 的边就可以了。

代码。

```cpp
#include <queue>
#include <cstdio>
#include <cstring>
#define int long long
const int N = 505;
struct twt {
    int x, y, k, d;
    bool operator < (twt b) const {
        return d > b.d;
    }
};
std::priority_queue<twt> que;
int R, C, a[N][N], b[N][N], dis[N][N][2];
void Dijkstra() {
    memset(dis, 0x3f, sizeof dis);
    que.push((twt){1, 1, 0});
    dis[1][1][0] = 0;
    while(!que.empty()) {
        twt now = que.top(); que.pop();
        int x = now.x, y = now.y, d = now.d, k = now.k;
      	if(x == R && y == C) break;
        if(now.d != dis[x][y][k]) continue;
        if(k == 0) {
            if(y < C && d + a[x][y] < dis[x][y+1][k]) {
                dis[x][y+1][k] = d + a[x][y];
                que.push((twt){x, y+1, 0, dis[x][y+1][k]});
            }
            if(y > 1 && d + a[x][y-1] < dis[x][y-1][k]) {
                dis[x][y-1][k] = d + a[x][y-1];
                que.push((twt){x, y-1, 0, dis[x][y-1][k]});
            }
            if(x < R && d + b[x][y] < dis[x+1][y][k]) {
                dis[x+1][y][k] = d + b[x][y];
                que.push((twt){x+1, y, 0, dis[x+1][y][k]});
            }
            if(d+1 < dis[x][y][1]) {
                dis[x][y][1] = d+1;
                que.push((twt){x, y, 1, dis[x][y][1]});
            }
        }
        else {
            if(x > 1 && d + 1 < dis[x-1][y][1]) {
                dis[x-1][y][1] = d+1;
                que.push((twt){x-1, y, 1, dis[x-1][y][1]});
            }
            if(d < dis[x][y][0]) {
                dis[x][y][0] = d;
                que.push((twt){x, y, 0, dis[x][y][0]});
            }
        }
    }
}
signed main() {
    scanf("%lld%lld", &R, &C);
    for(int i = 1; i <= R; i++)
        for(int j = 1; j < C; j++) scanf("%lld", &a[i][j]);
    for(int i = 1; i < R; i++)
        for(int j = 1; j <= C; j++) scanf("%lld", &b[i][j]);
    Dijkstra();
    printf("%lld", dis[R][C][0]);
    return 0;
}
```