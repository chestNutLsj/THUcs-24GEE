
前面章节详细地介绍了 unordered_set 容器的特定和用法，在此基础上，本节再介绍一个类似的 C++ STL 无序容器，即 unordered_multiset 容器。

所谓 “类似”，指的是 unordered_multiset 容器大部分的特性都和 unordered_set 容器相同，包括：
1. unordered_multiset 不以键值对的形式存储数据，而是直接存储数据的值；
2. 该类型容器底层采用的也是哈希表存储结构；
3.  unordered_multiset 容器内部存储的元素，其值不能被修改。

和 unordered_set 容器不同的是，unordered_multiset 容器可以同时存储多个值相同的元素，且这些元素会存储到哈希表中同一个桶（本质就是链表）上。

> 读者可以这样认为，unordered_multiset 除了能存储相同值的元素外，它和 unordered_set 容器完全相同。

另外值得一提的是，实现 unordered_multiset 容器的模板类并没有定义在以该容器名命名的文件中，而是和 unordered_set 容器共用同一个 `<unordered_set>` 头文件，并且也位于 std 命名空间。因此，如果程序中需要使用该类型容器，应包含如下代码：

```
#include <unordered_set>
using namespace std;
```

unordered_multiset 容器类模板的定义如下：

```
template < class Key,            //容器中存储元素的类型
           class Hash = hash<Key>,    //确定元素存储位置所用的哈希函数
           class Pred = equal_to<Key>,   //判断各个元素是否相等所用的函数
           class Alloc = allocator<Key>   //指定分配器对象的类型
           > class unordered_multiset;
```

> 需要说明的是，在 99% 的实际场景中，最多只需要使用前 3 个参数（各自含义如表 1 所示），最后一个参数保持默认值即可。

| 参数                          | 含义                                                                                                                 |
|-----------------------------|--------------------------------------------------------------------------------------------------------------------|
| Key                         | 确定容器存储元素的类型，如果读者将 unordered_multiset 看做是存储键和值相同的键值对的容器，则此参数则用于确定各个键值对的键和值的类型，因为它们是完全相同的，因此一定是同一数据类型的数据。            |
| Hash = hash&lt; Key&gt;     | 指定 unordered_multiset 容器底层存储各个元素时所使用的哈希函数。需要注意的是，默认哈希函数 hash&lt; Key&gt; 只适用于基本数据类型（包括 string 类型），而不适用于自定义的结构体或者类。 |
| Pred = equal_to&lt; Key&gt; | 用于指定 unordered_multiset 容器判断元素值相等的规则。默认情况下，使用 STL 标准库中提供的 equal_to&lt; key&gt; 规则，该规则仅支持可直接用 == 运算符做比较的数据类型。       |

> 总之，如果 unordered_multiset 容器中存储的元素为自定义的数据类型，则默认的哈希函数 hash\<key\> 以及比较函数 equal_to\<key\> 将不再适用，只能自己设计适用该类型的哈希函数和比较函数，并显式传递给 Hash 参数和 Pred 参数。至于如何实现自定义，后续章节会做详细讲解。

## 创建 unordered_multiset 容器

考虑到不同场景的需要，unordered_multiset 容器模板类共提供了以下 4 种创建 unordered_multiset 容器的方式。

1) 调用 unordered_multiset 模板类的默认构造函数，可以创建空的 unordered_multiset 容器。比如：
```
std::unordered_multiset<std::string> umset;
```
由此，就创建好了一个可存储 string 类型值的 unordered_multiset 容器，该容器底层采用默认的哈希函数 hash\<Key\> 和比较函数 equal_to\<Key\>。

2) 当然，在创建 unordered_multiset 容器的同时，可以进行初始化操作。比如：
```
std::unordered_multiset<std::string> umset{ " http://c.biancheng.net/c/" ,
" http://c.biancheng.net/java/" ,
" http://c.biancheng.net/linux/" };
```
通过此方法创建的 umset 容器中，内部存有 3 个 string 类型元素。

3) 还可以调用 unordered_multiset 模板中提供的复制（拷贝）构造函数，将现有 unordered_multiset 容器中存储的元素全部用于为新建 unordered_multiset 容器初始化。

例如，在第二种方式创建好 umset 容器的基础上，再创建并初始化一个 umset 2 容器：
```
std::unordered_multiset<std::string> umset 2 (umset);
```
由此，umap 2 容器中就包含有 umap 容器中所有的元素。

除此之外，C++ 11 标准中还向 unordered_multiset 模板类增加了移动构造函数，即以右值引用的方式，利用临时 unordered_multiset 容器中存储的所有元素，给新建容器初始化。例如：
```
//返回临时 unordered_multiset 容器的函数
std:: unordered_multiset <std::string> retumset () {
    std::unordered_multiset<std::string> tempumset{ " http://c.biancheng.net/c/" ,
                                                    " http://c.biancheng.net/java/" ,
                                                    " http://c.biancheng.net/linux/" };
    return tempumset;
}
//调用移动构造函数，创建 umset 容器
std::unordered_multiset<std::string> umset (retumset ());
```

> 注意，无论是调用复制构造函数还是拷贝构造函数，必须保证 2 个容器的类型完全相同。

4) 当然，如果不想全部拷贝，可以使用 unordered_multiset 类模板提供的迭代器，在现有 unordered_multiset 容器中选择部分区域内的元素，为新建 unordered_multiset 容器初始化。例如：
```
//传入 2 个迭代器，
std::unordered_multiset<std::string> umset 2 (++umset.begin (), umset.end ());
```
通过此方式创建的 umset 2 容器，其内部就包含 umset 容器中除第 1 个元素外的所有其它元素。

## unordered_multiset 的成员方法

值得一提的是，unordered_multiset 模板类中提供的成员方法，无论是种类还是数量，都和 unordered_set 类模板一样，如表 2 所示。

| 成员方法                | 功能                                                                                                                  |
|---------------------|---------------------------------------------------------------------------------------------------------------------|
| begin ()            | 返回指向容器中第一个元素的正向迭代器。                                                                                                 |
| end ();             | 返回指向容器中最后一个元素之后位置的正向迭代器。                                                                                            |
| cbegin ()           | 和 begin () 功能相同，只不过其返回的是 const 类型的正向迭代器。                                                                            |
| cend ()             | 和 end () 功能相同，只不过其返回的是 const 类型的正向迭代器。                                                                              |
| empty ()            | 若容器为空，则返回 true；否则 false。                                                                                            |
| size ()             | 返回当前容器中存有元素的个数。                                                                                                     |
| max_size ()         | 返回容器所能容纳元素的最大个数，不同的操作系统，其返回值亦不相同。                                                                                   |
| find (key)          | 查找以值为 key 的元素，如果找到，则返回一个指向该元素的正向迭代器；反之，则返回一个指向容器中最后一个元素之后位置的迭代器（如果 end () 方法返回的迭代器）。                                |
| count (key)         | 在容器中查找值为 key 的元素的个数。                                                                                                |
| equal_range (key)   | 返回一个 pair 对象，其包含 2 个迭代器，用于表明当前容器中值为 key 的元素所在的范围。                                                                   |
| emplace ()          | 向容器中添加新元素，效率比 insert () 方法高。                                                                                        |
| emplace_hint ()     | 向容器中添加新元素，效率比 insert () 方法高。                                                                                        |
| insert ()           | 向容器中添加新元素。                                                                                                          |
| erase ()            | 删除指定元素。                                                                                                             |
| clear ()            | 清空容器，即删除容器中存储的所有元素。                                                                                                 |
| swap ()             | 交换 2 个 unordered_multiset&nbsp; 容器存储的元素，前提是必须保证这 2 个容器的类型完全相等。                                                      |
| bucket_count ()     | 返回当前容器底层存储元素时，使用桶（一个线性链表代表一个桶）的数量。                                                                                  |
| max_bucket_count () | 返回当前系统中，容器底层最多可以使用多少桶。                                                                                              |
| bucket_size (n)     | 返回第 n 个桶中存储元素的数量。                                                                                                   |
| bucket (key)        | 返回值为 key 的元素所在桶的编号。                                                                                                 |
| load_factor ()      | 返回容器当前的负载因子。所谓负载因子，指的是的当前容器中存储元素的数量（size ()）和使用桶数（bucket_count ()）的比值，即 load_factor () = size () / bucket_count ()。 |
| max_load_factor ()  | 返回或者设置当前 unordered_multiset 容器的负载因子。                                                                                |
| rehash (n)          | 将当前容器底层使用桶的数量设置为 n。                                                                                                 |
| reserve ()          | 将存储桶的数量（也就是 bucket_count () 方法的返回值）设置为至少容纳 count 个元（不超过最大负载因子）所需的数量，并重新整理容器。                                        |
| hash_function ()    | 返回当前容器使用的哈希函数对象。                                                                                                    |

注意，和 unordered_set 容器一样，unordered_multiset 模板类也没有重载 \[\] 运算符，没有提供 at () 成员方法。不仅如此，无论是由哪个成员方法返回的迭代器，都不能用于修改容器中元素的值。

另外，对于互换 2 个相同类型 unordered_multiset 容器存储的所有元素，除了调用表 2 中的 swap () 成员方法外，STL 标准库也提供了 swap () 非成员函数。

下面的样例演示了表 2 中部分成员方法的用法：

```
#include <iostream>
#include <string>
#include <unordered_set>
using namespace std;
int main ()
{
    //创建一个空的 unordered_multiset 容器
    std::unordered_multiset<std::string> umset;
    //给 uset 容器添加数据
    umset.emplace (" http://c.biancheng.net/java/" );
    umset.emplace (" http://c.biancheng.net/c/" );
    umset.emplace (" http://c.biancheng.net/python/" );
    umset.emplace (" http://c.biancheng.net/c/" );
    //查看当前 umset 容器存储元素的个数
    cout << "umset size = " << umset.size () << endl;
    //遍历输出 umset 容器存储的所有元素
    for (auto iter = umset.begin (); iter != umset.end (); ++iter) {
        cout << *iter << endl;
    }
    return 0;
}
```

程序执行结果为：
```
umset size = 4  
http://c.biancheng.net/java/  
http://c.biancheng.net/c/  
http://c.biancheng.net/c/  
http://c.biancheng.net/python/
```
> 注意，表 2 中绝大多数成员方法的用法，都和 unordered_set 容器提供的同名成员方法相同，读者可翻阅前面的文章做详细了解，当然也可以到 [C++ STL 标准库](http://www.cplusplus.com/reference/unordered_set/unordered_multiset/)官网查询。