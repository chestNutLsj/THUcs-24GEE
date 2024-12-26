## [数数](https://www.luogu.com.cn/problem/U412490)

```cpp
/// 这个是暴力做法，只能混到20分
#include <iostream>
#include <vector>

using namespace std;
using LL = long long;

const LL constraint = 999999999999;

LL nums_specified_length_string(int n) {
    vector<int> nums(n, 1); // 初始化为最小的字符串
    LL          cnt = 0;

    while (true) {
        bool valid = true;
        // 检查是否有连续三个相同字符
        for (int i = 2; i < n; ++i) {
            if (nums[i] == nums[i - 1] && nums[i] == nums[i - 2]) {
                valid = false;
                break;
            }
        }
        if (valid) cnt++; // 符合条件的字符串数量加一

        // 生成下一个四进制数
        int carry = 1;
        for (int i = n - 1; i >= 0; --i) {
            nums[i] += carry;
            if (nums[i] == 4) {
                nums[i] = 1;
                carry   = 1;
            } else {
                carry = 0;
                break;
            }
        }

        if (carry == 1) break; // 已经到达最大四进制数
    }

    return cnt;
}

int main() {
    int t; // 查询次数
    cin >> t;
    while (t--) {
        int n; // 查询的字符串的长度
        cin >> n;
        LL res = nums_specified_length_string(n);
        if (res <= constraint)
            cout << res << endl;
        else {
            cout << "......" << res % 1000000000 << endl;
        }
    }
    return 0;
}

```

```cpp
// Mario佬的答案
#include <iostream>

using namespace std;
using LL = long long;

const int N        = 1000000;
const LL  mod      = 10000000000;
const LL  overflow = 10000000000000000;

LL f[N + 10];

LL add(LL a, LL b) { return (a + b >= mod) ? (a + b - mod) : (a + b); }

LL sub(LL a, LL b) { return (a >= b) ? (a - b) : (a + mod - b); }

LL mul(LL a, LL b) { return a * b % mod; }

int main() {
    f[1] = 3, f[2] = 9, f[3] = 24;
    for (int i = 4; i <= 37; i++)
        f[i] = 3 * f[i - 1] - 2 * f[i - 3];
    f[37] %= mod;
    for (int i = 38; i <= N; i++)
        f[i] = sub(mul(3, f[i - 1]), mul(2, f[i - 3]));

    int T, n;
    cin >> T;
    while (T--) {
        cin >> n;
        if (n <= 36)
            cout << f[n] << endl;
        else
            cout << "......" << f[n] << endl;
    }
}
```

```cpp
//赵佬的答案
#include <algorithm>
#include <cstdio>
using namespace std;

unsigned long long f[1000001][3][2];
void               calc(int n) {
    long long ret = 0;
    for (int j = 0; j < 3; ++j) ret += f[n][j][0] + f[n][j][1];
    if (n >= 37)
        printf("......%010lld\n", ret % 10000000000ll);
    else
        printf("%lld\n", ret);
}
int main() {
    f[1][0][0] = f[1][1][0] = f[1][2][0] = 1;
    for (int i = 2; i <= 1000000; ++i)
        for (int j = 0; j < 3; ++j) {
            for (int k = 0; k < 3; ++k)
                if (j != k) f[i][j][0] = (f[i][j][0] + f[i - 1][k][0] + f[i - 1][k][1]) % 100000000000000000ll;
            f[i][j][1] = f[i - 1][j][0];
        }
    int T, n;
    scanf("%d", &T);
    while (T--) {
        scanf("%d", &n);
        calc(n);
    }
}
```

## [粽子树](https://www.luogu.com.cn/problem/U412495)

```cpp
// 模拟题2 —— 粽子树（离散化方法）
// https://www.luogu.com.cn/problem/U412495
#include <algorithm>
#include <iostream>
#include <vector>

using namespace std;

const int N = 1e6 + 10, M = 2 * N;
int       h[N], e[M], ne[M], idx; // 树是无向稀疏图，用邻接表存之，
// h[N]: 邻接表的头节点数组，存储每个节点的第一条边的索引
// e[N]和ne[N]: 边数组和下一条边的索引数组，用于表示图中的边。e[i]表示边的终点，ne[i]指向下一条边
int         n;         // 节点数
int         kind[N];   // kind[i] 表示编号i的节点上的粽子的种类，不同的值代表不同的种类，注意种类数在int范围，而节点数在10^6范围，这是个经典的稀疏情形，需要hash或离散化
int         ans[N];    // ans[i]表示编号i的节点到根节点1的简单路径上粽子的种类数
int         cnt[N];    // 记录每种粽子在当前路径上的出现次数
bool        st[N];     // 记录dfs中是否已经扫描过
vector<int> kinds_set; // 用于离散化，存储所有不同粽子种类的列表

void init() {
    fill(h, h + N, -1);
    // fill(ans, ans + N, 0);
    // fill(kind, kind + N, 0);
    // fill(cnt, cnt + N, 0);
    fill(st, st + N, false);
    idx = 0;
}

int find(int x) { // 用二分查找，在粽子种类的去重列表中找到匹配者并返回其离散化后的位置
    int l = 0, r = kinds_set.size() - 1;
    while (l < r) {
        int mid = (l + r) >> 1;
        if (kinds_set[mid] >= x)
            r = mid;
        else
            l = mid + 1;
    }
    return l + 1;
}

void addEdge(int a, int b) { // 建立a-->b的边
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u, int fa) { // 从u号点开始dfs，fa避免在递归过程中重新访问父节点，根据树的连通性、kind[i]计算ans[i]
    if (st[u]) return;    // 不过本题是无环图，因此这一步并不必要，不会出现无限递归的情况
    st[u] = true;
    if (cnt[kind[u]])     // 如果当前节点u的粽子种类在此前的路径上出现过
        ans[u] = ans[fa]; // 则当前节点到根的路径上不同粽子种类数量与父亲相同
    else
        ans[u] = ans[fa] + 1; // 否则在父亲的种类数上+1

    cnt[kind[u]]++; // 将当前节点的粽子种类的计数+1，表示这种粽子种类在当前路径上的出现次数增加

    for (int i = h[u]; i != -1; i = ne[i]) { // 递归
        int j = e[i];
        dfs(j, u);
    }
    cnt[kind[u]]--; // 在递归返回之前，将当前节点的粽子种类的计数减少1（cnt[kind[u]]--）
    // 这是因为当离开一个节点时，这个节点的粽子种类不应再影响后续路径的计数。
    // 这样可以确保cnt数组准确地反映了从当前路径上每种粽子的实际出现次数。
}

int main() {
    ios::sync_with_stdio(false);
    cin >> n;
    init();
    for (int i = 0; i < n - 1; i++) {
        int u, v;
        cin >> u >> v;
        addEdge(u, v), addEdge(v, u); // 树是无向图
    }
    for (int i = 1; i <= n; i++) {
        cin >> kind[i];               // 节点编号从1开始
        kinds_set.push_back(kind[i]); // 离散化的准备
    }

    // 离散化
    sort(kinds_set.begin(), kinds_set.end());
    kinds_set.erase(unique(kinds_set.begin(), kinds_set.end()), kinds_set.end());
    for (int i = 1; i <= n; i++) kind[i] = find(kind[i]);

    dfs(n, 0); // 使用n作为根节点开始遍历
    for (int i = 1; i <= n; i++) {
        cout << ans[i] << " ";
    }
    cout << endl;
    return 0;
}
```

```cpp
// mario佬的答案
// unordered_map版本
#include <algorithm>
#include <iostream>
#include <unordered_map>

using namespace std;

const int N = 1000010;

int rd() {
    int  k = 0, f = 1;
    char c = getchar();
    while (c < '0' || c > '9') {
        if (c == '-') f = 0;
        c = getchar();
    }
    while (c >= '0' && c <= '9') {
        k = (k << 1) + (k << 3) + (c ^ 48);
        c = getchar();
    }
    return f ? k : -k;
}

void wr(int x) {
    if (x < 0)
        putchar('-'), x = -x;
    if (x > 9)
        wr(x / 10);
    putchar((x % 10) ^ '0');
}

struct edge {
    int to, nxt;
} e[N << 1];

int                     head[N], ecnt;
unordered_map<int, int> mp;
int                     a[N], cnt[N], ans[N], type_cnt, res;

void add_edge(int u, int v) {
    e[++ecnt].to = v, e[ecnt].nxt = head[u], head[u] = ecnt;
}

void dfs(int u, int fa) {
    if (!cnt[a[u]]) ++res;
    ++cnt[a[u]];
    ans[u] = res;
    for (int i = head[u]; i; i = e[i].nxt) {
        int v = e[i].to;
        if (v ^ fa) dfs(v, u);
    }
    --cnt[a[u]];
    if (!cnt[a[u]]) --res;
}

int main() {
    int n;
    n = rd();
    for (int i = 1; i < n; i++) {
        int u = rd(), v = rd();
        add_edge(u, v), add_edge(v, u);
    }
    for (int i = 1; i <= n; i++) {
        a[i] = rd();
        if (!mp.count(a[i])) mp[a[i]] = ++type_cnt;
        a[i] = mp[a[i]];
    }
    dfs(n, 0);
    for (int i = 1; i <= n; ++i, putchar(i > n ? '\n' : ' ')) wr(ans[i]);
}
```

```cpp
// 模拟题2 —— 粽子树（不用快速读写）
#include <algorithm>
#include <iostream>
#include <unordered_map>

using namespace std;

const int N = 1000010, M = 2 * N;
int       h[N], e[M], ne[M], idx; // 树是稀疏图，用邻接表存之，h[i]: 表示第 i 个节点的第一条边的 idx ，ne[i] : 表示与第 idx 条边同起点的下一条边的 idx，e[i] : 表示第 idx 条边的终点
int       n;                      // 节点数
int       a[N];                   // a[i] 表示编号i的节点上的粽子的种类，不同的值代表不同的种类
int       ans[N];                 // ans[i]表示编号i的节点到根节点1的简单路径上粽子的种类数
bool      st[N];                  // dfs中用作标记，表示该点已经被搜索过

void init() {
    fill(h, h + N, -1);
    fill(ans, ans + N, 0);
    idx = 0;
}

void addEdge(int a, int b) { // 建立a-->b的边
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

void dfs(int u, int fa, unordered_map<int, int> &cnt) { // 从u号点开始dfs，根据树的连通性、a[i]计算ans[i]
    st[u] = true;
    cnt[a[u]]++;
    ans[u] = cnt.size();

    for (int i = h[u]; i != -1; i = ne[i]) {
        int v = e[i];
        if (v == fa) continue;
        dfs(v, u, cnt);
    }
    cnt[a[u]]--;
    if (cnt[a[u]] == 0) cnt.erase(a[u]);
}

int main() {
    ios::sync_with_stdio(false);
    cin >> n;
    init();
    for (int i = 0; i < n - 1; i++) {
        int u, v;
        cin >> u >> v;
        addEdge(u, v), addEdge(v, u); // 树是无向图
    }
    for (int i = 1; i <= n; i++) cin >> a[i];
    unordered_map<int, int> cnt;
    dfs(n, 0, cnt);
    for (int i = 1; i <= n; i++) {
        cout << ans[i] << " ";
    }
    cout << endl;
    return 0;
}
```

## [公司](https://www.luogu.com.cn/problem/U412534)

```cpp
// 23真题1 —— 公司
// https://www.luogu.com.cn/problem/U412534
#include <algorithm>
#include <iostream>

using namespace std;
using LL  = long long;
using PLL = pair<LL, LL>;

const int    N = 1e5 + 10, M = 2 * N;
const double esp = 1e-6;
int          n, m;      // 公司的人数、泄露的工资情况数
LL           salary[N]; // 员工自己的工资
PLL          know[N];   // 员工了解到的其它人的工资情况，first表示知道的人数，second表示知道的总工资

int main() {
    cin >> n >> m;
    for (int i = 1; i <= n; i++) cin >> salary[i]; // 员工编号从1开始
    for (int i = 1; i <= m; i++) {
        int x, y;
        cin >> x >> y;               // x号雇员知道了y号雇员的工资
        know[x].first++;             // x号雇员知道的人数
        know[x].second += salary[y]; // x号雇员知道的工资的总数
    }

    int ans = 0;
    for (int i = 1; i <= n; i++) {
        if (know[i].first)                                                              // 如果雇员知道其他人的工资情况不为0条
            if ((((double) know[i].second / (double) know[i].first) - salary[i]) > esp) // 并且知道的平均工资高于自身工资
                ans++;                                                                  // 萌生离职的想法
    }
    cout << ans;

    return 0;
}
```

## [任务调度](https://www.luogu.com.cn/problem/U412641)

```cpp
// 23真题2 —— 任务调度
// https://www.luogu.com.cn/problem/U412641
#include <algorithm>
#include <iostream>
#include <list>
#include <map>

using namespace std;

int rd() {
    int  k = 0, f = 1;
    char c = getchar();
    while (c < '0' || c > '9') {
        if (c == '-') f = 0;
        c = getchar();
    }
    while (c >= '0' && c <= '9') {
        k = (k << 1) + (k << 3) + (c ^ 48);
        c = getchar();
    }
    return f ? k : -k;
}

void wr(int x) {
    if (x < 0)
        putchar('-'), x = -x;
    if (x > 9)
        wr(x / 10);
    putchar((x % 10) ^ '0');
}

int n, m;             // 需要执行的操作数、等待队列的最大容量
int opt, a, x;        // 任务的操作类型、重要性、要插入到的位置
int global_tasks = 0; // 记录全局任务数
struct Task {
    int id = -1; // 任务的编号，初始化任务时id都为-2
    int prior;   // 任务的重要程度
};
list<Task>                     wait_queue; // 任务的等待队列
map<int, list<Task>::iterator> task_map;   // 使用map来快速定位任务

int handle1(int prior) {        // 处理操作1 a ，指的是一个优先级为prior的任务到达，将其添加到等待队列尾部
    global_tasks++;             // 不论成功与否，都会导致全局任务id增加
    if (wait_queue.size() >= m) // 队列已满，添加新任务的操作失败
        return -1;

    wait_queue.push_back({global_tasks, prior});     // 否则就在队列末端添加任务
    task_map[global_tasks] = prev(wait_queue.end()); // 在map中存储任务的迭代器
    return global_tasks;
}

int handle2(int prior, int replace) { // 处理操作2 a x ，指将一个优先级为prior的任务插入到编号为replace的任务之前

    global_tasks++;                                                         // 不论成功与否，都会导致全局任务id增加
    if (wait_queue.size() >= m || task_map.find(replace) == task_map.end()) // 队列已满或未找到指定任务，就报错
        return -1;

    // 在编号replace的任务前插入
    auto it = wait_queue.insert(task_map[replace], {global_tasks, prior});

    task_map[global_tasks] = it; // 同样在map中存储任务迭代器
    return global_tasks;         // 返回新插入任务的id
}

int handle3() {
    if (wait_queue.empty()) return -1; // 如果队列为空，则报错

    int res = wait_queue.front().id; // 否则记录队首元素的id
    task_map.erase(res);             // 从map中删除该task
    wait_queue.pop_front();          // 删除队首元素，代表执行完毕任务
    return res;                      // 返回执行完毕的任务的id
}

int handle4() {
    if (wait_queue.empty()) return -1; // 如果队列为空，则报错

    auto max_it = max_element(wait_queue.begin(), wait_queue.end(), [](const Task &a, const Task &b) {
        return a.prior < b.prior;
    });

    int res = max_it->id; // 记录最大优先级任务的id
    task_map.erase(res);
    wait_queue.erase(max_it); // 从队列中删除，代表完成任务

    return res; // 返回完成任务的id
}

int main() {

    n = rd();
    m = rd();

    while (n--) {
        opt = rd();
        if (opt == 1) {
            a       = rd();
            int res = handle1(a);
            if (res == -1)
                puts("ERR");
            else {
                wr(res);
                putchar('\n');
            }
        } else if (opt == 2) {
            a       = rd();
            x       = rd();
            int res = handle2(a, x);
            if (res == -1)
                puts("ERR");
            else {
                wr(res);
                putchar('\n');
            }
        } else if (opt == 3) {
            int res = handle3();
            if (res == -1)
                puts("ERR");
            else {
                wr(res);
                putchar('\n');
            }
        } else {
            int res = handle4();
            if (res == -1)
                puts("ERR");
            else {
                wr(res);
                putchar('\n');
            }
        }
    }

    return 0;
}
```

```cpp
// 手搓双端队列+map
#include <cstdio>
#include <iostream>
#include <map>
using namespace std;

const int N = 1e6 + 10;

int           n, m;
int           e[N], l[N], r[N];
int           idx, count;
map<int, int> mp;

void init() {
    l[-1] = 0, r[0] = -1;
    idx = 1;
}

void add(int k, int x) {
    e[idx]  = x;
    l[idx]  = k;
    r[idx]  = r[k];
    l[r[k]] = idx;
    r[k]    = idx;
    idx++;
    count++;
}

void remove1() {
    if (!count) {
        printf("ERR\n");
        return;
    }
    int id  = r[0];
    int tmp = e[id];
    mp.erase(tmp);
    r[0]    = r[id];
    l[r[0]] = 0;
    count--;
    printf("%d\n", id);
    r[id] = -2;
}

void remove2() {
    if (!count) {
        printf("ERR\n");
        return;
    }
    auto tmp = mp.end();
    tmp--;
    int id = tmp->second;
    mp.erase(tmp->first);
    printf("%d\n", id);
    r[l[id]] = r[id];
    l[r[id]] = l[id];
    r[id]    = -2;
    count--;
}

int main(void) {

    cin >> n >> m;

    init();

    while (n--) {
        int x;
        cin >> x;
        if (x == 1) {
            int imp;
            cin >> imp;
            if (count == m) {
                printf("ERR\n");
                idx++;
            } else {

                mp[imp] = idx;
                printf("%d\n", idx);
                add(l[-1], imp);
            }
        } else if (x == 2) {
            int imp, x1;
            cin >> imp >> x1;
            bool flag = false;
            if (r[x1] == -2 || r[x1] == 0) flag = true;
            if (count == m || flag) {
                printf("ERR\n");
                idx++;
            } else {
                mp[imp] = idx;
                printf("%d\n", idx);
                add(l[x1], imp);
            }
        } else if (x == 3) {
            remove1();
        } else if (x == 4) {
            remove2();
        }
    }

    return 0;
}
```

```cpp
// 利用priority_queue
#include <bits/stdc++.h>
using namespace std;

int       ID = 0;
const int N  = 5e5 + 5;

struct Task {
    int a;
    int id;

    Task(int aa) {
        ID++;
        id = ID;
        a  = aa;
    }
};

map<int, list<Task>::iterator> id2it;
map<int, list<Task>::iterator> a2it;
priority_queue<int>            pq;
list<Task>                     l;

int n, m;

void print() {
    cout << "DBG:";
    for (auto i = l.begin(); i != l.end(); i++) {
        cout << "(" << i->id << ',' << i->a << ") ";
    }
    cout << endl;
}

void insert(int a) {
    Task t(a);
    if (l.size() == m)
        cout << "ERR" << endl;
    else {
        l.push_back(t);
        id2it[t.id] = std::prev(l.end());
        a2it[t.a]   = std::prev(l.end());
        pq.push(t.a);
        cout << t.id << endl;
    }
}

void insert_before(int a, int x) {
    Task t(a);
    if (l.size() == m || id2it.find(x) == id2it.end())
        cout << "ERR" << endl;
    else {
        auto it     = l.insert(id2it[x], t);
        id2it[t.id] = it;
        a2it[t.a]   = it;
        pq.push(t.a);
        cout << t.id << endl;
    }
}

void pop_front() {
    if (l.empty())
        cout << "ERR" << endl;
    else {
        auto &t = l.front();
        cout << t.id << endl;
        id2it.erase(t.id);
        a2it.erase(t.a);
        l.pop_front();
    }
}

void pop_top() {
    if (l.empty()) {
        cout << "ERR" << endl;
    } else {
        while (not pq.empty()) {
            auto a = pq.top();
            if (a2it.find(a) == a2it.end())
                pq.pop();
            else {
                auto it = a2it[a];
                cout << it->id << endl;
                id2it.erase(it->id);
                a2it.erase(it->a);
                l.erase(it);
                pq.pop();
                break;
            }
        }
    }
}

int main() {
    cin >> n >> m;
    while (n--) {
        int op, a, x;
        cin >> op;
        if (op == 1) {
            cin >> a;
            insert(a);
        } else if (op == 2) {
            cin >> a >> x;
            insert_before(a, x);
        } else if (op == 3) {
            pop_front();
        } else if (op == 4) {
            pop_top();
        }
        // print();
    }
}
```

## [总k次方差(easy)](https://www.luogu.com.cn/problem/U414204)

```cpp
// 22调剂题1——总k次方差(easy)
// https://www.luogu.com.cn/problem/U414204
#include <algorithm>
#include <cmath>
#include <iostream>

using namespace std;
using LL = long long;

int rd() {
    int  k = 0, f = 1;
    char c = getchar();
    while (c < '0' || c > '9') {
        if (c == '-') f = 0;
        c = getchar();
    }
    while (c >= '0' && c <= '9') {
        k = (k << 1) + (k << 3) + (c ^ 48);
        c = getchar();
    }
    return f ? k : -k;
}

void wr(int x) {
    if (x < 0)
        putchar('-'), x = -x;
    if (x > 9)
        wr(x / 10);
    putchar((x % 10) ^ '0');
}

const int N = 1e5 + 10, MOD = 998244353;
int       n, k; // n个数，k次方差
LL        a[N]; // 存放数组

int main() {
    n = rd(), k = rd(); // easy版的题目中k恒为1，因此只需要优化差的绝对值的总和即可
    for (int i = 1; i <= n; i++)
        a[i] = rd();

    stable_sort(a + 1, a + n + 1);

    // LL ans = 0;
    // for (int i = 1; i <= n; i++)
    //     for (int j = i + 1; j <= n; j++)
    //         ans = (ans + abs(a[i] - a[j])) % MOD;

    LL ans = 0;
    for (int i = 1; i <= n; i++) {
        ans = (ans + a[i] * (1ll * (i - 1)) - a[i] * (1ll * (n - i))) % MOD;
        ans = (ans + MOD) % MOD; // 确保结果为正
    }

    cout << (ans * 2) % MOD << endl; // 输出结果

    return 0;
}
```

```cpp
// 赵佬的前缀和思想
#include <algorithm>
#include <cstdio>
using namespace std;

int       n, a[100010], s;
long long ans;

int main() {
    scanf("%d%*d", &n);
    for (int i = 1; i <= n; ++i) scanf("%d", a + i), s += a[i];
    sort(a + 1, a + n + 1);

    // 排序后动态维护前缀和
    for (int i = 1; i <= n; ++i) ans = (ans + s - (1ll * a[i] * (n - i + 1))) % 998244353, s -= a[i];
    printf("%lld\n", ans << 1);
    return 0;
}
```

## [考试](https://www.luogu.com.cn/problem/U414149)

```cpp
// 22调剂 2 —— 考试
// https://www.luogu.com.cn/problem/U414149
#include <iostream>

using namespace std;
using LL = long long;

const int N = 1e4 + 10, MOD = 7654321;
int       t, n[N]; // 代表T组数据，n次考试的输入
LL        f[N][4]; // j有CA、Net、OS、Sig可选，分别下标为0,1,2,3

/*
CA-->Net-->CA-->Net-->CA
      \          \
       \          \-->OS
        \
         \->OS-->Net-->CA
            \     \
             \     \-->OS
              \
               \->Sig-->OS

显然是DP的思路，因此设状态f[i][j]，表示第i次考试的科目为j的考试安排的可能数
，因此j有CA、Net、OS、Sig可选。
   序号:  0   1   2   3

根据规律，可以得出，f[i][0]=f[i-1][1]
                    f[i][1]=f[i-1][0]+f[i-1][2]
                    f[i][2]=f[i-1][3]+f[i-1][1]
                    f[i][3]=f[i-1][2]

题目的答案就是 ans = f[n][0]+f[n][1]+f[n][2]+f[n][3];
 */

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(nullptr);

    cin >> t;
    int max_n = 0;
    for (int i = 0; i < t; i++) {
        cin >> n[i], max_n = max(max_n, n[i]);
    }
    fill(f[0], f[0] + N * 4, 0ll);
    f[1][0] = 1ll;

    for (int i = 2; i <= max_n; i++) {
        f[i][0] = f[i - 1][1] % MOD;
        f[i][1] = (f[i - 1][0] + f[i - 1][2]) % MOD;
        f[i][2] = (f[i - 1][3] + f[i - 1][1]) % MOD;
        f[i][3] = f[i - 1][2] % MOD;
    }

    for (int i = 0; i < t; i++) {
        cout << (1ll * (f[n[i]][0] + f[n[i]][1] + f[n[i]][2] + f[n[i]][3])) % MOD << endl;
    }

    return 0;
}
```

## [k叉树](https://www.luogu.com.cn/problem/U414310)

```cpp
// 22模拟题1 —— k叉树
// https://www.luogu.com.cn/problem/U414310
#include <iostream>
#include <queue>

using namespace std;

const int  N = 1e5 + 10, M = N * 2;
int        m, k;                       // m条边和k个儿子的限制条件
int        h[N], e[M], ne[M], idx = 0; // 最多m+1个节点，并最多组成一棵树，无向稀疏图
queue<int> que;                        // 用来BFS中做辅助队列
bool       st[N];                      // 用来BFS中判断是否遍历过
int        degree[N];                  // 记录每个节点的度数(出度+入度)

void addEdge(int a, int b) {
    e[idx] = b, ne[idx] = h[a], h[a] = idx++;
}

bool isTree(int u) { // 从节点u开始BFS，判断是否是树
    st[u] = true;
    que.push(u);

    int visited = 1; // 记录已遍历节点数，如果是树，其是极大无环图，因此将会等于节点总数
    while (que.size()) {
        int t = que.front();
        que.pop();

        for (int i = h[t]; i != -1; i = ne[i]) {
            int j = e[i];
            if (!st[j]) {
                st[j] = true;
                que.push(j);
                visited++;
            }
        }
    }
    if (visited == m + 1)
        return true;
    else
        return false;
}

int main() {
    fill(h, h + N, -1);
    cin >> m >> k;
    for (int i = 0; i < m; i++) {
        int a, b;
        cin >> a >> b;
        addEdge(a, b), addEdge(b, a);
        degree[a]++, degree[b]++;
    }

    if (!isTree(0)) {
        cout << "It's not a tree!" << endl;
        return 0;
    }

    for (int i = 0; i <= m; i++) { // 只要找到一个节点的度数比k+1大，就不会存在节点能够满足所有儿子数小于k
        if (degree[i] > k + 1) {
            cout << "No such a node!" << endl;
            return 0;
        }
    }

    for (int i = 0; i <= m; i++) { // 一旦找到第一个满足节点度数小于等于k的，就可以作为根，只有度数k+1的节点不能作为根
        if (degree[i] <= k) {
            cout << i << endl;
            return 0;
        }
    }
}
```

## [字符串](https://www.luogu.com.cn/problem/U276782)

```cpp
// 22真题1 —— 字符串
// https://www.luogu.com.cn/problem/U276782
#include <iostream>
#include <string>
#include <vector>

using namespace std;
using LL = long long;

const int N = 1e5 + 10;
int       n, m; // 字符串长度为n，子串长度限制为m

LL calc(LL t) {
    if (t < m) return 0ll;
    return 1ll * (t - m + 1) * (t - m + 2) / 2;
}

int main() {
    cin >> n >> m;

    vector<int> vec; // 记录输入字符串中纯1子串的长度，每遇到一个0，就换新的一个元素记录
    string      input;
    cin >> input;
    int len = 0;
    for (auto it = input.begin(); it != input.end(); it++) {
        if (*it == '1')
            len++;
        else {
            vec.push_back(len);
            len = 0;
        }
    }
    vec.push_back(len);

    LL ans = 0ll;
    for (auto &&i : vec) {
        ans += calc(i * 1ll);
    }
    cout << ans;
    return 0;
}
```