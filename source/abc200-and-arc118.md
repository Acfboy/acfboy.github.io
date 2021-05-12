---
title: 'ABC200 & ARC118 游记'
date: 2021-05-10 11:45:03
tags: [游记]
published: true
hideInList: false
feature: 
isTop: false
---
~~一周两场合订版~~

<!-- more -->

## ABC200

首先祝贺 AtCoder 顺利举行 $200$ 场 ABC 了！

然而这次我发挥得却特别糟糕，前三题都是手速极快地秒切的，仅用了六分钟……然后看到 C 题直接无脑 dp, 就翻车了，一直到最后都没有调处来——然后顺利掉分。

> 在序列中选出两个不同的子序列，使它们的和对 $200$ 取模后相等。

其实很简单，由抽屉原理，直接爆搜就可以了……

```cpp
#include <cstdio>
#include <map>
#include <vector>
#include <algorithm>
typedef std::vector<int> twt;
std::map<int, twt> map;
twt now;
int n, a[205];
int main() {
    scanf("%d", &n);
    for(int i = 1; i <= n; i++) scanf("%d", &a[i]), a[i] %= 200;
    for(int i = 1; i < (1 << std::min(n, 8)); i++) {
        now.clear(); int s = 0;
        for(int j = 1; j <= std::min(n, 8); j++) 
            if((1 << (j-1)) & i) s += a[j], s %= 200, now.push_back(j);
        if(map.count(s)) {
            twt pre;
            pre = map[s];
            printf("Yes\n%d ", (signed)pre.size());
            for(int k = 0; k < (signed)pre.size(); k++) printf("%d ", pre[k]);
            printf("\n%d ", (signed)now.size());
            for(int k = 0; k < (signed)now.size(); k++) printf("%d ", now[k]);
            return 0;
        }
        map[s] = now;
    }
    printf("No");
    return 0;
}
```

然后看我的无脑 dp 吧……因为诸多细节还调了一早上，最后对拍出来的。

```cpp
#include <cstdio>
#include <vector>
#include <algorithm>
const int N = 205, A = 1, B = 2, E = 3;
int n, a[N], f[N][N][N], pre[N][N][N];
int flag[N][N][N];
std::vector<int> anA, anB;
void trans1(int i, int j, int k) {
    if((flag[i+1][(j+a[i+1])%200][k] & 1) == 0 || (1 | flag[i][j][k]) == 3) {
        f[i+1][(j+a[i+1])%200][k] = 1;
        pre[i+1][(j+a[i+1])%200][k] = A;
        flag[i+1][(j+a[i+1])%200][k] = (1 | flag[i][j][k]);
    }
}
void trans2(int i, int j, int k) {
    if((flag[i+1][j][(k+a[i+1])%200] & 2) == 0 || (2 | flag[i][j][k]) == 3) {
        f[i+1][j][(k+a[i+1])%200] = 1;
        pre[i+1][j][(k+a[i+1])%200] = B;
        flag[i+1][j][(k+a[i+1])%200] = (2 | flag[i][j][k]);
    }
}
int main() {
    scanf("%d", &n);
    for(int i = 1; i <= n; i++) scanf("%d", &a[i]), a[i] %= 200;
    f[0][0][0] = 1;
    for(int i = 0; i < n; i++) 
        for(int j = 0; j < 200; j++)
            for(int k = 0; k < 200; k++)
                if(f[i][j][k]) {
                    if(flag[i][j][k] & 2) trans1(i, j, k), trans2(i, j, k);
                    else trans2(i, j, k), trans1(i, j, k);
                    if(pre[i+1][j][k] == 0) {
                        pre[i+1][j][k] = E;
                        flag[i+1][j][k] = flag[i][j][k];
                        f[i+1][j][k] = 1;
                     }
                }
    int ani = -1, anj = -1, ank = 0;
    for(int i = 1; i <= n; i++) {
        for(int j = 0; j < 200; j++)
            if(f[i][j][j] && flag[i][j][j] != 0) {
                ani = i, anj = j;
                break;
            }
        if(ani != -1) break;    
    }
    if(ani == -1) return puts("No"), 0; 
    ank = anj;
    for(int i = ani; i >= 1; i--) {
        if(pre[i][anj][ank] == E) continue;
        else if(pre[i][anj][ank] == A) anA.push_back(i), anj = (anj - a[i] + 200) % 200;
        else anB.push_back(i), ank = (ank - a[i] + 200) % 200;
    }
    std::sort(anA.begin(), anA.end());
    std::sort(anB.begin(), anB.end());
    if(anA.size() > anB.size()) std::swap(anA, anB);
    if(anA.size() == 0) {
    	if((signed)anB.size() != n) {
	        int add = -1;
	        for(int i = 0; i < (signed)anB.size(); i++) 
	            if(anB[i] != i+1) { add = i+1; break;}
	        if(anB[anB.size()-1] < n) add = n;
	        if(add == -1) return puts("No"), 0;
	        anA.push_back(add); anB.push_back(add);
	        std::sort(anA.begin(), anA.end());
	        std::sort(anB.begin(), anB.end());
	    }
	    else anA = anB, anA.erase(anA.begin());
    }
    puts("Yes");
    printf("%d ", (signed)anA.size());
    for(int i = 0; i < (signed)anA.size(); i++) printf("%d ", anA[i]);
    printf("\n%d ", (signed)anB.size());
    for(int i = 0; i < (signed)anB.size(); i++) printf("%d ", anB[i]);
    return 0;
}
```

不能这么无脑了，什么性质都不分析然后就杀鸡用牛刀。

其实这场比赛中还有一些值得改进的地方：

- 在比赛中后面心急了直接把 `trans1` 和 `trans2` 的东西压成逗号表达式写，结果因为有个分号调了很久。

- 也因此在最后几分钟发现问题修改时把 `flag` 写成了 `f`, 不然至少赛时数据能过。

所以：**不管情况多么紧急都要保持冷静遵守 SOP 啊！**

## ARC118

这场比赛的体验其实还是挺不错的，甚至拿到了我在 ARC 中的最高名次——结果居然还掉分了？可能是因为得分人数比较少。

### A

> 求第 $k$ 个不能用 $\lfloor \frac{100+t}{100}A \rfloor$ 表示的数。

很显然的，每当 $At$ 超过一个百的时候就会产生一次进位，但是不能直接去用这个计算第 $k$ 个是多少，因为进位之后会先构成后面的数。

所以二分一个 $A$, 然后看最早的  $\lfloor \frac{100+t}{100}x \rfloor \ge A$ 是多少，然后再计算出进了几次位。

```cpp
#include <cstdio>
#define int long long
int n, t;
bool check(int m) {
    int x = (m*100 + 99 + t) / (t+100);
    return x*t/100 >= n;
}
signed main() {
    scanf("%lld%lld", &t, &n);
    int l = 1, r = 10000000000000000, ans = 0;
    while(l <= r) {
        int mid = l + (r - l) / 2;
        if(check(mid)) ans = mid, r = mid - 1;
        else l = mid + 1;
    }
    printf("%lld", ans);
    return 0;
}
```

### B

> 构造一个 $B$ 使 $\left| \frac{b_i}{m} - \frac{a_i}{n} \right|$ 的最大值最小，且所有的 $b_i$ 和是 $m$。

最大值最小，就有二分的味道了。

那考虑怎么验证，发现 $\frac{a_i}{n}$ 是确定的，所以一个简单的不等式推一推就可以确定 $b_i$ 的范围，然后在看一看 $m$ 是不是满足和的范围就可以了。

但这样一写发现样例也过不了，因为这个答案很有可能会很小，随便一精度误差就没了。所以重新观察一下这个式子，发现我们只需要把不等式全乘上一个 $mn$ 就可以让二分的变成一个整数了，直接二分 $mnx$ 即可。

至于构造，开始让 $b_i$ 全部取到最小，然后再一个个能加的加上去就可以了。

代码。

```cpp
#include <cstdio>
#include <algorithm>
#define int long long
const int N = 100005;
int k, n, m, a[N], B[N], limL[N], limR[N], an;
void makeLim(int x) {
    for(int i = 1; i <= k; i++) {
        limL[i] = std::max((m*a[i]-x+n-1) / n, 0ll);
        limR[i] = (m*a[i]+x)/n;
    }
}
bool check(int x) {
    makeLim(x);
    int sumL = 0, sumR = 0;
    for(int i = 1; i <= k; i++) {
        if(limL[i] > limR[i]) return false;
        sumL += limL[i], sumR += limR[i];
    }
    return sumL <= m && m <= sumR;
}
signed main() {
    scanf("%lld%lld%lld", &k, &n, &m);
    for(int i = 1; i <= k; i++) scanf("%lld", &a[i]);
    int l = 1, r = 1000000000, ans = 0;
    while(l <= r) {
        int mid = (l+r) / 2;
        if(check(mid)) r = mid - 1, ans = mid;
        else l = mid + 1;
    }
    makeLim(ans);
    for(int i = 1; i <= k; i++) B[i] = limL[i], an += B[i];
    an = m - an;
    for(int i = 1; i <= k; i++) {
        if(an == 0) break;
        while(an > 0 && B[i] < limR[i]) B[i] ++, an --;
    }
    for(int i = 1; i <= k; i++) printf("%lld ", B[i]);
    return 0;
}
```

### C

从通过人数来看 C 似乎要比 B 简单一些（赛后看来确实如此），但比赛的时候我看 dblark 先过了 C 就先去想了会儿 C, 不过想歪了。

> 构造一个长 $n$ 的徐磊，每个数不超过 $10000$, 数两两不互质但全部放在一起就是互质的。

我想的构造方式是这样的：一堆 $2 \times 3 \times p$ 然后再来 $2x$ 和 $3x$， 然后要 $p$ 是后面的质数的积。

可是这样一来明显凑不满 $2500$ 个。比赛的时候做完 B 时间也没有多少了，最后就没能通过这题。

其实我的构造方案里有大量的浪费，为什么 $p$ 要是后面质数的积呢？

$$
\begin{aligned}
2  \times &3 \times 2 \\
2 \times &3 \times 3 \\
2 \times &3 \times 4 \\
2 &\times 5  \\
3 &\times 5  
\end{aligned}
$$

这样同样是满足要求的，然后就可以进一步发现把一个满足要求的答案里的一个 $a_i$ 拉出来乘几个再放回去一样是合法的。所以只需要设置一个开始数组，如 $\{6, 10, 15\}$ 然后再构造就行了，这样子可以构造出 $\lfloor\frac{10000}{6}\rfloor +\lfloor\frac{10000}{10}\rfloor +\lfloor\frac{10000}{15}\rfloor = 3332$ 个了。

代码。

```cpp
#include <cstdio>
const int N = 2505;
int n, ans[N] = {0, 6, 10, 15}, tot, flag[10005];
int main() {
	scanf("%d", &n);
	tot = 3;
	for(int i = 1; i <= 3; i++) {
		for(int j = ans[i]*2; j*ans[i] <= 10000; j *= ans[i]) {
			if(flag[j]) continue; flag[j] = 1;
			ans[++tot] = j;
			if(tot == n) break;
		}
		if(tot == n) break;
	}
	for(int i = 1; i <= n; i++) printf("%d ", ans[i]);
	return 0;
}
```