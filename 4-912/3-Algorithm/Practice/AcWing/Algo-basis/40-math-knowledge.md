---
publish: "true"
tags:
  - Algorithm
  - AcWing
date: 2024-02-28
---
## 质数

所有小于等于 1 的整数既不是质数也不是合数。

### 试除法求素数

>[!note] 质数的判定：试除法
>1) d|n 代表的含义是 d 能整除 n, (这里的 `|` 代表整除，注意 d 是较小的数，而 n 是较大的的数)
>2) 一个合数的约数总是成对出现的, 如果 d|n,那么 (n/d)|n, 因此我们判断一个数是否为质数的时候,只需要判断较小的那一个数能否整除 n 就行了, 即只需枚举 d<=(n/d), 即 dd<=n, d<=sqrt (n)就行了.
>3) `sqrt(n)` 这个函数执行的时候比较慢。

如果想要快速求出大于某个数的素数（这对设置散列表的表长非常有用），可以：

```cpp
for (int i = 100000;; i++) {
	bool flag = true;
	for (int j = 2; j * j <= i; j++) {
		if (i % j == 0) {
			flag = false;
			break;
		}
	}
	if (flag) {
		cout << i << endl;
		break;
	}
}

// 要判断一个数是不是素数，只需要：
bool is_prime(int x)
{
    if (x < 2) return false;
    for (int i = 2; i <= x / i; i ++ )
        if (x % i == 0)
            return false;
    return true;
}
```

### 分解质因数

**试除法**

> 用到的原理:唯一分解定理 (算数基本定理))

1) 特别要注意------分解质因数与质因数不一样！分解质因数是一个过程, 而质因数是一个数。
2) 一个合数分解而成的质因数最多只包含一个大于 $\sqrt{n}$ 的质因数(反证法, 若 n 可以被分解成两个大于 sqrt (n)的质因数, 则这两个质因数相乘的结果大于 n, 与事实矛盾)。
3) 当枚举到某一个数 i 的时候， n 的因子里面已经不包含 2-i-1 里面的数,如果 `n%i==0`, 则 i 的因子里面也已经不包含 2-i-1 里面的数, 因此每次枚举的数都是质数。
4) **算数基本定理** (唯一分解定理): 任何一个大于 1 的自然数 N, 如果 N 不为质数, 那么 N 可以唯一分解成有限个质数的乘积 $N=p_{1}^{a_{1}}\times p_{2}^{a_{2}}\times...\times p_{k}^{a_{k}}$，这里 $p_{1}<p_{2}<...<p_{k}$ 均为质数，其中指数 $a_i$ 是正整数。这样的分解称为 N 的标准分解式。最早证明是由欧几里得给出的，由陈述证明。此定理可推广至更一般的交换代数和代数数论。
5) 质因子（或质因数）在数论里是指能整除给定正整数的质数。根据算术基本定理，不考虑排列顺序的情况下，每个正整数都能够以唯一的方式表示成它的质因数的乘积。
6) 两个没有共同质因子的正整数称为互质。因为 1 没有质因子，1 与任何正整数（包括 1 本身）都是互质。
7) 只有一个质因子的正整数为质数。

```cpp
// 判断质因数
void divide(int x) 
{
    for (int i = 2; i <= x / i; i ++ )
        if (x % i == 0)
        {
            int s = 0;
            while (x % i == 0) x /= i, s ++ ;
            cout << i << ' ' << s << endl;
        }
    if (x > 1) cout << x << ' ' << 1 << endl;
    cout << endl;
}
/*从小到大的顺序输出其分解质因数后，每个质因数的底数和指数，每个底数和指数占一行。

每个正整数的质因数全部输出完毕后，输出一个空行。*/
```

### 朴素筛法

1). 做法: 把 2~(n-1)中的所有的数的倍数都标记上, 最后没有被标记的数就是质数.
(2). 原理: 假定有一个数 p 未被 2~(p-1)中的数标记过, 那么说明, 不存在 2~(p-1)中的任何一个数的倍数是 p,
也就是说 p 不是 2~(p-1)中的任何数的倍数, 也就是说 2~(p-1)中不存在 p 的约数, 因此, 根据质数的定义可知:
p 是质数.
(3). 调和级数: 当 n 趋近于正无穷的时候, 1/2+1/3+1/4+1/5+…+1/n=lnn+c.(c 是欧阳常数, 约等于 0.577 左右.).
(4). 底数越大, log 数越小
(4). 时间复杂度: 约为 O (nlogn);(注: 此处的 log 数特指以 2 为底的 log 数).

```cpp
int primes[N], cnt;     // primes[]存储所有素数
bool st[N];         // st[x]存储x是否被筛掉

void get_primes(int n)
{
    for (int i = 2; i <= n; i ++ )
    {
        if (st[i]) continue;
        primes[cnt ++ ] = i;
        for (int j = i + i; j <= n; j += i)
            st[j] = true;
    }
}
```

### 埃氏筛法

(1). 质数定理:1~n 中有 n/lnn 个质数.
(2). 原理: 在朴素筛法的过程中只用质数项去筛.
(3). 时间复杂度:粗略估计: O (n).实际: O (nlog (logn)).
(4). 1~n 中, 只计算质数项的话,”1/2+1/3+1/4+1/5+…+1/n”的大小约为 log (logn).

### 线性筛法

首先，线性筛法是对埃氏筛法的优化：因为即使埃氏筛法是朴素做法的优化版，但是还是会重复标记一个合数。
因此，线性筛法的核心步骤，就是避免将一个合数被标记多次，以此达到 $O (n)$ 的时间复杂度。体现在代码里面，就是：`if (i % primes[j] == 0) break;`。

y 总的解释是：`primes[j]` 一定是 i 的最小质因子。但是这句话并不清晰，或者说由这句话还可以得出一个结论：i 是个合数，并且之前已经被 primes[j] 筛过了。再换个说法，就是说 i 不论乘什么数，都是 primes[j] 的倍数。这就意味着，不需要再使用 i 去筛其他的数，因为 primes[j] 会代劳。
例如：如果 i 是 10，第一个质数是 2，那么 10 % 2 == 0。10 不论乘多少（得到 30, 50, 70, ......），都是第一个质数 2 的倍数，所以没必要用 10 去筛其他数，避免重复标记，因此 break 掉。

但是为什么要把 st[primes[j] * i] = true; 放在前面？我个人是这么理解的，例如 20，分解质因数为 2 * 2 * 5，但是因为每次只有一个质数和 i 相乘，所以只有在 i == 10 的情况下才能筛掉 20（分解为 2 * 10）。依此类推。

还有一个问题，为什么不用写 j < cnt？因为：
- 如果 i 是质数，那么在循环之前已经被加入到了 primes[] 中，所以一定会存在 i % primes[j] == 0 成立（两个相同的数相除，和为 1，余数为 0），break 掉。
- 如果 i 是合数，一定会存在一个最小质因子，使得 if 语句成立，break 掉；或者在这之前，因为不满足循环条件从而停止循环。

还需注意的是，此处的循环条件 primes[j] <= n / i 代表的不是 primes[j] <= sqrt (n)。取原式的等号，可以得到：primes[j] = n / i，因此可以得到 n = primes[j] * i。也就是说这个条件所代表的是：在 i 已经确定的情况下，primes[j] 为多少才能使得结果小于约束条件 n。因为在这个写法里，要求出的是 1~n 之间的质数。

```cpp
int primes[N], cnt;     // primes[]存储所有素数
bool st[N];         // st[x]存储x是否被筛掉

void get_primes(int n)
{
    for (int i = 2; i <= n; i ++ )
    {
        if (!st[i]) primes[cnt ++ ] = i;
        for (int j = 0; primes[j] <= n / i; j ++ )
        {
            st[primes[j] * i] = true;
            if (i % primes[j] == 0) break;
        }
    }
}
```

## 约数

### [869. 试除法求约数](https://www.acwing.com/problem/content/description/871/)

```cpp
vector<int> get_divisors(int x)
{
    vector<int> res;
    for (int i = 1; i <= x / i; i ++ )
        if (x % i == 0)
        {
            res.push_back(i);
            if (i != x / i) res.push_back(x / i);
        }
    sort(res.begin(), res.end());
    return res;
}
```

```cpp
// 试除法求约数

#include <algorithm>
#include <iostream>
#include <vector>
using namespace std;
int main() {
    int T;
    cin >> T;
    while (T--) {
        int n;
        cin >> n;
        vector<int> res;
        // 因为约数成对出现，所以只需要循环到根号x
        //  不要是用 i *i <= n，因为可能溢出
        for (int i = 1; i <= n / i; i++) {
            if (n % i == 0) {
                res.push_back(i);
                // 如果i * i = x,添加i即可，不用添加 x / i
                if (n / i != i)
                    res.push_back(n / i);
            }
        }
        sort(res.begin(), res.end());
        for (auto x : res) cout << x << " ";
        cout << endl;
    }
}
```

### [870. 约数的个数](https://www.acwing.com/problem/content/description/872/)

使用 `unordered_map` 来存储每个质因数及其出现的次数，而不是直接计算乘积的所有约数。约数的总数可以通过质因数分解的结果来计算：如果一个数的质因数分解是  $p_{1}^{a_{1}}\times p_{2}^{a_{2}}\times...\times p_{k}^{a_{k}}$，那么它的约数个数就是 $(a_{1}+1)\times(a_{2}+1)\times...\times(a_{1}+1)\times(a_{k}+1)$ 。

```cpp
#include <iostream>
#include <unordered_map>
using namespace std;
typedef long long LL;

const int N = 110;
int n, arr[N];
const int mod = 7 + 1e9;

// 对x进行质因数分解，并统计每个质因数的次数
unordered_map<LL, int> prime_factors(LL x) {
    unordered_map<LL, int> factors;
    for (LL i = 2; i <= x / i; i++) {
        while (x % i == 0) {
            factors[i]++;
            x /= i;
        }
    }
    if (x > 1) factors[x]++;
    return factors;
}

// 计算约数的数量
LL num_of_divisors(const unordered_map<LL, int>& factors) {
    LL ans = 1;
    for (auto& [prime, exp] : factors) {
        ans = ans * (exp + 1) % mod;
    }
    return ans;
}

int main() {
    cin >> n;
    unordered_map<LL, int> total_factors;
    for (int i = 0; i < n; i++) {
        cin >> arr[i];
        auto factors = prime_factors(arr[i]);
        for (auto& [prime, exp] : factors) {
            total_factors[prime] += exp;
        }
    }
    cout << num_of_divisors(total_factors) % mod;
    return 0;
}

```

>[!error] 注意不要使用暴力法，那样时间复杂度过高，一定会 TLE 的。要利用质因数与约数之间的关系来解答。

### [871. 约数之和](https://www.acwing.com/problem/content/873/)

是的，我们仍然可以利用质因数分解的方法来求解约数总和。如果一个数的质因数分解是  $p_{1}^{a_{1}}\times p_{2}^{a_{2}}\times...\times p_{k}^{a_{k}}$ ，那么它的约数总和可以用以下公式计算：
$$
\text{Sum} = \left (\sum_{i=0}^{a_1}p_1^i\right) \cdot \left (\sum_{i=0}^{a_2}p_2^i\right) \cdot \ldots \cdot \left (\sum_{i=0}^{a_k}p_k^i\right)
$$

其中，$\sum_{i=0}^{a_j}p_j^i$ 是一个等比数列的求和公式，可以简化为 $\frac{p_j^{a_j+1}-1}{p_j-1}$ ，如果 $p_j$ 是质数的话。

```cpp
#include <cmath>
#include <iostream>
#include <unordered_map>
using namespace std;
typedef long long LL;

const int N = 110;
int       n, arr[N];
const int mod = 7 + 1e9;

// 对x进行质因数分解，并统计每个质因数的次数
unordered_map<LL, int> prime_factors(LL x) {
    unordered_map<LL, int> factors;
    for (LL i = 2; i <= x / i; i++) {
        while (x % i == 0) {
            factors[i]++;
            x /= i;
        }
    }
    if (x > 1) factors[x]++;
    return factors;
}

// 快速幂算法，计算 a^b % m
LL quick_pow(LL a, LL b, LL m) {
    LL ans = 1;
    a      = a % m;
    while (b > 0) {
        if (b & 1) ans = ans * a % m;
        a = a * a % m;
        b >>= 1;
    }
    return ans;
}

// 计算等比数列之和：(p^(a+1) - 1) / (p - 1) % mod
LL sum_of_geometric_series(LL p, int a, LL mod) {
    LL numerator = quick_pow(p, a + 1, mod) - 1;     // 分子 p^(a+1) - 1
    if (numerator < 0) numerator += mod;             // 确保结果为正
    LL denominator = quick_pow(p - 1, mod - 2, mod); // 分母的逆元 (p-1)^(mod-2) % mod
    return numerator * denominator % mod;
}

// 计算约数的和
LL divisor_sum(const unordered_map<LL, int> &factors, LL mod) {
    LL ans = 1;
    for (auto &[prime, exp] : factors) {
        LL sum = sum_of_geometric_series(prime, exp, mod);
        ans    = ans * sum % mod;
    }
    return ans;
}

int main() {
    cin >> n;
    unordered_map<LL, int> total_factors;
    for (int i = 0; i < n; i++) {
        cin >> arr[i];
        auto factors = prime_factors(arr[i]);
        for (auto &[prime, exp] : factors) {
            total_factors[prime] += exp;
        }
    }
    cout << divisor_sum(total_factors, mod) % mod;
    return 0;
}
<center></center>
```

>[!tip] 为什么要手动实现一个快速幂函数，而不是调用 cmath 库中的 `pow` 函数呢？
>直接使用`%`运算符与浮点数`pow`函数的结果进行取模操作会导致编译错误，因为`%`运算符只适用于整数。另外，计算等比数列之和时应使用一个有效处理模运算的快速幂算法，而不是直接使用`pow`函数，因为后者是为浮点数设计的，且在涉及模运算时可能导致精度问题。
>在进行模运算时处理大数的幂次方，我们通常使用快速幂算法，并在每步都取模以避免溢出。

### 最大公约数（欧几里得算法）

```cpp
int gcd(int a, int b)
{
    return b ? gcd(b, a % b) : a;
}
```

![[40-math-knowledge-gcd.png]]

## 扩展欧几里得算法

1. **扩展欧几里得**：用于求解方程 ax+by=gcd (a, b) 的解

当 $b=0$ 时 $ax+by=a$ 故而 $x=1, y=0$ 
当 $b≠0$ 时，因为 $gcd (a, b)=gcd (b, a\%b)$ ，而
$$
\begin{aligned}
bx^{\prime} +(a\%b) y^{\prime}&=gcd (b, a\%b)\\
bx^{\prime}+(a−⌊a/b⌋∗b) y^{\prime}&=gcd (b, a\%b)\\
ay^{\prime}+b (x^{\prime}−⌊a/b⌋∗y^{\prime})&=gcd (b, a\%b)=gcd (a, b)
\end{aligned}
$$
故而
$$
x=y^{\prime}, y=x^′−⌊a/b⌋∗y^′
$$

因此可以采取递归算法 先求出下一层的 x′和 y′ 再利用上述公式回代即可

2. **对于求解更一般的方程 $ax+by=c$**

设 $d=gcd (a, b)$ 则其有解当且仅当 $d|c$（整除）。求解方法如下：用扩展欧几里得求出 $ax_0+by_0=d$ 的解，则 $a (x_{0}\times \frac{c}{d})+b (y_{0}\times \frac{c}{d})=c$ 。故而特解为 $x^{\prime}=x_{0}\times \frac{c}{d}, y^{′}=y_{0}\times \frac{c}{d}$ ，而**通解 = 特解 + 齐次解**，其中齐次解为方程 $ax+by=0$ 的解。故而通解为 $x=x^{′}+k\times \frac{b}{d}, y=y^{′}−k\times \frac{a}{d},k∈z$ ，若令 $t=\frac{b}{d}$ ，则对于 x 的最小非负整数解为 $(x^{′}\%t+t)\%t$ 。

3. **求解一次同余方程 $ax≡b\mod m$**
等价于求
$$
\begin{aligned}
&ax+my=b  
\end{aligned}
$$
此式有解的条件为 $gcd (a, m)|b$ （Bezout 定理），于是先用扩展欧几里得算出一组整数 $x_0,y_0$ 使得 $ax_{0}+my_{0}=gcd(a,m)$ ，然后 $x=x_{0}\times \frac{b}{gcd(a,m)}\%m$ 即是所求。

**特别的，当 b=1 且 a 与 m 互质时则所求的 x 即为 a 的[逆元](https://www.acwing.com/solution/acwing/content/8512/)。**

```cpp
// 求x, y，使得ax + by = gcd(a, b)
int exgcd(int a, int b, int &x, int &y)
{
    if (!b)
    {
        x = 1; y = 0;
        return a;
    }
    int d = exgcd(b, a % b, y, x);
    y -= (a/b) * x;
    return d;
}
```

### [878. 线性同余方程](https://www.acwing.com/problem/content/880/)

```cpp
// 线性同余方程
#include <iostream>

using namespace std;

const int N = 100010;

// 求x, y，使得ax + by = gcd(a, b)
int exgcd(int a, int b, int &x, int &y) {
    if (!b) {
        x = 1;
        y = 0;
        return a;
    }
    int d = exgcd(b, a % b, y, x);
    y -= (a / b) * x;
    return d;
}

int main() {
    int n;
    cin >> n;
    while (n--) {
        int a, b, m, x, y;
        cin >> a >> b >> m;
        int ans = exgcd(a, m, x, y);
        if (b % ans != 0)
            cout << "impossible" << endl;
        else {
            x = ( long long ) x * (b / ans) % m;
            cout << x << endl;
        }
    }
    return 0;
}
```

## 欧拉函数

欧拉函数 $\varphi(n)$ 指的是自然数 $1\sim n$ 之间与 $n$ 互质的元素的个数。而任何自然数都可以分解为质因数：
$$
n=p_{1}^{a_{1}}\times p_{2}^{a_{2}}\times...\times p_{k}^{a_{k}}$$
而欧拉函数则可以表示为：
$$
\varphi(n)=n\times(1-\frac{1}{p_{1}})\times(1-\frac{1}{p_{2}})\times...\times(1-\frac{1}{p_k})
$$

```cpp
// 朴素欧拉
int phi(int x)
{
    int res = x;
    for (int i = 2; i <= x / i; i ++ )
        if (x % i == 0)
        {
            res = res / i * (i - 1);
            while (x % i == 0) x /= i;
        }
    if (x > 1) res = res / x * (x - 1);

    return res;
}

// 筛法求欧拉
int primes[N], cnt;     // primes[]存储所有素数
int euler[N];           // 存储每个数的欧拉函数
bool st[N];         // st[x]存储x是否被筛掉


void get_eulers(int n) //制作从1~n的所有数的欧拉函数表
{
    euler[1] = 1;
    for (int i = 2; i <= n; i ++ )
    {
        if (!st[i])
        {
            primes[cnt ++ ] = i;
            euler[i] = i - 1;
        }
        for (int j = 0; primes[j] <= n / i; j ++ )
        {
            int t = primes[j] * i;
            st[t] = true;
            if (i % primes[j] == 0)
            {
                euler[t] = euler[i] * primes[j];
                break;
            }
            euler[t] = euler[i] * (primes[j] - 1);
        }
    }
}
```

>[!note] 筛法求欧拉的最大表长是多少？
>在 64MB 的空间限制下，最多能支持大约 $1.68 \times 10^7$ 个 `int` 类型的数据。因此，如果通过预先计算出欧拉函数表的方式来处理，n 的范围最大大约是 $1.68 \times 10^7$（即约 1 千 6 百万）。这是考虑到每个数字需要一个 `int` 类型的空间来存储其欧拉函数值的情况下得出的估算结果。

## 快速幂

### [875. 快速幂](https://www.acwing.com/problem/content/submission/877/)

```cpp
// Quick Power
#include <iostream>

using namespace std;
typedef unsigned long long ULL;

const int N = 100010;
int       n, arr[N];

ULL quic_power(int m, int k, int p) { // m^k mod p
    ULL res = 1 % p, t = m;
    while (k) {
        if (k & 1) res = res * t % p;
        t = t * t % p;
        k >>= 1;
    }
    return res;
}

int main() {
    cin >> n;
    while (n--) {
        int a, b, p;
        cin >> a >> b >> p;
        cout << quic_power(a, b, p) << endl;
    }
    return 0;
}
```

### [876. 快速幂求逆元](https://www.acwing.com/problem/content/878/)

**当 n 为质数时，可以用快速幂求逆元**：
a / b ≡ a * x (mod n)
两边同乘 b 可得 a ≡ a * b * x (mod n)
即 1 ≡ b * x (mod n)
同 b * x ≡ 1 (mod n)
由费马小定理可知，当 n 为质数时
b ^ (n - 1) ≡ 1 (mod n)
拆一个 b 出来可得 b * b ^ (n - 2) ≡ 1 (mod n)
故当 n 为质数时，b 的乘法逆元 x = b ^ (n - 2)

**当 n 不是质数时，可以用扩展欧几里得算法求逆元**：
a 有逆元的充要条件是 a 与 p 互质，所以 gcd (a, p) = 1
假设 a 的逆元为 x，那么有 a * x ≡ 1 (mod p)
等价：ax + py = 1
exgcd (a, p, x, y)

```cpp
// 快速幂求逆元
#include <iostream>
using namespace std;
typedef long long LL;

LL qmi(int a, int b, int p) {
    LL res = 1;
    while (b) {
        if (b & 1) res = res * a % p;
        a = ( LL ) a * a % p;
        b >>= 1;
    }
    return res;
}

int main() {
    int n;
    cin >> n;
    while (n--) {
        int a, p;
        cin >> a >> p;
        if (a % p == 0)
            puts("impossible");
        else
            cout << qmi(a, p - 2, p) << endl;
    }
    return 0;
}

//扩展欧几里得求逆元
#include <iostream>
using namespace std;
typedef long long LL;

int n;

int exgcd(int a, int b, int &x, int &y) {
    if (!b) {
        x = 1, y = 0;
        return a;
    }
    int d = exgcd(b, a % b, y, x);
    y -= a / b * x;
    return d;
}

int main() {
    cin >> n;
    while (n--) {
        int a, p, x, y;
        // if (a < p) swap(a, p);
        cin >> a >> p;
        int d = exgcd(a, p, x, y);
        if (d == 1)
            cout << (( LL ) x + p) % p << endl; // 保证x是正数
        else
            puts("impossible");
    }
    return 0;
}
```

## 中国剩余定理

中国剩余定理给出了以下的一元线性同余方程组：
$$
{\displaystyle (S):\quad \left\{{\begin{matrix}x\equiv a_{1}{\pmod {m_{1}}}\\x\equiv a_{2}{\pmod {m_{2}}}\\\vdots \qquad \qquad \qquad \\x\equiv a_{n}{\pmod {m_{n}}}\end{matrix}}\right.}$$
有解的判定条件，并用构造法给出了在有解情况下解的具体形式。

**中国剩余定理的说明**：假设整数 $m_1, m_2, ... , m_n$ 其中任两数互质，则对任意的整数：$a_1, a_2, ... , a_n$，方程组 $(S)$ 有解，并且通解可以用如下方式构造得到：

1. 设 ${\displaystyle M=m_{1}\times m_{2}\times \cdots \times m_{n}=\prod _{i=1}^{n}m_{i}}$ 是整数 $m_1, m_2, ... , m_n$ 的乘积，并设 ${\displaystyle M_{i}=M/m_{i},\;\;\forall i\in \{1,2,\cdots ,n\}}$ ，即 $M_i$ 是除了 $m_i$ 以外的 $n − 1$ 个整数的乘积。
2. 设 ${\displaystyle t_{i}=M_{i}^{-1}}$ 为 $M_i$ 模 $m_i$ 的逆元：${\displaystyle t_{i}M_{i}\equiv 1{\pmod {m_{i}}},\;\;\forall i\in \{1,2,\cdots ,n\}.}$
3. 方程组 $(S)$ 的通解形式为：${\displaystyle x=a_{1}t_{1}M_{1}+a_{2}t_{2}M_{2}+\cdots +a_{n}t_{n}M_{n}+kM=kM+\sum _{i=1}^{n}a_{i}t_{i}M_{i},\quad k\in \mathbb {Z} .}$ 在模 $M$ 的意义下，方程组 $(S)$ 只有一个解：${\displaystyle x=\sum _{i=1}^{n}a_{i}t_{i}M_{i}.}$

```cpp
LL CRT(int k, LL* a, LL* r) {
  LL n = 1, ans = 0;
  for (int i = 1; i <= k; i++) n = n * r[i];
  for (int i = 1; i <= k; i++) {
    LL m = n / r[i], b, y;
    exgcd(m, r[i], b, y);  // b * m mod r[i] = 1
    ans = (ans + a[i] * m * b % n) % n;
  }
  return (ans % n + n) % n;
}
```

### [204. 表达整数的奇怪方式](https://www.acwing.com/problem/content/206/)

```cpp
#include <iostream>

using namespace std;

typedef long long LL; // 数据范围比较大,所以用LL来存储

LL exgcd(LL a, LL b, LL &x, LL &y) {
    if (!b) {
        x = 1, y = 0;
        return a;
    }
    LL d = exgcd(b, a % b, y, x);
    y -= a / b * x;
    return d;
}

int main() {
    int n;
    LL  a1, m1;
    cin >> n >> a1 >> m1;
    LL x = 0;
    for (int i = 1; i < n; i++) {
        LL a2, m2;
        cin >> a2 >> m2;
        LL k1, k2;
        LL d = exgcd(a1, a2, k1, k2);
        if ((m2 - m1) % d) {
            x = -1;
            break;
        }
        k1 *= (m2 - m1) / d;
        // 因为此时k1是k1*a1+k2*a2=d的解,所以要乘上(m2-m1)/d的倍数大小
        LL t = abs(a2 / d);
        k1   = (k1 % t + t) % t;
        // 数据比较极端,所以只求k的最小正整数解
        m1 = k1 * a1 + m1;
        // m1在被赋值之后的值为当前"x"的值，此时赋值是为了方便下一轮的继续使用
        a1 = abs(a1 * a2 / d);
        // 循环结束时a1的值为当前所有的a1,a2,……an中的最小公倍数
    }
    if (x != -1)
        x = (m1 % a1 + a1) % a1;
    // 当循环结束时，此时的值应该与最小公倍数取模,以求得最小正整数解
    printf("%lld\n", x);
    return 0;
}
```

## 解线性方程组

### [883. 高斯消元法解线性方程组](https://www.acwing.com/problem/content/885/)

```cpp
// 高斯消元法解线性方程组
#include <cmath>
#include <iostream>

using namespace std;

const int N = 110;
int       n;
double    a[N][N]; // augmented matrix
double    eps = 1e-4;

int gauss() {
    int col, row;
    for (col = 0, row = 0; col < n; col++) {
        int t = row;
        for (int i = row; i < n; i++) // 找到绝对值最大的行
            if (fabs(a[i][col]) > fabs(a[t][col]))
                t = i;

        if (fabs(a[t][col]) < eps) continue;

        for (int i = col; i <= n; i++) swap(a[t][i], a[row][i]); // 将绝对值最大的行换到最顶端
        for (int i = n; i >= col; i--) a[row][i] /= a[row][col]; // 将当前行的首位变成1
        for (int i = row + 1; i < n; i++)                        // 用当前行将下面所有的列消成0
            if (fabs(a[i][col]) > eps)
                for (int j = n; j >= col; j--)
                    a[i][j] -= a[row][j] * a[i][col];

        row++;
    }

    if (row < n) {
        for (int i = row; i < n; i++)
            if (fabs(a[i][n]) > eps)
                return 2; // 无解
        return 1;         // 有无穷多组解
    }

    for (int i = n - 1; i >= 0; i--)
        for (int j = i + 1; j < n; j++)
            a[i][n] -= a[i][j] * a[j][n];

    return 0; // 有唯一解
}

int main() {
    cin >> n;
    for (int i = 0; i < n; i++)
        for (int j = 0; j < n + 1; j++)
            cin >> a[i][j];

    int res = gauss();
    if (res == 0) {
        for (int i = 0; i < n; i++)
            printf("%.2f\n", a[i][n]);

    } else if (res == 1)
        cout << "Infinite group solutions" << endl;
    else if (res == 2)
        cout << "No solution" << endl;
}
```

### [884. 高斯消元解异或线性方程组]( https://www.acwing.com/problem/content/886/ )

```cpp
// 高斯消元解异或线性方程组
#include <iostream>

using namespace std;

const int N = 110;
int       n;
int       a[N][N];

/*
核心思想: 异或-不进位的加法
那么等式与等式间的异或要一起进行才能保证等式左右两边依然是相等关系!
 a^b^c = x
   d^f = y
   则
 a^b^d^c^f = x^y
1 左下角消0
  1.1 枚举列
  1.2 找第一个非零行
  1.3 交换
  1.4 把同列下面行消零(异或)
2 判断3种情况
  2.1 唯一解
  2.2 秩<n
      2.2.1 有矛盾 无解
      2.2.2 无矛盾 无穷多解

// 左下角消
for(int i=r+1;i<n;i++)
    if(a[i][c])//漏了
        for(int j=n;j>=c;j--)//漏了
            a[i][j] ^= a[r][j];

for(int i=r;i<n;i++)
    if(a[i][c])
        t= i;//写成t=r

for(int i=n-1;i>=0;i--)
    for(int j=i+1;j<n;j++)//写成j=r+1
*/

int gauss() {
    int c, r;
    for (c = 0, r = 0; c < n; c++) {
        // 找主元
        int t = -1;
        for (int i = r; i < n; i++)
            if (a[i][c]) {
                t = i;
                break;
            }
        if (t == -1) continue;
        // 交换主元行
        for (int j = c; j <= n; j++) swap(a[r][j], a[t][j]);
        // 左下角消
        for (int i = r + 1; i < n; i++)
            if (a[i][c])                     // 漏了
                for (int j = n; j >= c; j--) // 漏了
                    a[i][j] ^= a[r][j];
        r++;
    }
    // 判断
    if (r < n) {
        for (int i = r; i < n; i++) // i=r
            if (a[i][n])
                return 2;
        return 1;
    }
    // 消右上角
    for (int i = n - 1; i >= 0; i--)
        for (int j = i + 1; j < n; j++)
            // 如果是0 就不用下面的a[j][j] 来^a[i][j]了
            // 如果不是0 才需要用第j行第j列a[j][j]来^第i行第j列a[i][j]
            // 进而进行整行row[i]^row[j] 间接导致 a[i][n]^a[j][n]
            if (a[i][j])
                a[i][n] ^= a[j][n];
    return 0;
}

int main() {
    cin >> n;
    for (int i = 0; i < n; i++)
        for (int j = 0; j <= n; j++)
            cin >> a[i][j];
    int t = gauss();
    if (t == 0) {
        for (int i = 0; i < n; i++) cout << a[i][n] << endl;
    } else if (t == 1)
        puts("Multiple sets of solutions");
    else
        puts("No solution");
    return 0;
}
```

## 组合数

### [885. 递推法求组合数](https://www.acwing.com/problem/content/887/)

```cpp
// c[a][b] 表示从a个苹果中选b个的方案数
for (int i = 0; i < N; i ++ )
    for (int j = 0; j <= i; j ++ )
        if (!j) c[i][j] = 1;
        else c[i][j] = (c[i - 1][j] + c[i - 1][j - 1]) % mod;
```

### [886. 预处理逆元法求组合数](https://www.acwing.com/problem/content/888/)

首先预处理出所有阶乘取模的余数 `fact[N]`，以及所有阶乘取模的逆元 `infact[N]` 。如果取模的数是质数，可以用费马小定理求逆元。

**费马小定理**（英语：Fermat's little theorem）是数论中的一个定理。假如 $a$ 是一个整数，$p$ 是一个质数，那么 $a^p-a$ 是 $p$ 的倍数，可以表示为
$$
{\displaystyle a^{p}\equiv a{\pmod {p}}}
$$
如果 $a$ 不是 $p$ 的倍数，这个定理也可以写成更加常用的一种形式
$$
{\displaystyle a^{p-1}\equiv 1{\pmod {p}}}
$$

费马小定理的逆叙述不成立，即假如 $a^p-a$ 是 $p$ 的倍数，$p$ 不一定是一个质数。例如 $2^{341}−2$ 是 $341$ 的倍数，但 $341=11×31$ ，不是质数。满足费马小定理的合数被称为费马伪素数。

费马小定理指出，如果 `p` 是一个质数，对于任意整数 `a`（`a` 不是 `p` 的倍数），有 `a^(p-1) ≡ 1 (mod p)`。这意味着 `a^(p-1) - 1` 是 `p` 的倍数。

进一步，由费马小定理可推导出 `a^(p-2) ≡ a^(-1) (mod p)`。这里的 `a^(-1)` 表示 `a` 模 `p` 的乘法逆元。乘法逆元的定义是在模运算的环境下，`a * a^(-1) ≡ 1 (mod p)`。这给出了一种计算逆元的有效方法，即 `a^(-1) = a^(p-2) mod p`。

在模 `p` 的算术中，如果存在两个数 `a` 和 `b` 满足 `a * b ≡ 1 (mod p)`，则称 `b` 是 `a` 的逆元。在很多算法中，我们需要使用除法，但是模运算下直接的除法不是很方便，此时可以通过乘以逆元来实现。计算逆元的一个通用方法是扩展欧几里得算法，但当模数 `p` 是质数时，可以直接使用费马小定理来计算逆元，如 `a^(-1) = a^(p-2) mod p`。

在计算组合数 `C(n, m) = n! / (m! * (n-m)!)` 时，**直接计算阶乘然后做除法在模运算的环境下是不可行的**，因为除法需要转换为乘以逆元。预先计算好从 `1` 到 `n` 的所有数的阶乘取模 `fact[i] = i! mod p` 和它们的逆元 `infact[i] = (i!)^(-1) mod p` 可以极大地简化组合数的计算，因为组合数可以改写为 `C(n, m) = fact[n] * infact[m] * infact[n-m] mod p`。

**如何使用这些概念求组合数**？

预处理阶段，我们计算并存储 `1` 到 `n` 的所有阶乘的模 `p` 值和它们的逆元。计算组合数 `C(n, m)` 时，我们直接使用预处理的结果，通过乘法和模运算来完成计算，避免了复杂的除法操作。

1. **预处理阶乘**：计算 `fact[i] = i! mod p`，直接连乘即可。
2. **预处理阶乘的逆元**：计算 `infact[i] = (i!)^(-1) mod p`，使用费马小定理 `infact[i] = fact[i]^(p-2) mod p` 或者通过递归关系 `infact[i] = infact[i-1] * qmi(i, p-2, p) mod p` 来计算。
3. **计算组合数**：通过预处理的结果，`C(n, m) = fact[n] * infact[m] * infact[n-m] mod p`。

这样，即可高效且准确地在模质数 `p` 的条件下计算组合数。

```cpp
#include <iostream>
using namespace std;
typedef long long LL;

const int N = 100010; // 根据问题需求调整N的大小
const int mod = 1e9 + 7;

LL fact[N], infact[N];

LL quick_pow(LL base, LL power) { //参数里还可以再添加一个取模的参数，用以覆盖mod这个变量
    LL result = 1;
    while (power) {
        if (power & 1) result = result * base % mod;
        base = base * base % mod;
        power >>= 1;
    }
    return result;
}

// 预处理阶乘的余数和阶乘逆元的余数
void init() {
    fact[0] = infact[0] = 1;
    for (int i = 1; i < N; i++) {
        fact[i] = fact[i - 1] * i % mod;
        infact[i] = infact[i - 1] * quick_pow(i, mod - 2) % mod;
    }
}

LL combination_num(LL n, LL m) {
    if (m > n) return 0;
    return fact[n] * infact[m] % mod * infact[n - m] % mod;
}

int main() {
    init(); // 预处理阶乘及其逆元
    int t;
    cin >> t;
    while (t--) {
        LL n, m;
        cin >> n >> m;
        cout << combination_num(n, m) << endl;
    }
    return 0;
}

```

### [887. Lucas 定理](https://www.acwing.com/problem/content/889/)

若 p 是质数，则对于任意整数 1 <= m <= n，有：
$$
C (n, m) = C (n \% p, m \% p) * C (\frac{n}{p},  \frac{m}{p}) \mod p
$$

```cpp
int qmi(int a, int k, int p)  // 快速幂模板
{
    int res = 1 % p;
    while (k)
    {
        if (k & 1) res = (LL)res * a % p;
        a = (LL)a * a % p;
        k >>= 1;
    }
    return res;
}

int C(int a, int b, int p)  // 通过定理求组合数C(a, b)
{
    if (a < b) return 0;

    LL x = 1, y = 1;  // x是分子，y是分母
    for (int i = a, j = 1; j <= b; i --, j ++ )
    {
        x = (LL)x * i % p;
        y = (LL) y * j % p;
    }

    return x * (LL)qmi(y, p - 2, p) % p;
}

int lucas(LL a, LL b, int p)
{
    if (a < p && b < p) return C(a, b, p);
    return (LL)C(a % p, b % p, p) * lucas(a / p, b / p, p) % p;
}
```

### [888. 分解质因数法求组合数](https://www.acwing.com/problem/content/890/)

当我们需要求出组合数的真实值，而非对某个数的余数时，分解质因数的方式比较好用：
1. 筛法求出范围内的所有质数
2. 通过 C (a, b) = a! / b! / (a - b)! 这个公式求出每个质因子的次数。 n! 中 p 的次数是 n / p + n / p^2 + n / p^3 + ...
3. 用高精度乘法将所有质因子相乘

```cpp
int primes[N], cnt;     // 存储所有质数
int sum[N];     // 存储每个质数的次数
bool st[N];     // 存储每个数是否已被筛掉


void get_primes(int n)      // 线性筛法求素数
{
    for (int i = 2; i <= n; i ++ )
    {
        if (!st[i]) primes[cnt ++ ] = i;
        for (int j = 0; primes[j] <= n / i; j ++ )
        {
            st[primes[j] * i] = true;
            if (i % primes[j] == 0) break;
        }
    }
}


int get(int n, int p)       // 求n！中的次数
{
    int res = 0;
    while (n)
    {
        res += n / p;
        n /= p;
    }
    return res;
}


vector<int> mul(vector<int> a, int b)       // 高精度乘低精度模板
{
    vector<int> c;
    int t = 0;
    for (int i = 0; i < a.size(); i ++ )
    {
        t += a[i] * b;
        c.push_back(t % 10);
        t /= 10;
    }

    while (t)
    {
        c.push_back(t % 10);
        t /= 10;
    }

    return c;
}

get_primes(a);  // 预处理范围内的所有质数

for (int i = 0; i < cnt; i ++ )     // 求每个质因数的次数
{
    int p = primes[i];
    sum[i] = get(a, p) - get(b, p) - get(a - b, p);
}

vector<int> res;
res.push_back(1);

for (int i = 0; i < cnt; i ++ )     // 用高精度乘法将所有质因子相乘
    for (int j = 0; j < sum[i]; j ++ )
        res = mul(res, primes[i]);

for (int i = res.size() - 1; i >= 0; i--) printf("%d", res[i]);

puts(""); // 输出高精度数据
```

### [889. 满足条件的01序列](https://www.acwing.com/problem/content/description/891/)

```cpp
// 满足条件的01序列
#include <iostream>
using namespace std;
// 给定 n个 0和 n个 1 满足任意前缀序列中 0的个数都不少于 1的个数
// 例如 111 000 1.000111 2.001101 3.001011 4.010011 5.010101
// 实际上可以通过画表格的方式求解 1代表向上走 0代表向右走 从(0, 0)到(6, 6)共有C12 6种走法
// 其中要满足题意那么所有路线都必须在(1, 0)到(5, 6)这条直线以下
// 也就是所有路线数量 - 经过(1, 0)到(5, 6)这条直线路线数量
// 将第一次经过(1, 0)到(5, 6)这条直线的点以后的路线做这条直线的轴对称 其终点变为(5, 7) 即所有不满足条件的路线都会到(5, 7)
// 那么就将原题转换为求(0, 0)到(5, 7)的路线个数 C12 5 满足题意的点为 C12 6 - C12 5
// 由以上可以推出2n条路线满足题意的路线有C2n n - C2n n-1条化简成C2n n / n + 1 这个数也称为卡特兰数
typedef long long LL;

const int N = 100010, mod = 1e9 + 7;
// 利用快速幂计算模mod后a的逆元(因为mod为质数 可以利用费马小定理计算逆元)
int qmi(int a, int b, int p) {
    int res = 1;
    while (b) {
        if (b & 1) res = ( LL ) res * a % mod;
        a = ( LL ) a * a % mod;
        b >>= 1;
    }
    return res;
}

int main(void) {
    int n;
    cin >> n;

    int a = 2 * n, b = n;
    int x = 1, y = 1;
    // 计算(a - b)! 和 b!
    for (int i = 0; i < b; i++) {
        x = ( LL ) x * (a - i) % mod;
        y = ( LL ) y * (i + 1) % mod;
    }
    // 最终结果为Ca b / n + 1  =>  (a - b)! * b!^-1 * (n + 1)^-1
    int res = ( LL ) x * qmi(y, mod - 2, mod) % mod * qmi(n + 1, mod - 2, mod) % mod;

    cout << res;
}
```

## 容斥原理

### [890. 能被整除的数](https://www.acwing.com/problem/content/892/)

```cpp
/*  容斥原理通过集合的角度来进行表示
    例：其中一个集合S_i可以表示为 1~n 中能被p_i 整除的数的个数，那么这个数量就是 n / p_i(p_i的倍数不超过n就是)
    当两个集合相交是表示可以被这两个集合同时整除，这里设置一个临时变量t表示这些表示集合的质数的乘积，n / t的数量就是可以同时整除这多个质数的数量
    其中1~n能被任意一个质数整除即可它的个数可以表示S_1 U S_2 U …… U S_m 就是这些集合维恩图组成的面积
    即可求解
    时间复杂度为m * 2 ^ m
*/
#include <algorithm>
#include <cstring>
#include <iostream>

using namespace std;
typedef long long LL;

const int N = 20;

int n, m;
int primes[N];

int main() {
    scanf("%d%d", &n, &m);

    for (int i = 0; i < m; i++) scanf("%d", &primes[i]);

    int res = 0;
    for (int i = 1; i < 1 << m; i++) {
        int cnt = 0, t = 1; // 其中cnt表示有多少个集合，t表示可以同时整除这些相乘质数的（质数乘积）

        for (int j = 0; j < m; j++) // 因为只算一次不用进行预处理
        {
            int p = primes[j];

            if (i >> j & 1) {         // p[j]这个集合存在
                if (( LL ) t * p > n) // 这个集合表示的质数乘积超过了n就不可能存在这样的n, 呜呜, 因为p的范围爆了int 所以要用long long 处理这个运算结果
                {
                    t = -1;
                    break;
                }
                cnt++;
                t *= p;
            }
        }

        if (t != -1) // 如果这个集合合法时
        {
            if (cnt & 1)
                res += n / t; // 奇数加
            else
                res -= n / t; // 偶数减
        }
    }

    printf("%d\n", res);
    return 0;
}
```

## 博弈论

### [891. NIM游戏](https://www.acwing.com/problem/content/893/)

