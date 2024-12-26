
priority_queue 容器适配器模拟的也是队列这种存储结构，即使用此容器适配器存储元素只能 “从一端进（称为队尾），从另一端出（称为队头）”，且每次只能访问 priority_queue 中位于队头的元素。

但是，priority_queue 容器适配器中元素的存和取，遵循的并不是 “First in, First out”（先入先出）原则，而是 “First in，Largest out” 原则。直白的翻译，指的就是先进队列的元素并不一定先出队列，而是优先级最大的元素最先出队列。

> 注意，“First in，Largest out” 原则是笔者为了总结 priority_queue 存取元素的特性自创的一种称谓，仅为了方便读者理解。

那么，priority_queue 容器适配器中存储的元素，优先级是如何评定的呢？很简单，每个 priority_queue 容器适配器在创建时，都制定了一种排序规则。根据此规则，该容器适配器中存储的元素就有了优先级高低之分。

举个例子，假设当前有一个 priority_queue 容器适配器，其制定的排序规则是按照元素值从大到小进行排序。根据此规则，自然是 priority_queue 中值最大的元素的优先级最高。

priority_queue 容器适配器为了保证每次从队头移除的都是当前优先级最高的元素，每当有新元素进入，它都会根据既定的排序规则找到优先级最高的元素，并将其移动到队列的队头；同样，当 priority_queue 从队头移除出一个元素之后，它也会再找到当前优先级最高的元素，并将其移动到队头。

基于 priority_queue 的这种特性，因此该容器适配器有被称为优先级队列。

> priority_queue 容器适配器 “First in，Largest out” 的特性，和它底层采用**堆结构**存储数据是分不开的。有关该容器适配器的底层实现，后续章节会进行深度剖析。

STL 中，priority_queue 容器适配器的定义如下：

```
template <typename T,
        typename Container=std::vector<T>,
        typename Compare=std::less<T> >
class priority_queue{
    //......
}
```

可以看到，priority_queue 容器适配器模板类最多可以传入 3 个参数，它们各自的含义如下：

* typename T：指定存储元素的具体类型；
* typename Container：指定 priority_queue 底层使用的基础容器，默认使用 vector 容器。

> 作为 priority_queue 容器适配器的底层容器，其必须包含 empty ()、size ()、front ()、push_back ()、pop_back () 这几个成员函数，序列式容器中只有 vector 和 deque 容器符合条件。

* typename Compare：指定容器中评定元素优先级所遵循的排序规则，默认使用 `std::less<T>` 按照元素值从大到小进行排序，还可以使用 `std::greater<T>` 按照元素值从小到大排序，但更多情况下是使用自定义的排序规则。

> 其中，std:: less\<T\> 和 std:: greater\<T\> 都是以函数对象的方式定义在 \<function\> 头文件中。关于如何自定义排序规则，后续章节会做详细介绍。


## 创建 priority_queue 
-----------------------

由于 priority_queue 容器适配器模板位于`<queue>`头文件中，并定义在 std 命名空间里，因此在试图创建该类型容器之前，程序中需包含以下 2 行代码：

```
#include <queue>
using namespace std;
```

创建 priority_queue 容器适配器的方法，大致有以下几种。  
1) 创建一个空的 priority_queue 容器适配器，第底层采用默认的 vector 容器，排序方式也采用默认的 std:: less\<T\> 方法：
```
std::priority_queue<int> values;
```

2) 可以使用普通数组或其它容器中指定范围内的数据，对 priority_queue 容器适配器进行初始化：
```
//使用普通数组
int values[]{4,1,3,2};
std::priority_queue<int>copy_values (values, values+4);//{4,2,3,1}

//使用序列式容器
std::array<int,4>values{ 4,1,3,2 };
std::priority_queue<int>copy_values (values.begin (), values.end ());//{4,2,3,1}
```
注意，以上 2 种方式必须保证数组或容器中存储的元素类型和 priority_queue 指定的存储类型相同。另外，用来初始化的数组或容器中的数据不需要有序，priority_queue 会自动对它们进行排序。

3) 还可以手动指定 priority_queue 使用的底层容器以及排序规则，比如：
```
int values[]{ 4,1,2,3 };
std::priority_queue<int, std::deque<int>, std::greater<int> >copy_values (values, values+4);//{1,3,2,4}
```
事实上，std:: less\<T\> 和 std:: greater\<T\> 适用的场景是有限的，更多场景中我们会使用自定义的排序规则。

> 由于自定义排序规则的方式不只一种，因此这部分知识将在后续章节做详细介绍。

## priority_queue 提供的成员函数
----------------------

priority_queue 容器适配器提供了下表所示的这些成员函数。

| 成员函数                                       | 功能                                                                                                        |
|--------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| empty ()                                   | 如果 priority_queue 为空的话，返回 true；反之，返回 false。                                                               |
| size ()                                    | 返回 priority_queue&nbsp; 中存储元素的个数。                                                                         |
| top ()                                     | 返回 priority_queue&nbsp; 中第一个元素的引用形式。                                                                      |
| push (const T&amp; obj)                    | 根据既定的排序规则，将元素 obj 的副本存储到 priority_queue&nbsp; 中适当的位置。                                                     |
| push (T&amp;&amp; obj)                     | 根据既定的排序规则，将元素 obj 移动存储到 priority_queue&nbsp; 中适当的位置。                                                      |
| emplace (Args&amp;&amp;... args)           | Args&amp;&amp;... args 表示构造一个存储类型的元素所需要的数据（对于类对象来说，可能需要多个数据构造出一个对象）。此函数的功能是根据既定的排序规则，在容器适配器适当的位置直接生成该新元素。 |
| pop ()                                     | 移除 priority_queue&nbsp; 容器适配器中第一个元素。                                                                      |
| swap (priority_queue&lt; T&gt;&amp; other) | 将两个 priority_queue 容器适配器中的元素进行互换，需要注意的是，进行互换的 2 个 priority_queue 容器适配器中存储的元素类型以及底层采用的基础容器类型，都必须相同。        |

> 和 queue 一样，priority_queue 也没有迭代器，因此访问元素的唯一方式是遍历容器，通过不断移除访问过的元素，去访问下一个元素。

下面的程序演示了表中部分成员函数的具体用法：

```
#include <iostream>
#include <queue>
#include <array>
#include <functional>
using namespace std;

int main ()
{
    //创建一个空的 priority_queue 容器适配器
    std::priority_queue<int>values;
    //使用 push () 成员函数向适配器中添加元素
    values.push (3);//{3}
    values.push (1);//{3,1}
    values.push (4);//{4,1,3}
    values.push (2);//{4,2,3,1}
    //遍历整个容器适配器
    while (! values.empty ())
    {
        //输出第一个元素并移除。
        std:: cout << values.top ()<<" ";
        values.pop ();//移除队头元素的同时，将剩余元素中优先级最大的移至队头
    }
    return 0;
}
```

运行结果为：
```
4 3 2 1
```

## 自定义排序

前面讲解 priority_queue 容器适配器时，还遗留一个问题，即当 \<function\> 头文件提供的排序方式（std:: less\<T\> 和 std:: greater\<T\>）不再适用时，如何自定义一个满足需求的排序规则。

首先，无论 priority_queue 中存储的是基础数据类型（int、double 等），还是 string 类对象或者自定义的类对象，都可以使用函数对象的方式自定义排序规则。例如：

```
#include <iostream>
#include <queue>

using namespace std;

template <typename T>
class cmp
{
public:
    bool operator ()(T a, T b)
    {
        return a > b;
    }
};

int main ()
{
    int a[] = { 4,2,3,5,6 };
    priority_queue<int,vector<int>,cmp<int> > pq (a, a+5);

    while (! pq.empty ())
    {
        cout << pq.top () << " ";
        pq.pop ();
    }
    return 0;
}
```

运行结果为：
```
2 3 4 5 6
```

注意，C++ 中的 struct 和 class 非常类似，前者也可以包含成员变量和成员函数，因此上面程序中，函数对象类 cmp 也可以使用 struct 关键字创建：

```
struct cmp
{
    bool operator ()(T a, T b)
    {
        return a > b;
    }
};
```

可以看到，通过在 cmp 类（结构体）重载的 () 运算符中自定义排序规则，并将其实例化后作为 priority_queue 模板的第 3 个参数传入，即可实现为 priority_queue 容器适配器自定义比较函数。

除此之外，当 priority_queue 容器适配器中存储的数据类型为结构体或者类对象（包括 string 类对象）时，还可以通过重载其 > 或者 < 运算符，间接实现自定义排序规则的目的。

> 注意，此方式仅适用于 priority_queue 容器中存储的为类对象或者结构体变量，也就是说，当存储类型为类的指针对象或者结构体指针变量时，此方式将不再适用，而只能使用函数对象的方式。

要想彻底理解这种方式的实现原理，首先要搞清楚 `std::less<T>` 和 `std::greater<T>` 各自的底层实现。实际上，`<function>` 头文件中的 `std::less<T>` 和 `std::greater<T>` ，各自底层实现采用的都是函数对象的方式。比如，`std::less<T>` 的底层实现代码为：

```
template <typename T>
struct less {
    bool operator ()(const T &_lhs, const T &_rhs) const {
        return _lhs < _rhs;
    }
};
```

`std::greater<T>` 的底层实现代码为：

```
template <typename T>
struct greater {
    bool operator ()(const T &_lhs, const T &_rhs) const {
        return _lhs > _rhs;
    }
};
```

可以看到，`std::less<T>` 和 `std::greater<T>` 底层实现的唯一不同在于，前者使用 < 号实现从大到小排序，后者使用 > 号实现从小到大排序。

那么，是否可以通过重载 <或者> 运算符修改 `std::less<T>` 和 `std::greater<T>` 的排序规则，从而间接实现自定义排序呢？答案是肯定的，举个例子：

```
#include <queue>
#include <iostream>

using namespace std;

class node {
public:
    node (int x = 0, int y = 0) : x (x), y (y) {}
    int x, y;
};

bool operator < (const node &a, const node &b) {
    if (a.x > b.x) return 1;
    else if (a.x == b.x)
        if (a.y >= b.y) return 1;
    return 0;
}

int main () {

    priority_queue<node> pq;
    pq.push (node (1, 2));
    pq.push (node (2, 2));
    pq.push (node (3, 4));
    pq.push (node (3, 3));
    pq.push (node (2, 3));
    
    cout << "x y" << endl;
    while (! pq.empty ()) {
        cout << pq.top (). x << " " << pq.top (). y << endl;
        pq.pop ();
    }

    return 0;
}
```

输出结果为：
```
x y  
1 2  
2 2  
2 3  
3 3  
3 4
```

可以看到，通过重载 < 运算符，使得 `std::less<T>` 变得适用了。

> 读者还可以自行尝试，通过重载 > 运算符，赋予 `std::greater<T>` 和之前不同的排序方式。

当然，也可以以友元函数或者成员函数的方式重载 > 或者 < 运算符。需要注意的是，以成员函数的方式重载 > 或者 < 运算符时，该成员函数必须声明为 const 类型，且参数也必须为 const 类型，至于参数的传值方式是采用按引用传递还是按值传递，都可以（建议采用按引用传递，效率更高）。

例如，将上面程序改为以成员函数的方式重载 < 运算符：
```
class node {
public:
    node (int x = 0, int y = 0) : x (x), y (y) {}
    int x, y;
    bool operator < (const node &b) const{
        if ((*this). x > b.x) return 1;
        else if ((*this). x == b.x)
            if ((*this). y >= b.y) return 1;
        return 0;
    }
};
```

同样，在以友元函数的方式重载 < 或者 > 运算符时，要求参数必须使用 const 修饰。例如，将上面程序改为以友元函数的方式重载 < 运算符。例如：

```
class node {
public:
    node (int x = 0, int y = 0) : x (x), y (y) {}
    int x, y;
    friend bool operator < (const node &a, const node &b);
};

bool operator < (const node &a, const node &b){
    if (a.x > b.x) return 1;
    else if (a.x == b.x)
        if (a.y >= b.y) return 1;
    return 0;
}
```

总的来说，以函数对象的方式自定义 priority_queue 的排序规则，适用于任何情况；而以重载 > 或者 < 运算符间接实现 priority_queue 自定义排序的方式，仅适用于 priority_queue 中存储的是结构体变量或者类对象（包括 string 类对象）。

## 深度剖析 PQ 的底层实现

priority_queue 优先级队列之所以总能保证优先级最高的元素位于队头，最重要的原因是其底层采用堆数据结构存储结构。

有读者可能会问，priority_queue 底层不是采用 vector 或 deque 容器存储数据吗，这里又说使用堆结构存储数据，它们之间不冲突吗？显然，它们之间是不冲突的。

首先，vector 和 deque 是用来存储元素的容器，而堆是一种数据结构，其本身无法存储数据，只能依附于某个存储介质，辅助其组织数据存储的先后次序。其次，priority_queue 底层采用 vector 或者 deque 作为基础容器，这毋庸置疑。但由于 vector 或 deque 容器并没有提供实现 priority_queue 容器适配器 “First in, Largest out” 特性的功能，因此 STL 选择使用堆来重新组织 vector 或 deque 容器中存储的数据，从而实现该特性。

> 注意，虽然不使用堆结构，通过编写算法调整 vector 或者 deque 容器中存储元素的次序，也能使其具备 “First in, Largest out” 的特性，但执行效率通常没有使用堆结构高。

那么，堆到底是什么，它又是怎样组织数据的呢？

## priority_queue 底层的堆存储结构
-----------------------

> 以下内容要求读者对数据结构中的树存储结构有一定的了解，如果没有，请先阅读《[树存储结构](http://www.cdsy.xyz/computer/programme/algorithm/20210307/cd161506658310404.html)》一章。

简单的理解堆，它在是完全二叉树的基础上，要求树中所有的父节点和子节点之间，都要满足既定的排序规则：
* 如果排序规则为从大到小排序，则表示堆的完全二叉树中，每个父节点的值都要不小于子节点的值，这种堆通常称为大顶堆；
* 如果排序规则为从小到大排序，则表示堆的完全二叉树中，每个父节点的值都要不大于子节点的值，这种堆通常称为小顶堆；

下图展示了一个由 {10,20,15,30,40,25,35,50,45} 这些元素构成的大顶堆和小顶堆。其中经大顶堆组织后的数据先后次序变为 {50,45,40,20,25,35,30,10,15}，而经小顶堆组织后的数据次序为 {10,20,15,25,50,30,40,35,45}。

![[heap-max-min.png]]

可以看到，大顶堆中，每个父节点的值都不小于子节点；同样在小顶堆中，每个父节点的值都不大于子节点。但需要注意的是，无论是大顶堆还是小顶堆，同一父节点下子节点的次序是不做规定的，这也是经大顶堆或小顶堆组织后的数据整体依然无序的原因。

可以确定的一点是，无论是通过大顶堆或者小顶堆，总可以筛选出最大或最小的那个元素（优先级最大），并将其移至序列的开头，此功能也正是 priority_queue 容器适配器所需要的。

为了验证 priority_queue 底层确实采用堆存储结构实现的，我们可以尝试用堆结合基础容器 vector 或 deque 实现 priority_queue。值得庆幸的是，STL 已经为我们封装好了可以使用堆存储结构的方法，它们都位于 `<algorithm>` 头文件中。下表中列出了常用的几个和堆存储结构相关的方法。

| 函数                                | 功能                                                                             |
|-----------------------------------|--------------------------------------------------------------------------------|
| make_heap (first, last, comp)     | 选择位于 \[first, last) 区域内的数据，并根据 comp 排序规则建立堆，其中 fist 和 last 可以是指针或者迭代器，默认是建立大顶堆。 |
| push_heap (first, last, comp)     | 当向数组或容器中添加数据之后，此数据可能会破坏堆结构，该函数的功能是重建堆。                                         |
| pop_heap (first, last, comp)      | 将位于序列头部的元素（优先级最高）移动序列尾部，并使 [first, last-1] 区域内的元素满足堆存储结构。                      |
| sort_heap (first, last, comp)     | 对 \[first, last) 区域内的元素进行堆排序，将其变成一个有序序列。                                        |
| is_heap_until (first, last, comp) | 发现 \[first, last) 区域内的最大堆。                                                      |
| is_heap (first, last, comp)       | 检查 \[first, last) 区域内的元素，是否为堆结构。                                                |

> 以上方法的实现，基于堆排序算法的思想，有关该算法的具体实现原理，可阅读《[堆排序](http://www.cdsy.xyz/computer/programme/algorithm/20210305/cd16149264218730.html)》一节做详细了解。

下面例子中，使用了表中的部分函数，并结合 vector 容器提供的成员函数，模拟了 priority_queue 容器适配器部分成员函数的底层实现：

```
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
void display (vector<int>& val) {
    for (auto v : val) {
        cout << v << " ";
    }
    cout << endl;
}
int main ()
{
    vector<int>values{ 2,1,3,4 };
    //建立堆
    make_heap (values.begin (), values.end ());//{4,2,3,1}
    display (values);
    //添加元素
    cout << "添加元素：\n";
    values. push_back (5);
    display (values);
    push_heap (values.begin (), values.end ());//{5,4,3,1,2}
    display (values);
    //移除元素
    cout << "移除元素：\n";
    pop_heap (values.begin (), values.end ());//{4,2,3,1,5}
    display (values);
    values. pop_back ();
    display (values);
    return 0;
}
```

运行结果为：
```
4 2 3 1  
添加元素：  
4 2 3 1 5  
5 4 3 1 2  
移除元素：  
4 2 3 1 5  
4 2 3 1
```

上面程序可以用 priority_queue 容器适配器等效替代：

```
#include <iostream>
#include <queue>
#include <vector>
using namespace std;
int main ()
{
    //创建优先级队列
    std::vector<int>values{ 2,1,3,4 };
    std::priority_queue<int>copy_values (values.begin (), values.end ());
    //添加元素
    copy_values.push (5);
    //移除元素
    copy_values.pop ();
    return 0;
}
```

如果调试此程序，查看各个阶段 priority_queue 中存储的元素，可以发现，它和上面程序的输出结果是一致。也就是说，此程序在创建 priority_queue 之后，其存储的元素依次为 {4,2,3,1}，同样当添加元素 5 之后，其存储的元素依次为 {5,4,3,1,2}，移除一个元素之后存储的元素依次为 {4,2,3,1}。