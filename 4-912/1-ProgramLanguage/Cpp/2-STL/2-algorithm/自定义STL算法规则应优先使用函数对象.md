作为一门面向对象的编程语言，使用 C++ 编写程序有一个缺点，即随着代码面向对象程度的提高，其执行效率反而会降低。例如，经实验证明几乎在所有情况下，**直接操作一个 double 类型变量的执行效率，要比操作一个含 double 类型成员属性的类对象更高。**

对于大多数读者来说，以上所说是很容易想通的，因为它符合我们对高级编程语言的认知。但本节要介绍的内容，一定程序上会打破这个认知。

前面章节中，我们学习了 STL 标准库中所有的排序算法，比如 sort()、stable_sort() 以及 nth_element() 等。不知读者有没有发现，这些排序算法都单独提供了带有 comp 参数的语法格式，借助此参数，我们可以自定义排序规则。

以 sort() 排序函数为例，其语法格式有以下 2 种：
```
//无 comp 参数
void sort (RandomAccessIterator first, RandomAccessIterator last);
//有 comp 参数
void sort (RandomAccessIterator first, RandomAccessIterator last, Compare comp);
```

显然仅从使用语法上看，它们唯一的区别在于，第 2 种多了一个 comp 参数。
> 事实上，对于 STL 标准库中的每个算法，只要用户需要自定义规则，该算法都会提供有带 comp 参数的语法格式。

本质上讲，comp 参数用于接收用户自定义的函数，其定义的方式有 2 种，既可以是普通函数，也可以是函数对象。例如：
```cpp
#include <iostream>     // std::cout
#include <algorithm>    // std::sort
#include <vector>       // std::vector
//以普通函数的方式实现自定义排序规则
inline bool mycomp(int i, int j) {
    return (i < j);
}
//以函数对象的方式实现自定义排序规则
class mycomp2 {
public:
    bool operator() (int i, int j) {
        return (i < j);
    }
};

int main() {
    std::vector<int> myvector{ 32, 71, 12, 45, 26, 80, 53, 33 };
    //调用普通函数定义的排序规则
    std::sort(myvector.begin(), myvector.end(), mycomp);
    //调用函数对象定义的排序规则
    //std::sort(myvector.begin(), myvector.end(), mycomp2());
   
    for (std::vector<int>::iterator it = myvector.begin(); it != myvector.end(); ++it) {
        std::cout << *it << ' ';
    }
    return 0;
}
```

程序执行结果为：
```
12 26 32 33 45 53 71 80
```

> 注意，为了提高执行效率，其函数都定义为内联函数（在类内部定义的函数本身就是内联函数）。至于为什么内联函数比普通函数的执行效率高，可阅读《[C++ inline内联函数](http://www.cdsy.xyz/computer/programme/vc/20210105/cd16098262027540.html)》一文。

要知道，函数对象可以理解为伪装成函数的对象，根据以往的认知，函数对象的执行效率应该不如普通函数。但事实恰恰相反，即便如上面程序那样，将普通函数定义为更高效的内联函数，其执行效率也无法和函数对象相比。

> 通过在 4 个不同的 STL 平台上，对包含 100 万个 double 类型数据的 vector 容器进行排序，最差情况下使用函数对象的执行效率要比普通内联函数高 50%，最好情况下则高 160%。

那么，是什么原因导致了它们执行效率上的差异呢？以 mycomp2() 函数对象为例，其 mycomp2::operator() 也是一个内联函数，编译器在对 sort() 函数进行实例化时会将该函数直接展开，这也就意味着，展开后的 sort() 函数内部不包含任何函数调用。

而如果使用 mycomp 作为参数来调用 sort() 函数，情形则大不相同。要知道，**C++ 并不能真正地将一个函数作为参数传递给另一个函数**，换句话说，*如果我们试图将一个函数作为参数进行传递，编译器会隐式地将它转换成一个指向该函数的指针，并将该指针传递过去*。

也就是说，上面程序中的如下代码：
```
std::sort(myvector.begin(), myvector.end(), mycomp);
```

并不是真正地将 mycomp 传递给 sort() 函数，它传递的仅是一个指向 mycomp() 函数的指针。当 sort() 函数被实例化时，编译器生成的函数声明如下所示：
```
std::sort(vector<int>::iterator first,
          vector<int>::iterator last,
          bool (*comp)(int, int));
```

可以看到，参数 comp 只是一个指向函数的指针，所以 sort() 函数内部每次调用 comp 时，编译器都会通过指针产生一个间接的函数调用。

> 也正是基于这个原因，**C++ sort() 函数要比 C 语言 qsort() 函数的执行效率更高**。读者可能会问，程序中 comp() 函数也是内联函数，为什么 C++ 不像函数对象那样去处理呢？具体原因我们无从得知，事实上也没必要关心，也许是编译器开发者觉得这种优化不值得去做。

除了效率上的优势之外，相比普通函数，以函数对象的方式自定义规则还有很多隐藏的优势。例如在某些特殊情况下，以普通函数的形式编写的代码看似非常合理，但就是无法通过编译，这也许是由于 STL 标准库的原因，也许是编译器缺陷所至，甚至两者都有可能。而使用函数对象的方式，则可以有效避开这些“坑”，而且还大大提升的代码的执行效率。

总之，以函数对象的方式为 STL 算法自定义规则，具有效率在内的诸多优势。当调用带有 comp 参数的 STL 算法时，除非调用 STL 标准库自带的比较函数，否则应优先以函数对象的方式自定义规则。