前面章节中，已经给大家系统地介绍了几个查找函数，如 find ()、find_if ()、search () 等。值得一提的是，这些函数的底层实现都采用的是顺序查找（逐个遍历）的方式，在某些场景中的执行效率并不高。例如，当指定区域内的数据处于有序状态时，如果想查找某个目标元素，更推荐使用二分查找的方法（相比顺序查找，二分查找的执行效率更高）。

幸运的是，除了前面讲过的几个函数外，C++ STL 标准库中还提供有 lower_bound ()、upper_bound ()、equal_range () 以及 binary_search () 这 4 个查找函数，它们的底层实现采用的都是二分查找的方式。

从本节开始，将给大家系统地讲解这 4 个二分查找函数的功能和用法，这里先从 lower_bound () 函数开始讲起。

## lower_bound ()

lower_bound () 函数用于在指定区域内查找不小于目标值的第一个元素。也就是说，使用该函数在指定范围内查找某个目标值时，最终查找到的不一定是和目标值相等的元素，还可能是比目标值大的元素。

lower_bound () 函数定义在 `<algorithm>` 头文件中，其语法格式有 2 种，分别为：

```
//在 [first, last) 区域内查找不小于 val 的元素
ForwardIterator lower_bound (ForwardIterator first, ForwardIterator last,
                             const T& val);
//在 [first, last) 区域内查找第一个不符合 comp 规则的元素
ForwardIterator lower_bound (ForwardIterator first, ForwardIterator last,
                             const T& val, Compare comp);
```

其中，first 和 last 都为正向迭代器，`[first, last)` 用于指定函数的作用范围；val 用于指定目标元素；comp 用于自定义比较规则，此参数可以接收一个包含 2 个形参（第二个形参值始终为 val）且返回值为 bool 类型的函数，可以是普通函数，也可以是函数对象。

> 实际上，第一种语法格式也设定有比较规则，只不过此规则无法改变，即使用 < 小于号比较 `[first, last)` 区域内某些元素和 val 的大小，直至找到一个不小于 val 的元素。这也意味着，如果使用第一种语法格式，则 `[first, last)` 范围的元素类型必须支持 < 运算符。

此外，该函数还会返回一个正向迭代器，当查找成功时，迭代器指向找到的元素；反之，如果查找失败，迭代器的指向和 last 迭代器相同。

再次强调，该函数仅适用于已排好序的序列。所谓 “已排好序”，指的是 `[first, last)` 区域内所有令 element<val（或者 comp (element, val)，其中 element 为指定范围内的元素）成立的元素都位于不成立元素的前面。✅ 注意，此处有序并不指完全有序，而是一种偏序关系

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::lower_bound
#include <vector>       // std::vector
using namespace std;
//以普通函数的方式定义查找规则
bool mycomp(int i,int j) { return i>j; }

//以函数对象的形式定义查找规则
class mycomp2 {
public:
    bool operator()(const int& i, const int& j) {
        return i>j;
    }
};

int main() {
    int a[5] = { 1,2,3,4,5 };
    //从 a 数组中找到第一个不小于 3 的元素
    int *p = lower_bound(a, a + 5, 3);
    cout << "*p = " << *p << endl;

    vector<int> myvector{ 4,5,3,1,2 };
    //根据 mycomp2 规则，从 myvector 容器中找到第一个违背 mycomp2 规则的元素
    vector<int>::iterator iter = lower_bound(myvector.begin(), myvector.end(),3,mycomp2());
    cout << "*iter = " << *iter;
    return 0;
}
```

程序执行结果为：
```
*p = 3  
*iter = 3
```

注意，myvector 容器中存储的元素看似是乱序的，但对于元素 3 来说，大于 3 的所有元素都位于其左侧，小于 3 的所有元素都位于其右侧，且查找规则选用的是 mycomp 2 ()，其查找的就是第一个不大于 3 的元素，因此 lower_bound () 函数是可以成功运行的。

C++ STL 标准库给出了 lower_bound () 函数底层实现的参考代码（如下所示），感兴趣的读者可自行研究，这里不再赘述：

```
template <class ForwardIterator, class T>
ForwardIterator lower_bound (ForwardIterator first, ForwardIterator last, const T& val)
{
    ForwardIterator it;
    iterator_traits<ForwardIterator>::difference_type count, step;
    count = distance(first,last);
    while (count>0)
    {
        it = first; step=count/2; advance (it,step);
        if (*it<val) {  
            first=++it;
            count-=step+1;
        }
        else count=step;
    }
    return first;
}
```

## upper_bound ()

上一节中，系统地介绍了 lower_bound () 二分法查找函数的功能和用法，在此基础上，本节再讲解一个功能类似的查找函数，即 upper_bound () 函数。

upper_bound () 函数定义在 `<algorithm>` 头文件中，用于在指定范围内查找大于目标值的第一个元素。该函数的语法格式有 2 种，分别是：
```
//查找[first, last)区域中第一个大于 val 的元素。
ForwardIterator upper_bound (ForwardIterator first, ForwardIterator last,
                             const T& val);
//查找[first, last)区域中第一个不符合 comp 规则的元素
ForwardIterator upper_bound (ForwardIterator first, ForwardIterator last,
                             const T& val, Compare comp);
```

其中，first 和 last 都为正向迭代器，`[first, last)` 用于指定该函数的作用范围；val 用于执行目标值；comp 作用自定义查找规则，此参数可接收一个包含 2 个形参（第一个形参值始终为 val）且返回值为 bool 类型的函数，可以是普通函数，也可以是函数对象。

> 实际上，第一种语法格式也设定有比较规则，即用 < 小于号比较 `[first, last)` 区域内某些元素和 val 的大小，直至找到一个大于 val 的元素，只不过此规则无法改变。这也意味着，如果使用第一种语法格式，则 `[first, last)` 范围的元素类型必须支持 < 运算符。

同时，该函数会返回一个正向迭代器，当查找成功时，迭代器指向找到的元素；反之，如果查找失败，迭代器的指向和 last 迭代器相同。

另外，由于 upper_bound () 底层实现采用的是二分查找的方式，因此该函数仅适用于 “已排好序” 的序列。注意，这里所说的 “已排好序”，并不要求数据完全按照某个排序规则进行升序或降序排序，而仅仅要求 `[first, last)` 区域内所有令 element<val（或者 comp (val, element）成立的元素都位于不成立元素的前面（其中 element 为指定范围内的元素）。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::upper_bound
#include <vector>       // std::vector
using namespace std;
//以普通函数的方式定义查找规则
bool mycomp(int i, int j) { return i > j; }
//以函数对象的形式定义查找规则
class mycomp2 {
public:
    bool operator()(const int& i, const int& j) {
        return i > j;
    }
};
int main() {
    int a[5] = { 1,2,3,4,5 };
    //从 a 数组中找到第一个大于 3 的元素
    int *p = upper_bound(a, a + 5, 3);
    cout << "*p = " << *p << endl;
    vector<int> myvector{ 4,5,3,1,2 };
    //根据 mycomp2 规则，从 myvector 容器中找到第一个违背 mycomp2 规则的元素
    vector<int>::iterator iter = upper_bound(myvector.begin(), myvector.end(), 3, mycomp2());
    cout << "*iter = " << *iter;
    return 0;
}
```

程序执行结果为：
```
*p = 4  
*iter = 1
```
> 借助输出结果可以看出，upper_bound () 函数的功能和 lower_bound () 函数不同，前者查找的是大于目标值的元素，而后者查找的不小于（大于或者等于）目标值的元素。

此程序中演示了 upper_bound () 函数的 2 种适用场景，其中 `a[5]` 数组中存储的为升序序列；而 myvector 容器中存储的序列虽然整体是乱序的，但对于目标元素 3 来说，所有符合 mycomp 2 (3, element) 规则的元素都位于其左侧，不符合的元素都位于其右侧，因此 upper_bound () 函数仍可正常执行。

C++ STL 标准库给出了 upper_bound () 函数底层实现的参考代码（如下所示），感兴趣的读者可自行研究，这里不再赘述：
```
template <class ForwardIterator, class T>
ForwardIterator upper_bound (ForwardIterator first, ForwardIterator last, const T& val)
{
    ForwardIterator it;
    iterator_traits<ForwardIterator>::difference_type count, step;
    count = std::distance(first,last);
    while (count>0)
    {
        it = first; step=count/2; std::advance (it,step);
        if (!(val<*it))  
            { first=++it; count-=step+1;  }
        else count=step;
    }
    return first;
}

```

## equal_range ()

equel_range () 函数定义在 `<algorithm>` 头文件中，用于在指定范围内查找等于目标值的所有元素。

值得一提的是，当指定范围内的数据支持用 <小于运算符直接做比较时，可以使用如下格式的 equel_range () 函数：

```
//找到 [first, last) 范围中所有等于 val 的元素
pair<ForwardIterator,ForwardIterator> equal_range (ForwardIterator first, ForwardIterator last, const T& val);
```

如果指定范围内的数据为自定义的类型（用结构体或类），就需要自定义比较规则，这种情况下可以使用如下格式的 equel_range () 函数：

```
//找到 [first, last) 范围内所有等于 val 的元素
pair<ForwardIterator,ForwardIterator> equal_range (ForwardIterator first, ForwardIterator last, const T& val, Compare comp);
```

以上 2 种格式中，first 和 last 都为正向迭代器，`[first, last)` 用于指定该函数的作用范围；val 用于指定目标值；comp 用于指定比较规则，此参数可接收一个包含 2 个形参（第二个形参值始终为 val）且返回值为 bool 类型的函数，可以是普通函数，也可以是函数对象。

同时，该函数会返回一个 pair 类型值，其包含 2 个正向迭代器。当查找成功时：

1. 第 1 个迭代器指向的是 `[first, last)` 区域中第一个等于 val 的元素；
2. 第 2 个迭代器指向的是 `[first, last)` 区域中第一个大于 val 的元素。

反之如果查找失败，则这 2 个迭代器要么都指向大于 val 的第一个元素（如果有），要么都和 last 迭代器指向相同。

需要注意的是，由于 equel_range () 底层实现采用的是二分查找的方式，因此该函数仅适用于 “已排好序” 的序列。所谓 “已排好序”，并不是要求 `[first, last)` 区域内的数据严格按照某个排序规则进行升序或降序排序，只要满足“所有令 element<val（或者 comp (element, val）成立的元素都位于不成立元素的前面（其中 element 为指定范围内的元素）” 即可。

举个例子：
```
#include <iostream>     
#include <algorithm>    
#include <vector>       
using namespace std;
bool mycomp(int i, int j) { return i > j; }
class mycomp2 {
public:
    bool operator()(const int& i, const int& j) {
        return i > j;
    }
};
int main() {
    int a[9] = { 1,2,3,4,4,4,5,6,7};
    
    pair<int*, int*> range = equal_range(a, a + 9, 4);
    cout << "a[9]：";
    for (int *p = range.first; p < range.second; ++p) {
        cout << *p << " ";
    }
    vector<int>myvector{ 7,8,5,4,3,3,3,3,2,1 };
    pair<vector<int>::iterator, vector<int>::iterator> range2;
    
    range2 = equal_range(myvector.begin(), myvector.end(), 3,mycomp2());
    cout << "\nmyvector：";
    for (auto it = range2.first; it != range2.second; ++it) {
        cout << *it << " ";
    }
    return 0;
}

```

程序执行结果为：
```
a[9]：4 4 4  
myvector：3 3 3 3
```
此程序中演示了 equal_range () 函数的 2 种适用场景，其中 `a[9]` 数组中存储的为升序序列；而 myvector 容器中存储的序列虽然整体是乱序的，但对于目标元素 3 来说，所有符合 mycomp 2 (element, 3) 规则的元素都位于其左侧，不符合的元素都位于其右侧，因此 equal_range () 函数仍可正常执行。

实际上，equel_range () 函数的功能完全可以看做是 lower_bound () 和 upper_bound () 函数的合体。C++ STL 标准库给出了 equel_range () 函数底层实现的参考代码（如下所示），感兴趣的读者可自行研究，这里不再赘述：
```
template <class ForwardIterator, class T>
pair<ForwardIterator,ForwardIterator> equal_range (ForwardIterator first, ForwardIterator last, const T& val)
{
    ForwardIterator it = std::lower_bound (first,last,val);
    return std::make_pair ( it, std::upper_bound(it,last,val) );
}
template<class ForwardIterator, class T, class Compare>
std::pair<ForwardIt,ForwardIt> equal_range(ForwardIterator first, ForwardIterator last, const T& val, Compare comp)
{
    ForwardIterator it = std::lower_bound (first,last,val,comp);
    return std::make_pair ( it, std::upper_bound(it,last,val,comp) );
}

```


## binary_search ()

binary_search () 函数定义在 `<algorithm>` 头文件中，用于查找指定区域内是否包含某个目标元素。

该函数有 2 种语法格式，分别为：
```
//查找 [first, last) 区域内是否包含 val
bool binary_search (ForwardIterator first, ForwardIterator last,
                      const T& val);
//根据 comp 指定的规则，查找 [first, last) 区域内是否包含 val
bool binary_search (ForwardIterator first, ForwardIterator last,
                      const T& val, Compare comp);
```

其中，first 和 last 都为正向迭代器，`[first, last)` 用于指定该函数的作用范围；val 用于指定要查找的目标值；comp 用于自定义查找规则，此参数可接收一个包含 2 个形参（第一个形参值为 val）且返回值为 bool 类型的函数，可以是普通函数，也可以是函数对象。

同时，该函数会返回一个 bool 类型值，如果 binary_search () 函数在 `[first, last)` 区域内成功找到和 val 相等的元素，则返回 true；反之则返回 false。

需要注意的是，由于 binary_search () 底层实现采用的是二分查找的方式，因此该函数仅适用于 “已排好序” 的序列。所谓 “已排好序”，并不是要求 `[first, last)` 区域内的数据严格按照某个排序规则进行升序或降序排序，只要满足“所有令 element<val（或者 comp (val, element）成立的元素都位于不成立元素的前面（其中 element 为指定范围内的元素）” 即可。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::binary_search
#include <vector>       // std::vector
using namespace std;
//以普通函数的方式定义查找规则
bool mycomp(int i, int j) { return i > j; }
//以函数对象的形式定义查找规则
class mycomp2 {
public:
    bool operator()(const int& i, const int& j) {
        return i > j;
    }
};
int main() {
    int a[7] = { 1,2,3,4,5,6,7 };
    //从 a 数组中查找元素 4
    bool haselem = binary_search(a, a + 9, 4);
    cout << "haselem：" << haselem << endl;

    vector<int>myvector{ 4,5,3,1,2 };
    //从 myvector 容器查找元素 3
    bool haselem2 = binary_search(myvector.begin(), myvector.end(), 3, mycomp2());
    cout << "haselem2：" << haselem2;
    return 0;
}
```

程序执行结果为：
```
haselem：1  
haselem 2：1
```
此程序中演示了 binary_search () 函数的 2 种适用场景，其中 `a[7]` 数组中存储的为升序序列；而 myvector 容器中存储的序列虽然整体是乱序的，但对于目标元素 3 来说，所有符合 mycomp 2 (element, 3) 规则的元素都位于其左侧，不符合的元素都位于其右侧，因此 binary_search () 函数仍可正常执行。

C++ STL 标准库给出了 binary_search () 函数底层实现的参考代码（如下所示），感兴趣的读者可自行研究，这里不再赘述：

```
template <class ForwardIterator, class T>
bool binary_search (ForwardIterator first, ForwardIterator last, const T& val)
{
    first = std::lower_bound(first,last,val);
    return (first!=last && !(val<*first));
}
template<class ForwardIt, class T, class Compare>
bool binary_search(ForwardIt first, ForwardIt last, const T& val, Compare comp)
{
    first = std::lower_bound(first, last, val, comp);
    return (!(first == last) && !(comp(val, *first)));
}

```