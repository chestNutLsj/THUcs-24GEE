---
publish: "true"
tags:
  - Algorithm
  - AcWing
date: 2024-02-24
---
## 链表

### 存储

在机试中不要使用动态创建节点再组成链表的方式，在链表较大时甚至直接超时：
```cpp
struct Node{
	int val;
	Node *next;
};

Node* head = new Node(); // 非常耗时
```

通常使用数组模拟链表的方式，这样的优势就是**快**，这种方式也称为静态链表，能够做到指针式的动态链表的所有操作：

![[20-data-struct-linkedtable-storage.png]]

### 单链表

单链表一般是邻接表的形式，用于存储图和树

```cpp
// head存储链表头，e[]存储节点的值，ne[]存储节点的next指针，idx表示当前用到了哪个节点
int head, e[N], ne[N], idx;

// 初始化
void init()
{
    head = -1; 
    idx = 0;   // 数组中所有位置都还未被分配
}

// 在链表头插入一个数a
void insertAsHead(int a)
{
    e[idx] = a, ne[idx] = head, head = idx ++ ;
}

void insertAsSucc(int k, int a){ //将x插入到下标为k的点的后面
	e[idx]=a;
	ne[idx]=ne[k];
	ne[k]=idx;
	idx++;
}

void removeSucc(int k){ //删除k位置后面的一个节点
	ne[k]=ne[ne[k]]; //不要在意浪费，这不是工程
}

// 将头结点删除，需要保证头结点存在
void removeHead()
{
    head = ne[head];
}
```

#### [826. 单链表](https://www.acwing.com/solution/content/16251/)

```cpp
int main() {
    int m;
    cin >> m;
    init();
    char opt;
    int  x1, x2;
    for (int i = 0; i < m; i++) {
        cin >> opt;
        if (opt == 'H') {
            cin >> x1;
            insertAsHead(x1);
        } else if (opt == 'I') {
            cin >> x1 >> x2;
            insertAsSucc(x1 - 1, x2);//同样的，第k个数，和下标不同，所以要减1
        } else {
            cin >> x1;
            if (x1 == 0) {
                removeHead();
            } else {
                removeSucc(x1 - 1); //注意删除第k个输入后面的数，那函数里放的是下标，k要减去1
            }
        }
    }
    for (int i = head; i != -1; i = ne[i]) cout << e[i] << ' ';
    cout << endl;
    return 0;
}
```

### [827. 双链表](https://www.acwing.com/problem/content/829/)

常用于优化特殊问题。

![[20-data-struct-double-linked.png]]

![[20-data-struct-double-linked-2.png]]

![[20-data-struct-double-linked-3.png]]

```cpp
#include <iostream>
#include <string>

using namespace std;

const int N = 100010;
// e[]表示节点的值，l[]表示节点的左指针，r[]表示节点的右指针，idx表示当前用到了哪个节点
int       e[N], l[N], r[N], idx;

void init() {
    r[0] = 1, l[1] = 0;
    idx = 2; //! idx 此时已经用掉两个点了
}

void insertAsRight(int k, int ele) {
    e[idx] = ele;
    l[idx] = k, r[idx] = r[k];
    l[r[k]] = idx, r[k] = idx;
    idx++;
} //! 当然在 k 的左边插入一个数可以直接调用这个函数，
  //! 等价于在 l[k] 的右边插入一个数 insertAsRight(l[k],x)

void removeAt(int k) {
    r[l[k]] = r[k];
    l[r[k]] = l[k];
}

int main() {
    int m;
    cin >> m;
    init();
    string opt;
    int    k, x;
    for (int i = 0; i < m; i++) {
        cin >> opt;
        if (opt == "L") {
            cin >> x;
            insertAsRight(0, x);
            //! 最左边插入就是在指向 0
            //! 的数的左边插入就可以了也就是可以直接在0的右边插入
        } else if (opt == "R") {
            cin >> x;
            insertAsRight(l[1], x);
            //! 0 和 1 只是代表头和尾所以最右边插入只要在指向 1
            //! 的那个点的右边插入就可以了
        } else if (opt == "D") {
            cin >> k;
            removeAt(k + 1); // removeAtk+1）这样头结点和尾结点就永远不会被删掉了
            //元素是从下标为2的位置开始加入，所以你第k个插入的元素，比如是第3个插入的元素，它的下标对应4,即:K+1
        } else if (opt == "IL") {
            cin >> k >> x;
            insertAsRight(l[k + 1], x);
        } else {
            cin >> k >> x;
            insertAsRight(k + 1, x);
        }
    }
    for (int i = r[0]; i != 1; i = r[i]) cout << e[i] << ' ';

    return 0;
}
```

## 栈

### [828. 模拟栈](https://www.acwing.com/problem/content/submission/830/)

同样使用数组模拟栈，并且栈的操作非常简单，使用栈顶指针完全能够实现：

```cpp
const int N = 100010;

int stack[N], sp = 0;

void push(int x) { stack[++sp] = x; }

void pop() { sp--; }

int top() { return stack[sp]; }

bool isEmpty() {
    if (sp > 0)
        return false;
    else
        return true;
}
```

#### [3302. 中缀表达式求值](https://www.acwing.com/problem/content/3305/)

```cpp
#include <iostream>
#include <stack>
#include <string>
#include <unordered_map>
using namespace std;

stack<int>  num;
stack<char> op;

// 优先级表
unordered_map<char, int> h {{'+', 1}, {'-', 1}, {'*', 2}, {'/', 2}};

void eval() // 求值
{
    int a = num.top(); // 第二个操作数
    num.pop();

    int b = num.top(); // 第一个操作数
    num.pop();

    char p = op.top(); // 运算符
    op.pop();

    int r = 0; // 结果

    // 计算结果
    if (p == '+') r = b + a;
    if (p == '-') r = b - a;
    if (p == '*') r = b * a;
    if (p == '/') r = b / a;

    num.push(r); // 结果入栈
}

int main() {
    string s; // 读入表达式
    cin >> s;

    for (int i = 0; i < s.size(); i++) {
        if (isdigit(s[i])) // 数字入栈
        {
            int x = 0, j = i; // 计算数字
            while (j < s.size() && isdigit(s[j])) {
                x = x * 10 + s[j] - '0';
                j++;
            }
            num.push(x); // 数字入栈
            i = j - 1;
        }
        // 左括号无优先级，直接入栈
        else if (s[i] == '(') // 左括号入栈
        {
            op.push(s[i]);
        }
        // 括号特殊，遇到左括号直接入栈，遇到右括号计算括号里面的
        else if (s[i] == ')') // 右括号
        {
            while (op.top() != '(') // 一直计算到左括号
                eval();
            op.pop(); // 左括号出栈
        } else {
            while (op.size() && h[op.top()] >= h[s[i]]) // 待入栈运算符优先级低，则先计算
                eval();
            op.push(s[i]); // 操作符入栈
        }
    }
    while (op.size()) eval();  // 剩余的进行计算
    cout << num.top() << endl; // 输出结果
    return 0;
}
```

### [830. 单调栈](https://www.acwing.com/problem/content/832/)

```cpp
// 常见模型：找出每个数左边离它最近的比它大/小的数
int tt = 0;
for (int i = 1; i <= n; i ++ )
{
    while (tt && check(stk[tt], i)) tt -- ;
    stk[ ++ tt] = i;
}
```

```cpp
#include <iostream>
using namespace std;
const int N = 100010;
int       stk[N], tt;

int main() {
    int n;
    cin >> n;
    while (n--) {
        int x;
        scanf("%d", &x);
        while (tt && stk[tt] >= x) tt--; // 如果栈顶元素大于当前待入栈元素，则出栈
        if (!tt)
            printf("-1 "); // 如果栈空，则没有比该元素小的值。
        else
            printf("%d ", stk[tt]); // 栈顶元素就是左侧第一个比它小的元素。
        stk[++tt] = x;
    }
    return 0;
}
```

```cpp
// STL implementation
/* int main() {
    int n;
    cin >> n;
    stack<int> stk; // 使用STL中的stack

    while (n--) {
        int x;
        cin >> x;                                         // 使用cin来代替scanf
        while (!stk.empty() && stk.top() >= x) stk.pop(); // 如果栈顶元素大于当前待入栈元素，则出栈
        if (stk.empty())
            cout << "-1 "; // 如果栈空，则没有比该元素小的值。
        else
            cout << stk.top() << " "; // 栈顶元素就是左侧第一个比它小的元素。
        stk.push(x);
    }
    return 0;
} */

int main() {
    int n;
    cin >> n;
    vector<int> stk; // 使用vector来模拟栈

    while (n--) {
        int x;
        cin >> x; // 直接使用cin读取输入，代替scanf
        while (!stk.empty() && stk.back() >= x)
            stk.pop_back(); // 如果栈顶元素大于当前待入栈元素，则出栈
        if (stk.empty())
            cout << "-1 "; // 如果栈空，则没有比该元素小的值。
        else
            cout << stk.back() << " "; // 栈顶元素就是左侧第一个比它小的元素。
        stk.push_back(x);              // 入栈操作
    }
    return 0;
}
```

## 队列

### [829. 模拟队列](https://www.acwing.com/problem/content/831/)

```cpp
/*****************单向队列******************/
const int N = 100010;
int       queue[N], head = 0, tail = -1;

void enqueue(int x) { queue[++tail] = x; }

void dequeue() { head++; }

int query() { return queue[head]; }

bool isEmpty() {
    if (head <= tail)
        return false;
    else
        return true;
}

/*****************循环单向队列******************/
// hh 表示队头，tt表示队尾的后一个位置
int q[N], hh = 0, tt = 0;

// 向队尾插入一个数
q[tt ++ ] = x;
if (tt == N) tt = 0;

// 从队头弹出一个数
hh ++ ;
if (hh == N) hh = 0;

// 队头的值
q[hh];

// 判断队列是否为空，如果hh != tt，则表示不为空
if (hh != tt)
{

}
```

### 单调队列

```cpp
//常见模型：找出滑动窗口中的最大值/最小值
int hh = 0, tt = -1;
for (int i = 0; i < n; i ++ )
{
    while (hh <= tt && check_out(q[hh])) hh ++ ;  // 判断队头是否滑出窗口
    while (hh <= tt && check(q[tt], i)) tt -- ;
    q[ ++ tt] = i;
}
```

数组实现的线性表，比 STL 的优势就是速度快、占用小，但是编译器开 O2 优化的话，STL 的速度也相差无几。

#### [154. 滑动窗口](https://www.acwing.com/problem/content/156/)

![[20-data-struct-slide-win.png]]

**解题思路**（以最大值为例）：

由于我们需要求出的是滑动窗口的最大值。
- 如果当前的滑动窗口中有两个下标 `i` 和 j ，其中 `i` 在 `j` 的左侧（i<j），并且 `i` 对应的元素不大于 `j` 对应的元素（`nums[i]≤nums[j]`），则：
	- 当滑动窗口向右移动时，只要 `i` 还在窗口中，那么 `j` 一定也还在窗口中。这是由于 `i` 在 `j` 的左侧所保证的。
	- 因此，由于 `nums[j]` 的存在，`nums[i]` 一定不会是滑动窗口中的最大值了，我们可以将 `nums[i]` 永久地移除。
- **因此我们可以使用一个队列存储所有还没有被移除的下标。在队列中，这些下标按照从小到大的顺序被存储，并且它们在数组 `nums` 中对应的值是严格单调递减的**。
- 当滑动窗口向右移动时，我们需要把一个新的元素放入队列中。
	- 为了保持队列的性质，我们会不断地将新的元素与队尾的元素相比较，如果新元素大于等于队尾元素，那么队尾的元素就可以被永久地移除，我们将其弹出队列。我们需要不断地进行此项操作，直到队列为空或者新的元素小于队尾的元素。
- 由于队列中下标对应的元素是严格单调递减的，因此此时队首下标对应的元素就是滑动窗口中的最大值。
- 窗口向右移动的时候。因此我们还需要不断从队首弹出元素保证队列中的所有元素都是窗口中的，因此当队头元素在窗口的左边的时候，弹出队头。

```cpp
#include <algorithm>
#include <cstring>
#include <deque>
#include <iostream>
using namespace std;

const int N = 1000010;
int       a[N];

int main() {
    int n, k;
    cin >> n >> k;
    for (int i = 1; i <= n; i++) cin >> a[i]; // 读入数据
    deque<int> q;
    for (int i = 1; i <= n; i++) {
        while (q.size() && q.back() > a[i]) // 新进入窗口的值小于队尾元素，则队尾出队列
            q.pop_back();
        q.push_back(a[i]);                       // 将新进入的元素入队
        if (i - k >= 1 && q.front() == a[i - k]) // 若队头是否滑出了窗口，队头出队
            q.pop_front();
        if (i >= k) // 当窗口形成，输出队头对应的值
            cout << q.front() << " ";
    }
    q.clear();
    cout << endl;

    // 最大值亦然
    for (int i = 1; i <= n; i++) {
        while (q.size() && q.back() < a[i]) q.pop_back();
        q.push_back(a[i]);
        if (i - k >= 1 && a[i - k] == q.front()) q.pop_front();
        if (i >= k) cout << q.front() << " ";
    }
}
```

## 字符串模式匹配

### [831. KMP](https://www.acwing.com/problem/content/833/)

```cpp
// s[]是长文本，p[]是模式串，n是s的长度，m是p的长度
// 求模式串的Next数组：
for (int i = 2, j = 0; i <= m; i ++ )
{
    while (j && p[i] != p[j + 1]) j = ne[j];
    if (p[i] == p[j + 1]) j ++ ;
    ne[i] = j;
}

// 匹配
for (int i = 1, j = 0; i <= n; i ++ )
{
    while (j && s[i] != p[j + 1]) j = ne[j];
    if (s[i] == p[j + 1]) j ++ ;
    if (j == m)
    {
        j = ne[j];
        // 匹配成功后的逻辑
    }
}
```

```cpp
#include <iostream>

using namespace std;

const int N = 1000010;
char      p[N], s[N]; // 用 p 来匹配 s
// “next” 数组，若第 i 位存储值为 k
// 说明 p[0...i] 内最长相等前后缀的前缀的最后一位下标为 k
// 即 p[0...k] == p[i-k...i]
int ne[N];
int n, m; // n 是模板串长度 m 是模式串长度

int main() {
    cin >> n >> p >> m >> s;

    // p[0...0] 的区间内一定没有相等前后缀
    ne[0] = -1;

    // 构造模板串的 next 数组
    for (int i = 1, j = -1; i < n; i++) {
        while (j != -1 && p[i] != p[j + 1]) {
            // 若前后缀匹配不成功
            // 反复令 j 回退，直至到 -1 或是 s[i] == s[j + 1]
            j = ne[j];
        }
        if (p[i] == p[j + 1]) {
            j++; // 匹配成功时，最长相等前后缀变长，最长相等前后缀前缀的最后一位变大
        }
        ne[i] = j; // 令 ne[i] = j，以方便计算 next[i + 1]
    }

    // kmp start !
    for (int i = 0, j = -1; i < m; i++) {
        while (j != -1 && s[i] != p[j + 1]) { j = ne[j]; }
        if (s[i] == p[j + 1]) {
            j++; // 匹配成功时，模板串指向下一位
        }
        if (j == n - 1) // 模板串匹配完成，第一个匹配字符下标为 0，故到 n - 1
        {
            // 匹配成功时，文本串结束位置减去模式串长度即为起始位置
            cout << i - j << ' ';

            // 模板串在模式串中出现的位置可能是重叠的
            // 需要让 j 回退到一定位置，再让 i 加 1 继续进行比较
            // 回退到 ne[j] 可以保证 j 最大，即已经成功匹配的部分最长
            j = ne[j];
        }
    }

    return 0;
}
```

```cpp
int match ( char* P, char* T ) {  //KMP算法
	int* next = buildNext ( P ); //构造next表
	int n = ( int ) strlen ( T ), i = 0; //文本串指针
	int m = ( int ) strlen ( P ), j = 0; //模式串指针
	while ( j < m  && i < n ) { //自左向右逐个比对字符  
	    if ( 0 > j || T[i] == P[j] ) //若匹配，或P已移出最左侧（两个判断的次序不可交换）
	        { i ++;  j ++; } //则转到下一字符
	    else //否则
	        j = next[j]; //模式串右移（注意：文本串不用回退）

	delete [] next; //释放next表
	return i - j;
	}
}

int* buildNext ( char* P ) { //构造模式串P的next表
	size_t m = strlen ( P ), j = 0; //“主”串指针
	int* next = new int[m]; int t = next[0] = -1; //next表，首项必为-1
	while ( j < m - 1 )
	    if ( 0 > t || P[t] == P[j] ) { //匹配
	        ++t; ++j; next[j] = t; //则递增赋值：此处可改进...
	    } else //否则
	        t = next[t]; //继续尝试下一值得尝试的位置
	return next;
}

// 返回数组
int *buildNext(char *P) {
    size_t m = strlen(P), j = 0;
    int   *N = new int[m];
    int    t = N[0] = -1;
    while (j < m - 1) {
        if (0 > t || P[j] == P[t]) {
            j++;
            t++;
            N[j] = (P[j] != P[t] ? t : N[t]);
        } else {
            t = N[t];
        }
    }
    return N;
}

vector<int> match(char *P, char *S) {
    int        *next = buildNext(P);
    int         n = strlen(S), i = 0;
    int         m = strlen(P), j = 0;
    vector<int> positions;

    while (i < n) {
        if (j < 0 || S[i] == P[j]) {
            i++;
            j++;
        } else {
            j = next[j];
        }
        if (j == m) {
            positions.push_back(i - j); // 存储找到匹配的起始位置
            j = next[j-1];                // 继续查找下一个可能的匹配
        }
    }
    delete[] next; // 修正：释放next表应在函数末尾
    return positions;
}
```

## Trie 树

高效地存储和查找字符串集合的数据结构：
- 存储：从 root 开始，逐个存储字符串，每层存放一个字符，在字符串结束时做结束标记；
- 查找：从 root 开始，从上到下检验节点是否与给定字符串的对应字符匹配；

![[20-data-struct-trie.png]]

```cpp
int son[N][26], cnt[N], idx;
// 0号点既是根节点，又是空节点
// son[][]存储树中每个节点的子节点
// cnt[]存储以每个节点结尾的单词数量

// 插入一个字符串
void insert(char *str)
{
    int p = 0;
    for (int i = 0; str[i]; i ++ )
    {
        int u = str[i] - 'a';
        if (!son[p][u]) son[p][u] = ++ idx;
        p = son[p][u];
    }
    cnt[p] ++ ;
}

// 查询字符串出现的次数
int query(char *str)
{
    int p = 0;
    for (int i = 0; str[i]; i ++ )
    {
        int u = str[i] - 'a';
        if (!son[p][u]) return 0;
        p = son[p][u];
    }
    return cnt[p];
}
```

### [最大异或对](https://www.acwing.com/problem/content/145/)

```cpp
#include <algorithm>
#include <iostream>

using namespace std;
typedef long long LL;

const int N = 100010, M = 32 * N;
int       son[M][2], idx;
LL        input[N];

void insert(int x) {
    int p = 0; // Trie root
    for (int i = 30; i >= 0; i--) {
        int u = x >> i & 1;                // 取x的第i位的二进制数
        if (!son[p][u]) son[p][u] = ++idx; // 插入过程中没有发现孩子节点，则设之
        p = son[p][u];                     // 指针指向下一层
    }
}

int search(int x) {
    int p = 0, res = 0;
    for (int i = 30; i >= 0; i--) {
        int u = x >> i & 1; // 从最高位开始查找
        if (son[p][!u]) {
            // 如果当前层有对应的不相同的数，p指针就指向不同数的地址；
            p   = son[p][!u];
            res = res * 2 + 1;
        } else {
            p   = son[p][u];
            res = res * 2 + 0;
        }
    }
    return res;
}

int main() {
    cin.tie(0);
    int n;
    cin >> n;
    idx = 0;
    for (int i = 0; i < n; i++) {
        cin >> input[i];
        insert(input[i]);
    }
    int res = 0;
    for (int i = 0; i < n; i++) {
        res = max(res, search(input[i])); /// search(a[i])查找的是a[i]值的最大与或值
    }
    cout << res << endl;

    return 0;
}
```

## 并查集

优势：
1. 将两个集合合并
2. 询问两个元素是否在一个集合当中

```cpp
/*************Init****************/
int fa[MAXN]; // 存每个节点的父亲
inline void init(int n)
{ // 初始化，每个节点初始时的父亲都是自身（自环）
    for (int i = 1; i <= n; ++i)
        fa[i] = i;
}

/*************Find****************/
int find(int x) //递归地查找最顶层的父亲
{
    if(fa[x] == x)
        return x;
    else
        return find(fa[x]);
}

/*************Merge****************/
inline void merge(int i, int j)
{
    fa[find(i)] = find(j); // 将i与j合并到同一个集合
}

/*************Erase****************/
struct dsu {
  vector<size_t> pa, size;

  explicit dsu(size_t size_) : pa(size_ * 2), size(size_ * 2, 1) {
    iota(pa.begin(), pa.begin() + size_, size_);
    iota(pa.begin() + size_, pa.end(), size_);
  }

  void erase(size_t x) {
    --size[find(x)];
    pa[x] = x;
  }
};


/*************Move****************/
void dsu::move(size_t x, size_t y) {
  auto fx = find(x), fy = find(y);
  if (fx == fy) return;
  pa[x] = fy;
  --size[fx], ++size[fy];
}

/*************Compress****************/
int find(int x) // 路径压缩查找
{
    if(x == fa[x])
        return x;
    else{
        fa[x] = find(fa[x]); //父节点设为根节点
        return fa[x];        //返回父节点
    }
}

/************Rank-based Merge************/
inline void init(int n) // 按秩合并
{
    for (int i = 1; i <= n; ++i)
    {
        fa[i] = i;
        rank[i] = 1;
    }
}
inline void merge(int i, int j)
{
    int x = find(i), y = find(j);    //先找到两个根节点
    if (rank[x] <= rank[y])
        fa[x] = y;
    else
        fa[y] = x;
    if (rank[x] == rank[y] && x != y)
        rank[y]++;                   //如果深度相同且根节点不同，则新的根节点的深度+1
}
```

### [837. 连通块中点的数量](https://www.acwing.com/problem/content/839/)

```cpp
// 连通块中点的数量
#include <iostream>

using namespace std;

const int N = 100010;
int       fa[N];
int       Size[N];

inline void init(int n) {
    for (int i = 1; i <= n; i++) {
        fa[i]   = i;
        Size[i] = 1;
    }
}

int find(int x) {
    if (fa[x] == x)
        return x;
    else {
        fa[x] = find(fa[x]);
        return fa[x];
    }
}

inline void merge(int a, int b) {
    // fa[find(a)] = find(b);
    // Size[find(b)] += Size[find(a)];
    /* 上面的两行代码是有错误的：
    1. 父节点错误更新：当你调用fa[find(a)] =find(b);之后，find(a)已经不再是a的直接父节点了，
    因为它已经被更新为b的父节点。因此，后续的Size[find(a)]+= Size[find(b)];实际上是错误的，
    因为此时find(a)可能不再代表原来的a所在集合的根节点。

    2. 集合大小错误累加：正确的做法应该是在确定了两个集合的根节点之后，
    将被合并的集合大小加到目标集合上，并且只在两个集合的根节点不同的情况下执行合并操作。
    */
    int rootA = find(a);
    int rootB = find(b);

    if (rootA != rootB) {
        fa[rootA] = rootB;          // 将rootA的父节点设置为rootB
        Size[rootB] += Size[rootA]; // 更新rootB的连通块大小
    }
}

int main() {
    int n, m;
    cin >> n >> m;
    init(n);
    string opt;
    while (m--) {
        cin >> opt;
        if (opt == "C") {
            int a, b;
            cin >> a >> b;
            if (a != b) { merge(a, b); }

        } else if (opt == "Q1") {
            int a, b;
            cin >> a >> b;
            if (find(a) == find(b) || a == b)
                cout << "Yes" << endl;
            else
                cout << "No" << endl;
        } else {
            int a;
            cin >> a;
            cout << Size[find(a)] << endl;
        }
    }
    return 0;
}
```

### [240. 食物链](https://www.acwing.com/problem/content/242/)

```cpp
// 食物链
#include <iostream>

using namespace std;

const int N = 50010;
int       fa[N];   // 只要命题里给出了动物之间的关系，就放到一个集合里
int       dist[N]; // 通过与根的距离来确定食物链，
                   // 距离1表示可以吃，距离2表示被吃，距离3表示同类
                   // 所以取模操作非常重要，mod 3=0就是同类，其余类推
                   // 节点find后路径压缩，并维护总距离
int lies = 0;      // 记录假话的数量

inline void init(int n) {
    for (int i = 1; i <= n; i++) {
        fa[i] = i; // dist[]就不必初始化了，因为设置时就是全0
    }
}

int find(int x) {
    if (fa[x] != x) {           // 只有祖先不是自己时（代表着已经加入了集合），才需要下述操作
        int tmp = find(fa[x]);  // 先暂存最早祖先
        dist[x] += dist[fa[x]]; // 更新当前节点到最早祖先的距离
        fa[x] = tmp;            // 路径压缩
    }
    return fa[x];
}

int main() {
    int n, k;
    cin >> n >> k;
    init(n); // 对n个动物的关系进行初始化
    int opt, x, y;
    while (k--) {
        cin >> opt >> x >> y;
        if (x > n || y > n) { // 假话情况2
            lies++;
        } else if (opt == 2 && x == y) { // 假话情况3
            lies++;
        } else {
            int px = find(x), py = find(y);                // 分别找到x、y的最早祖先，之后就可以判断命题是否已经出现过（出现过才会有逻辑悖论）
            if (opt == 1) {                                // 判断x和y是同类的命题是否正确
                if (px == py && (dist[x] - dist[y]) % 3) { // 如果祖先相同，说明已在同一集合
                    lies++;                                // 同时到祖先的距离都能模3，说明是同类，就不会导致lies增加，
                                                           // 否则就会存在吃与被吃的关系，lies增加
                } else if (px != py) {                     // 祖先不同，说明还没在同一集合（即这个二者之间的关系还未知，自然不会出错）
                    fa[px]   = py;                         // 先合并，将py作为合并后集合的根
                    dist[px] = dist[y] - dist[x];          // 再维护距离，示意图如下
                    /* px -------------------- py
                       |        dist[px]=?     |
                       |                       |
                       |                       |
                       |dist[x]                |dist[y]
                       |                       |
                       |                       |
                       x                       y

                       x,y,px,py都是同类，说明(dist[x]+?-dist[y])%3 = 0 ，因此?=dist[px]=dist[y]-dist[x]
                     */
                }
            } else {                                         // 判断x吃y的命题是否符合逻辑
                if (px == py && (dist[x] - dist[y] - 1) % 3) // 如果命题已经出现过
                    lies++;                                  // 但是距离之差再减1取模后仍为1，说明x被y吃，因此出现悖论，lies++
                else if (px != py) {
                    fa[px]   = py;
                    dist[px] = dist[y] + 1 - dist[x];
                }
            }
        }
    }
    cout << lies;
    return 0;
}
```

## 二叉堆

### 手动建堆

```cpp
// heap sort
#include <iostream>

using namespace std;

const int N = 100010;
int       h[N];  // h[N]存储堆中的值, h[1]是堆顶，x的左儿子是2x, 右儿子是2x + 1
int       ph[N]; // ph[k]存储第k个插入的点在堆中的位置
int       hp[N]; // hp[k]存储堆中下标是k的点是第几个插入的
int       hsize; // 堆的规模

void heap_swap(int a, int b) { // 交换两个点，及其映射关系
    swap(ph[hp[a]], ph[hp[b]]);
    swap(hp[a], hp[b]);
    swap(h[a], h[b]);
}

void down(int u) { // 下滤
    int t = u;
    if (u * 2 <= hsize && h[u * 2] < h[t]) t = u * 2; // 从指定节点开始，比较节点与其子节点的值，
    if (u * 2 + 1 <= hsize && h[u * 2 + 1] < h[t]) t = u * 2 + 1;
    // if (u * 2 <= hsize && h[u * 2] > h[t]) t = u * 2; 
    //if (u * 2 + 1 <= hsize && h[u * 2 + 1] > h[t]) t = u * 2 + 1;
    //只需要修改<为>，就可以得到大顶堆
    if (u != t) { // 将其与两个子节点中较小的一个交换，直到节点值小于其子节点或已经是叶子节点
        heap_swap(u, t);
        down(t);
    }
}

void up(int u) {                       // 上滤
    while (u / 2 && h[u] < h[u / 2]) { // 从指定节点开始，如果节点值小于父节点的值，则与父节点交换
    //只要把判断条件的<改为>，就可以得到大顶堆
        heap_swap(u, u / 2);           // 直到到达根节点或父节点的值小于当前节点
        u >>= 1;
    }
}

int main() {
    int n, m;
    cin >> n >> m;
    hsize = n;
    for (int i = 1; i <= n; i++) {
        cin >> h[i];
    }
    for (int i = n / 2; i; i--) { // 自底向上建堆：从最后一个非叶子节点开始（即n/2），向上至根节点，依次执行down操作
        down(i);                  // 这个过程确保每个子堆都满足小顶堆的性质。
    }
    while (m--) {
        cout << h[1] << " ";
        h[1] = h[hsize--]; // 用最后一个元素替换根节点：将堆的最后一个元素移动到根节点，同时堆大小减一。
        down(1);
    }
    return 0;
}
```

#### [839. 模拟堆](https://www.acwing.com/problem/content/841/)

```cpp
// 模拟堆
#include <iostream>

using namespace std;

const int N = 100010;
int       h[N];      // h[N]存储堆中的值, h[1]是堆顶，x的左儿子是2x, 右儿子是2x + 1
int       ph[N];     // ph[k]存储第k个插入的点在堆中的位置
int       hp[N];     // hp[k]存储堆中下标是k的点是第几个插入的
int       hsize = 0; // 堆的规模

void heap_swap(int a, int b) { // 交换两个点，及其映射关系
    swap(ph[hp[a]], ph[hp[b]]);
    swap(hp[a], hp[b]);
    swap(h[a], h[b]);
}

void down(int u) { // 下滤
    int t = u;
    if (u * 2 <= hsize && h[u * 2] < h[t]) t = u * 2; // 从指定节点开始，比较节点与其子节点的值，
    if (u * 2 + 1 <= hsize && h[u * 2 + 1] < h[t]) t = u * 2 + 1;
    if (u != t) { // 将其与两个子节点中较小的一个交换，直到节点值小于其子节点或已经是叶子节点
        heap_swap(u, t);
        down(t);
    }
}

void up(int u) { // 上滤
    while (u / 2 && h[u] < h[u / 2]) {
        heap_swap(u, u / 2);
        u >>= 1;
    }
}

int main() {
    int n, m = 0;
    cin >> n;
    while (n--) {
        string opt;
        int    k, x;
        cin >> opt;
        if (opt == "I") { // 插入x
            cin >> x;
            h[++hsize] = x;
            m++;
            ph[m]     = hsize; // 更新ph数组
            hp[hsize] = m;     // 更新hp数组
            up(hsize);
        } else if (opt == "PM") {
            cout << h[1] << endl;
        } else if (opt == "DM") {
            heap_swap(1, hsize);
            hsize--;
            down(1);
        } else if (opt == "D") {
            cin >> k;
            int tmp = ph[k];
            heap_swap(tmp, hsize);
            hsize--;
            down(tmp);
            up(tmp);

        } else {
            cin >> k >> x;
            int tmp = ph[k];
            h[tmp]  = x;
            down(tmp);
            up(tmp);
        }
    }
    return 0;
}
```

### Priority_Queue

```cpp
#include <iostream>
#include <queue>
#include <vector>

using namespace std;

// 定义一个小顶堆（改成less就是大顶堆）
priority_queue<int, vector<int>, greater<int>> minHeap;

int main() {
    int n, m;
    cin >> n >> m;

    // 读取所有元素并添加到小顶堆中
    for (int i = 0; i < n; i++) {
        int x;
        cin >> x;
        minHeap.push(x);
    }

    // 输出前m个最小的元素
    for (int i = 0; i < m; i++) {
        // 由于是小顶堆，所以堆顶是最小的元素
        cout << minHeap.top() << " ";
        minHeap.pop(); // 移除堆顶元素，下一个最小的元素将移动到堆顶
    }

    return 0;
}

```

## 哈希表

### 哈希操作

```cpp
// 拉链法
int h[N], e[N], ne[N], idx;

// 向哈希表中插入一个数
void insert(int x)
{
	int k = (x % N + N) % N; // 让余数变为正数，k就是哈希值，C++中对负数取模比较诡异
	e[idx] = x; // 链表的插入操作
	ne[idx] = h[k];
	h[k] = idx ++ ;
}

// 在哈希表中查询某个数是否存在
bool find(int x)
{
	int k = (x % N + N) % N;
	for (int i = h[k]; i != -1; i = ne[i])
		if (e[i] == x)
			return true;

	return false;
}

//开放寻址法
//经验上说，这里数组长度要开到题目的数据范围的2~3倍
int h[N];

// 如果x在哈希表中，返回x的下标；如果x不在哈希表中，返回x应该插入的位置
int find(int x)
{
	int t = (x % N + N) % N;
	while (h[t] != null && h[t] != x) //这里null设置一个不在数据范围内的任意数即可，作为位置为空的标记
	{
		t ++ ;
		if (t == N) t = 0;
	}
	return t;
}
```

>[!warning] 注意初始化 `h[N]` 数组
>在 main 函数开始处，应当首先对 `h[N]` 数组进行初始化，否则会导致 `insert` 和 `find` 函数行为异常：在独立链表法实现的哈希表中，`h[]`数组用于存储各个链表的头节点索引，因此在使用之前需要将其初始化为某个不可能的索引值，通常是`-1`，表示空链表。如果不进行这一步，`h[]`数组的初始值将是未定义的，这可能导致你的`find`函数遍历未初始化的链表，从而极大地增加计算时间，特别是当`h[k]`偶然被初始化为一个非常大的值时。
>
>**如何进行初始化**？
>在 C++中建议使用 `fill(h, h + N, -1);` 语句进行初始化。在C++中，`fill`函数是一个标准库函数，用于给容器或数组中的所有元素赋相同的值。其原型定义在头文件`<algorithm>`中，形式如下：
> ```cpp
> void fill (ForwardIterator first, ForwardIterator last, const T& value);
> ```
>- `first` 和 `last` 是迭代器，分别指向要填充范围的起始位置和终点位置（不包括 `last`）。
>- `value` 是要赋给指定范围内所有元素的值。
>
>**`fill`函数与`memset`的优势**
>C 中有一个臭名昭著的初始化函数 `memset` ，为什么不用它？主要考虑以下几点：
>- **类型安全**：`fill`是类型安全的，可以用于任何数据类型的容器或数组，而`memset`仅适用于字符数组或需要按字节操作的场景。`memset`在处理非POD（Plain Old Data）类型时可能会导致未定义行为。
>- **通用性**：`fill`可以用于非连续存储的容器（如`std::list`），而`memset`只能用于连续存储的内存块（如C风格数组和`std::vector`的底层存储）。
>- **易用性**：`fill`直接作用于容器和数组，不需要考虑数据的字节大小，而`memset`需要指定要填充的字节数，使用不当容易出错。
>
>**C++11以后更推荐使用`fill`**
>自C++11以来，标准库更推荐使用`std::fill`和其他标准算法，原因如下：
>- **现代C++的编程范式**：C++11及后续标准鼓励使用标准库提供的高级抽象，以编写更安全、更简洁和更易于理解的代码。
>- **自动类型推导和范围循环**：C++11引入的自动类型推导（如`auto`关键字）和范围`for`循环与标准库算法（如`std::fill`）搭配使用，可以简化代码和提高可读性。
>- **泛型编程**：`std::fill`等算法支持泛型编程，使代码更加灵活和通用。

```cpp
// 840. 模拟散列表
// 独立链法
int main() {
    fill(h, h + N, -1);
    int  n, k;
    char opt;
    cin >> n;

    while (n--) {
        cin >> opt >> k;
        if (opt == 'I') {
            insert(k);
        } else {
            if (find(k)) {
                cout << "Yes" << endl;
            } else {
                cout << "No" << endl;
            }
        }
    }
    return 0;
}

// 开放地址法
const int N = 300007; // N用find_prime得到
int       h[N];
int       null = 400007;

int find_prime(int x) { // 找到大于x的第一个质数
    int i = x;
    for (;; i++) {
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
    return i;
}

int main() {
    fill(h, h + N, null);
    int n, x;
    cin >> n;
    string opt;
    while (n--) {
        cin >> opt >> x;
        int pos = find(x);
        if (opt == "I") {
            h[pos] = x;
        } else {
            if (h[pos] == x) {
                cout << "Yes" << endl;
            } else
                cout << "No" << endl;
        }
    }

    return 0;
}
```

### [841. 字符串哈希](https://www.acwing.com/problem/content/843/)

核心思想：将字符串看成 P 进制数，P 的经验值是 131 或 13331，取这两个值的冲突概率低
小技巧：取模的数用 2^64，这样直接用 unsigned long long 存储，溢出的结果就是取模的结果

```cpp
// 字符串哈希
#include <iostream>

using namespace std;
typedef unsigned long long ULL;

const int N = 100003, P = 131; // P是素数，常取131、13331
ULL       h[N], p[N];          // h[k]存储字符串前k个字母的哈希值, p[k]存储 P^k mod 2^64
int       n, m;                // n是字符串长度，m是查询次数
string    str;

void init() {
    p[0] = 1;
    for (int i = 1; i <= n; i++) {
        h[i] = h[i - 1] * P + str[i - 1]; // 在C++中，std::string的索引是从0开始的，但是在init函数中，h和p数组的计算是从1开始的，而且str[i]直接使用i作为索引，这会导致索引偏移错误。应该将str[i]改为str[i - 1]以正确匹配字符串中的字符。
        p[i] = p[i - 1] * P;
    }
}

ULL get_interval_hash(int l, int r) {
    return h[r] - h[l - 1] * p[r - l + 1];
}

int main() {
    cin >> n >> m >> str;
    init();
    while (m--) {
        int l1, l2, r1, r2;
        cin >> l1 >> r1 >> l2 >> r2;
        if (get_interval_hash(l1, r1) == get_interval_hash(l2, r2))
            cout << "Yes" << endl;
        else
            cout << "No" << endl;
    }
    return 0;
}
```

注意，不能将字母映射为 0 。

## STL

### vector

```
size ()  返回元素个数
empty ()  返回是否为空
clear ()  清空
front ()/back ()
push_back ()/pop_back ()
begin ()/end ()
[] 支持随机寻址
支持比较运算，按元素的字典序比较
```

### pair<int, int>
```
first, 第一个元素
second, 第二个元素
支持比较运算，以 first 为第一关键字，以 second 为第二关键字（字典序）
make_pair ()
```

### string，字符串

```
size ()/length ()  返回字符串长度
empty ()
clear ()
substr (起始下标，(子串长度))  返回子串
c_str ()  返回字符串所在字符数组的起始地址
```

### queue, 队列

```
size ()
empty ()
push ()  向队尾插入一个元素
front ()  返回队头元素
back ()  返回队尾元素
pop ()  弹出队头元素
```


### priority_queue, 优先队列，默认是大根堆

```
size ()
empty ()
push ()  插入一个元素
top ()  返回堆顶元素
pop ()  弹出堆顶元素
定义成小根堆的方式：priority_queue<int, vector<int>, greater<int>> q;
```

### stack, 栈

```
size ()
empty ()
push ()  向栈顶插入一个元素
top ()  返回栈顶元素
pop ()  弹出栈顶元素
```

### deque, 双端队列

```
size ()
empty ()
clear ()
front ()/back ()
push_back ()/pop_back ()
push_front ()/pop_front ()
begin ()/end ()
[]
```

### set, map, multiset, multimap,

基于平衡二叉树（红黑树），动态维护有序序列

```
size ()
empty ()
clear ()
begin ()/end ()
++, -- 返回前驱和后继，时间复杂度 O (logn)

set/multiset
	insert ()  插入一个数
	find ()  查找一个数
	count ()  返回某一个数的个数
	erase ()
		(1) 输入是一个数 x，删除所有 x   O (k + logn)
		(2) 输入一个迭代器，删除这个迭代器
	lower_bound ()/upper_bound ()
		lower_bound (x)  返回大于等于 x 的最小的数的迭代器
		upper_bound (x)  返回大于 x 的最小的数的迭代器
map/multimap
	insert ()  插入的数是一个 pair
	erase ()  输入的参数是 pair 或者迭代器
	find ()
	[]  注意 multimap 不支持此操作。 时间复杂度是 O (logn)
	lower_bound ()/upper_bound ()
```

### unordered_set, unordered_map, unordered_multiset, unordered_multimap

哈希表

```
和上面类似，增删改查的时间复杂度是 O (1)
不支持 lower_bound ()/upper_bound ()， 迭代器的++，--
```

### bitset, 圧位

```
bitset<10000> s;
~, &, |, ^
>>, <<
==, !=
[]

count ()  返回有多少个 1

any ()  判断是否至少有一个 1
none ()  判断是否全为 0

set ()  把所有位置成 1
set (k, v)  将第 k 位变成 v
reset ()  把所有位变成 0
flip ()  等价于~
flip (k) 把第 k 位取反
```

![[1-bitset#*Init Syntax*]]