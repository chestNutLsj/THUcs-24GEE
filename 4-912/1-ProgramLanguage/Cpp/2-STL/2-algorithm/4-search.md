## search ()

find_end () 函数用于在序列 A 中查找序列 B 最后一次出现的位置。那么，如果想知道序列 B 在序列 A 中第一次出现的位置，该如何实现呢？可以借助 search () 函数。

search () 函数定义在 `<algorithm>` 头文件中，其功能恰好和 find_end () 函数相反，用于在序列 A 中查找序列 B 第一次出现的位置。

例如，仍以如下两个序列为例：
```
序列 A：1,2,3,4,5,1,2,3,4,5  
序列 B：1,2,3
```

可以看到，序列 B 在序列 A 中出现了 2 次。借助 find_end () 函数，我们可以找到序列 A 中最后一个{1,2,3}；而借助 search () 函数，我们可以找到序列 A 中第 1 个 {1,2,3}。

和 find_end () 相同，search () 函数也提供有以下 2 种语法格式：
```
//查找 [first1, last1) 范围内第一个 [first2, last2) 子序列
ForwardIterator search (ForwardIterator first1, ForwardIterator last1,
                        ForwardIterator first2, ForwardIterator last2);
                        
//查找 [first1, last1) 范围内，和 [first2, last2) 序列满足 pred 规则的第一个子序列
ForwardIterator search (ForwardIterator first1, ForwardIterator last1,
                        ForwardIterator first2, ForwardIterator last2,
                        BinaryPredicate pred);
```

其中，各个参数的含义分别为：
* first 1、last 1：都为正向迭代器，其组合 \[first 1, last 1\) 用于指定查找范围（也就是上面例子中的序列 A）；
* first 2、last 2：都为正向迭代器，其组合 \[first 2, last 2\) 用于指定要查找的序列（也就是上面例子中的序列 B）；
* pred：用于自定义查找规则。该规则实际上是一个包含 2 个参数且返回值类型为 bool 的函数（第一个参数接收 \[first 1, last 1\) 范围内的元素，第二个参数接收 \[first 2, last 2\) 范围内的元素）。函数定义的形式可以是普通函数，也可以是函数对象。

> 实际上，第一种语法格式也可以看做是包含一个默认的 pred 参数，该参数指定的是一种相等规则，即在 \[first 1, last 1\) 范围内查找和 \[first 2, last 2\) 中各个元素对应相等的子序列；而借助第二种语法格式，我们可以自定义一个当前场景需要的匹配规则。

同时，search () 函数会返回一个正向迭代器，当函数查找成功时，该迭代器指向查找到的子序列中的第一个元素；反之，如果查找失败，则该迭代器的指向和 last 1 迭代器相同。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::search
#include <vector>       // std::vector
using namespace std;
//以普通函数的形式定义一个匹配规则
bool mycomp1(int i, int j) {
    return (i%j == 0);
}

//以函数对象的形式定义一个匹配规则
class mycomp2 {
public:
    bool operator()(const int& i, const int& j) {
        return (i%j == 0);
    }
};

int main() {
    vector<int> myvector{ 1,2,3,4,8,12,18,1,2,3 };
    int myarr[] = { 1,2,3 };
    //调用第一种语法格式
    vector<int>::iterator it = search(myvector.begin(), myvector.end(), myarr, myarr + 3);
    if (it != myvector.end()) {
        cout << "第一个{1,2,3}的起始位置为：" << it - myvector.begin() << ",*it = " << *it << endl;
    }

    int myarr2[] = { 2,4,6 };
    //调用第二种语法格式
    it = search(myvector.begin(), myvector.end(), myarr2, myarr2 + 3, mycomp2());
    if (it != myvector.end()) {
        cout << "第一个{2,3,4}的起始位置为：" << it - myvector.begin() << ",*it = " << *it;
    }
    return 0;
}
```

程序执行结果为：
```
第一个 {1,2,3} 的起始位置为：0,*it = 1  
第一个 {2,3,4} 的起始位置为：3,*it = 4
```

通过程序的执行结果可以看到，
- 第 22 行代码借助 search () 函数找到了 myvector 容器中第一个 {1,2,3}，并返回了一个指向元素 1 的迭代器（其下标位置为 0）。
- 而在第 29 行中，search () 函数使用的是第 2 种格式，其自定义了 mycomp 2 匹配规则，即在 myvector 容器中找到第一个连续的 3 个元素，它们能分别被 2、4、6 整除。显然，myvector 容器中符合要求的子序列有 2 个，分别为 {4,8,12} 和 {8,12,18}，但 search () 函数只会查找到第一个，并返回指向元素 4 的迭代器（其下标为 3）。

> 注意，search () 函数的第一种语法格式，其底层是借助 == 运算符实现的。这意味着，如果 \[first 1, last 1\] 和 \[first 2, last 2\] 区域内的元素为自定义的类对象或结构体变量时，使用该函数之前需要对 == 运算符进行重载。

C++ STL 标准库官方给出了 search () 函数底层实现的参考代码，感兴趣的读者可自行分析，这里不再做过多描述：

```
template<class ForwardIterator1, class ForwardIterator2>
ForwardIterator1 search(ForwardIterator1 first1, ForwardIterator1 last1,
  ForwardIterator2 first2, ForwardIterator2 last2)
{
    if (first2 == last2) return first1;
    while (first1 != last1)
    {
        ForwardIterator1 it1 = first1;
        ForwardIterator2 it2 = first2;
        while (*it1 == *it2) {    
            if (it2 == last2) return first1;
            if (it1 == last1) return last1;
            ++it1; ++it2;
        }
        ++first1;
    }
    return last1;
}

```

## search_n ()

上一节中，已经详细介绍了 search () 函数的功能和用法。在此基础上，本节再介绍一个功能类似的函数，即 search_n () 函数。

和 search () 一样，search_n () 函数也定义在 `<algorithm>` 头文件中，用于在指定区域内查找第一个符合要求的子序列。不同之处在于，前者查找的子序列中可包含多个不同的元素，而后者查找的只能是包含多个相同元素的子序列。

关于 search () 函数和 search_n () 函数的区别，给大家举个例子，下面有 3 个序列：
```
序列 A：1,2,3,4,4,4,1,2,3,4,4,4  
序列 B：1,2,3  
序列 C：4,4,4
```
如果想查找序列 B 在序列 A 中第一次出现的位置，就只能使用 search () 函数；而如果想查找序列 C 在序列 A 中第一次出现的位置，既可以使用 search () 函数，也可以使用 search_n () 函数。

search_n () 函数的语法格式如下：
```
//在 [first, last] 中查找 count 个 val 第一次连续出现的位置
ForwardIterator search_n (ForwardIterator first, ForwardIterator last,
                          Size count, const T& val);
//在 [first, last] 中查找第一个序列，该序列和 count 个 val 满足 pred 匹配规则
ForwardIterator search_n ( ForwardIterator first, ForwardIterator last,
                           Size count, const T& val, BinaryPredicate pred );
```

其中，各个参数的含义分别为：
* first、last：都为正向迭代器，其组合 \[first, last\) 用于指定查找范围（也就是上面例子中的序列 A）；
* count、val：指定要查找的元素个数和元素值，以上面的序列 B 为例，该序列实际上就是 3 个元素 4，其中 count 为 3，val 为 4；
* pred：用于自定义查找规则。该规则实际上是一个包含 2 个参数且返回值类型为 bool 的函数（第一个参数接收 \[first, last\) 范围内的元素，第二个参数接收 val）。函数定义的形式可以是普通函数，也可以是函数对象。

> 实际上，第一种语法格式也可以看做是包含一个默认的 pred 参数，该参数指定的是一种相等规则，即在 \[first, last\) 范围内查找和 count 个 val 相等的子序列；而借助第二种语法格式，我们可以自定义一个当前场景需要的匹配规则。

同时，search_n () 函数会返回一个正向迭代器，当函数查找成功时，该迭代器指向查找到的子序列中的第一个元素；反之，如果查找失败，则该迭代器的指向和 last 迭代器相同。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::search_n
#include <vector>       // std::vector
using namespace std;
//以普通函数的形式定义一个匹配规则
bool mycomp1(int i, int j) {
    return (i%j == 0);
}

//以函数对象的形式定义一个匹配规则
class mycomp2 {
public:
    bool operator()(const int& i, const int& j) {
        return (i%j == 0);
    }
};

int main() {
    int a[] = { 1,2,3,4,4,4,1,2,3,4,4,4 };
    //调用第一种语法格式,查找 myvector 容器中第一个 {4,4,4}
    int * it = search_n(a, a+12, 3, 4);
    if (it != a+12) {
        cout << "one：" << it - a << ",*it = " << *it << endl;
    }

    vector<int> myvector{1,2,4,8,3,4,6,8};
    //调用第二种语法格式，以自定义的 mycomp2 作为匹配规则，查找 myvector 容器中和 {16,16,16} 满足 mycomp2 规则的序列
    vector<int>::iterator iter = search_n(myvector.begin(), myvector.end(), 3, 2, mycomp2());
    if (iter != myvector.end()) {
        cout << "two：" << iter - myvector.begin() << ",*iter = " << *iter;
    }
    return 0;
}
```

程序执行结果为：
```
one：3,*it = 4  
two：1,*iter = 2
```
程序中先后调用了 2 种语法格式的 search_n () 函数，
- 其中第 28 行代码中，search_n () 函数不再采用默认的相等匹配规则，而是采用了自定义了 mycomp 2 匹配规则。这意味着，该函数会去 myvector 容器中查找一个子序列，该序列中的 3 个元素都满足和 2 有 (i%j == 0) 的关系。显然，myvector 容器中符合条件的子序列有 2 个，分别为 {2,4,8} 和 {4,6,8}，但 search_n () 函数只会查找到 {2,4,8}。

> 注意，search_n () 函数的第一种语法格式，其底层是借助 == 运算符实现的。这意味着，如果 \[first, last\] 区域内的元素为自定义的类对象或结构体变量时，使用此格式的 search_n () 函数之前，需要对 == 运算符进行重载。

C++ STL 标准库官方给出了 search_n () 函数底层实现的参考代码，感兴趣的读者可自行分析，这里不再做过多描述：
```
template<class ForwardIterator, class Size, class T>
ForwardIterator search_n (ForwardIterator first, ForwardIterator last,
                            Size count, const T& val)
{
    ForwardIterator it, limit;
    Size i;
    limit=first; std::advance(limit,std::distance(first,last)-count);
    while (first!=limit)
    {
        it = first; i=0;
        while (*it==val)       
        { ++it; if (++i==count) return first; }
        ++first;
    }
    return last;
}

```

