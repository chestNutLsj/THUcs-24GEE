
STL list 容器，又称双向链表容器，即该容器的底层是以双向链表的形式实现的。这意味着，list 容器中的元素可以分散存储在内存空间里，而不是必须存储在一整块连续的内存空间中。

![[list-elems-store.png]]

可以看到，list 容器中各个元素的前后顺序是靠指针来维系的，每个元素都配备了 2 个指针，分别指向它的前一个元素和后一个元素。其中第一个元素的前向指针总为 null，因为它前面没有元素；同样，尾部元素的后向指针也总为 null。

基于这样的存储结构，list 容器具有一些其它容器（array、vector 和 deque）所不具备的优势，即它可以在序列已知的任何位置快速插入或删除元素（时间复杂度为 `O(1)`）。并且在 list 容器中移动元素，也比其它容器的效率高。

使用 list 容器的缺点是，它不能像 array 和 vector 那样，通过位置直接访问元素。举个例子，如果要访问 list 容器中的第 6 个元素，它不支持 `容器对象名[6]` 这种语法格式，正确的做法是从容器中第一个元素或最后一个元素开始遍历容器，直到找到该位置。

> 实际场景中，如何需要对序列进行大量添加或删除元素的操作，而直接访问元素的需求却很少，这种情况建议使用 list 容器存储序列。

list 容器以模板类 list\<T\>（T 为存储元素的类型）的形式在 `<list>` 头文件中，并位于 std 命名空间中。因此，在使用该容器之前，代码中需要包含下面两行代码：

```
#include <list>
using namespace std;
```

## list 容器的创建
----------

根据不同的使用场景，有以下 5 种创建 list 容器的方式供选择。

1) 创建一个没有任何元素的空 list 容器：
```
std::list<int> values;
```

和空 array 容器不同，空的 list 容器在创建之后仍可以添加元素，因此创建 list 容器的方式很常用。

2) 创建一个包含 n 个元素的 list 容器：
```
std::list<int> values (10);
```

通过此方式创建 values 容器，其中包含 10 个元素，每个元素的值都为相应类型的默认值（int 类型的默认值为 0）。

3) 创建一个包含 n 个元素的 list 容器，并为每个元素指定初始值。例如：
```
std::list<int> values (10, 5);
```

如此就创建了一个包含 10 个元素并且值都为 5 个 values 容器。

4) 在已有 list 容器的情况下，通过拷贝该容器可以创建新的 list 容器。例如：
```
std::list<int> value1 (10);
std::list<int> value2 (value1);
```

注意，采用此方式，必须保证新旧容器存储的元素类型一致。

5) 通过拷贝其他类型容器（或者普通数组）中指定区域内的元素，可以创建新的 list 容器。例如：
```
//拷贝普通数组，创建 list 容器
int a[] = { 1,2,3,4,5 };
std::list<int> values (a, a+5);

//拷贝其它类型的容器，创建 list 容器
std::array<int, 5> arr{ 11,12,13,14,15 };
std::list<int>values (arr.begin ()+2, arr.end ());//拷贝 arr 容器中的{13,14,15}
```


## list 容器可用的成员函数
--------------

下表罗列出了 list 模板类提供的所有成员函数以及各自的功能。

| 成员函数             | 功能                                                   |
|------------------|------------------------------------------------------|
| begin ()         | 返回指向容器中第一个元素的双向迭代器。                                  |
| end ()           | 返回指向容器中最后一个元素所在位置的下一个位置的双向迭代器。                       |
| rbegin ()        | 返回指向最后一个元素的反向双向迭代器。                                  |
| rend ()          | 返回指向第一个元素所在位置前一个位置的反向双向迭代器。                          |
| cbegin ()        | 和 begin () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改元素。      |
| cend ()          | 和 end () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改元素。        |
| crbegin ()&nbsp; | 和 rbegin () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改元素。     |
| crend ()         | 和 rend () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改元素。       |
| empty ()         | 判断容器中是否有元素，若无元素，则返回 true；反之，返回 false。                |
| size ()          | 返回当前容器实际包含的元素个数。                                     |
| max_size ()      | 返回容器所能包含元素个数的最大值。这通常是一个很大的值，一般是 $2^{32}-1$ ，所以我们很少会用到这个函数。 |
| front ()         | 返回第一个元素的引用。                                          |
| back ()          | 返回最后一个元素的引用。                                         |
| assign ()        | 用新元素替换容器中原有内容。                                       |
| emplace_front () | 在容器头部生成一个元素。该函数和 push_front () 的功能相同，但效率更高。          |
| push_front ()    | 在容器头部插入一个元素。                                         |
| pop_front ()     | 删除容器头部的一个元素。                                         |
| emplace_back ()  | 在容器尾部直接生成一个元素。该函数和 push_back () 的功能相同，但效率更高。         |
| push_back ()     | 在容器尾部插入一个元素。                                         |
| pop_back ()      | 删除容器尾部的一个元素。                                         |
| emplace ()       | 在容器中的指定位置插入元素。该函数和 insert () 功能相同，但效率更高。             |
| insert ()&nbsp;  | 在容器中的指定位置插入元素。                                       |
| erase ()         | 删除容器中一个或某区域内的元素。                                     |
| swap ()          | 交换两个容器中的元素，必须保证这两个容器中存储的元素类型是相同的。                    |
| resize ()        | 调整容器的大小。                                             |
| clear ()         | 删除容器存储的所有元素。                                         |
| splice ()        | 将一个 list 容器中的元素插入到另一个容器的指定位置。                        |
| remove (val)     | 删除容器中所有等于 val 的元素。                                   |
| remove_if ()     | 删除容器中满足条件的元素。                                        |
| unique ()        | 删除容器中相邻的重复元素，只保留一个。                                  |
| merge ()         | 合并两个事先已排好序的 list 容器，并且合并之后的 list 容器依然是有序的。           |
| sort ()          | 通过更改容器中元素的位置，将它们进行排序。                                |
| reverse ()       | 反转容器中元素的顺序。                                          |

除此之外，C++ 11 标准库还新增加了 begin () 和 end () 这 2 个函数，和 list 容器包含的 begin () 和 end () 成员函数不同，标准库提供的这 2 个函数的操作对象，既可以是容器，还可以是普通数组。当操作对象是容器时，它和容器包含的 begin () 和 end () 成员函数的功能完全相同；如果操作对象是普通数组，则 begin () 函数返回的是指向数组第一个元素的指针，同样 end () 返回指向数组中最后一个元素之后一个位置的指针（注意不是最后一个元素）。

list 容器还有一个`std:: swap (x , y)`非成员函数（其中 x 和 y 是存储相同类型元素的 list 容器），它和 swap () 成员函数的功能完全相同，仅使用语法上有差异。

如下代码演示了表 2 中部分成员函数的用法：
```
#include <iostream>
#include <list>
using namespace std;

int main()
{
    //创建空的 list 容器
    std::list<double> values;
    //向容器中添加元素
    values.push_back(3.1);
    values.push_back(2.2);
    values.push_back(2.9);
    cout << "values size：" << values.size() << endl;
    //对容器中的元素进行排序
    values.sort();
    //使用迭代器输出list容器中的元素
    for (std::list<double>::iterator it = values.begin(); it != values.end(); ++it) {
        std::cout << *it << " ";
    }
    return 0;
}
```

运行结果为：
```
values size：3  
2.2 2.9 3.1
```

## list 迭代器及用法

只有运用迭代器，才能访问 list 容器中存储的各个元素。list 模板类提供了如下表所示的这些迭代器函数。

| 迭代器函数           | 功能                                                     |
|-----------------|--------------------------------------------------------|
| begin ()        | 返回指向容器中第一个元素的双向迭代器（正向迭代器）。                             |
| end ()          | 返回指向容器中最后一个元素所在位置的下一个位置的双向迭代器。（正向迭代器）。                 |
| rbegin ()&nbsp; | 返回指向最后一个元素的反向双向迭代器。                                    |
| rend ()         | 返回指向第一个元素所在位置前一个位置的反向双向迭代器。                            |
| cbegin ()       | 和 begin () 功能相同，只不过在其基础上，正向迭代器增加了 const 属性，即不能用于修改元素。  |
| cend ()         | 和 end () 功能相同，只不过在其基础上，正向迭代器增加了 const 属性，即不能用于修改元素。    |
| crbegin ()      | 和 rbegin () 功能相同，只不过在其基础上，反向迭代器增加了 const 属性，即不能用于修改元素。 |
| crend ()        | 和 rend () 功能相同，只不过在其基础上，反向迭代器增加了 const 属性，即不能用于修改元素。   |

![[list-iter.png]]

> 注意，list 容器的底层实现结构为双向链表，图 2 这种表示仅是为了方便理解各个迭代器函数的功能。

前面章节已经详细介绍了 array、vector、deque 容器的迭代器，和它们相比，list 容器迭代器最大的不同在于，其配备的迭代器类型为双向迭代器，而不再是随机访问迭代器。

这意味着，假设 p1 和 p2 都是双向迭代器，则它们支持使用 `++p1`、 `p1++`、 `p1--`、 `p1++`、 `*p1`、 `p1==p2` 以及 `p1!=p2` 运算符，但不支持以下操作（其中 i 为整数）：
* `p1[i]`：不能通过下标访问 list 容器中指定位置处的元素。
* `p1-=i`、`p1+=i`、`p1+i`、`p1-i`：双向迭代器 p1 不支持使用 -=、+=、+、- 运算符。
* `p1<p2`、`p1>p2`、`p1<=p2`、`p1>=p2`：双向迭代器 p1、p2 不支持使用 <、 >、 <=、 >= 比较运算符。

下面这个程序演示了如何使用迭代器遍历 list 容器中的各个元素。

```
#include <iostream>
#include <list>
using namespace std;

int main()
{
    //创建 list 容器
    std::list<char> values{'h','t','t','p',':','/','/','c','.','b','i','a','n','c','h','e','n','g','.','n','e','t'};
    //使用begin()/end()迭代器函数对输出list容器中的元素
    for (std::list<char>::iterator it = values.begin(); it != values.end(); ++it) {
        std::cout << *it;
    }
    cout << endl;
    //使用 rbegin()/rend()迭代器函数输出 lsit 容器中的元素
    for (std::list<char>::reverse_iterator it = values.rbegin(); it != values.rend();++it) {
        std::cout << *it;
    }
    return 0;
}
```

输出结果为：
```
http://c.biancheng.net  
ten. gnehcnaib. c//:ptth
```

> 注意，程序中比较迭代器之间的关系，用的是 != 运算符，因为它不支持 < 等运算符。另外在实际场景中，所有迭代器函数的返回值都可以传给使用 auto 关键字定义的变量，因为编译器可以自行判断出该迭代器的类型。

值得一提的是，list 容器在进行插入（`insert ()`）、接合（`splice ()`）等操作时，都不会造成原有的 list 迭代器失效，甚至进行删除操作，而只有指向被删除元素的迭代器失效，其他迭代器不受任何影响。

举个例子：

```
#include <iostream>
#include <list>
using namespace std;

int main()
{
    //创建 list 容器
    std::list<char> values{'h','t','t','p',':','/','/','c','.','b','i','a','n','c','h','e','n','g','.','n','e','t'};
    //创建 begin 和 end 迭代器
    std::list<char>::iterator begin = values.begin();
    std::list<char>::iterator end = values.end();
    //头部和尾部插入字符 '1'
    values.insert(begin, '1');
    values.insert(end, '1');
    while (begin != end)
    {
        std::cout << *begin;
        ++begin;
    }
    return 0;
}
```

运行结果为：
```
http://c.biancheng.net1
```
可以看到，在进行插入操作之后，仍使用先前创建的迭代器遍历容器，虽然程序不会出错，但由于插入位置的不同，可能会遗漏新插入的元素。

## 底层存储结构

前面在讲 STL list 容器时提到，该容器的底层是用双向链表实现的，甚至一些 STL 版本中（比如 SGI STL），list 容器的底层实现使用的是双向循环链表。

![[list-node-composition.png]]

> 图中 node 表示链表的头指针。

如图所示，使用链表存储数据，并不会将它们存储到一整块连续的内存空间中。恰恰相反，各元素占用的存储空间（又称为节点）是独立的、分散的，它们之间的线性关系通过指针来维持。

### list 容器节点结构
-----------

通过上图可以看到，双向链表的各个节点中存储的不仅仅是元素的值，还应包含 2 个指针，分别指向前一个元素和后一个元素。

通过查看 list 容器的源码实现，其对节点的定义如下：

```
template<typename T,...>
struct __List_node{
    //...
    __list_node<T>* prev;
    __list_node<T>* next;
    T myval;
    //...
}
```

可以看到，list 容器定义的每个节点中，都包含 `*prev`、`*next` 和 `myval`。其中，prev 指针用于指向前一个节点；next 指针用于指向后一个节点；myval 用于存储当前元素的值。

### list 容器迭代器的底层实现
---------------

和 array、vector 这些容器迭代器的实现方式不同，由于 list 容器的元素并不是连续存储的，所以该容器迭代器中，必须包含一个可以指向 list 容器的指针，并且该指针还可以借助重载的 `*`、`++`、`--`、`==`、 `!= ` 等运算符，实现迭代器正确的递增、递减、取值等操作。

因此，list 容器迭代器的实现代码如下：

```
template<tyepname T,...>
struct __list_iterator{
    __list_node<T>* node;
    //...
    //重载 == 运算符
    bool operator==(const __list_iterator& x){return node == x.node;}
    //重载 != 运算符
    bool operator!=(const __list_iterator& x){return node != x.node;}
    //重载 * 运算符，返回引用类型
    T* operator *() const {return *(node).myval;}
    //重载前置 ++ 运算符
    __list_iterator<T>& operator ++(){
        node = (*node).next;
        return *this;
    }
    //重载后置 ++ 运算符
    __list_iterator<T>& operator ++(int){
        __list_iterator<T> tmp = *this;
        ++(*this);
        return tmp;
    }
    //重载前置 -- 运算符
    __list_iterator<T>& operator--(){
        node = (*node).prev;
        return *this;
    }
    //重载后置 -- 运算符
    __list_iterator<T> operator--(int){
        __list_iterator<T> tmp = *this;
        --(*this);
        return tmp;
    }
    //...
}
```

可以看到，迭代器的移动就是通过操作节点的指针实现的。

### list 容器的底层实现
------------

本节开头提到，不同版本的 STL 标准库中，list 容器的底层实现并不完全一致，但原理基本相同。这里以 SGI STL 中的 list 容器为例，讲解该容器的具体实现过程。

SGI STL 标准库中，list 容器的底层实现为双向循环链表，相比双向链表结构的好处是在构建 list 容器时，只需借助一个指针即可轻松表示 list 容器的首尾元素。

如下是 SGI STL 标准库中对 list 容器的定义：

```
template <class T,...>
class list
{
    //...
    //指向链表的头节点，并不存放数据
    __list_node<T>* node;
    //...以下还有list 容器的构造函数以及很多操作函数
}
```

另外，为了更方便的实现 list 模板类提供的函数，该模板类在构建容器时，会刻意在容器链表中添加一个空白节点，并作为 list 链表的首个节点（又称头节点）。

> 使用双向链表实现的 list 容器，其内部通常包含 2 个指针，并分别指向链表中头部的空白节点和尾部的空白节点（也就是说，其包含 2 个空白节点）。

比如，我们经常构造空的 list 容器，其用到的构造函数如下所示：

```
list() { empty_initialize(); }
// 用于空链表的建立
void empty_initialize()
{
    node = get_node();//初始化节点
    node->next = node; // 前置节点指向自己
    node->prev = node; // 后置节点指向自己
}
```

显然，即便是创建空的 list 容器，它也包含有 1 个节点。

除此之外，list 模板类中还提供有带参的构造函数，它们的实现过程大致分为以下 2 步：
* 调用 empty_initialize () 函数，构造带有头节点的空 list 容器链表；
* 将各个参数按照次序插入到空的 list 容器链表中。

由此可以总结出，list 容器实际上就是一个带有头节点的双向循环链表。如图所示，此为存有 2 个元素的 list 容器：

![[list-with-two-nodes.png]]

在此基础上，通过借助 node 头节点，就可以实现 list 容器中的所有成员函数，比如：

```
//begin()成员函数
__list_iterator<T> begin(){return (*node).next;}
//end()成员函数
__list_iterator<T> end(){return node;}
//empty()成员函数
bool empty() const{return (*node).next == node;}
//front()成员函数
T& front() {return *begin();}
//back()成员函数
T& back() {return *(--end();)}
//...
```

## list 访问元素

不同于之前学过的 [STL](http://c.biancheng.net/stl/) 容器，访问 list 容器中存储元素的方式很有限，即要么使用 front () 和 back () 成员函数，要么使用 list 容器迭代器。

> list 容器不支持随机访问，未提供下标操作符 \[\] 和 at () 成员函数，也没有提供 data () 成员函数。

通过 front () 和 back () 成员函数，可以分别获得 list 容器中第一个元素和最后一个元素的引用形式。举个例子：

```
#include <iostream>
#include <list>
using namespace std;

int main()
{
    std::list<int> mylist{ 1,2,3,4 };
    int &first = mylist.front();
    int &last = mylist.back();
    cout << first << " " << last << endl;
    first = 10;
    last = 20;
    cout << mylist.front() << " " << mylist.back() << endl;
    return 0;
}
```

输出结果为：
```
1 4  
10 20
```

可以看到，通过 front () 和 back () 的返回值，我们不仅能分别获取当前 list 容器中的首尾元素，必要时还能修改它们的值。

除此之外，如果想访问 list 容存储的其他元素，就只能使用 list 容器的迭代器。例如：

```
#include <iostream>
#include <list>
using namespace std;
int main()
{
    const std::list<int> mylist{1,2,3,4,5};
    auto it = mylist.begin();
    cout << *it << " ";
    ++it;
    while (it!=mylist.end())
    {
        cout << *it << " ";
        ++it;  
    }
    return 0;
}
```

运行结果为：
```
1 2 3 4 5
```

值得一提的是，对于非 const 类型的 list 容器，迭代器不仅可以访问容器中的元素，也可以对指定元素的值进行修改。

> 当然，对于修改容器指定元素的值，list 模板类提供有专门的成员函数 assign ()

## 添加元素

前面章节介绍了如何创建 list 容器，在此基础上，本节继续讲解如何向现有 list 容器中添加或插入新的元素。

list 模板类中，与 “添加或插入新元素” 相关的成员方法有如下几个：
* push_front ()：向 list 容器首个元素前添加新元素；
* push_back ()：向 list 容器最后一个元素后添加新元素；
* emplace_front ()：在容器首个元素前直接生成新的元素；
* emplace_back ()：在容器最后一个元素后直接生成新的元素；
* emplace ()：在容器的指定位置直接生成新的元素；
* insert ()：在指定位置插入新元素；
* splice ()：将其他 list 容器存储的多个元素添加到当前 list 容器的指定位置处。

以上这些成员方法中，除了 insert () 和 splice () 方法有多种语法格式外，其它成员方法都仅有 1 种语法格式，下面程序演示了它们的具体用法。

```
#include <iostream>
#include <list>
using namespace std;

int main()
{
    std::list<int> values{1,2,3};
    values.push_front(0);//{0,1,2,3}
    values.push_back(4); //{0,1,2,3,4}

    values.emplace_front(-1);//{-1,0,1,2,3,4}
    values.emplace_back(5);  //{-1,0,1,2,3,4,5}
   
    //emplace(pos,value),其中 pos 表示指明位置的迭代器，value为要插入的元素值
    values.emplace(values.end(), 6);//{-1,0,1,2,3,4,5,6}
    for (auto p = values.begin(); p != values.end(); ++p) {
        cout << *p << " ";
    }
    return 0;
}
```

输出结果为：
```
-1,0,1,2,3,4,5,6
```
### list insert () 成员方法
------------------

insert () 成员方法的语法格式有 4 种，如表所示。

| 语法格式                                     | 用法说明                                                                                                 |
|------------------------------------------|------------------------------------------------------------------------------------------------------|
| iterator insert (pos, elem)              | 在迭代器 pos 指定的位置之前插入一个新元素 elem，并返回表示新插入元素位置的迭代器。                                                       |
| iterator insert (pos, n, elem)           | 在迭代器 pos 指定的位置之前插入 n 个元素 elem，并返回表示第一个新插入元素位置的迭代器。                                                   |
| iterator insert (pos, first, last)&nbsp; | 在迭代器 pos 指定的位置之前，插入其他容器（例如&nbsp; array、vector、deque 等）中位于 \[first, last) 区域的所有元素，并返回表示第一个新插入元素位置的迭代器。 |
| iterator insert (pos, initlist)          | 在迭代器 pos 指定的位置之前，插入初始化列表（用大括号 { } 括起来的多个元素，中间有逗号隔开）中所有的元素，并返回表示第一个新插入元素位置的迭代器。                       |

下面的程序演示了如何使用 insert () 方法向 list 容器中插入元素。

```
#include <iostream>
#include <list>
#include <array>
using namespace std;
int main()
{
    std::list<int> values{ 1,2 };
    //第一种格式用法
    values.insert(values.begin() , 3);//{3,1,2}

    //第二种格式用法
    values.insert(values.end(), 2, 5);//{3,1,2,5,5}

    //第三种格式用法
    std::array<int, 3>test{ 7,8,9 };
    values.insert(values.end(), test.begin(), test.end());//{3,1,2,5,5,7,8,9}

    //第四种格式用法
    values.insert(values.end(), { 10,11 });//{3,1,2,5,5,7,8,9,10,11}

    for (auto p = values.begin(); p != values.end(); ++p)
    {
        cout << *p << " ";
    }
    return 0;
}
```

输出结果为：
```
3 1 2 5 5 7 8 9 10 11
```

学到这里，读者有没有发现，同样是实现插入元素的功能，无论是 push_front ()、push_back () 还是 insert ()，都有以 emplace 为名且功能和前者相同的成员函数。这是因为，后者是 [C++](http://c.biancheng.net/cplus/) 11 标准新添加的，在大多数场景中，都可以完全替代前者实现同样的功能。更重要的是，实现同样的功能，emplace 系列方法的执行效率更高。

### list splice () 成员方法
------------------

和 insert () 成员方法相比，splice () 成员方法的作用对象是其它 list 容器，其功能是将其它 list 容器中的元素添加到当前 list 容器中指定位置处。

splice () 成员方法的语法格式有 3 种，如表所示。

| 语法格式                                                                         | 功能                                                                                                                                                               |
|------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| void splice (iterator position, list&amp; x);                                | position 为迭代器，用于指明插入位置；x 为另一个 list 容器。此格式的 splice () 方法的功能是，将 x 容器中存储的所有元素全部移动当前 list 容器中 position 指明的位置处。                                                       |
| void splice (iterator position, list&amp; x, iterator i);                    | position 为迭代器，用于指明插入位置；x 为另一个 list 容器；i 也是一个迭代器，用于指向 x 容器中某个元素。此格式的 splice () 方法的功能是将 x 容器中 i 指向的元素移动到当前容器中 position 指明的位置处。                                     |
| void splice (iterator position, list&amp; x, iterator first, iterator last); | position 为迭代器，用于指明插入位置；x 为另一个 list 容器；first 和 last 都是迭代器，[fist, last) 用于指定 x 容器中的某个区域。此格式的 splice () 方法的功能是将 x 容器 [first, last) 范围内所有的元素移动到当前容器 position 指明的位置处。 |

我们知道，list 容器底层使用的是链表存储结构，splice () 成员方法移动元素的方式是，将存储该元素的节点从 list 容器底层的链表中摘除，然后再链接到当前 list 容器底层的链表中。这意味着，当使用 splice () 成员方法将 x 容器中的元素添加到当前容器的同时，该元素会从 x 容器中删除。

下面程序演示了 splice () 成员方法的用法：

```
#include <iostream>
#include <list>
using namespace std;
int main()
{
    //创建并初始化 2 个 list 容器
    list<int> mylist1{ 1,2,3,4 }, mylist2{10,20,30};
    list<int>::iterator it = ++mylist1.begin(); //指向 mylist1 容器中的元素 2
   
    //调用第一种语法格式
    mylist1.splice(it, mylist2); // mylist1: 1 10 20 30 2 3 4
                                 // mylist2:
                                 // it 迭代器仍然指向元素 2，只不过容器变为了 mylist1

    //调用第二种语法格式，将 it 指向的元素 2 移动到 mylist2.begin() 位置处
    mylist2.splice(mylist2.begin(), mylist1, it);   // mylist1: 1 10 20 30 3 4
                                                    // mylist2: 2
                                                    // it 仍然指向元素 2
   
    //调用第三种语法格式，将 [mylist1.begin(),mylist1.end())范围内的元素移动到 mylist.begin() 位置处                  
    mylist2.splice(mylist2.begin(), mylist1, mylist1.begin(), mylist1.end());//mylist1:
                                                                             //mylist2:1 10 20 30 3 4 2
   
    cout << "mylist1 包含 " << mylist1.size() << "个元素" << endl;
    cout << "mylist2 包含 " << mylist2.size() << "个元素" << endl;
    //输出 mylist2 容器中存储的数据
    cout << "mylist2:";
    for (auto iter = mylist2.begin(); iter != mylist2.end(); ++iter) {
        cout << *iter << " ";
    }
    return 0;
}
```

程序执行结果为：
```
mylist 1 包含 0 个元素  
mylist 2 包含 7 个元素  
mylist2:1 10 20 30 3 4 2
```

## 删除元素

对 list 容器存储的元素执行删除操作，需要借助该容器模板类提供的成员函数。幸运的是，相比其它 STL 容器模板类，list 模板类提供了更多用来实现此操作的成员函数（如表所示）。

| 成员函数         | 功能                                              |
|--------------|-------------------------------------------------|
| pop_front () | 删除位于 list 容器头部的一个元素。                            |
| pop_back ()  | 删除位于 list 容器尾部的一个元素。                            |
| erase ()     | 该成员函数既可以删除 list 容器中指定位置处的元素，也可以删除容器中某个区域内的多个元素。 |
| clear ()     | 删除 list 容器存储的所有元素。                              |
| remove (val) | 删除容器中所有等于 val 的元素。                              |
| unique ()    | 删除容器中相邻的重复元素，只保留一份。                             |
| remove_if () | 删除容器中满足条件的元素。                                   |

其中，pop_front ()、pop_back () 和 clear () 的用法非常简单，这里仅给出一个样例，不再过多解释：

```
#include <iostream>
#include <list>
using namespace std;
int main()
{
    list<int>values{ 1,2,3,4 };
   
    //删除当前容器中首个元素
    values.pop_front();//{2,3,4}
   
    //删除当前容器最后一个元素
    values.pop_back();//{2,3}
   
    //清空容器，删除容器中所有的元素
    values.clear(); //{}
   
    for (auto begin = values.begin(); begin != values.end(); ++begin)
    {
        cout << *begin << " ";
    }
    return 0;
}
```

运行程序，可以看到输出结果为 “空”。

### erase
erase () 成员函数有以下 2 种语法格式：
```
iterator erase (iterator position);  
iterator erase (iterator first, iterator last);
```

利用第一种语法格式，可实现删除 list 容器中 position 迭代器所指位置处的元素，例如：

```
#include <iostream>
#include <list>
using namespace std;
int main()
{
    list<int>values{ 1,2,3,4,5 };
    //指向元素 1 的迭代器
    auto del = values.begin();
    //迭代器右移，改为指向元素 2
    ++del;
    values.erase(del); //{1,3,4,5}

    for (auto begin = values.begin(); begin != values.end(); ++begin)
    {
        cout << *begin << " ";
    }
    return 0;
}
```

运行结果为：
```
1 3 4 5
```

利用第二种语法格式，可实现删除 list 容器中 first 迭代器和 last 迭代器限定区域内的所有元素（包括 first 指向的元素，但不包括 last 指向的元素）。例如：

```
#include <iostream>
#include <list>
using namespace std;
int main()
{
    list<int>values{ 1,2,3,4,5 };
    //指定删除区域的左边界
    auto first = values.begin();
    ++first;//指向元素 2

    //指向删除区域的右边界
    auto last = values.end();
    --last;//指向元素 5

    //删除 2、3 和 4
    values.erase(first, last);

    for (auto begin = values.begin(); begin != values.end(); ++begin)
    {
        cout << *begin << " ";
    }
    return 0;
}
```

运行结果为：
```
1 5
```

### remove
erase () 成员函数是按照被删除元素所在的位置来执行删除操作，如果想根据元素的值来执行删除操作，可以使用 remove () 成员函数。例如：

```
#include <iostream>
#include <list>
using namespace std;
int main()
{
    list<char>values{'a','b','c','d'};
    values.remove('c');
    for (auto begin = values.begin(); begin != values.end(); ++begin)
    {
        cout << *begin << " ";
    }
    return 0;
}
```

运行结果为：
```
a b d
```

### unique
unique () 函数也有以下 2 种语法格式：
```
void unique ()  
void unique（BinaryPredicate）// 传入一个二元谓词函数
```
以上 2 种格式都能实现去除 list 容器中相邻重复的元素，仅保留一份。但第 2 种格式的优势在于，我们能自定义去重的规则，例如：

```
#include <iostream>
#include <list>
using namespace std;
//二元谓词函数
bool demo(double first, double second)
{
    return (int(first) == int(second));
}

int main()
{
    list<double> mylist{ 1,1.2,1.2,3,4,4.5,4.6 };
    //删除相邻重复的元素，仅保留一份
    mylist.unique();//{1, 1.2, 3, 4, 4.5, 4.6}

    for (auto it = mylist.begin(); it != mylist.end(); ++it)
        cout << *it << ' ';
    cout << endl;
    //demo 为二元谓词函数，是我们自定义的去重规则
    mylist.unique(demo);

    for (auto it = mylist.begin(); it != mylist.end(); ++it)
        std::cout << *it << ' ';
    return 0;
}
```

运行结果为：
```
1 1.2 3 4 4.5 4.6  
1 3 4
```

> 注意，除了以上一定谓词函数的方式，还可以使用 [lamba 表达式](http://c.biancheng.net/view/433.html)以及[函数对象](http://c.biancheng.net/view/354.html)的方式定义。

可以看到，通过调用无参的 unique ()，仅能删除相邻重复（也就是相等）的元素，而通过我们自定义去重的规则，可以更好的满足在不同场景下去重的需求。

### remove_if
除此之外，通过将自定义的谓词函数（不限定参数个数）传给 remove_if () 成员函数，list 容器中能使谓词函数成立的元素都会被删除。举个例子：

```
#include <iostream>
#include <list>
using namespace std;

int main()
{
    std::list<int> mylist{ 15, 36, 7, 17, 20, 39, 4, 1 };
    //删除 mylist 容器中能够使 lamba 表达式成立的所有元素。
    mylist.remove_if([](int value) {return (value < 10); }); //{15 36 17 20 39}

    for (auto it = mylist.begin(); it != mylist.end(); ++it)
        std::cout << ' ' << *it;

    return 0;
}
```

运行结果为：
```
15 36 17 20 39
```

## 问题：empty 和 size 都可以用来判空，哪个更好？

到目前为止，我们已经了解了 C++ STL 标准库中 vector、deque 和 list 这 3 个容器的功能和具体用法。学习过程中，读者是否想过一个问题，即这些容器的模板类中都提供了 empty () 成员方法和 size () 成员方法，它们都可以用来判断容器是否为空。

换句话说，假设有一个容器 cont，则此代码：

```
if(cont.size() == 0)
```

本质上和如下代码是等价的：

```
if(cont.empty())
```

那么，在实际场景中，到底应该使用哪一种呢？

建议使用 empty () 成员方法。理由很简单，无论是哪种容器，只要其模板类中提供了 empty () 成员方法，使用此方法都可以保证在 O (1) 时间复杂度内完成对 “容器是否为空” 的判断；但对于 list 容器来说，使用 size () 成员方法判断“容器是否为空”，可能要消耗 O (n) 的时间复杂度。

> 注意，这个结论不仅适用于 vector、deque 和 list 容器，后续还会讲解更多容器的用法，该结论也依然适用。

那么，为什么 list 容器这么特殊呢？这和 list 模板类提供了独有的 splice () 成员方法有关。

### 深度剖析选用 empty () 的原因
------------------

做一个大胆的设想，假设我们自己就是 list 容器的设计者。从始至终，我们的目标都是让 list 成为标准容器，并被广泛使用，因此打造 “高效率的 list” 成为我们奋斗的目标。

在实现 list 容器的过程中我们发现，用户经常需要知道当前 list 容器中存有多少个元素，于是我们想让 size () 成员方法的执行效率达到 O (1)。为了实现这个目的，我们必须重新设计 list，使它总能以最快的效率知道自己含有多少个元素。

> 要想令 size () 的执行效率达到 O (1)，最直接的实现方式是：在 list 模板类中设置一个专门用于统计存储元素数量的 size 变量，其位于所有成员方法的外部。与此同时，在每一个可用来为容器添加或删除元素的成员方法中，添加对 size 变量值的更新操作。由此，size () 成员方法只需返回 size 这个变量即可（时间复杂度为 O (1)）。

不仅如此，由于 list 容器底层采用的是链式存储结构（也就是链表），该结构最大的特点就是，某一链表中存有元素的节点，无需经过拷贝就可以直接链接到其它链表中，且整个过程只需要消耗 O (1) 的时间复杂度。考虑到很多用户之所以选用 list 容器，就是看中了其底层存储结构的这一特性。因此，作为 list 容器设计者的我们，自然也想将 splice () 方法的时间复杂度设计为 O (1)。

这里就产生了一个矛盾，即如果将 size () 设计为 O (1) 时间复杂度，则由于 splice () 成员方法会修改 list 容器存储元素的个数，因此该方法中就需要添加更新 size 变量的代码（更新方式无疑是通过遍历链表来实现），这也就意味着 splice () 成员方法的执行效率将无法达到 O (1)；反之，如果将 splice () 成员方法的执行效率提高到 O (1)，则 size () 成员方法将无法实现 O (1) 的时间复杂度。

> 也就是说，list 容器中的 size () 和 splice () 总有一个要做出让步，即只能实现其中一个方法的执行效率达到 O (1)。

值得一提的是，不同版本的 STL 标准库，其底层解决此冲突的抉择是不同的。以本教程所用的 C++ STL 标准模板库为例，其选择将 splice () 成员方法的执行效率达到 O (1)，而 size () 成员方法的执行效率为 O (n)。当然，有些版本的 STL 标准库中，选择将 size () 方法的执行效率设计为 O (1)。

但不论怎样，选用 empty () 判断容器是否为空，效率总是最高的。所以，如果程序中需要判断当前容器是否为空，应优先考虑使用 empty ()。