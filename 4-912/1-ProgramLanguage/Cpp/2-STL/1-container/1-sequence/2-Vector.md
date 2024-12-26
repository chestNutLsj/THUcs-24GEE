---
banner: "![[vector.png]]"
banner_x: 0.5
---

## vector 介绍
vector 容器是 STL 中最常用的容器之一，它和 array 容器非常类似，都可以看做是对 C++ 普通数组的 “升级版”。不同之处在于，array 实现的是静态数组（容量固定的数组），而 vector 实现的是一个动态数组，即可以进行元素的插入和删除，在此过程中，vector 会动态调整所占用的内存空间，整个过程无需人工干预。

vector 常被称为向量容器，因为该容器擅长在尾部插入或删除元素，在常量时间内就可以完成，时间复杂度为 `O(1)`；而对于在容器头部或者中部插入或删除元素，则花费时间要长一些（移动元素需要耗费时间），时间复杂度为线性阶 `O(n)`。

vector 容器以类模板 vector\<T\>（ T 表示存储元素的类型）的形式定义在 \<vector\> 头文件中，并位于 std 命名空间中。因此，在创建该容器之前，代码中需包含如下内容：

```
#include <vector>
using namespace std;
```

> 注意，std 命名空间也可以在使用 vector 容器时额外注明，两种方式都可以。

## 创建 vector 容器的几种方式
-----------------

创建 vector 容器的方式有很多，大致可分为以下几种。

1) 如下代码展示了如何创建存储 double 类型元素的一个 vector 容器：

```
std::vector<double> values;
```

注意，这是一个空的 vector 容器，因为容器中没有元素，所以没有为其分配空间。当添加第一个元素（比如使用 push_back () 函数）时，vector 会自动分配内存。

在创建好空容器的基础上，还可以像下面这样通过调用 reserve () 成员函数来增加容器的容量：

```cpp
values.reserve(20);
```

这样就设置了容器的内存分配，即至少可以容纳 20 个元素。注意，如果 vector 的容量在执行此语句之前，已经大于或等于 20 个元素，那么这条语句什么也不做；另外，调用 reserve () 不会影响已存储的元素，也不会生成任何元素，即 values 容器内此时仍然没有任何元素。

> 还需注意的是，如果调用 reserve () 来增加容器容量，之前创建好的任何迭代器（例如开始迭代器和结束迭代器）都可能会失效，这是因为，为了增加容器的容量，vector\<T\> 容器的元素可能已经被复制或移到了新的内存地址。所以后续再使用这些迭代器时，最好重新生成一下。

2) 除了创建空 vector 容器外，还可以在创建的同时指定初始值以及元素个数，比如：

```
std::vector<int> primes {2, 3, 5, 7, 11, 13, 17, 19};
```

这样就创建了一个含有 8 个素数的 vector 容器。

3) 在创建 vector 容器时，也可以指定元素个数：

```
std::vector<double> values (20);
```

如此，values 容器开始时就有 20 个元素，它们的默认初始值都为 0。

> 注意，圆括号 () 和大括号 {} 是有区别的，前者（例如 (20) ）表示元素的个数，而后者（例如 {20} ） 则表示 vector 容器中只有一个元素 20。

如果不想用 0 作为默认值，也可以指定一个其它值，例如：

```
std::vector<double> values (20, 1.0);
```

第二个参数指定了所有元素的初始值，因此这 20 个元素的值都是 1.0。

值得一提的是，圆括号 () 中的 2 个参数，既可以是常量，也可以用变量来表示，例如：

```
int num=20;
double value =1.0;
std::vector<double> values (num, value);
```

4) 通过存储元素类型相同的其它 vector 容器，也可以创建新的 vector 容器，例如：

```
std::vector<char>value 1 (5, 'c');
std::vector<char>value 2 (value 1);
```

由此，value 2 容器中也具有 5 个字符'c'。在此基础上，如果不想复制其它容器中所有的元素，可以用一对[指针](http://c.biancheng.net/c/80/)或者迭代器来指定初始值的范围，例如：

```
int array[]={1,2,3};
std::vector<int>values (array, array+2);//values 将保存{1,2}
std::vector<int>value 1{1,2,3,4,5};
std::vector<int>value 2 (std:: begin (value 1), std:: begin (value 1)+3);//value 2 保存{1,2,3}
```

由此，value 2 容器中就包含了 {1,2,3} 这 3 个元素。

## vector 容器包含的成员函数
----------------

相比 array 容器，vector 提供了更多了成员函数供我们使用，它们各自的功能如下表所示。

| 函数成员              | 函数功能                                                      |
|-------------------|-----------------------------------------------------------|
| begin ()          | 返回指向容器中第一个元素的迭代器。                                         |
| end ()            | 返回指向容器最后一个元素所在位置后一个位置的迭代器，通常和 begin () 结合使用。              |
| rbegin ()         | 返回指向最后一个元素的迭代器。                                           |
| rend ()           | 返回指向第一个元素所在位置前一个位置的迭代器。                                   |
| cbegin ()         | 和&nbsp; begin () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改元素。     |
| cend ()           | 和 end () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改元素。             |
| crbegin ()        | 和 rbegin () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改元素。          |
| crend ()          | 和 rend () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改元素。            |
| size ()           | 返回实际元素个数。                                                 |
| max_size ()       | 返回元素个数的最大值。这通常是一个很大的值，一般是 232-1，所以我们很少会用到这个函数。            |
| resize ()         | 改变实际元素的个数。                                                |
| capacity ()       | 返回当前容量。                                                   |
| empty ()          | 判断容器中是否有元素，若无元素，则返回 true；反之，返回 false。                     |
| reserve ()        | 增加容器的容量。                                                  |
| shrink \_to\_fit () | 将内存减少到等于当前元素实际所使用的大小。                                     |
| operator[ ]       | 重载了&nbsp;[ ] 运算符，可以向访问数组中元素那样，通过下标即可访问甚至修改 vector 容器中的元素。 |
| at ()             | 使用经过边界检查的索引访问元素。                                          |
| front ()          | 返回第一个元素的引用。                                               |
| back ()           | 返回最后一个元素的引用。                                              |
| data ()           | 返回指向容器中第一个元素的指针。                                          |
| assign ()         | 用新元素替换原有内容。                                               |
| push_back ()      | 在序列的尾部添加一个元素。                                             |
| pop_back ()       | 移出序列尾部的元素。                                                |
| insert ()         | 在指定的位置插入一个或多个元素。                                          |
| erase ()          | 移出一个元素或一段元素。                                              |
| clear ()          | 移出所有的元素，容器大小变为 0。                                         |
| swap ()           | 交换两个容器的所有元素。                                              |
| emplace ()        | 在指定的位置直接生成一个元素。                                           |
| emplace_back ()   | 在序列尾部生成一个元素。                                              |

vector 容器还有一个 std:: swap (x , y) 非成员函数（其中 x 和 y 是存储相同类型元素的  vector 容器），它和 swap () 成员函数的功能完全相同，仅使用语法上有差异。

如下代码演示了表 1 中部分成员函数的用法：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    //初始化一个空vector容量
    vector<char>value;
    //向value容器中的尾部依次添加 S、T、L 字符
    value.push_back('S');
    value.push_back('T');
    value.push_back('L');
    //调用 size() 成员函数容器中的元素个数
    printf("元素个数为：%d\n", value.size());
    //使用迭代器遍历容器
    for (auto i = value.begin(); i < value.end(); i++) {
        cout << *i << " ";
    }
    cout << endl;
    //向容器开头插入字符
    value.insert(value.begin(), 'C');
    cout << "首个元素为：" << value.at(0) << endl;
    return 0;
}
```

输出结果为：
```
元素个数为：3  
S T L  
首个元素为：C
```

## vector 的迭代器
vector 容器的迭代器也是随机访问迭代器，并且 vector 模板类提供的操作迭代器的成员函数也和 array 容器一样

| 成员函数       | 功能                                                                            |
|------------|-------------------------------------------------------------------------------|
| begin ()   | 返回指向容器中第一个元素的正向迭代器；如果是 const 类型容器，在该函数返回的是常量正向迭代器。                            |
| end ()     | 返回指向容器最后一个元素之后一个位置的正向迭代器；如果是 const 类型容器，在该函数返回的是常量正向迭代器。此函数通常和 begin () 搭配使用。 |
| rbegin ()  | 返回指向最后一个元素的反向迭代器；如果是 const 类型容器，在该函数返回的是常量反向迭代器。                              |
| rend ()    | 返回指向第一个元素之前一个位置的反向迭代器。如果是 const 类型容器，在该函数返回的是常量反向迭代器。此函数通常和 rbegin () 搭配使用。   |
| cbegin ()  | 和 begin () 功能类似，只不过其返回的迭代器类型为常量正向迭代器，不能用于修改元素。                                |
| cend ()    | 和 end () 功能相同，只不过其返回的迭代器类型为常量正向迭代器，不能用于修改元素。                                  |
| crbegin () | 和 rbegin () 功能相同，只不过其返回的迭代器类型为常量反向迭代器，不能用于修改元素。                               |
| crend ()   | 和 rend () 功能相同，只不过其返回的迭代器类型为常量反向迭代器，不能用于修改元素。                                 |

![[vector-visit.png]]

> 值得一提的是，以上函数在实际使用时，其返回值类型都可以使用 auto 关键字代替，编译器可以自行判断出该迭代器的类型。

### vector 容器迭代器的基本用法
-----------------

vector 容器迭代器最常用的功能就是遍历访问容器中存储的元素。

首先来看 begin () 和 end () 成员函数，它们分别用于指向「首元素」和「尾元素 +1」 的位置，下面程序演示了如何使用 begin () 和 end () 遍历 vector 容器并输出其中的元素：

```
#include <iostream>
//需要引入 vector 头文件
#include <vector>
using namespace std;
int main()
{
    vector<int>values{1,2,3,4,5};
    auto first = values.begin();
    auto end = values.end();
    while (first != end)
    {
        cout << *first << " ";
        ++first;
    }
    return 0;
}
```

输出结果为：
```
1 2 3 4 5
```

可以看到，迭代器对象是由 vector 对象的成员函数 begin () 和 end () 返回的。我们可以像使用普通指针那样上使用它们。比如代码中，在保存了元素值后，使用前缀 `++` 运算符对 first 进行自增，当 first 等于 end 时，所有的元素都被设完值，循环结束。

与此同时，还可以使用全局的 begin () 和 end () 函数来从容器中获取迭代器，比如将上面代码中第 8、9 行代码用如下代码替换：

```
auto first = std::begin(values);
auto end = std::end (values);
```

cbegin ()/cend () 成员函数和 begin ()/end () 唯一不同的是，前者返回的是 const 类型的正向迭代器，这就意味着，由 cbegin () 和 cend () 成员函数返回的迭代器，可以用来遍历容器内的元素，也可以访问元素，但是不能对所存储的元素进行修改。

举个例子：

```
#include <iostream>
//需要引入 vector 头文件
#include <vector>
using namespace std;
int main()
{
    vector<int>values{1,2,3,4,5};
    auto first = values.cbegin();
    auto end = values.cend();
    while (first != end)
    {
        //*first = 10;不能修改元素
        cout << *first << " ";
        ++first;
    }
    return 0;
}
```

程序第 12 行，由于 first 是 const 类型的迭代器，因此不能用于修改容器中元素的值。

vector 模板类中还提供了 rbegin () 和 rend () 成员函数，分别表示指向最后一个元素和第一个元素前一个位置的随机访问迭代器，又称它们为反向迭代器。

> 需要注意的是，在使用反向迭代器进行 ++ 或 -- 运算时，++ 指的是迭代器向左移动一位，-- 指的是迭代器向右移动一位，即这两个运算符的功能也 “互换” 了。

反向迭代器用于以逆序的方式遍历容器中的元素。例如：

```
#include <iostream>
//需要引入 vector 头文件
#include <vector>
using namespace std;
int main()
{
    vector<int>values{1,2,3,4,5};
    auto first = values.rbegin();
    auto end = values.rend();
    while (first != end)
    {
        cout << *first << " ";
        ++first;
    }
    return 0;
}
```

运行结果为：
```
5 4 3 2 1
```
可以看到，从最后一个元素开始循环，遍历输出了容器中的所有元素。结束迭代器指向第一个元素之前的位置，所以当 first 指向第一个元素并 +1 后，循环就结朿了。

当然，在上面程序中，我们也可以使用 [for 循环](http://c.biancheng.net/view/172.html)：

```
for (auto first = values.rbegin(); first != values.rend(); ++first) {
    cout << *first << " ";
}
```

crbegin ()/crend () 组合和 rbegin ()/crend () 组合唯一的区别在于，前者返回的迭代器为 const 类型，即不能用来修改容器中的元素，除此之外在使用上和后者完全相同。

### vector 容器迭代器的独特之处
-----------------

和 array 容器不同，vector 容器可以随着存储元素的增加，自行申请更多的存储空间。因此，在创建 vector 对象时，我们可以直接创建一个空的 vector 容器，并不会影响后续使用该容器。

但这会产生一个问题，即在初始化空的 vector 容器时，不能使用迭代器。也就是说，如下初始化 vector 容器的方法是不行的：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int>values;
    int val = 1;
    for (auto first = values.begin(); first < values.end(); ++first, val++) {
        *first = val;
        //初始化的同时输出值
        cout << *first;
    }
    return 0;
}
```

运行程序可以看到，什么也没有输出。这是因为，对于空的 vector 容器来说，begin () 和 end () 成员函数返回的迭代器是相等的，即它们指向的是同一个位置。

> 对于空的 vector 容器来说，可以通过调用 push_back () 或者借助 resize () 成员函数实现初始化容器的目的。

除此之外，vector 容器在申请更多内存的同时，容器中的所有元素可能会被复制或移动到新的内存地址，这会导致之前创建的迭代器失效。

举个例子：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int>values{1,2,3};
    cout << "values 容器首个元素的地址：" << values.data() << endl;
    auto first = values.begin();
    auto end = values.end();
    //增加 values 的容量
    values.reserve(20);
    cout << "values 容器首个元素的地址：" << values.data() << endl;
    while (first != end) {
        cout << *first;
        ++first;
    }
    return 0;
}
```

运行程序，显示如下信息并崩溃：
```
values 容器首个元素的地址：0096 DFE 8  
values 容器首个元素的地址：00965560
```
可以看到，values 容器在增加容量之后，首个元素的存储地址发生了改变，此时再使用先前创建的迭代器，显然是错误的。因此，为了保险起见，**每当 vector 容器的容量发生变化时，我们都要对之前创建的迭代器重新初始化一遍**：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int>values{1,2,3};
    cout << "values 容器首个元素的地址：" << values.data() << endl;
    auto first = values.begin();
    auto end = values.end();
    //增加 values 的容量
    values.reserve(20);
    cout << "values 容器首个元素的地址：" << values.data() << endl;
    first = values.begin();
    end = values.end();
    while (first != end) {
        cout << *first ;
        ++first;
    }
    return 0;
}
```

运行结果为：
```
values 容器首个元素的地址：0164 DBE 8  
values 容器首个元素的地址：01645560  
123
```

## vector 访问元素

### 访问 vector 容器中单个元素
-----------------

首先，vector 容器可以向普通数组那样访问存储的元素，甚至对指定下标处的元素进行修改，比如：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int> values{1,2,3,4,5};
    //获取容器中首个元素
    cout << values[0] << endl;
    //修改容器中下标为 0 的元素的值
    values[0] = values[1] + values[2] + values[3] + values[4];
    cout << values[0] << endl;
    return 0;
}
```

运行结果为：
```
1  
14
```

> 显然，vector 的索引从 0 开始，这和普通数组一样。通过使用索引，总是可以访问到 vector 容器中现有的元素。

值得一提的是，`容器名[n]` 这种获取元素的方式，需要确保下标 n 的值不会超过容器的容量（可以通过 capacity () 成员函数获取），否则会发生越界访问的错误。幸运的是，和 array 容器一样，vector 容器也提供了 at () 成员函数，当传给 at () 的索引会造成越界时，会抛出 `std::out_of_range` 异常。

举个例子：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int> values{1,2,3,4,5};
    //获取容器中首个元素
    cout << values.at(0) << endl;
    //修改容器中下标为 0 的元素的值
    values.at(0) = values.at(1) + values.at(2) + values.at(3) + values.at(4);
    cout << values.at(0) << endl;
    //下面这条语句会发生 out_of_range 异常
    //cout << values.at(5) << endl;
    return 0;
}
```

运行结果为：
```
1  
14
```

除此之外，vector 容器还提供了 2 个成员函数，即 front () 和 back ()，它们分别返回 vector 容器中第一个和最后一个元素的引用，通过利用这 2 个函数返回的引用，可以访问（甚至修改）容器中的首尾元素。

举个例子：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int> values{1,2,3,4,5};
    cout << "values 首元素为：" << values.front() << endl;
    cout << "values 尾元素为：" << values.back() << endl;
    //修改首元素
    values.front() = 10;
    cout <<"values 新的首元素为：" << values.front() << endl;
    //修改尾元素
    values.back() = 20;
    cout << "values 新的尾元素为：" << values.back() << endl;
    return 0;
}
```

输出结果为：
```
values 首元素为：1  
values 尾元素为：5  
values 新的首元素为：10  
values 新的尾元素为：20
```
另外，vector 容器还提供了 data () 成员函数，该函数的功能是返回指向容器中首个元素的[指针](http://c.biancheng.net/c/80/)。通过该指针也可以访问以及修改容器中的元素。比如：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int> values{1,2,3,4,5};
    //输出容器中第 3 个元素的值
    cout << *(values.data() + 2) << endl;
    //修改容器中第 2 个元素的值
    *(values.data() + 1) = 10;
    cout << *(values.data() + 1) << endl;
    return 0;
}
```

运行结果为：
```
3  
10
```

### 访问 vector 容器中多个元素
-----------------

如果想访问 vector 容器中多个元素，可以借助 size () 成员函数，该函数可以返回 vector 容器中实际存储的元素个数。例如：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int> values{1,2,3,4,5};
    //从下标 0 一直遍历到 size()-1 处
    for (int i = 0; i < values.size(); i++) {
        cout << values[i] << " ";
    }
    return 0;
}
```

运行结果为：
```
1 2 3 4 5
```

注意，这里不要使用 capacity () 成员函数，因为它返回的是 vector 容器的容量，而不是实际存储元素的个数，这两者是有差别的。

或者也可以使用基于范围的循环，此方式将会逐个遍历容器中的元素。比如：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int> values{1,2,3,4,5};
    for (auto&& value : values)
        cout << value << " ";
    return 0;
}
```

运行结果为：
```
1 2 3 4 5
```

另外还可以使用 vector 迭代器遍历 vector 容器，这里以 begin ()/end () 为例：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int> values{1,2,3,4,5};
    for (auto first = values.begin(); first < values.end(); ++first) {
        cout << *first << " ";
    }
    return 0;
}
```

运行结果为：
```
1 2 3 4 5
```

## 添加元素

要知道，向 vector 容器中添加元素的唯一方式就是使用它的成员函数，如果不调用成员函数，非成员函数既不能添加也不能删除元素。这意味着，vector 容器对象必须通过它所允许的函数去访问，迭代器显然不行。

在前文已经列出了 vector 容器提供的所有成员函数，在这些成员函数中，可以用来给容器中添加元素的函数有 2 个，分别是 push_back () 和 emplace_back () 函数。

> 有读者可能认为还有 insert () 和 emplace () 成员函数，严格意义上讲，这 2 个成员函数的功能是向容器中的指定位置插入元素，后续章节会对它们做详细的介绍。

### push_back ()
-----------

该成员函数的功能是在 vector 容器尾部添加一个元素，用法也非常简单，比如：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int> values{};
    values.push_back(1);
    values.push_back(2);
    for (int i = 0; i < values.size(); i++) {
        cout << values[i] << " ";
    }
    return 0;
}
```

程序中，第 7 行代码表示向 values 容器尾部添加一个元素，但由于当前 values 容器是空的，因此新添加的元素 1 无疑成为了容器中首个元素；第 8 行代码实现的功能是在现有元素 1 的后面，添加元素 2。

运行程序，输出结果为：
```
1 2
```

### emplace_back ()
--------------

该函数是 [C++](http://c.biancheng.net/cplus/) 11 新增加的，其功能和 push_back () 相同，都是在 vector 容器的尾部添加一个元素。

emplace_back () 成员函数的用法也很简单，这里直接举个例子：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int> values{};
    values.emplace_back(1);
    values.emplace_back(2);
    for (int i = 0; i < values.size(); i++) {
        cout << values[i] << " ";
    }
    return 0;
}
```

运行结果为：
```
1 2
```
读者可能会发现，以上 2 段代码，只是用 emplace_back () 替换了 push_back ()，既然它们实现的功能是一样的，那么 C++ 11 标准中为什么要多此一举呢？

>[! note] emplace_back () 和 push_back () 的区别
>emplace_back () 和 push_back () 的区别，就在于底层实现的机制不同。
>- push_back () 向容器尾部添加元素时，首先会创建这个元素，然后再将这个元素拷贝或者移动到容器中（如果是拷贝的话，事后会自行销毁先前创建的这个元素）；
>- 而 emplace_back () 在实现时，则是直接在容器尾部创建这个元素，省去了拷贝或移动元素的过程。

为了让大家清楚的了解它们之间的区别，我们创建一个包含类对象的 vector 容器，如下所示：

```
#include <vector> 
#include <iostream> 
using namespace std;
class testDemo
{
public:
    testDemo(int num):num(num){
        std::cout << "调用构造函数" << endl;
    }
    testDemo(const testDemo& other) :num(other.num) {
        std::cout << "调用拷贝构造函数" << endl;
    }
    testDemo(testDemo&& other) :num(other.num) {
        std::cout << "调用移动构造函数" << endl;
    }
private:
    int num;
};

int main()
{
    cout << "emplace_back:" << endl;
    std::vector<testDemo> demo1;
    demo1.emplace_back(2);  

    cout << "push_back:" << endl;
    std::vector<testDemo> demo2;
    demo2.push_back(2);
}
```

运行结果为：
```
emplace_back:  
调用构造函数  
push_back:  
调用构造函数  
调用移动构造函数
```

在此基础上，读者可尝试将 testDemo 类中的移动构造函数注释掉，再运行程序会发现，运行结果变为：
```
emplace_back:  
调用构造函数  
push_back:  
调用构造函数  
调用拷贝构造函数
```

由此可以看出，push_back () 在底层实现时，会优先选择调用移动构造函数，如果没有才会调用拷贝构造函数。

显然完成同样的操作，push_back () 的底层实现过程比 emplace_back () 更繁琐，换句话说，emplace_back () 的执行效率比 push_back () 高。因此，在实际使用时，建议大家优先选用 emplace_back ()。

> 由于 emplace_back () 是 C++ 11 标准新增加的，如果程序要兼顾之前的版本，还是应该使用 push_back ()。

## 插入元素

vector 容器提供了 insert () 和 emplace () 这 2 个成员函数，用来实现在容器指定位置处插入元素，本节将对它们的用法做详细的讲解。

### insert ()
--------

insert () 函数的功能是在 vector 容器的指定位置插入一个或多个元素。该函数的语法格式有多种，如表 1 所示。

| 语法格式                                 | 用法说明                                                                                                                                          |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| iterator insert (pos, elem)              | 在迭代器 pos 指定的位置之前插入一个新元素 elem，并返回表示新插入元素位置的迭代器。                                                                |
| iterator insert (pos, n, elem)           | 在迭代器 pos 指定的位置之前插入 n 个元素 elem，并返回表示第一个新插入元素位置的迭代器。                                                           |
| iterator insert (pos, first, last)&nbsp; | 在迭代器 pos 指定的位置之前，插入其他容器（不仅限于 vector）中位于 \[first, last) 区域的所有元素，并返回表示第一个新插入元素位置的迭代器。         |
| iterator insert (pos, initlist)          | 在迭代器 pos 指定的位置之前，插入初始化列表（用大括号 {} 括起来的多个元素，中间有逗号隔开）中所有的元素，并返回表示第一个新插入元素位置的迭代器。 |

下面的例子，演示了如何使用 insert () 函数向 vector 容器中插入元素。

```
#include <iostream> 
#include <vector> 
#include <array> 
using namespace std;
int main()
{
    std::vector<int> demo{1,2};
    //第一种格式用法
    demo.insert(demo.begin() + 1, 3);//{1,3,2}

    //第二种格式用法
    demo.insert(demo.end(), 2, 5);//{1,3,2,5,5}

    //第三种格式用法
    std::array<int,3>test{ 7,8,9 };
    demo.insert(demo.end(), test.begin(), test.end());//{1,3,2,5,5,7,8,9}

    //第四种格式用法
    demo.insert(demo.end(), { 10,11 });//{1,3,2,5,5,7,8,9,10,11}

    for (int i = 0; i < demo.size(); i++) {
        cout << demo[i] << " ";
    }
    return 0;
}
```

运行结果为：
```
1 3 2 5 5 7 8 9 10 11
```

### emplace ()
---------

emplace () 是 [C++](http://c.biancheng.net/cplus/) 11 标准新增加的成员函数，用于在 vector 容器指定位置之前插入一个新的元素。

> 再次强调，emplace () 每次只能插入一个元素，而不是多个。

该函数的语法格式如下：
```
iterator emplace (const_iterator pos, args...);
```

其中，pos 为指定插入位置的迭代器；args... 表示与新插入元素的构造函数相对应的多个参数；该函数会返回表示新插入元素位置的迭代器。

> 简单的理解 args...，即被插入元素的构造函数需要多少个参数，那么在 emplace () 的第一个参数的后面，就需要传入相应数量的参数。

举个例子：

```
#include <vector>
#include <iostream>
using namespace std;

int main()
{
    std::vector<int> demo1{1,2};
    //emplace() 每次只能插入一个 int 类型元素
    demo1.emplace(demo1.begin(), 3);
    for (int i = 0; i < demo1.size(); i++) {
        cout << demo1[i] << " ";
    }
    return 0;
}
```

运行结果为：
```
3 1 2
```

既然 emplace () 和 insert () 都能完成向 vector 容器中插入新元素，那么谁的运行效率更高呢？答案是 emplace ()。在说明原因之前，通过下面这段程序，就可以直观看出两者运行效率的差异：

```
#include <vector>
#include <iostream>
using namespace std;
class testDemo
{
public:
    testDemo(int num) :num(num) {
        std::cout << "调用构造函数" << endl;
    }
    testDemo(const testDemo& other) :num(other.num) {
        std::cout << "调用拷贝构造函数" << endl;
    }
    testDemo(testDemo&& other) :num(other.num) {
        std::cout << "调用移动构造函数" << endl;
    }

    testDemo& operator=(const testDemo& other);
private:
    int num;
};
testDemo& testDemo::operator=(const testDemo& other) {
    this->num = other.num;
    return *this;
}
int main()
{
    cout << "insert:" << endl;
    std::vector<testDemo> demo2{};
    demo2.insert(demo2.begin(), testDemo(1));

    cout << "emplace:" << endl;
    std::vector<testDemo> demo1{};
    demo1.emplace(demo1.begin(), 1);
    return 0;
}
```

运行结果为：
```
insert:  
调用构造函数  
调用移动构造函数  
emplace:  
调用构造函数
```

> 注意，当拷贝构造函数和移动构造函数同时存在时，insert () 会优先调用移动构造函数。

可以看到，通过 insert () 函数向 vector 容器中插入 testDemo 类对象，需要调用类的构造函数和移动构造函数（或拷贝构造函数）；而通过 emplace () 函数实现同样的功能，只需要调用构造函数即可。

简单的理解，就是 emplace () 在插入元素时，是在容器的指定位置直接构造元素，而不是先单独生成，再将其复制（或移动）到容器中。因此，在实际使用中，推荐大家优先使用 emplace ()。

## 删除元素

前面提到，无论是向现有 vector 容器中访问元素、添加元素还是插入元素，都只能借助 vector 模板类提供的成员函数，但删除 vector 容器的元素例外，完成此操作除了可以借助本身提供的成员函数，还可以借助一些全局函数。

基于不同场景的需要，删除 vecotr 容器的元素，可以使用下表中所示的函数（或者函数组合）。

| 函数                     | 功能                                                                                                  |
|------------------------|-----------------------------------------------------------------------------------------------------|
| pop_back ()            | 删除 vector 容器中最后一个元素，该容器的大小（size）会减 1，但容量（capacity）不会发生改变。                                           |
| erase (pos)            | 删除 vector 容器中 pos 迭代器指定位置处的元素，并返回指向被删除元素下一个位置元素的迭代器。该容器的大小（size）会减 1，但容量（capacity）不会发生改变。           |
| swap (beg)、pop_back () | 先调用 swap () 函数交换要删除的目标元素和容器最后一个元素的位置，然后使用 pop_back () 删除该目标元素。                                      |
| erase (beg, end)       | 删除 vector 容器中位于迭代器 \[beg, end) 指定区域内的所有元素，并返回指向被删除区域下一个位置元素的迭代器。该容器的大小（size）会减小，但容量（capacity）不会发生改变。 |
| remove ()              | 删除容器中所有和指定元素值相等的元素，并返回指向最后一个元素下一个位置的迭代器。值得一提的是，调用该函数不会改变容器的大小和容量。                                   |
| clear ()               | 删除 vector 容器中所有的元素，使其变成空的 vector 容器。该函数会改变 vector 的大小（变为 0），但不是改变其容量。                               |

下面就表中罗列的这些函数，一一讲解它们的具体用法。
### pop_back
pop_back () 成员函数的用法非常简单，它不需要传入任何的参数，也没有返回值。举个例子：

```
#include <vector>
#include <iostream>
using namespace std;

int main()
{
    vector<int>demo{ 1,2,3,4,5 };
    demo.pop_back();
    //输出 dmeo 容器新的size
    cout << "size is :" << demo.size() << endl;
    //输出 demo 容器新的容量
    cout << "capacity is :" << demo.capacity() << endl;
    for (int i = 0; i < demo.size(); i++) {
        cout << demo[i] << " ";
    }
    return 0;
}
```

运行结果为：
```
size is : 4  
capacity is : 5  
1 2 3 4
```

可以发现，相比原 demo 容器，新的 demo 容器删除了最后一个元素 5，容器的大小减了 1，但容量没变。

### erase
如果想删除 vector 容器中指定位置处的元素，可以使用 erase () 成员函数，该函数的语法格式为：
```
iterator erase (pos);
```

其中，pos 为指定被删除元素位置的迭代器，同时该函数会返回一个指向删除元素所在位置下一个位置的迭代器。

下面的例子演示了 erase () 函数的具体用法：

```
#include <vector>
#include <iostream>
using namespace std;

int main()
{
    vector<int>demo{ 1,2,3,4,5 };
    auto iter = demo.erase(demo.begin() + 1);//删除元素 2
    //输出 dmeo 容器新的size
    cout << "size is :" << demo.size() << endl;
    //输出 demo 容器新的容量
    cout << "capacity is :" << demo.capacity() << endl;
    for (int i = 0; i < demo.size(); i++) {
        cout << demo[i] << " ";
    }
    //iter迭代器指向元素 3
    cout << endl << *iter << endl;
    return 0;
}
```

运行结果为：
```
size is : 4  
capacity is : 5  
1 3 4 5  
3
```

通过结果不能看出，erase () 函数在删除元素时，会将删除位置后续的元素陆续前移，并将容器的大小减 1。

### swap + pop_back
另外，如果不在意容器中元素的排列顺序，可以结合 swap () 和 pop_back () 函数，同样可以实现删除容器中指定位置元素的目的。

> 注意，swap () 函数在头文件 `<algorithm>` 和 `<utility>` 中都有定义，使用时引入其中一个即可。

例如：

```
#include <vector>
#include <iostream>
#include <algorithm>
using namespace std;

int main()
{
    vector<int>demo{ 1,2,3,4,5 };
    //交换要删除元素和最后一个元素的位置
    swap(*(std::begin(demo)+1),*(std::end(demo)-1));//等同于 swap(demo[1],demo[4])
   
    //交换位置后的demo容器
    for (int i = 0; i < demo.size(); i++) {
        cout << demo[i] << " ";
    }
    demo.pop_back();
    cout << endl << "size is :" << demo.size() << endl;
    cout << "capacity is :" << demo.capacity() << endl;
    //输出demo 容器中剩余的元素
    for (int i = 0; i < demo.size(); i++) {
        cout << demo[i] << " ";
    }
    return 0;
}
```

运行结果为：
```
1 5 3 4 2  
size is : 4  
capacity is : 5  
1 5 3 4
```

### erase (first, last)
当然，除了删除容器中单个元素，还可以删除容器中某个指定区域内的所有元素，同样可以使用 erase () 成员函数实现。该函数有 2 种基本格式，前面介绍了一种，这里使用另一种：
```
iterator erase (iterator first, iterator last);
```

其中 first 和 last 是指定被删除元素区域的迭代器，同时该函数会返回指向此区域之后一个位置的迭代器。

举个例子：

```
#include <vector>
#include <iostream>
using namespace std;

int main()
{
    std::vector<int> demo{ 1,2,3,4,5 };
    //删除 2、3
    auto iter = demo.erase(demo.begin()+1, demo.end() - 2);
    cout << "size is :" << demo.size() << endl;
    cout << "capacity is :" << demo.capacity() << endl;

    for (int i = 0; i < demo.size(); i++) {
        cout << demo[i] << " ";
    }
    return 0;
}
```

运行结果为：
```
size is : 3  
capacity is : 5  
1 4 5
```

可以看到，和删除单个元素一样，删除指定区域内的元素时，也会将该区域后续的元素前移，并缩小容器的大小。

### remove
如果要删除容器中和指定元素值相同的所有元素，可以使用 remove () 函数，该函数定义在 `<algorithm>` 头文件中。例如：

```
#include <vector>
#include <iostream>
#include <algorithm>
using namespace std;

int main()
{
    vector<int>demo{ 1,3,3,4,3,5 };
    auto iter = std::remove(demo.begin(), demo.end(), 3);

    cout << "size is :" << demo.size() << endl;
    cout << "capacity is :" << demo.capacity() << endl;
    //输出剩余的元素
    for (auto first = demo.begin(); first < iter;++first) {
        cout << *first << " ";
    }
    return 0;
}
```

运行结果为：
```
size is : 6  
capacity is : 6  
1 4 5
```

注意，在对容器执行完 remove () 函数之后，由于该函数并没有改变容器原来的大小和容量，因此无法使用之前的方法遍历容器，而是需要向程序中那样，借助 remove () 返回的迭代器完成正确的遍历。

> remove () 的实现原理是，在遍历容器中的元素时，一旦遇到目标元素，就做上标记，然后继续遍历，直到找到一个非目标元素，即用此元素将最先做标记的位置覆盖掉，同时将此非目标元素所在的位置也做上标记，等待找到新的非目标元素将其覆盖。因此，如果将上面程序中 demo 容器的元素全部输出，得到的结果为 `1 4 5 4 3 5`。

另外还可以看到，既然通过 remove () 函数删除掉 demo 容器中的多个指定元素，该容器的大小和容量都没有改变，其剩余位置还保留了之前存储的元素。我们可以使用 erase () 成员函数删掉这些 "无用" 的元素。

比如，修改上面的程序：

```
#include <vector>
#include <iostream>
#include <algorithm>
using namespace std;

int main()
{
    vector<int>demo{ 1,3,3,4,3,5 };
    auto iter = std::remove(demo.begin(), demo.end(), 3);
    demo.erase(iter, demo.end());
    cout << "size is :" << demo.size() << endl;
    cout << "capacity is :" << demo.capacity() << endl;
    //输出剩余的元素
    for (int i = 0; i < demo.size();i++) {
        cout << demo[i] << " ";
    }
    return 0;
}
```

运行结果为：
```
size is : 3  
capacity is : 6  
1 4 5
```

> remove () 用于删除容器中指定元素时，常和 erase () 成员函数搭配使用。

### clear
如果想删除容器中所有的元素，则可以使用 clear () 成员函数，例如：

```
#include <vector>
#include <iostream>
#include <algorithm>
using namespace std;

int main()
{
    vector<int>demo{ 1,3,3,4,3,5 };
    //交换要删除元素和最后一个元素的位置
    demo.clear();
    cout << "size is :" << demo.size() << endl;
    cout << "capacity is :" << demo.capacity() << endl;
    return 0;
}
```

运行结果为：
```
size is : 0  
capacity is : 6
```

## vector capacity 与 size 的区别

很多初学者分不清楚 vector 容器的容量（capacity）和大小（size）之间的区别，甚至有人认为它们表达的是一个意思。本节将对 vector 容量和大小各自的含义做一个详细的介绍。

vector 容器的容量（用 capacity 表示），指的是在不分配更多内存的情况下，容器可以保存的最多元素个数；而 vector 容器的大小（用 size 表示），指的是它实际所包含的元素个数。

对于一个 vector 对象来说，通过该模板类提供的 capacity () 成员函数，可以获得当前容器的容量；通过 size () 成员函数，可以获得容器当前的大小。例如：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    std::vector<int>value{ 2,3,5,7,11,13,17,19,23,29,31,37,41,43,47 };
    value.reserve(20);
    cout << "value 容量是：" << value.capacity() << endl;
    cout << "value 大小是：" << value.size() << endl;
    return 0;
}
```

程序输出结果为：
```
value 容量是：20  
value 大小是：15
```

结合该程序的输出结果，下图可以更好的说明 vector 容器容量和大小之间的关系。

![[vector-size-capacity.png]]

显然，vector 容器的大小不能超出它的容量，在大小等于容量的基础上，只要增加一个元素，就必须分配更多的内存。注意，这里的 “更多” 并不是 1 个。换句话说，当 vector 容器的大小和容量相等时，如果再向其添加（或者插入）一个元素，vector 往往会申请多个存储空间，而不仅仅只申请 1 个。

>一旦 vector 容器的内存被重新分配，则和 vector 容器中元素相关的所有引用、指针以及迭代器，都可能会失效，最稳妥的方法就是重新生成。

举个例子：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int>value{ 2,3,5,7,11,13,17,19,23,29,31,37,41,43,47 };
    cout << "value 容量是：" << value.capacity() << endl;
    cout << "value 大小是：" << value.size() << endl;
    printf("value首地址：%p\n", value.data());
    value.push_back(53);
    cout << "value 容量是(2)：" << value.capacity() << endl;
    cout << "value 大小是(2)：" << value.size() << endl;
    printf("value首地址： %p", value.data());
    return 0;
}
```

运行结果为：
```
value 容量是：15  
value 大小是：15  
value 首地址：01254 D 40  
value 容量是 (2)：22  
value 大小是 (2)：16  
value 首地址： 01254 E 80
```

可以看到，向 “已满” 的 vector 容器再添加一个元素，整个 value 容器的存储位置发生了改变，同时 vector 会一次性申请多个存储空间（具体多少，取决于底层算法的实现）。这样做的好处是，可以很大程度上减少 vector 申请空间的次数，当后续再添加元素时，就可以节省申请空间耗费的时间。

> 因此，对于 vector 容器而言，当增加新的元素时，有可能很快完成（即直接存在预留空间中）；也有可能会慢一些（扩容之后再放新元素）。

### 修改 vector 容器的容量和大小
------------------

另外，通过前面的学习我们知道，可以调用 reserve () 成员函数来增加容器的容量（但并不会改变存储元素的个数）；而通过调用成员函数 resize () 可以改变容器的大小，并且该函数也可能会导致 vector 容器容量的增加。比如说：

```
#include <iostream>
#include <vector>
using namespace std;
int main()
{
    vector<int>value{ 2,3,5,7,11,13,17,19,23,29,31,37,41,43,47 };
    cout << "value 容量是：" << value.capacity() << endl;
    cout << "value 大小是：" << value.size() << endl;
    value.reserve(20);
    cout << "value 容量是(2)：" << value.capacity() << endl;
    cout << "value 大小是(2)：" << value.size() << endl;
    //将元素个数改变为 21 个，所以会增加 6 个默认初始化的元素
    value.resize(21);
    //将元素个数改变为 21 个，新增加的 6 个元素默认值为 99。
    //value.resize(21,99);
    //当需要减小容器的大小时，会移除多余的元素。
    //value.resize(20);
    cout << "value 容量是(3)：" << value.capacity() << endl;
    cout << "value 大小是(3)：" << value.size() << endl;
    return 0;
}
```

运行结果为：
```
value 容量是：15  
value 大小是：15  
value 容量是 (2)：20  
value 大小是 (2)：15  
value 容量是 (3)：30  
value 大小是 (3)：21
```

> 程序中给出了关于 resize () 成员函数的 3 种不同的用法，有兴趣的读者可自行查看不同用法的运行结果。

可以看到，仅通过 reserve () 成员函数增加 value 容器的容量，其大小并没有改变；但通过 resize () 成员函数改变 value 容器的大小，它的容量可能会发生改变。另外需要注意的是，通过 resize () 成员函数减少容器的大小（多余的元素会直接被删除），不会影响容器的容量。

### vector 容器容量和大小的数据类型
-------------------

在实际场景中，我们可能需要将容器的容量和大小保存在变量中，要知道 vector\<T\> 对象的容量和大小类型都是 vector\<T\>:: size_type 类型。因此，当定义一个变量去保存这些值时，可以如下所示：

```
vector<int>:: size_type cap = value.capacity ();
vector<int>:: size_type size = value.size ();
```

size_type 类型是定义在由 vector 类模板生成的 vecotr 类中的，它表示的真实类型和操作系统有关，在 32 位架构下普遍表示的是 unsigned int 类型，而在 64 位架构下普通表示 unsigned long 类型。

当然，我们还可以使用 auto 关键字代替 vector\<int\>:: size_type，比如：

```
auto cap = value.capacity ();
auto size = value.size ();
```

## 底层实现原理

STL 众多容器中，vector 是最常用的容器之一，其底层所采用的数据结构非常简单，就只是一段连续的线性内存空间。

通过分析 vector 容器的源代码不难发现，它就是使用 3 个迭代器（可以理解成指针）来表示的：

```
//_Alloc 表示内存分配器，此参数几乎不需要我们关心
template <class _Ty, class _Alloc = allocator<_Ty>>
class vector{
    ...
protected:
    pointer _Myfirst;
    pointer _Mylast;
    pointer _Myend;
};
```

其中，`_Myfirst` 指向的是 vector 容器对象的起始字节位置；`_Mylast` 指向当前最后一个元素的末尾字节；`_Myend` 指向整个 vector 容器所占用内存空间的末尾字节。

![[vector-base-pointer.png]]

如上图所示，通过这 3 个迭代器，就可以表示出一个已容纳 2 个元素，容量为 5 的 vector 容器。

在此基础上，将 3 个迭代器两两结合，还可以表达不同的含义，例如：
* `_Myfirst` 和 `_Mylast` 可以用来表示 vector 容器中目前已被使用的内存空间；
* `_Mylast` 和 `_Myend` 可以用来表示 vector 容器目前空闲的内存空间；
* `_Myfirst` 和 `_Myend` 可以用表示 vector 容器的容量。

> 对于空的 vector 容器，由于没有任何元素的空间分配，因此 `_Myfirst`、`_Mylast` 和 `_Myend` 均为 null。

通过灵活运用这 3 个迭代器，vector 容器可以轻松的实现诸如首尾标识、大小、容器、空容器判断等几乎所有的功能，比如：

```
template <class _Ty, class _Alloc = allocator<_Ty>>
class vector{
public：
    iterator begin() {return _Myfirst;}
    iterator end() {return _Mylast;}
    size_type size() const {return size_type(end() - begin());}
    size_type capacity() const {return size_type(_Myend - begin());}
    bool empty() const {return begin() == end();}
    reference operator[] (size_type n) {return *(begin() + n);}
    reference front() { return *begin();}
    reference back() {return *(end()-1);}
    ...
};
```

### vector 扩大容量的本质
--------------

另外需要指明的是，当 vector 的大小和容量相等（`size==capacity`）也就是满载时，如果再向其添加元素，那么 vector 就需要扩容。vector 容器扩容的过程需要经历以下 3 步：
1. 完全弃用现有的内存空间，重新申请更大的内存空间；
2. 将旧内存空间中的数据，按原有顺序移动到新的内存空间中；
3. 最后将旧的内存空间释放。

> 这也就解释了，为什么 vector 容器在进行扩容后，与其相关的指针、引用以及迭代器可能会失效的原因。

由此可见，vector 扩容是非常耗时的。为了降低再次分配内存空间时的成本，每次扩容时 vector 都会申请比用户需求量更多的内存空间（这也就是 vector 容量的由来，即 capacity>=size），以便后期使用。

> vector 容器扩容时，不同的编译器申请更多内存空间的量是不同的。以 VS 为例，它会扩容现有容器容量的 50%。

## 如何避免 vector 进行不必要的扩容？

前面提到，我们可以将 vector 容器看做是一个动态数组。换句话说，在不超出 vector 最大容量限制（max_size () 成员方法的返回值）的前提下，该类型容器可以自行扩充容量来满足用户存储更多元素的需求。

值得一提的是，vector 容器扩容的整个过程，和 realloc () 函数的实现方法类似，大致分为以下 4 个步骤：

1. 分配一块大小是当前 vector 容量几倍的新存储空间。注意，多数 STL 版本中的 vector 容器，其容器都会以 2 的倍数增长，也就是说，每次 vector 容器扩容，它们的容量都会提高到之前的 2 倍；
2. 将 vector 容器存储的所有元素，依照原有次序从旧的存储空间复制到新的存储空间中；
3. 析构掉旧存储空间中存储的所有元素；
4. 释放旧的存储空间。

通过以上分析不难看出，vector 容器的扩容过程是非常耗时的，并且当容器进行扩容后，之前和该容器相关的所有指针、迭代器以及引用都会失效。因此在使用 vector 容器过程中，我们应尽量避免执行不必要的扩容操作。

要实现这个目标，可以借助 vector 模板类中提供的 reserve () 成员方法。不过在讲解如何用 reserve () 方法避免 vector 容器进行不必要的扩容操作之前，vector 模板类中还提供有几个和 reserve () 功能类似的成员方法，很容易混淆，这里有必要为读者梳理一下，如下表所示。

| 成员方法        | 功能                                                                                                                                                                                            |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| size ()     | 告诉我们当前 vector 容器中已经存有多少个元素，但仅通过此方法，无法得知 vector 容器有多少存储空间。                                                                                                                                     |
| capacity () | 告诉我们当前 vector 容器总共可以容纳多少个元素。如果想知道当前 vector 容器有多少未被使用的存储空间，可以通过 capacity ()-size () 得知。注意，如果 size () 和 capacity () 返回的值相同，则表明当前 vector 容器中没有可用存储空间了，这意味着，下一次向 vector 容器中添加新元素，将导致 vector 容器扩容。 |
| resize (n)  | 强制 vector 容器必须存储 n 个元素，注意，如果 n 比 size () 的返回值小，则容器尾部多出的元素将会被析构（删除）；如果 n 比 size () 大，则 vector 会借助默认构造函数创建出更多的默认值元素，并将它们存储到容器末尾；如果 n 比 capacity () 的返回值还要大，则 vector 会先扩增，在添加一些默认值元素。            |
| reserve (n) | 强制 vector 容器的容量至少为 n。注意，如果 n 比当前 vector 容器的容量小，则该方法什么也不会做；反之如果 n 比当前 vector 容器的容量大，则 vector 容器就会扩容。                                                                                           |

通过对以上几个成员方法功能的分析，我们可以总结出一点，即只要有新元素要添加到 vector 容器中而恰好此时 vector 容器的容量不足时，该容器就会自动扩容。

因此，避免 vector 容器执行不必要的扩容操作的关键在于，在使用 vector 容器初期，就要将其容量设为足够大的值。换句话说，在 vector 容器刚刚构造出来的那一刻，就应该借助 reserve () 成员方法为其扩充足够大的容量。

举个例子，假设我们想创建一个包含 1~1000 的 vector\<int\>，通常会这样实现：

```
vector<int>myvector;
for (int i = 1; i <= 1000; i++) {
    myvector. push_back (i);
}
```

值得一提的是，上面代码的整个循环过程中，vector 容器会进行 2~10 次自动扩容（多数的 STL 标准库版本中，vector 容器通常会扩容至当前容量的 2 倍，而这里 1000≈ $2^{10}$），程序的执行效率可想而知。

在上面程序的基础上，下面代码演示了如何使用 reserve () 成员方法尽量避免 vector 容器执行不必要的扩容操作：

```
vector<int>myvector;
myvector.reserve (1000);
cout << myvector.capacity ();
for (int i = 1; i <= 1000; i++) {
    myvector. push_back (i);
}
```

相比前面的代码实现，整段程序在运行过程中，vector 容器的容量仅扩充了 1 次，执行效率大大提高。

当然在实际场景中，我们可能并不知道 vector 容器到底要存储多少个元素。这种情况下，可以先预留出足够大的空间，当所有元素都存储到 vector 容器中之后，再去除多余的容量。

> 关于怎样去除 vector 容器多余的容量，可以借助该容器模板类提供的 shrink_to_fit () 成员方法，另外后续还会讲解如何使用 swap () 成员方法去除 vector 容器多余的容量，两种方法都可以。

## 去除 vector 多余容量

上一节中，遗留了一个问题，即如何借助 swap () 成员方法去除 vector 容器中多余的容量？本节将就此问题给读者做详细的讲解。

我们知道，在使用 vector 容器的过程中，其容器会根据需要自行扩增。比如，使用 push_back ()、insert ()、emplace () 等成员方法向 vector 容器中添加新元素时，如果当前容器已满（即 size () == capacity ()），则它会自行扩容以满足添加新元素的需求。当然，还可以调用 reserve () 成员方法来手动提升当前 vector 容器的容量。

举个例子（程序一）：

```
#include <iostream>
#include <vector>
using namespace std;

int main()
{
    vector<int>myvector;
    cout << "1、当前 myvector 拥有 " << myvector.size() << " 个元素，容量为 " << myvector.capacity() << endl;
    //利用 myvector 容器存储 10 个元素
    for (int i = 1; i <= 10; i++) {
        myvector.push_back(i);
    }
    cout << "2、当前 myvector 拥有 " << myvector.size() << " 个元素，容量为 " << myvector.capacity() << endl;
    //手动为 myvector 扩容
    myvector.reserve(1000);
    cout << "3、当前 myvector 拥有 " << myvector.size() << " 个元素，容量为 " << myvector.capacity() << endl;
    return 0;
}
```

程序执行结果为：
```
1、当前 myvector 拥有 0 个元素，容量为 0  
2、当前 myvector 拥有 10 个元素，容量为 13  
3、当前 myvector 拥有 10 个元素，容量为 1000
```

除了可以添加元素外，vector 模板类中还提供了 pop_back ()、erase ()、clear () 等成员方法，可以轻松实现删除容器中已存储的元素。但需要注意得是，借助这些成员方法只能删除指定的元素，容器的容量并不会因此而改变。

例如在程序一的基础上，末尾（return 0 之前）添加如下语句：

```
myvector.erase(myvector.begin());
cout << "4、当前 myvector 拥有 " << myvector.size() << " 个元素，容量为 " << myvector.capacity() << endl;

myvector.pop_back();
cout << "5、当前 myvector 拥有 " << myvector.size() << " 个元素，容量为 " << myvector.capacity() << endl;

myvector.clear();
cout << "6、当前 myvector 拥有 " << myvector.size() << " 个元素，容量为 " << myvector.capacity() << endl;
```

此段代码的执行结果为：
```
4、当前 myvector 拥有 9 个元素，容量为 1000  
5、当前 myvector 拥有 8 个元素，容量为 1000  
6、当前 myvector 拥有 0 个元素，容量为 1000
```

显然，myvector 容器存储的元素个数在减少，但容量并不会减小。

### shrink_to_fit
幸运的是，myvector 模板类中提供有一个 shrink_to_fit () 成员方法，该方法的功能是将当前 vector 容器的容量缩减至和实际存储元素的个数相等。例如，在程序一的基础上，添加如下语句：

```
myvector.shrink_to_fit();
cout << "7、当前 myvector 拥有 " << myvector.size() << " 个元素，容量为 " << myvector.capacity() << endl;
```

该语句的执行结果为：
```
7、当前 myvector 拥有 10 个元素，容量为 10
```

显然，myvector 容器的容量由 1000 缩减到了 10。

### swap
除此之外，vector 模板类中还提供有 swap () 成员方法，该方法的基础功能是交换 2 个相同类型的 vector 容器（交换容量和存储的所有元素），但其也能用于去除 vector 容器多余的容量。

如果想用 swap () 成员方法去除当前 vector 容器多余的容量时，可以套用如下的语法格式：
```
vector\<T\>(x).swap (x);
```
其中，x 指当前要操作的容器，T 为该容器存储元素的类型。

下面程序演示了此语法格式的 swap () 方法的用法和功能：

```
#include <iostream>
#include <vector>
using namespace std;

int main ()
{
    vector<int>myvector;
    //手动为 myvector 扩容
    myvector.reserve (1000);
    cout << "1、当前 myvector 拥有 " << myvector.size () << " 个元素，容量为 " << myvector.capacity () << endl;
    //利用 myvector 容器存储 10 个元素
    for (int i = 1; i <= 10; i++) {
        myvector. push_back (i);
    }
    //将 myvector 容量缩减至 10
    vector<int>(myvector).swap (myvector);
    cout << "2、当前 myvector 拥有 " << myvector.size () << " 个元素，容量为 " << myvector.capacity () << endl;
    return 0;
}
```

程序执行结果为：
```
1、当前 myvector 拥有 0 个元素，容量为 1000  
2、当前 myvector 拥有 10 个元素，容量为 10
```

显然，第 16 行代码成功将 myvector 容器的容量 1000 修改为 10，此行代码的执行流程可细分为以下 3 步：

1) 先执行 vector\<int\>(myvector)，此表达式会调用 vector 模板类中的拷贝构造函数，从而创建出一个临时的 vector 容器（后续称其为 temp vector）。

值得一提的是，tempvector 临时容器并不为空，因为我们将 myvector 作为参数传递给了复制构造函数，该函数会将 myvector 容器中的所有元素拷贝一份，并存储到 tempvector 临时容器中。

> 注意，vector 模板类中的拷贝构造函数只会为拷贝的元素分配存储空间。换句话说，tempvector 临时容器中没有空闲的存储空间，其容量等于存储元素的个数。

2) 然后借助 swap () 成员方法对 temp vector 临时容器和 myvector 容器进行调换，此过程不仅会交换 2 个容器存储的元素，还会交换它们的容量。换句话说经过 swap () 操作，myvetor 容器具有了 tempvector 临时容器存储的所有元素和容量，同时 tempvector 也具有了原 myvector 容器存储的所有元素和容量。

3) 当整条语句执行结束时，临时的 tempvector 容器会被销毁，其占据的存储空间都会被释放。注意，这里释放的其实是原 myvector 容器占用的存储空间。

经过以上 3 个步骤，就成功的将 myvector 容器的容量由 100 缩减至 10。

#### 利用 swap () 方法清空 vector 容器

在以上内容的学习过程中，如果读者善于举一反三，应该不难想到，swap () 方法还可以用来清空 vector 容器。

当 swap () 成员方法用于清空 vector 容器时，可以套用如下的语法格式：
```
vector\<T\>().swap (x);
```
其中，x 指当前要操作的容器，T 为该容器存储元素的类型。

注意，和上面语法格式唯一的不同之处在于，这里没有为 vector\<T\>() 表达式传递任何参数。这意味着，此表达式将调用 vector 模板类的默认构造函数，而不再是复制构造函数。也就是说，此格式会先生成一个空的 vector 容器，再借助 swap () 方法将空容器交换给 x，从而达到清空 x 的目的。

下面程序演示了此语法格式的 swap () 方法的用法和功能：

```
#include <iostream>
#include <vector>
using namespace std;

int main ()
{
    vector<int>myvector;
    //手动为 myvector 扩容
    myvector.reserve (1000);
    cout << "1、当前 myvector 拥有 " << myvector.size () << " 个元素，容量为 " << myvector.capacity () << endl;
    //利用 myvector 容器存储 10 个元素
    for (int i = 1; i <= 10; i++) {
        myvector. push_back (i);
    }
    //清空 myvector 容器
    vector<int>(). swap (myvector);
    cout << "2、当前 myvector 拥有 " << myvector.size () << " 个元素，容量为 " << myvector.capacity () << endl;
    return 0;
}
```

程序执行结果为：
```
1、当前 myvector 拥有 0 个元素，容量为 1000  
2、当前 myvector 拥有 0 个元素，容量为 0
```

## vector\<bool\> 不是存储 bool 类型的容器

前面章节中，已经详细介绍了 vector\<T\> 容器的功能和用法。特别需要提醒的是，在使用 vector 容器时，要尽量避免使用该容器存储 bool 类型的元素，即避免使用 vector\<bool\>。

具体来讲，不推荐使用 vector\<bool\> 的原因有以下 2 个：
1.  严格意义上讲，vector\<bool\> 并不是一个 STL 容器；
2.  vector\<bool\> 底层存储的并不是 bool 类型值。

读者可能会感到有些困惑，别着急，继续往下读。

### vector \<bool\> 不是容器
-----------------

值得一提的是，对于是否为 STL 容器，C++ 标准库中有明确的判断条件，其中一个条件是：如果 cont 是包含对象 T 的 STL 容器，且该容器中重载了 [] 运算符（即支持 operator[]），则以下代码必须能够被编译：

```
T *p = &cont[0];
```

此行代码的含义是，借助 operator\[\] 获取一个 cont\<T\> 容器中存储的 T 对象，同时将这个对象的地址赋予给一个 T 类型的指针。

这就意味着，如果 vector\<bool\> 是一个 STL 容器，则下面这段代码是可以通过编译的：

```
//创建一个 vector<bool> 容器
vector<bool>cont{0,1};
//试图将指针 p 指向 cont 容器中第一个元素
bool *p = &cont[0];
```

但不幸的是，此段代码不能通过编译。原因在于 vector\<bool\> 底层采用了独特的存储机制。

实际上，为了节省空间，vector\<bool\> 底层在存储各个 bool 类型值时，每个 bool 值都只使用一个比特位（二进制位）来存储。也就是说在 vector\<bool\> 底层，一个字节可以存储 8 个 bool 类型值。在这种存储机制的影响下，operator\[\] 势必就需要返回一个指向单个比特位的引用，但显然这样的引用是不存在的。

> C++ 标准中解决这个问题的方案是，令 operator\[\] 返回一个代理对象（proxy object）。有关代理对象，由于不是本节重点，这里不再做描述，有兴趣的读者可自行查阅相关资料。

同样对于指针来说，其指向的最小单位是字节，无法另其指向单个比特位。综上所述可以得出一个结论，即上面第 2 行代码中，用 = 赋值号连接 `bool *p` 和 `&cont[0]` 是矛盾的。

由于 vector\<bool\> 并不完全满足 C++ 标准中对容器的要求，所以严格意义上来说它并不是一个 STL 容器。可能有读者会问，既然 vector\<bool\> 不完全是一个容器，为什么还会出现在 C++ 标准中呢？

这和一个雄心勃勃的试验有关，还要从前面提到的代理对象开始说起。由于代理对象在 C++ 软件开发中很受欢迎，引起了 C++ 标准委员会的注意，他们决定以开发 vector\<bool\> 作为一个样例，来演示 STL 中的容器如何通过代理对象来存取元素，这样当用户想自己实现一个基于代理对象的容器时，就会有一个现成的参考模板。

然而开发人员在实现 vector\<bool\> 的过程中发现，既要创建一个基于代理对象的容器，同时还要求该容器满足 C++ 标准中对容器的所有要求，是不可能的。由于种种原因，这个试验最终失败了，但是他们所做过的尝试（即开发失败的 vector\<bool\>）遗留在了 C++ 标准中。

> 至于将 vector\<bool\> 遗留到 C++ 标准中，是无心之作，还是有意为之，这都无关紧要，重要的是让读者明白，vector\<bool\> 不完全满足 C++ 标准中对容器的要求，尽量避免在实际场景中使用它！

### 如何避免使用 vector\<bool\>
-------------------

那么，如果在实际场景中需要使用 vector\<bool\> 这样的存储结构，该怎么办呢？很简单，可以选择使用 deque\<bool\> 或者 bitset 来替代 vector\<bool\>。

要知道，deque 容器几乎具有 vecotr 容器全部的功能（拥有的成员方法也仅差 reserve () 和 capacity ()），而且更重要的是，deque 容器可以正常存储 bool 类型元素。

还可以考虑用 bitset 代替 vector\<bool\>，其本质是一个模板类，可以看做是一种类似数组的存储结构。和后者一样，bitset 只能用来存储 bool 类型值，且底层存储机制也采用的是用一个比特位来存储一个 bool 值。

和 vector 容器不同的是，bitset 的大小在一开始就确定了，因此不支持插入和删除元素；另外 bitset 不是容器，所以不支持使用迭代器。

> 有关 bitset 的用法，感兴趣的读者可查阅 C++ 官方提供的 [bitset 使用手册](http://www.cplusplus.com/reference/bitset/bitset/)。