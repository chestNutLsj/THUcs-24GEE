---
publish:
---
## Introduction
本章将介绍介绍动态规划（Dynamic Programming, DP）及其解决的问题、根据其设计的算法及优化。

动态规划是一种通过把原问题分解为相对简单的子问题的方式求解复杂问题的方法。

由于动态规划并不是某种具体的算法，而是一种解决特定问题的方法，因此它会出现在各式各样的数据结构中，与之相关的题目种类也更为繁杂。

在 OI 中，计数等非最优化问题的递推解法也常被不规范地称作 DP，因此本章将它们一并列出。事实上，动态规划与其它类型的递推的确有很多相似之处，学习时可以注意它们之间的异同。

[Dynamic programming - Wikipedia](https://en.wikipedia.org/wiki/Dynamic_programming?useskin=vector)

## Basis

### 引入：数字三角形问题

> [! note] [IOI1994 数字三角形]([https://www.luogu.com.cn/problem/P1216](https://www.luogu.com.cn/problem/P1216))
> 给定一个 r 行的数字三角形（$r \leq 1000$），需要找到一条从最高点到底部任意处结束的路径，使路径经过数字的和最大。每一步可以走到当前点左下方的点或右下方的点。 
>```plain  
>         7   
>       3   8   
>     8   1   0   
>   2   7   4   4   
> 4   5   2   6   5   
>```   
>
>在上面这个例子中，最优路径是 $7 \to 3 \to 8 \to 7 \to 5$。
 
最简单粗暴的思路是尝试所有的路径。因为路径条数是 $O(2^r)$ 级别的，这样的做法无法接受。

注意到这样一个事实，一条最优的路径，它的每一步决策都是最优的。

以例题里提到的最优路径为例，只考虑前四步 $7 \to 3 \to 8 \to 7$，不存在一条从最顶端到 $4$ 行第 $2$ 个数的权值更大的路径。

而对于每一个点，它的下一步决策只有两种：往左下角或者往右下角（如果存在）。因此只需要记录当前点的最大权值，用这个最大权值执行下一步决策，来更新后续点的最大权值。

这样做还有一个好处：我们成功缩小了问题的规模，将一个问题分成了多个规模更小的问题。要想得到从顶端到第 $r$ 行的最优方案，只需要知道从顶端到第 $r-1$ 行的最优方案的信息就可以了。

这时候还存在一个问题：子问题间重叠的部分会有很多，同一个子问题可能会被重复访问多次，效率还是不高。解决这个问题的方法是把每个子问题的解存储下来，通过记忆化的方式限制访问顺序，确保每个子问题只被访问一次。

上面就是动态规划的一些基本思路。下面将会更系统地介绍动态规划的思想。

### 动态规划原理

能用动态规划解决的问题，需要满足三个条件：最优子结构，无后效性和子问题重叠。

#### 最优子结构

具有最优子结构也可能是适合用贪心的方法求解。

注意要确保我们考察了最优解中用到的所有子问题。

1.  证明问题最优解的第一个组成部分是做出一个选择；
2.  对于一个给定问题，在其可能的第一步选择中，假定你已经知道哪种选择才会得到最优解。你现在并不关心这种选择具体是如何得到的，只是假定已经知道了这种选择；
3.  给定可获得的最优解的选择后，确定这次选择会产生哪些子问题，以及如何最好地刻画子问题空间；
4.  证明作为构成原问题最优解的组成部分，每个子问题的解就是它本身的最优解。方法是反证法，考虑加入某个子问题的解不是其自身的最优解，那么就可以从原问题的解中用该子问题的最优解替换掉当前的非最优解，从而得到原问题的一个更优的解，从而与原问题最优解的假设矛盾。

要保持子问题空间尽量简单，只在必要时扩展。

最优子结构的不同体现在两个方面：

1.  原问题的最优解中涉及多少个子问题；
2.  确定最优解使用哪些子问题时，需要考察多少种选择。

子问题图中每个定点对应一个子问题，而需要考察的选择对应关联至子问题顶点的边。

#### 无后效性

已经求解的子问题，不会再受到后续决策的影响。

#### 子问题重叠

如果有大量的重叠子问题，我们可以用空间将这些子问题的解存储下来，避免重复求解相同的子问题，从而提升效率。

#### 基本思路

对于一个能用动态规划解决的问题，一般采用如下思路解决：

1.  将原问题划分为若干 **阶段**，每个阶段对应若干个子问题，提取这些子问题的特征（称之为 **状态**）；
2.  寻找每一个状态的可能 **决策**，或者说是各状态间的相互转移方式（用数学的语言描述就是 **状态转移方程**）。
3.  按顺序求解每一个阶段的问题。

如果用图论的思想理解，我们建立一个 有向无环图 ，每个状态对应图上一个节点，决策对应节点间的连边。这样将问题转变为在 DAG 上寻找最长(短)路径的问题。

### 最长公共子序列问题

>[! note] Longest Common Subsequence
>给定一个长度为 $n$ 的序列 $A$ 和一个 长度为 $m$ 的序列 $B$（$n,m \leq 5000$），求出一个最长的序列，使得该序列既是 $A$ 的子序列，也是 $B$ 的子序列。

一个简要的例子：字符串 `abcde` 与字符串 `acde` 的公共子序列有 `a`、`c`、`d`、`e`、`ac`、`ad`、`ae`、`cd`、`ce`、`de`、`ade`、`ace`、`cde`、`acde`，最长公共子序列的长度是 4。

设 $f(i,j)$ 表示只考虑 $A$ 的前 $i$ 个元素，$B$ 的前 $j$ 个元素时的最长公共子序列的长度，求这时的最长公共子序列的长度就是 **子问题**。$f(i,j)$ 就是我们所说的 **状态**，则 $f(n,m)$ 是最终要达到的状态，即为所求结果。

对于每个 $f(i,j)$，存在三种决策：如果 $A_i=B_j$，则可以将它接到公共子序列的末尾；另外两种决策分别是跳过 $A_i$ 或者 $B_j$。状态转移方程如下：

$$
f(i,j)=\begin{cases}f(i-1,j-1)+1&A_i=B_j\\\max(f(i-1,j),f(i,j-1))&A_i\ne B_j\end{cases}
$$

可参考 [SourceForge 的 LCS 交互网页](http://lcs-demo.sourceforge.net/) 来更好地理解 LCS 的实现过程。

该做法的时间复杂度为 $O(nm)$。

另外，本题存在 $O\left(\dfrac{nm}{w}\right)$ 的算法[^1]。有兴趣的同学可以自行探索。

```cpp
int a[MAXN], b[MAXM], f[MAXN][MAXM];

int dp() {
  for (int i = 1; i <= n; i++)
    for (int j = 1; j <= m; j++)
      if (a[i] == b[j])
        f[i][j] = f[i - 1][j - 1] + 1;
      else
        f[i][j] = std::max(f[i - 1][j], f[i][j - 1]);
  return f[n][m];
}
```

### 最长不下降子序列问题

>[! note]
>给定一个长度为 $n$ 的序列 $A$（$n \leq 5000$），求出一个最长的 $A$ 的子序列，满足该子序列的后一个元素不小于前一个元素。

#### 算法一

设 $f(i)$ 表示以 $A_i$ 为结尾的最长不下降子序列的长度，则所求为 $\max_{1 \leq i \leq n} f(i)$。

计算 $f(i)$ 时，尝试将 $A_i$ 接到其他的最长不下降子序列后面，以更新答案。于是可以写出这样的状态转移方程：$f(i)=\max_{1 \leq j < i, A_j \leq A_i} (f(j)+1)$。

容易发现该算法的时间复杂度为 $O(n^2)$。

```cpp
int a[MAXN], d[MAXN];

int dp() {
  d[1] = 1;
  int ans = 1;
  for (int i = 2; i <= n; i++) {
	d[i] = 1;
	for (int j = 1; j < i; j++)
	  if (a[j] <= a[i]) {
		d[i] = max(d[i], d[j] + 1);
		ans = max(ans, d[i]);
	  }
  }
  return ans;
}
```


```python
a = [0] * MAXN
d = [0] * MAXN
def dp ():
	d[1] = 1
	ans = 1
	for i in range (2, n + 1):
		for j in range (1, i):
			if a[j] <= a[i]:
				d[i] = max (d[i], d[j] + 1)
				ans = max (ans, d[i])
	return ans
```

#### 算法二[^2]

当 $n$ 的范围扩大到 $n \leq 10^5$ 时，第一种做法就不够快了，下面给出了一个 $O(n \log n)$ 的做法。

首先，定义 $a_1 \dots a_n$ 为原始序列，$d$ 为当前的不下降子序列，$len$ 为子序列的长度，那么 $d_{len}$ 就是长度为 $len$ 的不下降子序列末尾元素。

初始化：$d_1=a_1,len=1$。

现在我们已知最长的不下降子序列长度为 1，那么我们让 $i$ 从 2 到 $n$ 循环，依次求出前 $i$ 个元素的最长不下降子序列的长度，循环的时候我们只需要维护好 $d$ 这个数组还有 $len$ 就可以了。**关键在于如何维护。**

考虑进来一个元素 $a_i$：

1.  元素大于等于 $d_{len}$，直接将该元素插入到 $d$ 序列的末尾。
2.  元素小于 $d_{len}$，找到 **第一个** 大于它的元素，用 $a_i$ 替换它。

参考代码如下：

```cpp
for (int i = 0; i < n; ++i) scanf("%d", a + i);
memset(dp, 0x1f, sizeof dp);
mx = dp[0];
for (int i = 0; i < n; ++i) {
  *std::upper_bound(dp, dp + n, a[i]) = a[i];
}
ans = 0;
while (dp[ans] != mx) ++ans;
```

```python
dp = [0x1f1f1f1f] * MAXN
mx = dp[0]
for i in range(0, n):
	bisect.insort_left(dp, a[i], 0, len(dp))
ans = 0
while dp[ans] != mx:
	ans += 1
```

## Memory-based search
记忆化搜索是一种通过记录已经遍历过的状态的信息，从而避免对同一状态重复遍历的搜索实现方式。

因为记忆化搜索确保了每个状态只访问一次，它也是一种常见的动态规划实现方式。

### 引入：采药总价值最大问题
>[! note] [NOIP2005 采药]([https://www.luogu.com.cn/problem/P1048](https://www.luogu.com.cn/problem/P1048))
>山洞里有 $M$ 株不同的草药，采每一株都需要一些时间 $t_i$，每一株也有它自身的价值 $v_i$。给你一段时间 $T$，在这段时间里，你可以采到一些草药。让采到的草药的总价值最大。
>
>$1 \leq T \leq 10^3$ ，$1 \leq t_i,v_i,M \leq 100$

#### 朴素的 DFS 做法

很容易实现这样一个朴素的搜索做法：在搜索时记录下当前准备选第几个物品、剩余的时间是多少、已经获得的价值是多少这三个参数，然后枚举当前物品是否被选，转移到相应的状态。

```cpp
int n, t;
int tcost[103], mget[103];
int ans = 0;

void dfs(int pos, int tleft, int tans) {
  if (tleft < 0) return;
  if (pos == n + 1) {
	ans = max(ans, tans);
	return;
  }
  dfs(pos + 1, tleft, tans);
  dfs(pos + 1, tleft - tcost[pos], tans + mget[pos]);
}

int main() {
  cin >> t >> n;
  for (int i = 1; i <= n; i++) cin >> tcost[i] >> mget[i];
  dfs(1, t, 0);
  cout << ans << endl;
  return 0;
}
```


```python
tcost = [0] * 103
mget = [0] * 103
ans = 0
def dfs(pos, tleft, tans):
	global ans
	if tleft < 0:
		return
	if pos == n + 1:
		ans = max(ans, tans)
		return
	dfs(pos + 1, tleft, tans)
	dfs(pos + 1, tleft - tcost[pos], tans + mget[pos])
t, n = map(lambda x:int(x), input().split())
for i in range(1, n + 1):
	tcost[i], mget[i] = map(lambda x:int(x), input().split())
dfs(1, t, 0)
print(ans)
```

这种做法的时间复杂度是指数级别的，并不能通过本题。

#### 优化

上面的做法为什么效率低下呢？因为同一个状态会被访问多次。

如果我们每查询完一个状态后将该状态的信息存储下来，再次需要访问这个状态就可以直接使用之前计算得到的信息，从而避免重复计算。这充分利用了动态规划中很多问题具有大量重叠子问题的特点，属于用空间换时间的「记忆化」思想。

具体到本题上，我们在朴素的 DFS 的基础上，增加一个数组 `mem` 来记录每个 `dfs(pos,tleft)` 的返回值。刚开始把 `mem` 中每个值都设成 `-1`（代表没求解过）。每次需要访问一个状态时，如果相应状态的值在 `mem` 中为 `-1`，则递归访问该状态。否则我们直接使用 `mem` 中已经存储过的值即可。

通过这样的处理，我们确保了每个状态只会被访问一次，因此该算法的的时间复杂度为 $O(TM)$。

```cpp
int n, t;
int tcost[103], mget[103];
int mem[103][1003];

int dfs(int pos, int tleft) {
  if (mem[pos][tleft] != -1)
	return mem[pos][tleft];  // 已经访问过的状态，直接返回之前记录的值
  if (pos == n + 1) return mem[pos][tleft] = 0;
  int dfs1, dfs2 = -INF;
  dfs1 = dfs(pos + 1, tleft);
  if (tleft >= tcost[pos])
	dfs2 = dfs(pos + 1, tleft - tcost[pos]) + mget[pos];  // 状态转移
  return mem[pos][tleft] = max(dfs1, dfs2);  // 最后将当前状态的值存下来
}

int main() {
  memset(mem, -1, sizeof(mem));
  cin >> t >> n;
  for (int i = 1; i <= n; i++) cin >> tcost[i] >> mget[i];
  cout << dfs(1, t) << endl;
  return 0;
}
```


```python
tcost = [0] * 103
mget = [0] * 103
mem = [[-1 for i in range(1003)] for j in range(103)]
def dfs(pos, tleft):
	if mem[pos][tleft] != -1:
		return mem[pos][tleft]
	if pos == n + 1:
		mem[pos][tleft] = 0
		return mem[pos][tleft]
	dfs1 = dfs2 = -INF
	dfs1 = dfs(pos + 1, tleft)
	if tleft >= tcost[pos]:
		dfs2 = dfs(pos + 1, tleft - tcost[pos]) + mget[pos]
	mem[pos][tleft] = max(dfs1, dfs2)
	return mem[pos][tleft]
t, n = map(lambda x:int(x), input().split())
for i in range(1, n + 1):
	tcost[i], mget[i] = map(lambda x:int(x), input().split())
print(dfs(1, t))
```

### 与递推的联系与区别

在求解动态规划的问题时，记忆化搜索与递推的代码，在形式上是高度类似的。这是由于它们使用了相同的状态表示方式和类似的状态转移。也正因为如此，一般来说两种实现的时间复杂度是一样的。

下面给出的是递推实现的代码（为了方便对比，没有添加滚动数组优化），通过对比可以发现二者在形式上的类似性。

```cpp
const int maxn = 1010;
int n, t, w[105], v[105], f[105][1005];

int main() {
  cin >> n >> t;
  for (int i = 1; i <= n; i++) cin >> w[i] >> v[i];
  for (int i = 1; i <= n; i++)
    for (int j = 0; j <= t; j++) {
      f[i][j] = f[i - 1][j];
      if (j >= w[i])
        f[i][j] = max(f[i][j], f[i - 1][j - w[i]] + v[i]);  // 状态转移方程
    }
  cout << f[n][t];
  return 0;
}
```

在求解动态规划的问题时，记忆化搜索和递推，都确保了同一状态至多只被求解一次。而它们实现这一点的方式则略有不同：递推通过设置明确的访问顺序来避免重复访问，记忆化搜索虽然没有明确规定访问顺序，但通过给已经访问过的状态打标记的方式，也达到了同样的目的。

与递推相比，记忆化搜索因为不用明确规定访问顺序，在实现难度上有时低于递推，且能比较方便地处理边界情况，这是记忆化搜索的一大优势。但与此同时，记忆化搜索难以使用滚动数组等优化，且由于存在递归，运行效率会低于递推。因此应该视题目选择更适合的实现方式。

### 如何写记忆化搜索

#### 方法一

1.  把这道题的 dp 状态和方程写出来
2.  根据它们写出 dfs 函数
3.  添加记忆化数组

举例：

$dp_{i} = \max\{dp_{j}+1\}\quad (1 \leq j < i \land a_{j}<a_{i})$（最长上升子序列）

转为

```cpp
int dfs(int i) {
  if (mem[i] != -1) return mem[i];
  int ret = 1;
  for (int j = 1; j < i; j++)
	if (a[j] < a[i]) ret = max(ret, dfs(j) + 1);
  return mem[i] = ret;
}

int main() {
  memset(mem, -1, sizeof(mem));
  // 读入部分略去
  int ret = 0;
  for (int j = 1; j <= n; j++) {
	ret = max(ret, dfs(j));
  }
  cout << ret << endl;
}
```


```python
def dfs(i):
	if mem[i] != -1:
		return mem[i]
	ret = 1
	for j in range(1, i):
		if a[j] < a[i]:
			ret = max(ret, dfs(j) + 1)
	mem[i] = ret
	return mem[i]
```

### 方法二

1.  写出这道题的暴搜程序（最好是 dfs）
2.  将这个 dfs 改成「无需外部变量」的 dfs
3.  添加记忆化数组

举例：本文中「采药」的例子

## 细分 DP 问题
接下来更加细节的 DP 问题，转至算法一章的 OIWiki 中：[[10-knapsack]]、[[20-interval]]


[^1]: [位运算求最长公共子序列 - -Wallace- - 博客园](https://www.cnblogs.com/-Wallace-/p/bit-lcs.html)
[^2]: [最长不下降子序列nlogn算法详解 - lvmememe - 博客园](https://www.cnblogs.com/itlqs/p/5743114.html)