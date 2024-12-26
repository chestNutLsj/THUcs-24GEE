
前面章节讲解了 map 容器和 multimap 容器的用法，类似地，STL 标准库中还提供有 set 和 multiset 这 2 个容器，它们也属于关联式容器。不过，本节先讲解 set 容器，后续章节再讲解 multiset 容器。

和 map、multimap 容器不同，使用 set 容器存储的各个键值对，要求**键 key 和值 value 必须相等**。

举个例子，如下有 2 组键值对数据：
```
{<'a', 1>, <'b', 2>, <'c', 3>}  
{<'a', 'a'>, <'b', 'b'>, <'c', 'c'>}
```
显然，第一组数据中各键值对的键和值不相等，而第二组中各键值对的键和值对应相等。对于 set 容器来说，只能存储第 2 组键值对，而无法存储第一组键值对。

基于 set 容器的这种特性，当使用 set 容器存储键值对时，只需要为其提供各键值对中的 value 值（也就是 key 的值）即可。仍以存储上面第 2 组键值对为例，只需要为 set 容器提供 {'a','b','c'} ，该容器即可成功将它们存储起来。

通过前面的学习我们知道，map、multimap 容器都会自行根据键的大小对存储的键值对进行排序，set 容器也会如此，只不过 set 容器中各键值对的键 key 和值 value 是相等的，*根据 key 排序，也就等价为根据 value 排序*。

另外，**使用 set 容器存储的各个元素的值必须各不相同**。更重要的是，从语法上讲 set 容器并没有强制对存储元素的类型做 const 修饰，即 set 容器中存储的元素的值是可以修改的。但是，C++ 标准为了防止用户修改容器中元素的值，对所有可能会实现此操作的行为做了限制，使得在正常情况下，用户是无法做到修改 set 容器中元素的值的。

> 对于初学者来说，切勿尝试直接修改 set 容器中已存储元素的值，这很有可能破坏 set 容器中元素的有序性，*最正确的修改 set 容器中元素值的做法是：先删除该元素，然后再添加一个修改后的元素*。

值得一提的是，set 容器定义于 `<set>` 头文件，并位于 std 命名空间中。因此如果想在程序中使用 set 容器，该程序代码应先包含如下语句：

```
#include <set>
using namespace std;
```

set 容器的类模板定义如下：

```
template < class T,                        // 键 key 和值 value 的类型
           class Compare = less<T>,        // 指定 set 容器内部的排序规则
           class Alloc = allocator<T>      // 指定分配器对象的类型
           > class set;
```

注意，由于 set 容器存储的各个键值对，其键和值完全相同，也就意味着它们的类型相同，因此 set 容器类模板的定义中，仅有第 1 个参数用于设定存储数据的类型。

> 对于 set 类模板中的 3 个参数，后 2 个参数自带默认值，且几乎所有场景中只需使用前 2 个参数，第 3 个参数不会用到。

## 创建 set 容器的几种方法
------------------

常见的创建 set 容器的方法，大致有以下 5 种。

1) 调用默认构造函数，创建空的 set 容器。比如：
```
std::set<std::string> myset;
```
由此就创建好了一个 set 容器，该容器采用默认的 `std::less<T>` 规则，会对存储的 string 类型元素做升序排序。注意，由于 set 容器支持随时向内部添加新的元素，因此创建空 set 容器的方法是经常使用的。

2) 除此之外，set 类模板还支持在创建 set 容器的同时，对其进行初始化。例如：
```
std::set<std::string> myset{"http://c.biancheng.net/java/", 
"http://c.biancheng.net/stl/",
"http://c.biancheng.net/python/"};
```
由此即创建好了包含 3 个 string 元素的 myset 容器。由于其采用默认的 std::less\<T\> 规则，因此其内部存储 string 元素的顺序如下所示：
```
" http://c.biancheng.net/java/"  
" http://c.biancheng.net/python/"  
" http://c.biancheng.net/stl/"
```

3) set 类模板中还提供了拷贝（复制）构造函数，可以实现在创建新 set 容器的同时，将已有 set 容器中存储的所有元素全部复制到新 set 容器中。
例如，在第 2 种方式创建的 myset 容器的基础上，执行如下代码：
```
std::set<std::string> copyset (myset);
//等同于
//std::set<std::string> copyset = myset
```
该行代码在创建 copyset 容器的基础上，还会将 myset 容器中存储的所有元素，全部复制给 copyset 容器一份。

另外，C++ 11 标准还为 set 类模板新增了移动构造函数，其功能是实现创建新 set 容器的同时，利用临时的 set 容器为其初始化。比如：
```
set<string> retSet () {
    std::set<std::string> myset{ " http://c.biancheng.net/java/" ,
                            " http://c.biancheng.net/stl/" ,
                            " http://c.biancheng.net/python/" };
    return myset;
}
std::set<std::string> copyset (retSet ());
//或者
//std::set<std::string> copyset = retSet ();
```
注意，由于 retSet () 函数的返回值是一个临时 set 容器，因此在初始化 copyset 容器时，其内部调用的是 set 类模板中的移动构造函数，而非拷贝构造函数。

> 显然，无论是调用复制构造函数还是调用拷贝构造函数，都必须保证这 2 个容器的类型完全一致。

4) 在第 3 种方式的基础上，set 类模板还支持取已有 set 容器中的部分元素，来初始化新 set 容器。例如：
```
std::set<std::string> myset{ " http://c.biancheng.net/java/" ,
                    " http://c.biancheng.net/stl/" ,
                    " http://c.biancheng.net/python/" };
std::set<std::string> copyset (++myset.begin (), myset.end ());
```
由此初始化的 copyset 容器，其内部仅存有如下 2 个 string 字符串：
```
" http://c.biancheng.net/python/"  
" http://c.biancheng.net/stl/"
```

5) 以上几种方式创建的 set 容器，都采用了默认的`std::less<T>`规则。其实，借助 set 类模板定义中第 2 个参数，我们完全可以手动修改 set 容器中的排序规则。比如：
```
std::set<std::string,std::greater<string> > myset{
    " http://c.biancheng.net/java/" ,
    " http://c.biancheng.net/stl/" ,
    " http://c.biancheng.net/python/" };
```

通过选用 std::greater\<string\> 降序规则，myset 容器中元素的存储顺序为:
```
" http://c.biancheng.net/stl/"  
" http://c.biancheng.net/python/"  
" http://c.biancheng.net/java/"
```

## set 容器包含的成员方法
---------------------

下表列出了 set 容器提供的常用成员方法以及各自的功能。

| 成员方法              | 功能                                                                                                                                                                             |
|-------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| begin ()          | 返回指向容器中第一个（注意，是已排好序的第一个）元素的双向迭代器。如果 set 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                                                                  |
| end ()            | 返回指向容器最后一个元素（注意，是已排好序的最后一个）所在位置后一个位置的双向迭代器，通常和 begin () 结合使用。如果 set 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                                      |
| rbegin ()         | 返回指向最后一个（注意，是已排好序的最后一个）元素的反向双向迭代器。如果 set 容器用 const 限定，则该方法返回的是 const 类型的反向双向迭代器。                                                                                               |
| rend ()           | 返回指向第一个（注意，是已排好序的第一个）元素所在位置前一个位置的反向双向迭代器。如果 set 容器用 const 限定，则该方法返回的是 const 类型的反向双向迭代器。                                                                                        |
| cbegin ()         | 和 begin () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的元素值。                                                                                                                         |
| cend ()           | 和 end () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的元素值。                                                                                                                           |
| crbegin ()        | 和 rbegin () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的元素值。                                                                                                                        |
| crend ()          | 和 rend () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的元素值。                                                                                                                          |
| find (val)        | 在 set 容器中查找值为 val 的元素，如果成功找到，则返回指向该元素的双向迭代器；反之，则返回和 end () 方法一样的迭代器。另外，如果 set 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                            |
| lower_bound (val) | 返回一个指向当前 set 容器中第一个大于或等于 val 的元素的双向迭代器。如果 set 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                                                            |
| upper_bound (val) | 返回一个指向当前 set 容器中第一个大于 val 的元素的迭代器。如果 set 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                                                                 |
| equal_range (val) | 该方法返回一个 pair 对象（包含 2 个双向迭代器），其中 pair. first 和 lower_bound () 方法的返回值等价，pair. second 和 upper_bound () 方法的返回值等价。也就是说，该方法将返回一个范围，该范围中包含的值为 val 的元素（set 容器中各个元素是唯一的，因此该范围最多包含一个元素）。 |
| empty ()          | 若容器为空，则返回 true；否则 false。                                                                                                                                                       |
| size ()           | 返回当前 set 容器中存有元素的个数。                                                                                                                                                           |
| max_size ()       | 返回 set 容器所能容纳元素的最大个数，不同的操作系统，其返回值亦不相同。                                                                                                                                         |
| insert ()         | 向 set 容器中插入元素。                                                                                                                                                                 |
| erase ()          | 删除 set 容器中存储的元素。                                                                                                                                                               |
| swap ()           | 交换 2 个 set 容器中存储的所有元素。这意味着，操作的 2 个 set 容器的类型必须相同。                                                                                                                              |
| clear ()          | 清空 set 容器中所有的元素，即令 set 容器的 size () 为 0。                                                                                                                                        |
| emplace ()        | 在当前 set 容器中的指定位置直接构造新元素。其效果和 insert () 一样，但效率更高。                                                                                                                               |
| emplace_hint ()   | 在本质上和 emplace () 在 set 容器中构造新元素的方式是一样的，不同之处在于，使用者必须为该方法提供一个指示新元素生成位置的迭代器，并作为该方法的第一个参数。                                                                                         |
| count (val)       | 在当前 set 容器中，查找值为 val 的元素的个数，并返回。注意，由于 set 容器中各元素的值是唯一的，因此该函数的返回值最大为 1。                                                                                                         |

下面程序演示了部分成员函数的用法：

```
#include <iostream>
#include <set>
#include <string>
using namespace std;

int main ()
{
    //创建空 set 容器
    std::set<std::string> myset;
    //空 set 容器不存储任何元素
    cout << "1、myset size = " << myset.size () << endl;
    //向 myset 容器中插入新元素
    myset.insert (" http://c.biancheng.net/java/" );
    myset.insert (" http://c.biancheng.net/stl/" );
    myset.insert (" http://c.biancheng.net/python/" );
    cout << "2、myset size = " << myset.size () << endl;
    //利用双向迭代器，遍历 myset
    for (auto iter = myset.begin (); iter != myset.end (); ++iter) {
        cout << *iter << endl;
    }
    return 0;
}
```

程序执行结果为：
```
1、myset size = 0  
2、myset size = 3  
http://c.biancheng.net/java/  
http://c.biancheng.net/python/  
http://c.biancheng.net/stl/
```

## set 的迭代器

和 map 容器不同，set 容器类模板中未提供 at () 成员函数，也未对 \[\] 运算符进行重载。因此，要想访问 set 容器中存储的元素，只能借助 set 容器的迭代器。

值得一提的是，C++ STL 标准库为 set 容器配置的迭代器类型为**双向迭代器**。这意味着，假设 p 为此类型的迭代器，则其只能进行 `++p`、` p++ `、` --p `、` p-- `、`*p ` 操作，并且 2 个双向迭代器之间做比较，也只能使用 `==` 或者 `!= ` 运算符。

在 set 容器类模板提供的所有成员函数中，返回迭代器的成员函数如下表所示。

| 成员方法              | 功能                                                                                                                                                                             |
|-------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| begin ()          | 返回指向容器中第一个（注意，是已排好序的第一个）元素的双向迭代器。如果 set 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                                                                  |
| end ()            | 返回指向容器最后一个元素（注意，是已排好序的最后一个）所在位置后一个位置的双向迭代器，通常和 begin () 结合使用。如果 set 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                                      |
| rbegin ()         | 返回指向最后一个（注意，是已排好序的最后一个）元素的反向双向迭代器。如果 set 容器用 const 限定，则该方法返回的是 const 类型的反向双向迭代器。                                                                                               |
| rend ()           | 返回指向第一个（注意，是已排好序的第一个）元素所在位置前一个位置的反向双向迭代器。通常和 rbegin () 结合使用。如果 set 容器用 const 限定，则该方法返回的是 const 类型的反向双向迭代器。                                                                     |
| cbegin ()         | 和 begin () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的元素值。                                                                                                                         |
| cend ()           | 和 end () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的元素值。                                                                                                                           |
| crbegin ()        | 和 rbegin () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的元素值。                                                                                                                        |
| crend ()          | 和 rend () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的元素值。                                                                                                                          |
| find (val)        | 在 set 容器中查找值为 val 的元素，如果成功找到，则返回指向该元素的双向迭代器；反之，则返回和 end () 方法一样的迭代器。另外，如果 set 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                            |
| lower_bound (val) | 返回一个指向当前 set 容器中第一个大于或等于 val 的元素的双向迭代器。如果 set 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                                                            |
| upper_bound (val) | 返回一个指向当前 set 容器中第一个大于 val 的元素的迭代器。如果 set 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                                                                 |
| equal_range (val) | 该方法返回一个 pair 对象（包含 2 个双向迭代器），其中 pair. first 和 lower_bound () 方法的返回值等价，pair. second 和 upper_bound () 方法的返回值等价。也就是说，该方法将返回一个范围，该范围中包含的值为 val 的元素（set 容器中各个元素是唯一的，因此该范围最多包含一个元素）。 |

> 注意，以上成员函数返回的迭代器，指向的只是 set 容器中存储的元素，而不再是键值对。另外，以上成员方法返回的迭代器，无论是 const 类型还是非 const 类型，都不能用于修改 set 容器中的值。

![[set-iterator.png]]

> 其中，Ei 表示 set 容器中存储的各个元素，它们的值各不相同。

下面程序以 begin ()/end () 为例，演示了如何使用图中相关迭代器遍历 set 容器：

```
#include <iostream>
#include <set>
#include <string>
using namespace std;

int main()
{
    //创建并初始化set容器
    std::set<std::string> myset{ "http://c.biancheng.net/java/",
                                 "http://c.biancheng.net/stl/",
                                 "http://c.biancheng.net/python/"
    };
    //利用双向迭代器，遍历myset
    for (auto iter = myset.begin(); iter != myset.end(); ++iter) {
        cout << *iter << endl;
    }
    return 0;
}
```

程序执行结果为：
```
http://c.biancheng.net/java/  
http://c.biancheng.net/python/  
http://c.biancheng.net/stl/
```

> 再次强调，正如程序第 15 行代码所示的那样，因为 iter 迭代器指向的是 set 容器存储的某个元素，而不是键值对，因此通过 `*iter` 可以直接获取该迭代器指向的元素的值。

除此之外，如果只想遍历 set 容器中指定区域内的部分数据，则可以借助 find ()、lower_bound () 以及 upper_bound () 实现。通过调用它们，可以获取一个指向指定元素的迭代器。

需要特别指出的是，equal_range (val) 函数的返回值是一个 *pair* 类型数据，其包含 2 个迭代器，表示 set 容器中和指定参数 val 相等的元素所在的区域，但由于 set 容器中存储的元素各不相等，因此该函数返回的这 2 个迭代器所表示的范围中，最多只会包含 1 个元素。

举个例子：

```
#include <iostream>
#include <set>
#include <string>
using namespace std;

int main()
{
    //创建并初始化set容器
    std::set<std::string> myset{ "http://c.biancheng.net/java/",
                                 "http://c.biancheng.net/stl/",
                                 "http://c.biancheng.net/python/"
    };
   
    set<string>::iterator iter = myset.find("http://c.biancheng.net/python/");
    for (;iter != myset.end();++iter)
    {
        cout << *iter << endl;
    }
    return 0;
}
```

程序执行结果为：
```
http://c.biancheng.net/python/  
http://c.biancheng.net/stl/
```

> 值得一提的是，虽然 C++ STL 标准中，set 类模板中包含 lower_bound ()、upper_bound ()、equal_range () 这 3 个成员函数，但它们更适用于 multiset 容器，几乎不会用于操作 set 容器。

## set 中插入元素

通过前面的学习，我们已经学会如何创建一个 set 容器。在此基础上，如果想向 set 容器中继续添加元素，可以借助 set 类模板提供的 insert () 方法。

为满足不同场景的需要，C++11 标准的 set 类模板中提供了多种不同语法格式的 insert () 成员方法，它们各自的功能和用法如下所示。

### insert 直接插入一个值
1) 只要给定目标元素的值，insert () 方法即可将该元素添加到 set 容器中，其语法格式如下：
```
// 普通引用方式传参  
pair<iterator,bool> insert (const value_type& val);  
// 右值引用方式传参  
pair<iterator,bool> insert (value_type&& val);
```
其中，val 表示要添加的新元素，该方法的返回值为 pair 类型。

可以看到，以上 2 种语法格式的 insert () 方法，返回的都是 pair 类型的值，其包含 2 个数据，一个迭代器和一个 bool 值：
* 当向 set 容器添加元素成功时，该迭代器指向 set 容器新添加的元素，bool 类型的值为 true；
* 如果添加失败，即证明原 set 容器中已存有相同的元素，此时返回的迭代器就指向容器中相同的此元素，同时 bool 类型的值为 false。

举个例子：

```
#include <iostream>
#include <set>
#include <string>
using namespace std;

int main()
{
    //创建并初始化set容器
    std::set<std::string> myset;
    //准备接受 insert() 的返回值
    pair<set<string>::iterator, bool> retpair;
    //采用普通引用传值方式
    string str = "http://c.biancheng.net/stl/";
    retpair = myset.insert(str);
    cout << "iter->" << *(retpair.first) << " " << "bool = " << retpair.second << endl;
    //采用右值引用传值方式
    retpair = myset.insert("http://c.biancheng.net/python/");
    cout << "iter->" << *(retpair.first) << " " << "bool = " << retpair.second << endl;
    return 0;
}
```

程序执行结果为：
```
iter-> http://c.biancheng.net/stl/ bool = 1  
iter-> http://c.biancheng.net/python/ bool = 1
```
通过观察输出结果不难看出，程序中两次借助 insert () 方法向 set 容器中添加元素，都成功了。

### insert 插入到具体位置
2) insert () 还可以指定将新元素插入到 set 容器中的具体位置，其语法格式如下：
```
// 以普通引用的方式传递 val 值  
iterator insert (const_iterator position, const value_type& val);  
// 以右值引用的方式传递 val 值  
iterator insert (const_iterator position, value_type&& val);
```

以上 2 种语法格式中，insert () 函数的返回值为迭代器：
* 当向 set 容器添加元素成功时，该迭代器指向容器中新添加的元素；
* 当添加失败时，证明原 set 容器中已有相同的元素，*该迭代器就指向 set 容器中相同的这个元素*。

举个例子：
```
#include <iostream>
#include <set>
#include <string>
using namespace std;

int main()
{
    //创建并初始化set容器
    std::set<std::string> myset;
    //准备接受 insert() 的返回值
    set<string>::iterator iter;

    //采用普通引用传值方式
    string str = "http://c.biancheng.net/stl/";
    iter = myset.insert(myset.begin(),str);
    cout << "myset size =" << myset.size() << endl;

    //采用右值引用传值方式
    iter = myset.insert(myset.end(),"http://c.biancheng.net/python/");
    cout << "myset size =" << myset.size() << endl;
    return 0;
}
```

程序执行结果为：
```
myset size =1  
myset size =2
```

> 注意，使用 insert () 方法将目标元素插入到 set 容器指定位置后，如果该元素破坏了容器内部的有序状态，set 容器还会自行对新元素的位置做进一步调整。也就是说，insert () 方法中指定新元素插入的位置，并不一定就是该元素最终所处的位置。

### insert 其它 set 指定区域
3) insert () 方法支持向当前 set 容器中插入其它 set 容器指定区域内的所有元素，只要这 2 个 set 容器存储的元素类型相同即可。

insert () 方法的语法格式如下：
```
template <class InputIterator>  
  void insert (InputIterator first, InputIterator last);
```
其中 first 和 last 都是迭代器，它们的组合 `[first, last)` 可以表示另一 set 容器中的一块区域，该区域包括 first 迭代器指向的元素，但不包含 last 迭代器指向的元素。

举个例子：
```
#include <iostream>
#include <set>
#include <string>
using namespace std;

int main ()
{
    //创建并初始化 set 容器
    std::set<std::string> myset{ " http://c.biancheng.net/stl/" ,
                                " http://c.biancheng.net/python/" ,
                                " http://c.biancheng.net/java/" };
    //创建一个同类型的空 set 容器
    std::set<std::string> otherset;
    //利用 myset 初始化 otherset
    otherset.insert (++myset.begin (), myset.end ());
    //输出 otherset 容器中的元素
    for (auto iter = otherset.begin (); iter != otherset.end (); ++iter) {
        cout << *iter << endl;
    }
    return 0;
}
```

程序执行结果为：
```
http://c.biancheng.net/python/  
http://c.biancheng.net/stl/
```
注意，程序第 15 行在初始化 otherset 容器时，选取的是 myset 容器中从第 2 个元素开始（包括此元素）直到容器末尾范围内的所有元素，所以程序输出结果中只有 2 个字符串。

### insert 多个元素
4) 采用如下格式的 insert () 方法，可实现一次向 set 容器中添加多个元素：
```
void insert ( {E 1, E 2,..., En} );
```
其中，Ei 表示新添加的元素。

举个例子：
```
#include <iostream>
#include <set>
#include <string>
using namespace std;

int main ()
{
    //创建并初始化 set 容器
    std::set<std::string> myset;
    //向 myset 中添加多个元素
    myset.insert ({ " http://c.biancheng.net/stl/" ,
        " http://c.biancheng.net/python/" ,
        " http://c.biancheng.net/java/" });

    for (auto iter = myset.begin (); iter != myset.end (); ++iter) {
        cout << *iter << endl;
    }
    return 0;
}
```

程序执行结果为：
```
http://c.biancheng.net/java/  
http://c.biancheng.net/python/  
http://c.biancheng.net/stl/
```
以上的讲解，即为 set 类模板中 insert () 成员方法的全部用法。指的一提的是，C++ 11 标准的 set 类模板中，还提供有另外 2 个成员方法，分别为 emplace () 和 emplace_hint () 方法，借助它们不但能实现向 set 容器添加新元素的功能，其实现效率也比 insert () 成员方法更高。

> 有关 set 类模板中 emplace () 和 emplace_hint () 方法的用法，后续章节会做详细介绍。

### emplace 插入值

emplace() 方法的语法格式如下：
```
template <class... Args>  
  pair<iterator,bool> emplace (Args&&... args);
```
其中，参数 (Args&&... args) 指的是，只需要传入构建新元素所需的数据即可，该方法可以自行利用这些数据构建出要添加的元素。比如，若 set 容器中存储的元素类型为自定义的结构体或者类，则在使用 emplace() 方法向容器中添加新元素时，构造新结构体变量（或者类对象）需要多少个数据，就需要为该方法传入相应个数的数据。  
  
另外，该方法的返回值类型为 pair 类型，其包含 2 个元素，一个迭代器和一个 bool 值：
- 当该方法将目标元素成功添加到 set 容器中时，其返回的迭代器指向新插入的元素，同时 bool 值为 true；
- 当添加失败时，则表明原 set 容器中已存在相同值的元素，此时返回的迭代器指向容器中具有相同键的这个元素，同时 bool 值为 false。

下面程序演示 emplace() 方法的具体用法：
```
#include <iostream>
#include <set>
#include <string>
using namespace std;
int main()
{
    //创建并初始化 set 容器
    std::set<string>myset;
    //向 myset 容器中添加元素
    pair<set<string, string>::iterator, bool> ret = myset.emplace("http://c.biancheng.net/stl/");
    cout << "myset size = " << myset.size() << endl;
    cout << "ret.iter = <" << *(ret.first) << ", " << ret.second << ">" << endl;
    return 0;
}
```
程序执行结果为：

```
myset size = 1  
ret.iter = <http://c.biancheng.net/stl/, 1>
```
显然，从执行结果可以看出，通过调用 emplace() 方法，成功向空 myset 容器中添加了一个元素，并且该方法的返回值中就包含指向新添加元素的迭代器。

### emplace_hint 插入元素
emplace_hint() 方法的功能和 emplace() 类似，其语法格式如下：
```
template <class... Args>  
  iterator emplace_hint (const_iterator position, Args&&... args);
```

和 emplace() 方法相比，有以下 2 点不同：
- 该方法需要额外传入一个迭代器，用来指明新元素添加到 set 容器的具体位置（新元素会添加到该迭代器指向元素的前面）；
- 返回值是一个迭代器，而不再是 pair 对象。当成功添加元素时，返回的迭代器指向新添加的元素；反之，如果添加失败，则迭代器就指向 set 容器和要添加元素的值相同的元素。

下面程序演示 emplace_hint() 方法的用法：
```
#include <iostream>
#include <set>
#include <string>
using namespace std;
int main()
{
    //创建并初始化 set 容器
    std::set<string>myset;
    //在 set 容器的指定位置添加键值对
    set<string>::iterator iter = myset.emplace_hint(myset.begin(), "http://c.biancheng.net/stl/");
    cout << "myset size = " << myset.size() << endl;
    cout << *iter << endl;
    return 0;
}
```
程序执行结果为：

```
myset size = 1  
http://c.biancheng.net/stl/
```
注意，和 insert() 方法一样，虽然 emplace_hint() 方法中指定了添加新元素的位置，但 set 容器为了保持数据的有序状态，可能会移动其位置。

## 删除元素

如果想删除 set 容器存储的元素，可以选择用 erase () 或者 clear () 成员方法。

set 类模板中，erase () 方法有 3 种语法格式，分别如下：
```
// 删除 set 容器中值为 val 的元素  
size_type erase (const value_type& val);  
// 删除 position 迭代器指向的元素  
iterator  erase (const_iterator position);  
// 删除 [first, last) 区间内的所有元素  
iterator  erase (const_iterator first, const_iterator last);
```
其中，第 1 种格式的 erase () 方法，其返回值为一个整数，表示成功删除的元素个数；后 2 种格式的 erase () 方法，返回值都是迭代器，其指向的是 set 容器中删除元素之后的第一个元素。

> 注意，如果要删除的元素就是 set 容器最后一个元素，则 erase () 方法返回的迭代器就指向新 set 容器中最后一个元素之后的位置（等价于 end () 方法返回的迭代器）。

下面程序演示了以上 3 种 erase () 方法的用法：

```
#include <iostream>
#include <set>
#include <string>
using namespace std;
int main()
{
    //创建并初始化 set 容器
    std::set<int>myset{1,2,3,4,5};
    cout << "myset size = " << myset.size() << endl;
   
    //1) 调用第一种格式的 erase() 方法
    int num = myset.erase(2); //删除元素 2，myset={1,3,4,5}
    cout << "1、myset size = " << myset.size() << endl;
    cout << "num = " << num << endl;

    //2) 调用第二种格式的 erase() 方法
    set<int>::iterator iter = myset.erase(myset.begin()); //删除元素 1，myset={3,4,5}
    cout << "2、myset size = " << myset.size() << endl;
    cout << "iter->" << *iter << endl;

    //3) 调用第三种格式的 erase() 方法
    set<int>::iterator iter2 = myset.erase(myset.begin(), --myset.end());//删除元素 3,4，myset={5}
    cout << "3、myset size = " << myset.size() << endl;
    cout << "iter2->" << *iter2 << endl;
    return 0;
}
```

程序执行结果为：
```
myset size = 5  
1、myset size = 4  
num = 1  
2、myset size = 3  
iter->3  
3、myset size = 1  
iter 2->5
```
如果需要删除 set 容器中存储的所有元素，可以使用 clear () 成员方法。该方法的语法格式如下：
```
void clear ();
```
显然，该方法不需要传入任何参数，也没有任何返回值。

举个例子：

```
#include <iostream>
#include <set>
#include <string>
using namespace std;
int main()
{
    //创建并初始化 set 容器
    std::set<int>myset{1,2,3,4,5};
    cout << "1、myset size = " << myset.size() << endl;
    //清空 myset 容器
    myset.clear();
    cout << "2、myset size = " << myset.size() << endl;
    return 0;
}
```

程序执行结果为：
```
1、myset size = 5  
2、myset size = 0
```

