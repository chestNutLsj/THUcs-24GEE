
## partition ()
partition 可直译为 “分组”，partition () 函数**可根据用户自定义的筛选规则，重新排列指定区域内存储的数据，使其分为 2 组**，第一组为符合筛选条件的数据，另一组为不符合筛选条件的数据。

举个例子，假设有一个数组 `a[9]`，其存储数据如下：
```
1 2 3 4 5 6 7 8 9
```

在此基础上，如果设定筛选规则为 `i%2=0`（其中 i 即代指数组 a 中的各个元素），则借助 partition () 函数，`a[9]` 数组中存储数据的顺序可能变为：
```
1 9 3 7 5 6 4 8 2
```
其中 {1,9,3,7,5} 为第一组，{6,4,8,2} 为第二组。显然前者中的各个元素都符合筛选条件，而后者则都不符合。由此还可看出，partition () 函数只会根据筛选条件将数据进行分组，并不关心分组后各个元素具体的存储位置。

> 如果想在分组之后仍不改变各元素之间的相对位置，可以选用 stable_partition () 函数。有关此函数的功能和用法，本节后续会做详细讲解。

值得一提得是，partition () 函数定义于 `<algorithm>` 头文件中，因此在使用该函数之前，程序中应先引入此头文件：

```
#include <algorithm> 
```

如下为 partition () 函数的语法格式：
```
ForwardIterator partition (ForwardIterator first,
                           ForwardIterator last,
                           UnaryPredicate pred);
```

其中，first 和 last 都为正向迭代器，其组合 `[first, last)` 用于指定该函数的作用范围；pred 用于指定筛选规则。

> 所谓筛选规则，其本质就是一个可接收 1 个参数且返回值类型为 bool 的函数，可以是普通函数，也可以是一个函数对象。

同时，partition () 函数还会返回一个正向迭代器，其指向的是两部分数据的分界位置，更确切地说，**指向的是第二组数据中的第 1 个元素**。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::partition
#include <vector>       // std::vector
using namespace std;
//以普通函数的方式定义partition()函数的筛选规则
bool mycomp(int i) { return (i % 2) == 0; }

//以函数对象的形式定义筛选规则
class mycomp2 {
public:
    bool operator()(const int& i) {
        return (i%2 == 0);
    }
};

int main() {
    std::vector<int> myvector{1,2,3,4,5,6,7,8,9};
    std::vector<int>::iterator bound;
    //以 mycomp2 规则，对 myvector 容器中的数据进行分组
    bound = std::partition(myvector.begin(), myvector.end(), mycomp2());
    for (std::vector<int>::iterator it = myvector.begin(); it != myvector.end(); ++it) {
        cout << *it << " ";
    }
    cout << "\nbound = " << *bound;
    return 0;
}
```

程序执行结果为：
```
8 2 6 4 5 3 7 1 9  
bound = 5
```

可以看到，程序中借助 partition () 对 myvector 容器中的数据进行了再加工，基于 mycomp 2 () 筛选规则，能够被 2 整除的元素位于第 1 组，不能被 2 整除的元素位于第 2 组。

同时，parition () 函数会返回一个迭代器，通过观察程序的执行结果可以看到，该迭代器指向的是元素 5，同时也是第 2 组数据中的第 1 个元素。

> [C++ STL partition() 函数官方](http://www.cplusplus.com/reference/algorithm/partition/)给出了该函数底层实现的参考代码，感兴趣的读者可自行前往分析，这里不再做过多描述。

## stable_partition ()
-------------------------

前面提到，partition () 函数只负责对指定区域内的数据进行分组，并不保证各组中元素的相对位置不发生改变。而如果想在分组的同时保证不改变各组中元素的相对位置，可以使用 stable_partition () 函数。

也就是说，stable_partition () 函数可以保证对指定区域内数据完成分组的同时，不改变各组内元素的相对位置。

仍以数组 `a[9]` 举例，其存储的数据如下：
```
1 2 3 4 5 6 7 8 9
```
假定筛选规则为 `i%2=0`（其中 i 即代指数组 a 中的各个元素），则借助 stable_partition () 函数，`a[9]` 数组中存储数据的顺序为：
```
2 4 6 8 1 3 5 7 9
```
其中 {2,4,6,8} 为一组，{1,3,5,7,9} 为另一组。通过和先前的 `a[9]` 对比不难看出，各个组中元素的相对位置没有发生改变。

> 所谓元素的相对位置不发生改变，以 {2,4,6,8} 中的元素 4 为例，在原 `a[9]` 数组中，该元素位于 2 的右侧，6 和 8 的左侧；在经过 stable_partition () 函数处理后的 `a[9]` 数组中，元素 4 仍位于 2 的右侧，6 和 8 的左侧。因此，该元素的相对位置确实没有发生改变。

stable_partition () 函数定义在 `<algorithm>` 头文件中，其语法格式如下：

```
BidirectionalIterator stable_partition (BidirectionalIterator first,
                                        BidirectionalIterator last,
                                        UnaryPredicate pred);
```

其中，first 和 last 都为双向迭代器，其组合 `[first, last)` 用于指定该函数的作用范围；pred 用于指定筛选规则。

同时，stable_partition () 函数还会返回一个双向迭代器，其指向的是两部分数据的分界位置，更确切地说，指向的是第二组数据中的第 1 个元素。

举个例子：
```
#include <iostream>     
#include <algorithm>    
#include <vector>       
using namespace std;
bool mycomp(int i) { return (i % 2) == 1; }
class mycomp2 {
public:
    bool operator()(const int& i) {
        return (i%2 == 1);
    }
};
int main() {
    std::vector<int> myvector{1,2,3,4,5,6,7,8,9};
    std::vector<int>::iterator bound;
    
    bound = std::stable_partition(myvector.begin(), myvector.end(), mycomp);
    for (std::vector<int>::iterator it = myvector.begin(); it != myvector.end(); ++it) {
        cout << *it << " ";
    }
    cout << "\nbound = " << *bound;
    return 0;
}

```

程序执行结果为：
```
1 3 5 7 9 2 4 6 8  
bound = 2
```

## partition_copy ()

上一节中，已经详细介绍了 partition () 和 stable_partition () 函数的功能和用法。不知道读者是否发现，这 2 个函数在实现功能时，都直接修改了原序列中元素的存储位置。

而在某些场景中，我们需要类似 partition () 或者 stable_partition () 函数 “分组” 的功能，但并不想对原序列做任何修改。这种情况下，就可以考虑使用 partition_copy () 函数。

和 stable_partition () 一样，partition_copy () 函数也能按照某个筛选规则对指定区域内的数据进行 “分组”，并且分组后不会改变各个元素的相对位置。更重要的是，partition_copy () 函数不会对原序列做修改，而是以复制的方式将序列中各个元组“分组” 到其它的指定位置存储。

举个例子，有如下一个数组 `a[10]`：
```
1 2 3 4 5 6 7 8 9
```
假设筛选条件为 `i%2==0`（也就是筛选出偶数），如果借助 stable_partition () 函数，则数组 `a[10]` 中元素的存储顺序会变成：
```
2 4 6 8 1 3 5 7 9
```
而如果选用同样的筛选规则，使用 partition_copy () 函数还需要为其配备 2 个存储空间（例如 `b[10]` 和 `c[10]`），其中 `b[10]` 用于存储符合筛选条件的偶数，而 `c[10]` 用于存储不符合筛选条件的奇数，也就是说，partition_copy () 函数执行的最终结果为：
```
a[10]: 1 2 3 4 5 6 7 8 9  
b[10]: 2 4 6 8  
c[10]: 1 3 5 7 9
```
> 注意，这里仅展示了 b[10] 和 c[10] 数组中存储的有效数据。

partition_copy () 函数定义在 `<algorithm>` 头文件中，其语法格式如下：

```
pair<OutputIterator1,OutputIterator2> partition_copy (
                    InputIterator first, InputIterator last,
                    OutputIterator1 result_true, OutputIterator2 result_false,
                    UnaryPredicate pred);
```

其中，各个参数的含义为：
* first、last：都为输入迭代器，其组合 `[first, last)` 用于指定该函数处理的数据区域；
* result_true：为输出迭代器，其用于指定某个存储区域，以存储满足筛选条件的数据；
* result_false：为输出迭代器，其用于指定某个存储区域，以存储满足筛选条件的数据；
* pred：用于指定筛选规则，其本质就是接收一个具有 1 个参数且返回值类型为 bool 的函数。注意，该函数既可以是普通函数，还可以是一个函数对象。

除此之外，该函数还会返回一个 pair 类型值，其包含 2 个迭代器，第一个迭代器指向的是 result_true 区域内最后一个元素之后的位置；第二个迭代器指向的是 result_false 区域内最后一个元素之后的位置

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::partition_copy
#include <vector>       // std::vector
using namespace std;
//以普通函数的方式定义筛选规则
bool mycomp(int i) { return (i % 2) == 0; }

//以函数对象的形式定义筛选规则
class mycomp2 {
public:
    bool operator()(const int& i) {
        return (i % 2 == 0);
    }
};

int main() {
    vector<int> myvector{ 1,2,3,4,5,6,7,8,9 };
    int b[10] = { 0 }, c[10] = { 0 };
    //以 mycomp 规则，对 myvector 容器中的数据进行分组，这里的 mycomp 还可以改为 mycomp2()，即以 mycomp2 为筛选规则
    pair<int*, int*> result= partition_copy(myvector.begin(), myvector.end(), b, c, mycomp);
    cout << "b[10]：";
    for (int *p = b; p < result.first; p++) {
        cout << *p << " ";
    }
    cout << "\nc[10]：";
    for (int *p = c; p < result.second; p++) {
        cout << *p << " ";
    }
    return 0;
}
```

程序执行结果为：
```
b[10]：2 4 6 8  
c[10]：1 3 5 7 9
```
> 程序中仅演示了如何用数组来存储 partition_copy () 函数分组后的数据，当然也可以用容器来存储。

C++ 标准库中还给出了 partition_copy () 函数底层实现的参考代码，感兴趣的读者可自行研究，这里不再进行过多赘述。

```
template <class InputIterator, class OutputIterator1,
          class OutputIterator2, class UnaryPredicate pred>
          pair<OutputIterator1,OutputIterator2>
partition_copy (InputIterator first, InputIterator last,
                OutputIterator1 result_true, OutputIterator2 result_false,
                UnaryPredicate pred)
{
    while (first!=last) {
        if (pred(*first)) {
            *result_true = *first;
            ++result_true;
        }
        else {
            *result_false = *first;
            ++result_false;
        }
        ++first;
    }
    return std::make_pair (result_true,result_false);
}

```

## partition_point ()

在前面章节中，我们系统学习了 partition ()、stable_partition () 和 partition_copy () 这 3 个函数，它们的功能本质上都是根据某个筛选规则对指定范围内的数据进行分组（即符合条件的为一组，不符合条件的为另一组），并且反馈给我们两组数据之间的分界位置。

事实上，有些数据本身就已经是按照某个筛选规则分好组的，例如：

```
1,2,3,4,5,6,7      <-- 根据规则 i<4，{1,2,3} 为一组，{4,5,6,7} 为另一组
2,4,6,8,1,3,5,7,9  <-- 根据规则 i%2=0，{2,4,6,8} 为一组，{1,3,5,7,9} 为另一组
```

类似上面这样已经 “分好组” 的数据，在使用时会有一个问题，即不知道两组数据之间的分界在什么位置。有读者可能想到，再调用一次 partition ()、stale_partition () 或者 partition_copy () 不就可以了吗？这种方法确实可行，但对已经分好组的数据再进行一次分组，是没有任何必要的。

实际上，对于如何在已分好组的数据中找到分界位置，C++ 11 标准库提供了专门解决此问题的函数，即 partition_point () 函数。

partition_point () 函数定义在 `<algorithm>` 头文件中，其语法格式为：

```
ForwardIterator partition_point (ForwardIterator first, ForwardIterator last,
                                 UnaryPredicate pred);
```

其中，first 和 last 为正向迭代器，`[first, last)` 用于指定该函数的作用范围；pred 用于指定数据的筛选规则。

> 所谓筛选规则，其实就是包含 1 个参数且返回值类型为 bool 的函数，此函数可以是一个普通函数，也可以是一个函数对象。

同时，该函数会返回一个正向迭代器，该迭代器指向的是 `[first, last]` 范围内第一个不符合 pred 筛选规则的元素。

举个例子：
```
#include <iostream>     
#include <algorithm>    
#include <vector>       
using namespace std;
bool mycomp(int i) { return (i % 2) == 0; }
class mycomp2 {
public:
    bool operator()(const int& i) {
        return (i % 2 == 0);
    }
};
int main() {
    vector<int> myvector{ 2,4,6,8,1,3,5,7,9 };
    
    vector<int>::iterator iter = partition_point(myvector.begin(), myvector.end(),mycomp);
    
    for (auto it = myvector.begin(); it != iter; ++it) {
        cout << *it << " ";
    }
    cout << "\n";
    
    for (auto it = iter; it != myvector.end(); ++it) {
        cout << *it << " ";
    }
    cout << "\n*iter = " << *iter;
    return 0;
}

```

程序执行结果为：
```
2 4 6 8  
1 3 5 7 9  
*iter = 1
```
通过分析程序并结合输出结果可以看到，partition_point () 返回了一个指向元素 1 的迭代器，而该元素为 myvector 容器中第一个不符合 mycomp 规则的元素，同时其也可以第二组数据中第一个元素。

值得一提的是，C++ 11 标准库中给出了 partition_point () 函数底层实现的参考代码（如下所示），感兴趣的读者可自行分析，这里不再进行赘述：

```
template <class ForwardIterator, class UnaryPredicate>
ForwardIterator partition_point (ForwardIterator first, ForwardIterator last,
                                   UnaryPredicate pred)
{
    auto n = distance(first,last);
    while (n>0)
    {
        ForwardIterator it = first;
        auto step = n/2;
        std::advance (it,step);
        if (pred(*it)) { first=++it; n-=step+1; }
        else n=step;
    }
    return first;
}

```