
插入迭代器适配器（insert_iterator），简称插入迭代器或者插入器，其功能就是向指定容器中插入元素。值得一提的是，根据插入位置的不同，C++ STL 标准库提供了 3 种插入迭代器，如表所示。

| 迭代器适配器                | 功能                                                                                 |
|-----------------------|------------------------------------------------------------------------------------|
| back_insert_iterator  | 在指定容器的尾部插入新元素，但前提必须是提供有 push_back () 成员方法的容器（包括&nbsp; vector、deque 和 list）。        |
| front_insert_iterator | 在指定容器的头部插入新元素，但前提必须是提供有 push_front () 成员方法的容器（包括&nbsp; list、deque 和 forward_list）。 |
| insert_iterator       | 在容器的指定位置之前插入新元素，前提是该容器必须提供有 insert () 成员方法。                                        |

接下来，将逐个对表中这 3 种插入迭代器的用法做详细的讲解。

## back_insert_iterator 迭代器
--------------------------------

back_insert_iterator 迭代器可用于在指定容器的末尾处添加新元素。

需要注意的是，由于此类型迭代器的底层实现需要调用指定容器的 push_back () 成员方法，这就意味着，此类型迭代器并不适用于 STL 标准库中所有的容器，它只适用于包含 push_back () 成员方法的容器。

> C++ STL 标准库中，提供有 push_back () 成员方法的容器包括 vector、deque 和 list。

另外，back_insert_iterator 迭代器定义在 `<iterator>` 头文件，并位于 std 命名空间中，因此在使用该类型迭代器之前，程序应包含如下语句：

```
#include <iterator>
using namespace std;
```

和反向迭代器不同，back_insert_iterator 插入迭代器的定义方式仅有一种，其语法格式如下：
```
std::back_insert_iterator<Container> back_it (container);
```

其中，Container 用于指定插入的目标容器的类型；container 用于指定具体的目标容器。

举个例子：
```
//创建一个 vector 容器
std::vector<int> foo;
//创建一个可向 foo 容器尾部添加新元素的迭代器
std::back_insert_iterator< std::vector<int> > back_it (foo);
```

在此基础上，back_insert_iterator 迭代器模板类中还对赋值运算符（=）进行了重载，借助此运算符，我们可以直接将新元素插入到目标容器的尾部。例如：
```
#include <iostream>
#include <iterator>
#include <vector>
using namespace std;
int main () {
    //创建一个 vector 容器
    std::vector<int> foo;
    //创建一个可向 foo 容器尾部添加新元素的迭代器
    std::back_insert_iterator< std::vector<int> > back_it (foo);
    //将 5 插入到 foo 的末尾
    back_it = 5;
    //将 4 插入到当前 foo 的末尾
    back_it = 4;
    //将 3 插入到当前 foo 的末尾
    back_it = 3;
    //将 6 插入到当前 foo 的末尾
    back_it = 6;
    //输出 foo 容器中的元素
    for (std::vector<int>:: iterator it = foo.begin (); it != foo.end (); ++it)
        std:: cout << *it << ' ';
    return 0;
}
```

程序执行结果为：
```
5 4 3 6
```

通过借助赋值运算符，我们依次将 5、4、3、6 插入到 foo 容器中的末尾。这里需要明确的是，每次插入新元素时，该元素都会插入到当前 foo 容器的末尾。换句话说，程序中 11-17 行的每个赋值语句，都可以分解为以下这 2 行代码：
```
//pos 表示指向容器尾部的迭代器，value 表示要插入的元素
pos = foo.insert (pos, value);
++pos;
```

可以看到，每次将新元素插入到容器的末尾后，原本指向容器末尾的迭代器会后移一位，重新指向容器新的末尾。

除此之外，C++ STL 标准库为了方便用户创建 back_insert_iterator 类型的插入迭代器，提供了 back_inserter () 函数，其语法格式如下：
```
template <class Container>  
    back_insert_iterator<Container> back_inserter (Container& x);
```
其中，Container 表示目标容器的类型。

显然在使用该函数时，只需要为其传递一个具体的容器（vector、deque 或者 list）做参数，此函数即可返回一个 back_insert_iterator 类型的插入迭代器。因此，上面程序中的第 9 行代码，可替换成如下语句：
```
std::back_insert_iterator< std::vector<int> > back_it = back_inserter (foo);
```
通过接收 back_inserter () 的返回值，我们也可以得到完全一样的 back_it 插入迭代器。

> 有关此类型迭代器的底层实现，[C++ STL back_insert_iterator 官方手册](http://www32.cplusplus.com/reference/iterator/back_insert_iterator/)给出了具体的实现代码，有兴趣的读者可自行前往查看。

## front_insert_iterator 迭代器
---------------------------------

和 back_insert_iterator 正好相反，front_insert_iterator 迭代器的功能是向目标容器的头部插入新元素。

并且，由于此类型迭代器的底层实现需要借助目标容器的 push_front () 成员方法，这意味着，只有包含 push_front () 成员方法的容器才能使用该类型迭代器。

> C++ STL 标准库中，提供有 push_front () 成员方法的容器，仅有 **deque**、**list** 和 **forward_list**。

另外，front_insert_iterator 迭代器定义在 `<iterator>` 头文件，并位于 std 命名空间中，因此在使用该类型迭代器之前，程序应包含如下语句：

```
#include <iterator>
using namespace std;
```

值得一提的是，定义 front_insert_iterator 迭代器的方式和 back_insert_iterator 完全相同，并且 C++ STL 标准库也提供了 front_inserter () 函数来快速创建该类型迭代器。

举个例子：
```
#include <iostream>
#include <iterator>
#include <forward_list>
using namespace std;
int main () {
    //创建一个 forward_list 容器
    std::forward_list<int> foo;
    //创建一个前插入迭代器
    //std::front_insert_iterator< std::forward_list<int> > front_it (foo);
    std::front_insert_iterator< std::forward_list<int> > front_it = front_inserter (foo);
    //向 foo 容器的头部插入元素
    front_it = 5;
    front_it = 4;
    front_it = 3;
    front_it = 6;
    for (std::forward_list<int>:: iterator it = foo.begin (); it != foo.end (); ++it)
        std:: cout << *it << ' ';
    return 0;
}
```
程序执行结果为：
```
6 3 4 5
```
> 同样，[C++ STL back_insert_iterator 官方手册](http://www32.cplusplus.com/reference/iterator/back_insert_iterator/)也给出了此类型迭代器底层实现的参考代码，有兴趣的读者可自行前往查看。

## insert_iterator 迭代器
---------------------------

当需要向容器的任意位置插入元素时，就可以使用 insert_iterator 类型的迭代器。

需要说明的是，该类型迭代器的底层实现，需要调用目标容器的 insert () 成员方法。但幸运的是，STL 标准库中所有容器都提供有 insert () 成员方法，因此 insert_iterator 是**唯一可用于关联式容器**的插入迭代器。

和前 2 种插入迭代器一样，insert_iterator 迭代器也定义在 `<iterator>` 头文件，并位于 std 命名空间中，因此在使用该类型迭代器之前，程序应包含如下语句：

```
#include <iterator>
using namespace std;
```

不同之处在于，定义 insert_iterator 类型迭代器的语法格式如下：
```
std::insert_iterator<Container> insert_it (container, it);
```
其中，Container 表示目标容器的类型，参数 container 表示目标容器，而 it 是一个基础迭代器，表示新元素的插入位置。

和前 2 种插入迭代器相比，insert_iterator 迭代器除了定义时需要多传入一个参数，它们的用法完全相同。除此之外，C++ STL 标准库中还提供有 inserter () 函数，可以快速创建 insert_iterator 类型迭代器。

举个例子：
```
#include <iostream>
#include <iterator>
#include <list>
using namespace std;
int main () {
    //初始化为 {5,5}
    std::list<int> foo (2,5);
    //定义一个基础迭代器，用于指定要插入新元素的位置
    std::list<int>:: iterator it = ++foo.begin ();
    //创建一个 insert_iterator 迭代器
    //std::insert_iterator< std::list<int> > insert_it (foo, it);
    std::insert_iterator< std::list<int> > insert_it = inserter (foo, it);
    //向 foo 容器中插入元素
    insert_it = 1;
    insert_it = 2;
    insert_it = 3;
    insert_it = 4;
    //输出 foo 容器存储的元素
    for (std::list<int>:: iterator it = foo.begin (); it != foo.end (); ++it)
        std:: cout << *it << ' ';
    return 0;
}
```

程序执行结果为：
```
5 1 2 3 4 5
```

需要注意的是，如果 insert_iterator 迭代器的目标容器为关联式容器，由于该类型容器内部会自行对存储的元素进行排序，因此我们指定的插入位置只起到一个提示的作用，即帮助关联式容器从指定位置开始，搜索正确的插入位置。但是，如果提示位置不正确，会使的插入操作的效率更加糟糕。

> [C++ STL insert_iterator 官方手册](http://www32.cplusplus.com/reference/iterator/insert_iterator/)中给出了此类型迭代器底层实现的参考代码，有兴趣的读者可自行前往查看。

## 总结
本节讲解了 3 种插入迭代器，虽然它们都可以借助重载的赋值运算符，实现向目标容器的指定位置插入新元素，但在实际应用中，它们通常和 copy () 函数连用，即作为 copy () 函数的第 3 个参数。