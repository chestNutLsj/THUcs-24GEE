
## find ()
find () 函数本质上是一个模板函数，用于在指定范围内查找和目标元素值相等的第一个元素。

如下为 find () 函数的语法格式：
```
InputIterator find (InputIterator first, InputIterator last, const T& val);
```

其中，first 和 last 为输入迭代器，\[first, last\) 用于指定该函数的查找范围；val 为要查找的目标元素。

> 正因为 first 和 last 的类型为输入迭代器，因此该函数**适用于所有的序列式容器**。

另外，该函数会返回一个输入迭代器，当 find () 函数查找成功时，其指向的是在 \[first, last\) 区域内查找到的第一个目标元素；如果查找失败，则该迭代器的指向和 last 相同。

值得一提的是，find () 函数的底层实现，其实就是用 `==` 运算符将 val 和 \[first, last\) 区域内的元素**逐个进行比对**。这也就意味着，\[first, last\) 区域内的元素必须支持 `==` 运算符。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::find
#include <vector>       // std::vector
using namespace std;
int main() {
    //find() 函数作用于普通数组
    char stl[] ="http://c.biancheng.net/stl/";
    //调用 find() 查找第一个字符 'c'
    char * p = find(stl, stl + strlen(stl), 'c');
    //判断是否查找成功
    if (p != stl + strlen(stl)) {
        cout << p << endl;
    }
    //find() 函数作用于容器
    std::vector<int> myvector{ 10,20,30,40,50 };
    std::vector<int>::iterator it;

    it = find(myvector.begin(), myvector.end(), 30);
    if (it != myvector.end())
        cout << "查找成功：" << *it;
    else
        cout << "查找失败";
    return 0;
}
```

程序执行结果为：
```
c.biancheng. net/stl/  
查找成功：30
```

可以看到，find () 函数除了可以作用于序列式容器，还可以作用于普通数组。

对于 find () 函数的底层实现，C++ 标准库中给出了参数代码，感兴趣的读者可自行研究：
```
template<class InputIterator, class T>
InputIterator find (InputIterator first, InputIterator last, const T& val)
{
    while (first!=last) {
        if (*first==val) return first;
        ++first;
    }
    return last;
}
```

## find_if

值得一提的是，find_if () 和 find_if_not () 函数都定义在 `<algorithm>` 头文件中。因此在使用它们之前，程序中要先引入此头文件：

```
#include <algorithm> 
```


和 find () 函数相同，find_if () 函数也用于在指定区域内执行查找操作。不同的是，前者需要明确指定要查找的元素的值，而后者则**允许自定义查找规则**。

所谓自定义查找规则，实际上指的是有一个形参且返回值类型为 bool 的函数。值得一提的是，该函数可以是一个普通函数（又称为一元谓词函数），比如：

```
bool mycomp(int i){
	return ((i%2)==1);
}
```

上面的 mycomp () 就是一个一元谓词函数，其可用来判断一个整数是奇数还是偶数。

> 如果读者想更深层次地了解 C++ 谓词函数，可阅读《[[什么是Cpp的谓词函数？]]》一节。

也可以是一个[[STL背景知识介绍#^60913c|函数对象]]，比如：

```
//以函数对象的形式定义一个 find_if() 函数的查找规则
class mycomp2 {
public:
    bool operator()(const int& i) {
        return ((i % 2) == 1);
    }
};
```
此函数对象的功能和 mycomp () 函数一样。

确切地说，**find_if () 函数会根据指定的查找规则，在指定区域内查找第一个符合该函数要求（使函数返回 true）的元素**。

find_if () 函数的语法格式如下：
```
InputIterator find_if (InputIterator first, InputIterator last, UnaryPredicate pred);
```
其中，first 和 last 都为输入迭代器，其组合 \[first, last\) 用于指定要查找的区域；pred 用于自定义查找规则。

> 值得一提的是，由于 first 和 last 都为输入迭代器，意味着该函数适用于所有的序列式容器。甚至当采用适当的谓词函数时，该函数还适用于所有的关联式容器（包括哈希容器）。

同时，该函数会返回一个输入迭代器，当查找成功时，该迭代器指向的是第一个符合查找规则的元素；反之，如果 find_if () 函数查找失败，则该迭代器的指向和 last 迭代器相同。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::find_if
#include <vector>       // std::vector
using namespace std;
//自定义一元谓词函数
bool mycomp(int i) {
    return ((i % 2) == 1);
}
//以函数对象的形式定义一个 find_if() 函数的查找规则
class mycomp2 {
public:
    bool operator()(const int& i) {
        return ((i % 2) == 1);
    }
};
int main() {
    vector<int> myvector{ 4,2,3,1,5 };
    //调用 find_if() 函数，并以 IsOdd() 一元谓词函数作为查找规则
    vector<int>::iterator it = find_if(myvector.begin(), myvector.end(), mycomp2());
    cout << "*it = " << *it;
    return 0;
}
```

程序执行结果为：
```
*it = 3
```
结合程序执行结果不难看出，对于 myvector 容器中的元素 4 和 2 来说，它们都无法使 `(i%2)==1` 这个表达式成立，因此 mycomp 2 () 返回 false；而对于元素 3 来说，它可以使 mycomp 2 () 函数返回 true，因此，find_if () 函数找到的第一个元素就是元素 3。

值得一提的是，[C++ STL find_if() 官网](http://www.cplusplus.com/reference/algorithm/find_if/)给出了 find_if () 函数底层实现的参考代码（如下所示），感兴趣的读者可自行分析，这里不做过多描述：

```
template<class InputIterator, class UnaryPredicate>
InputIterator find_if (InputIterator first, InputIterator last, UnaryPredicate pred)
{
    while (first!=last) {
        if (pred(*first)) return first;
        ++first;
    }
    return last;
}
```

## find_if_not ()

find_if_not () 函数和 find_if () 函数的功能恰好相反，通过上面的学习我们知道，find_if () 函数用于查找符合谓词函数规则的第一个元素，而 find_if_not () 函数则用于查找第一个不符合谓词函数规则的元素。

find_if_not () 函数的语法规则如下所示：
```
InputIterator find_if_not (InputIterator first, InputIterator last, UnaryPredicate pred);
```
其中，first 和 last 都为输入迭代器，\[first, last\) 用于指定查找范围；pred 用于自定义查找规则。

> 和 find_if () 函数一样，find_if_not () 函数也适用于所有的容器，包括所有序列式容器和关联式容器。

同样，该函数也会返回一个输入迭代器，**当 find_if_not () 函数查找成功时，该迭代器指向的是查找到的那个元素；反之，如果查找失败，该迭代器的指向和 last 迭代器相同**。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::find_if_not
#include <vector>       // std::vector
using namespace std;
//自定义一元谓词函数
bool mycomp(int i) {
    return ((i % 2) == 1);
}

int main() {
    vector<int> myvector{4,2,3,1,5};
    //调用 find_if() 函数，并以 mycomp() 一元谓词函数作为查找规则
    vector<int>::iterator it = find_if_not(myvector.begin(), myvector.end(), mycomp);
    cout << "*it = " << *it;
    return 0;
}
```

程序执行结果为：
```
*it = 4
```
可以看到，由于第一个元素 4 就不符合 `(i%2)==1`，因此 find_if_not () 成功找到符合条件的元素，并返回一个指向该元素的迭代器。

find_if_not () 函数的底层实现和 find_if () 函数非常类似，[C++ STL find_if_not() 官网](http://www.cplusplus.com/reference/algorithm/find_if_not/)给出了该函数底层实现的参考代码，感兴趣的读者可自行分析，这里不做过多描述：

```
template<class InputIterator, class UnaryPredicate>
InputIterator find_if_not (InputIterator first, InputIterator last, UnaryPredicate pred)
{
    while (first!=last) {
        if (!pred(*first)) return first;
        ++first;
    }
    return last;
}
```

## find_end ()

find_end () 函数定义在 `<algorithm>` 头文件中，常用于在序列 A 中查找序列 B 最后一次出现的位置。例如，有如下 2 个序列：
```
序列 A：1,2,3,4,5,1,2,3,4,5  
序列 B：1,2,3
```
通过观察不难发现，序列 B 在序列 A 中出现了 2 次，而借助 find_end () 函数，可以轻松的得到序列 A 中最后一个 {1,2,3}。

find_end () 函数的语法格式有 2 种：
```
//查找序列 [first1, last1) 中最后一个子序列 [first2, last2)
ForwardIterator find_end (ForwardIterator first1, ForwardIterator last1,
                          ForwardIterator first2, ForwardIterator last2);
//查找序列 [first2, last2) 中，和 [first2, last2) 序列满足 pred 规则的最后一个子序列
ForwardIterator find_end (ForwardIterator first1, ForwardIterator last1,
                          ForwardIterator first2, ForwardIterator last2,
                          BinaryPredicate pred);
```

其中，各个参数的含义如下：
* first 1、last 1：都为正向迭代器，其组合 \[first 1, last 1\) 用于指定查找范围（也就是上面例子中的序列 A）；
* first 2、last 2：都为正向迭代器，其组合 \[first 2, last 2\) 用于指定要查找的序列（也就是上面例子中的序列 B）；
* pred：用于自定义查找规则。该规则实际上是一个包含 2 个参数且返回值类型为 bool 的函数（第一个参数接收 \[first 1, last 1\) 范围内的元素，第二个参数接收 \[first 2, last 2\) 范围内的元素）。函数定义的形式可以是普通函数，也可以是函数对象。

> 实际上，第一种语法格式也可以看做是包含一个默认的 pred 参数，该参数指定的是一种相等规则，即在 \[first 1, last 1\) 范围内查找和 \[first 2, last 2\) 中各个元素对应相等的子序列；而借助第二种语法格式，我们可以自定义一个当前场景需要的匹配规则。

同时，find_end () 函数会返回一个正向迭代器，当函数查找成功时，该迭代器指向**查找到的子序列中的第一个元素**；反之，如果查找失败，则该迭代器的指向和 last1 迭代器相同。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::find_end
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
    vector<int>::iterator it = find_end(myvector.begin(), myvector.end(), myarr, myarr + 3);
    if (it != myvector.end()) {
        cout << "最后一个{1,2,3}的起始位置为：" << it - myvector.begin() << ",*it = " << *it << endl;
    }

    int myarr2[] = { 2,4,6 };
    //调用第二种语法格式
    it = find_end(myvector.begin(), myvector.end(), myarr2, myarr2 + 3, mycomp2());
    if (it != myvector.end()) {
        cout << "最后一个{2,3,4}的起始位置为：" << it - myvector.begin() << ",*it = " << *it;
    }
    return 0;
}
```
程序执行结果为：
```
匹配 {1,2,3} 的起始位置为：7,*it = 1  
匹配 {2,3,4} 的起始位置为：4,*it = 8
```

上面程序中共调用了 2 次 find_end () 函数：
1. 第 22 行代码：调用了第一种语法格式的 find_end () 函数，其功能是在 myvector 容器中查找和 {1,2,3} 相等的最后一个子序列，显然最后一个 {1,2,3} 中元素 1 的位置下标为 7（myvector 容器下标从 0 开始）；
2. 第 29 行代码：调用了第二种格式的 find_end () 函数，其匹配规则为 mycomp 2，即在 myvector 容器中找到最后一个子序列，该序列中的元素能分别被 {2、4、6} 中的元素整除。显然，myvector 容器中 {4,8,12} 和 {8,12,18} 都符合，该函数会找到后者并返回一个指向元素 8 的迭代器。

> 注意，find_end () 函数的第一种语法格式，其底层是借助 == 运算符实现的。这意味着，如果 \[first 1, last 1\] 和 \[first 2, last 2\] 区域内的元素为自定义的类对象或结构体变量时，**使用该函数之前需要对 == 运算符进行重载**。

C++ STL 标准库官方给出了 find_end () 函数底层实现的参考代码，感兴趣的读者可自行分析，这里不再做过多描述：
```
template<class ForwardIterator1, class ForwardIterator2>
ForwardIterator1 find_end(ForwardIterator1 first1, ForwardIterator1 last1,
  ForwardIterator2 first2, ForwardIterator2 last2)
{
    if (first2 == last2) return last1;  // specified in C++11

    ForwardIterator1 ret = last1;

    while (first1 != last1)
    {
        ForwardIterator1 it1 = first1;
        ForwardIterator2 it2 = first2;
        while (*it1 == *it2) {    // or: while (pred(*it1,*it2)) for version (2)
            ++it1; ++it2;
            if (it2 == last2) { ret = first1; break; }
            if (it1 == last1) return ret;
        }
        ++first1;
    }
    return ret;
}
```

## find_first_of ()

在某些情境中，我们可能需要在 A 序列中查找和 B 序列中任意元素相匹配的第一个元素，这时就可以使用 find_first_of () 函数。

find_first_of () 函数定义于 `<algorithm>` 头文件中，因此使用该函数之前，程序中要先引入此头文件：

```
#include <algorithm> 
```

find_first_of () 函数有 2 种语法格式，分别是：
```
//以判断两者相等作为匹配规则
InputIterator find_first_of (InputIterator first1, InputIterator last1,
                             ForwardIterator first2, ForwardIterator last2);
//以 pred 作为匹配规则
InputIterator find_first_of (InputIterator first1, InputIterator last1,
                             ForwardIterator first2, ForwardIterator last2,
                             BinaryPredicate pred);
```

其中，各个参数的含义如下：
* first 1、last 1：都为输入迭代器，它们的组合 \[first 1, last 1\) 用于指定该函数要查找的范围；
* first 2、last 2：都为正向迭代器，它们的组合 \[first 2, last 2\) 用于指定要进行匹配的元素所在的范围；
* pred：可接收一个包含 2 个形参且返回值类型为 bool 的函数，该函数可以是普通函数（又称为二元谓词函数），也可以是函数对象。

find_first_of () 函数用于在 \[first 1, last 1\) 范围内查找和 \[first 2, last 2\) 中任何元素相匹配的第一个元素。如果匹配成功，该函数会返回一个指向该元素的输入迭代器；反之，则返回一个和 last 1 迭代器指向相同的输入迭代器。

值得一提的是，不同语法格式的匹配规则也是不同的：
* 第 1 种语法格式：逐个取 \[first 1, last 1\) 范围内的元素（假设为 A），和 \[first 2, last 2\) 中的每个元素（假设为 B）做 A == B 运算，如果成立则匹配成功；
* 第 2 种语法格式：逐个取 \[first 1, last 1\) 范围内的元素（假设为 A），和 \[first 2, last 2\) 中的每个元素（假设为 B）一起代入 pred (A, B) 谓词函数，如果函数返回 true 则匹配成功。

> 注意，当采用第一种语法格式时，如果 \[first 1, last 1\) 或者 \[first 2, last 2\) 范围内的元素类型为自定义的类对象或者结构体变量，此时应对 == 运算符进行重载，使其适用于当前场景。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::find_first_of
#include <vector>       // std::vector
using namespace std;
//自定义二元谓词函数，作为 find_first_of() 函数的匹配规则
bool mycomp(int c1, int c2) {
    return (c2 % c1 == 0);
}
//以函数对象的形式定义一个 find_first_of() 函数的匹配规则
class mycomp2 {
public:
    bool operator()(const int& c1, const int& c2) {
        return (c2 % c1 == 0);
    }
};
int main() {
    char url[] = "http://c.biancheng.net/stl/";
    char ch[] = "stl";
    //调用第一种语法格式，找到 url 中和 "stl" 任一字符相同的第一个字符
    char *it = find_first_of(url, url + 27, ch, ch + 4);

    if (it != url + 27) {
        cout << "*it = " << *it << '\n';
    }

    vector<int> myvector{ 5,7,3,9 };
    int inter[] = { 4,6,8 };
    //调用第二种语法格式，找到 myvector 容器中和 3、5、7 任一元素有 c2%c1=0 关系的第一个元素
    vector<int>::iterator iter = find_first_of(myvector.begin(), myvector.end(), inter, inter + 3, mycomp2());
    if (iter != myvector.end()) {
        cout << "*iter = " << *iter;
    }
    return 0;
}
```

程序执行结果为：
```
*it = t  
*iter = 3
```

此程序给读者演示了 find_first_of () 函数 2 种语法格式的用法：
- 其中第 20 行代码中 find_first_of () 函数发挥的功能是，在 url 字符数组中逐个查找和's'、't'、'l' 这 3 个字符相等的字符，显然 url 数组第 2 个字符't' 就符合此规则。
- 在第 29 行代码中，find_first_of () 会逐个提取 myvector 容器中的每个元素（假设为 A），并尝试和 inter 数组中的每个元素（假设为 B）一起带入 mycomp 2 (A, B) 函数对象中。显然，当将 myvector 容器中的元素 3 和 inter 数组中的元素 6 带入该函数时，c2 % c1=0 表达式第一次成立。

C++ STL 标准库给出了 find_first_of () 函数底层实现的参考代码，感兴趣的读者可自行分析：
```
template<class InputIt, class ForwardIt, class BinaryPredicate>
InputIt find_first_of(InputIt first, InputIt last,
                      ForwardIt s_first, ForwardIt s_last,
                      BinaryPredicate p)
{
    for (; first != last; ++first) {
        for (ForwardIt it = s_first; it != s_last; ++it) {
            //第二种语法格式换成 if (p(*first, *it))
            if (p(*first, *it)) {
                return first;
            }
        }
    }
    return last;
}
```

## adjacent_find ()

adjacent_find () 函数用于在指定范围内查找 2 个连续相等的元素。该函数的语法格式为：
```
//查找 2 个连续相等的元素
ForwardIterator adjacent_find (ForwardIterator first, ForwardIterator last);
//查找 2 个连续满足 pred 规则的元素
ForwardIterator adjacent_find (ForwardIterator first, ForwardIterator last,
                               BinaryPredicate pred);
```
其中，first 和 last 都为正向迭代器，其组合 \[first, last\) 用于指定该函数的查找范围；pred 用于接收一个包含 2 个参数且返回值类型为 bool 的函数，以实现自定义查找规则。

另外，该函数会返回一个正向迭代器，当函数查找成功时，该迭代器指向的是连续相等元素的第 1 个元素；而如果查找失败，该迭代器的指向和 last 迭代器相同。

值得一提的是，adjacent_find () 函数定义于 `<algorithm>` 头文件中，因此使用该函数之前，程序中要先引入此头文件。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::adjacent_find
#include <vector>       // std::vector
using namespace std;
//以创建普通函数的形式定义一个查找规则
bool mycomp1(int i, int j) {
    return (i == j);
}
//以函数对象的形式定义一个查找规则
class mycomp2{
public:
    bool operator()(const int& _Left, const int& _Right){
        return (_Left == _Right);
    }
};
int main() {
    std::vector<int> myvector{ 5,20,5,30,30,20,10,10,20 };
    //调用第一种语法格式
    std::vector<int>::iterator it = adjacent_find(myvector.begin(), myvector.end());

    if (it != myvector.end()) {
        cout << "one : " << *it << '\n';
    }
    //调用第二种格式，也可以使用 mycomp1
    it = adjacent_find(++it, myvector.end(), mycomp2());

    if (it != myvector.end()) {
        cout << "two : " << *it;
    }
    return 0;
}
```

程序执行结果为：
```
one : 30  
two : 10
```

可以看到，程序中调用了 2 次 adjacent_find () 函数：
* 第 19 行：使用该函数的第一种语法格式，查找整个 myvector 容器中首个连续 2 个相等的元素，显然最先找到的是 30；
* 第 25 行：使用该函数的第二种语法格式，查找 {30,20,10,10,20} 部分中是否有连续 2 个符合 mycomp 2 规则的元素。不过，程序中自定义的 mycomp 1 或 mycomp 2 查找规则也是查找 2 个连续相等的元素，因此最先找到的是元素 10。

> 注意，对于第一种语法格式的 adjacent_find () 函数，其底层使用的是 == 运算符来判断连续 2 个元素是否相等。这意味着，如果指定区域内的元素类型为自定义的类对象或者结构体变量时，需要先对 == 运算符进行重载，然后才能使用此函数。

C++ STL 标准库官方给出了 adjacent_find () 函数底层实现的参考代码，感兴趣的读者可自行分析，这里不再做过多描述：
```
template <class ForwardIterator>
ForwardIterator adjacent_find (ForwardIterator first, ForwardIterator last)
{
    if (first != last)
    {
        ForwardIterator next=first; ++next;
        while (next != last) {
            if (*first == *next)     // 或者 if (pred(*first,*next)), 对应第二种语法格式
                return first;
        ++first; ++next;
        }
    }
    return last;
}
```