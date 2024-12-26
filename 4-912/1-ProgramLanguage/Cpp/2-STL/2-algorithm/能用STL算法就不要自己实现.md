
前面章节已经介绍了很多算法函数，比如 find ()、merge ()、sort () 等。不知读者有没有发现，每个算法函数都至少要用一对迭代器来指明作用区间，并且为了实现自己的功能，每个函数内部都势必会对指定区域内的数据进行遍历操作。

举几个例子，find () 函数会对指定区域的数据逐个进行遍历，确认其是否为要查找的目标元素；merge () 函数内部也会分别对 2 个有序序列做逐个遍历，从而将它们合并为一个有序序列；sort () 函数在对指定区域内的元素进行排序时，其底层也会遍历每个元素。

事实上，虽然这些算法函数的内部实现我们不得而知，但无疑都会用到循环结构。可以这么说，STL 标准库中几乎所有的算法函数，其底层都是借助循环结构实现的。

在此基础上，由于 STL 标准库使用场景很广，因此很多需要手动编写循环结构实现的功能，用 STL 算法函数就能完成。举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::for_each
#include <string>       // std::string
#include <vector>       // std::vector
#include <functional>
using namespace std;

class Address {
public:
    Address(string url) :url(url) {};
    void display() {
        cout << "url:" << this->url << endl;
    }
private:
    string url;
};

int main() {
    vector<Address>adds{ Address("http://www.cdsy.xyz/computer/programme/stl/"),
                         Address("http://www.cdsy.xyz/computer/programme/java/"),
                         Address("http://www.cdsy.xyz/computer/programme/Python/") };
    //手动编写循环结构
    cout << "first：\n";
    for (auto it = adds.begin(); it != adds.end(); ++it) {
        (*it).display();
    }
    //调用 STL 标准库中的算法函数
    cout << "second：\n";
    for_each(adds.begin(), adds.end(), mem_fun_ref(&Address::display));
    return 0;
}
```

程序执行结果为：
```
first：  
url: http://www.cdsy.xyz/computer/programme/stl/  
url: http://www.cdsy.xyz/computer/programme/java/  
url: http://www.cdsy.xyz/computer/programme/Python/  
second：  
url: http://www.cdsy.xyz/computer/programme/stl/  
url: http://www.cdsy.xyz/computer/programme/java/  
url: http://www.cdsy.xyz/computer/programme/Python/
```

可以看到，对于输出 adds 容器中存储的元素，除了可以手动编写循环结构实现，还可以使用 STL 标准库提供的 for_each () 函数。

那么，手动编写循环结构和调用 STL 算法函数相比，哪种实现方式更好呢？毫无疑问，**直接调用算法会更好**，理由有以下几个：

1. 算法函数通常比自己写的循环结构效率更高；
2. 自己写循环比使用算法函数更容易出错；
3. 相比自己编写循环结构，直接调用算法函数的代码更加简洁明了。
4. 使用算法函数编写的程序，可扩展性更强，更容易维护；

后面 3 个理由相信读者很容易理解，接下来重点讲一下 “为什么算法函数的效率更高”。

## 为什么 STL 算法效率更高
--------------

仍以上面程序为例，如下是我们手动编写的循环代码：
```
for (auto it = adds.begin(); it != adds.end(); ++it) {
    (*it).display();
}
```

此段代码中，每一次循环都要执行一次 end () 方法，事实上该方法并不需要多次调用，因为它的值自始至终都没有发生改变。也就是说，end () 方法只需要调用一次足矣，for_each () 函数就对这一点进行了优化：
```
for_each(adds.begin(), adds.end(), mem_fun_ref(&Address::display));
```

可以看到，通过将 end () 方法作为参数传入 for_each () 函数，该方法只执行了 1 次。当然，这也仅是众多优化中的一处。事实上，STL 标准库的开发者对每个算法函数的底层实现代码都多了优化，使它们的执行效率达到最高。

有读者可能会说，难道我们自己对循环结构进行优化不行吗？可以，但是其执行效率仍无法和算法函数相提并论。

一方面，STL 开发者可以根据他们对容器底层的了解，对整个遍历过程进行优化，而这是我们难以做到的。以 deque 容器为例，该容器底层会将数据存储在多个大小固定的连续空间中。对于这些连续空间的遍历，只有 STL 开发者才知道这些连续空间的大小，才知道如何控制指针逐个遍历这些连续空间。

另一方面，某些 STL 函数的底层实现使用了复杂的科学计算方法，并不是普通 C++ 程序员能驾驭的。例如，在实现对某个序列进行排序时，我们很难编写出比 sort () 函数更高效的代码。

> 总之，STL 开发者比使用者更了解内部的实现细节，他们会充分利用这些知识来对算法进行优化。

当然，只有熟悉 STL 标准库提供的函数，才能在实际编程时想到使用它们。作为一个专业的 C++ 程序员，我们必须熟悉 STL 标准库中的每个算法函数，并清楚它们各自的功能。

> C++ STL 标准库中包含 70 多个算法函数，如果考虑到函数的重载，大约有 100 多个不同的函数模板。本章仅介绍一些常用的算法函数，如果想了解全部的 STL 算法，读者可参考 [C++ STL 标准库官网](http://www.cplusplus.com/reference/algorithm/)。