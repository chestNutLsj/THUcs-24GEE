
前面章节中，对 set 容器做了详细的讲解。回忆一下，set 容器具有以下几个特性：

* 不再以键值对的方式存储数据，因为 set 容器专门用于存储键和值相等的键值对，因此该容器中真正存储的是各个键值对的值（value）；
* set 容器在存储数据时，会根据各元素值的大小对存储的元素进行排序（默认做升序排序）；
* 存储到 set 容器中的元素，虽然其类型没有明确用 const 修饰，但正常情况下它们的值是无法被修改的；
* set 容器存储的元素必须互不相等。

在此基础上，STL 标准库中还提供有一个和 set 容器相似的关联式容器，即 multiset 容器。所谓 “相似”，是指 multiset 容器遵循 set 容器的前 3 个特性，仅在第 4 条特性上有差异。和 set 容器不同的是，multiset 容器可以存储多个值相同的元素。

> 也就是说，multiset 容器和 set 容器唯一的差别在于，multiset 容器允许存储多个值相同的元素，而 set 容器中只能存储互不相同的元素。

和 set 类模板一样，multiset 类模板也定义在 `<set>` 头文件，并位于 std 命名空间中。这意味着，如果想在程序中使用 multiset 容器，该程序代码应包含如下语句：

```
#include <set>
using namespace std;
```

> 注意，第二行代码不是必需的，如果不用，则后续程序中在使用 multiset 容器时，需手动注明 std 命名空间（强烈建议初学者使用）。

multiset 容器类模板的定义如下所示：

```
template < class T,                        // 存储元素的类型
           class Compare = less<T>,        // 指定容器内部的排序规则
           class Alloc = allocator<T> >    // 指定分配器对象的类型
           > class multiset;
```

显然，multiset 类模板有 3 个参数，其中后 2 个参数自带有默认值。值得一提的是，在实际使用中，我们最多只需要使用前 2 个参数即可，第 3 个参数不会用到。

## 创建 multiset 容器
---------------------

创建 multiset 容器，无疑需要调用 multiset 类模板中的构造函数。值得一提的是，multiset 类模板提供的构造函数，和 set 类模板中提供创建 set 容器的构造函数，是完全相同的。这意味着，创建 set 容器的方式，也同样适用于创建 multiset 容器。

考虑到一些读者可能并未系统学习 set 容器，因此这里还是对 multiset 容器的创建做一下详细的介绍。

multiset 类模板中提供了 5 种构造函数，也就代表有 5 种创建 multiset 容器的方式，分别如下。

1) 调用默认构造函数，创建空的 multiset 容器。比如：
```
std::multiset<std::string> mymultiset;
```
由此就创建好了一个 mymultiset 容器，该容器采用默认的 `std::less<T>` 规则，会对存储的 string 类型元素做升序排序。
> 注意，由于 multiset 容器支持随时向内部添加新的元素，因此创建空 multiset 容器的方法比较常用。

2) 除此之外，multiset 类模板还支持在创建 multiset 容器的同时，对其进行初始化。例如：
```
std::multiset<std::string> mymultiset{ "http://c.biancheng.net/java/",
                                       "http://c.biancheng.net/stl/",
                                       "http://c.biancheng.net/python/" };
```
由此即创建好了包含 3 个 string 元素的 mymultiset 容器。由于其采用默认的 `std::less<T>` 规则，因此其内部存储 string 元素的顺序如下所示：
```
" http://c.biancheng.net/java/"  
" http://c.biancheng.net/python/"  
" http://c.biancheng.net/stl/"
```

3) multiset 类模板中还提供了拷贝（复制）构造函数，可以实现在创建新 multiset 容器的同时，将已有 multiset 容器中存储的所有元素全部复制到新 multiset 容器中。

例如，在第 2 种方式创建的 mymultiset 容器的基础上，执行如下代码：
```
std::multiset<std::string> copymultiset(mymultiset);
//等同于
//std::multiset<std::string> copymultiset = mymultiset;
```
该行代码在创建 copymultiset 容器的基础上，还会将 mymultiset 容器中存储的所有元素，全部复制给 copymultiset 容器一份。

另外，C++ 11 标准还为 multiset 类模板新增了移动构造函数，其功能是实现创建新 multiset 容器的同时，利用临时的 multiset 容器为其初始化。比如：
```
multiset<string> retMultiset() {
    std::multiset<std::string> tempmultiset{ "http://c.biancheng.net/java/",
                            "http://c.biancheng.net/stl/",
                            "http://c.biancheng.net/python/" };
    return tempmultiset;
}
std::multiset<std::string> copymultiset(retMultiset());
//等同于
//std::multiset<std::string> copymultiset = retMultiset();
```
注意，由于 retMultiset () 函数的返回值是一个临时 multiset 容器，因此在初始化 copymultiset 容器时，其内部调用的是 multiset 类模板中的移动构造函数，而非拷贝构造函数。
> 显然，无论是调用复制构造函数还是调用拷贝构造函数，都必须保证这 2 个容器的类型完全一致。

4) 在第 3 种方式的基础上，multiset 类模板还支持取已有 multiset 容器中的部分元素，来初始化新 multiset 容器。例如：
```
std::multiset<std::string> mymultiset{ "http://c.biancheng.net/java/",
"http://c.biancheng.net/stl/",
"http://c.biancheng.net/python/" };
std::set<std::string> copymultiset(++mymultiset.begin(), mymultiset.end());
```
以上初始化的 copyset 容器，其内部仅存有如下 2 个 string 字符串：
```
" http://c.biancheng.net/python/"  
" http://c.biancheng.net/stl/"
```

5) 以上几种方式创建的 multiset 容器，都采用了默认的 `std::less<T>` 规则。其实，借助 multiset 类模板定义中的第 2 个参数，我们完全可以手动修改 multiset 容器中的排序规则。
下面样例中，使用了 STL 标准库提供的 std::greater\<T\> 排序方法，作为 multiset 容器内部的排序规则：

```
std::multiset<std::string, std::greater<string> > mymultiset{
    " http://c.biancheng.net/java/" ,
    " http://c.biancheng.net/stl/" ,
    " http://c.biancheng.net/python/" };
```
通过选用`std::greater<string>`降序规则，mymultiset 容器中元素的存储顺序为:
```
" http://c.biancheng.net/stl/"  
" http://c.biancheng.net/python/"  
" http://c.biancheng.net/java/"
```

## C ++ multiset 容器提供的成员方法
----------------------

multiset 容器提供的成员方法，和 set 容器提供的完全一样，如下表所示。

| 成员方法              | 功能                                                                                                                                               |
|-------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| begin ()          | 返回指向容器中第一个（注意，是已排好序的第一个）元素的双向迭代器。如果 multiset 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                               |
| end ()            | 返回指向容器最后一个元素（注意，是已排好序的最后一个）所在位置后一个位置的双向迭代器，通常和 begin () 结合使用。如果 multiset 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                   |
| rbegin ()         | 返回指向最后一个（注意，是已排好序的最后一个）元素的反向双向迭代器。如果 multiset 容器用 const 限定，则该方法返回的是 const 类型的反向双向迭代器。                                                            |
| rend ()           | 返回指向第一个（注意，是已排好序的第一个）元素所在位置前一个位置的反向双向迭代器。如果 multiset 容器用 const 限定，则该方法返回的是 const 类型的反向双向迭代器。                                                     |
| cbegin ()         | 和 begin () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的元素值。                                                                                           |
| cend ()           | 和 end () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的元素值。                                                                                             |
| crbegin ()        | 和 rbegin () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的元素值。                                                                                          |
| crend ()          | 和 rend () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的元素值。                                                                                            |
| find (val)        | 在 multiset 容器中查找值为 val 的元素，如果成功找到，则返回指向该元素的双向迭代器；反之，则返回和 end () 方法一样的迭代器。另外，如果 multiset 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                    |
| lower_bound (val) | 返回一个指向当前 multiset 容器中第一个大于或等于 val 的元素的双向迭代器。如果 multiset 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                    |
| upper_bound (val) | 返回一个指向当前 multiset 容器中第一个大于 val 的元素的迭代器。如果 multiset 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                         |
| equal_range (val) | 该方法返回一个 pair 对象（包含 2 个双向迭代器），其中 pair. first 和 lower_bound () 方法的返回值等价，pair. second 和 upper_bound () 方法的返回值等价。也就是说，该方法将返回一个范围，该范围中包含所有值为 val 的元素。 |
| empty ()          | 若容器为空，则返回 true；否则 false。                                                                                                                         |
| size ()           | 返回当前 multiset 容器中存有元素的个数。                                                                                                                        |
| max_size ()       | 返回 multiset 容器所能容纳元素的最大个数，不同的操作系统，其返回值亦不相同。                                                                                                      |
| insert ()         | 向 multiset 容器中插入元素。                                                                                                                              |
| erase ()          | 删除 multiset 容器中存储的指定元素。                                                                                                                          |
| swap ()           | 交换 2 个 multiset 容器中存储的所有元素。这意味着，操作的 2 个 multiset 容器的类型必须相同。                                                                                      |
| clear ()          | 清空 multiset 容器中所有的元素，即令 multiset 容器的 size () 为 0。                                                                                                |
| emplace ()        | 在当前 multiset 容器中的指定位置直接构造新元素。其效果和 insert () 一样，但效率更高。                                                                                            |
| emplace_hint ()   | 本质上和 emplace () 在 multiset 容器中构造新元素的方式是一样的，不同之处在于，使用者必须为该方法提供一个指示新元素生成位置的迭代器，并作为该方法的第一个参数。                                                       |
| count (val)       | 在当前 multiset 容器中，查找值为 val 的元素的个数，并返回。                                                                                                            |

> 注意，虽然 multiset 容器和 set 容器拥有的成员方法完全相同，但由于 multiset 容器允许存储多个值相同的元素，因此诸如 count ()、find ()、lower_bound ()、upper_bound ()、equal_range () 等方法，更常用于 multiset 容器。

下面程序演示了表中部分成员函数的用法：

```
#include <iostream>
#include <set>
#include <string>
using namespace std;

int main () {
    std::multiset<int> mymultiset{1,2,2,2,3,4,5};
    cout << "multiset size = " << mymultiset.size () << endl;
    cout << "multiset count (2) =" << mymultiset.count (2) << endl;
    //向容器中添加元素 8
    mymultiset.insert (8);
    //删除容器中所有值为 2 的元素
    int num = mymultiset.erase (2);
    cout << "删除了 " << num << " 个元素 2" << endl;
    //输出容器中存储的所有元素
    for (auto iter = mymultiset.begin (); iter != mymultiset.end (); ++iter) {
        cout << *iter << " ";
    }
    return 0;
}
```

程序执行结果为：
```
multiset size = 7  
multiset count (2) =3  
删除了 3 个元素 2  
1 3 4 5 8
```
