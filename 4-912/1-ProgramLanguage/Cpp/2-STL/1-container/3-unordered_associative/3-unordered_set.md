
我们知道，C++ 11 为 STL 标准库增添了 4 种无序（哈希）容器，前面已经对 unordered_map 和 unordered_multimap 容器做了详细的介绍，本节再讲解一种无序容器，即 unordered_set 容器。

unordered_set 容器，可直译为 “无序 set 容器”，即 unordered_set 容器和 set 容器很像，唯一的区别就在于 set 容器会自行对存储的数据进行排序，而 unordered_set 容器不会。

总的来说，unordered_set 容器具有以下几个特性：
1. 不再以键值对的形式存储数据，而是直接存储数据的值；
2. 容器内部存储的各个元素的值都互不相等，且不能被修改。
3. 不会对内部存储的数据进行排序

> 对于 unordered_set 容器不以键值对的形式存储数据，读者也可以这样认为，即 unordered_set 存储的都是键和值相等的键值对，为了节省存储空间，该类容器在实际存储时选择只存储每个键值对的值。

另外，实现 unordered_set 容器的模板类定义在 `<unordered_set>` 头文件，并位于 std 命名空间中。这意味着，如果程序中需要使用该类型容器，则首先应该包含如下代码：

```
#include <unordered_set>
using namespace std;
```

unordered_set 容器的类模板定义如下：

```
template < class Key,            //容器中存储元素的类型
           class Hash = hash<Key>,    //确定元素存储位置所用的哈希函数
           class Pred = equal_to<Key>,   //判断各个元素是否相等所用的函数
           class Alloc = allocator<Key>   //指定分配器对象的类型
           > class unordered_set;
```

可以看到，以上 4 个参数中，只有第一个参数没有默认值，这意味着如果我们想创建一个 unordered_set 容器，至少需要手动传递 1 个参数。事实上，在 99% 的实际场景中最多只需要使用前 3 个参数（各自含义如表 1 所示），最后一个参数保持默认值即可。

| 参数                          | 含义                                                                                                                                      |
|-----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| Key                         | 确定容器存储元素的类型，如果读者将 unordered_set 看做是存储键和值相同的键值对的容器，则此参数则用于确定各个键值对的键和值的类型，因为它们是完全相同的，因此一定是同一数据类型的数据。                                      |
| Hash = hash&lt; Key&gt;     | 指定 unordered_set 容器底层存储各个元素时，所使用的哈希函数。需要注意的是，默认哈希函数 hash&lt; Key&gt; 只适用于基本数据类型（包括 string 类型），而不适用于自定义的结构体或者类。                          |
| Pred = equal_to&lt; Key&gt; | unordered_set&nbsp; 容器内部不能存储相等的元素，而衡量 2 个元素是否相等的标准，取决于该参数指定的函数。 默认情况下，使用 STL 标准库中提供的 equal_to&lt; key&gt; 规则，该规则仅支持可直接用 == 运算符做比较的数据类型。 |

> 注意，如果 unordered_set 容器中存储的元素为**自定义的数据类型，则默认的哈希函数 hash\<key\> 以及比较函数 equal_to\<key\> 将不再适用**，只能自己设计适用该类型的哈希函数和比较函数，并显式传递给 Hash 参数和 Pred 参数。至于如何实现自定义，后续章节会做详细讲解。

## 创建 unordered_set 容器

前面介绍了如何创建 unordered_map 和 unordered_multimap 容器，值得一提的是，创建它们的所有方式完全适用于 unordereded_set 容器。不过，考虑到一些读者可能尚未学习其它无序容器，因此这里还是讲解一下创建 unordered_set 容器的几种方法。

1) 通过调用 unordered_set 模板类的默认构造函数，可以创建空的 unordered_set 容器。比如：
```
std::unordered_set<std::string> uset;
```
由此，就创建好了一个可存储 string 类型值的 unordered_set 容器，该容器底层采用默认的哈希函数 hash\<Key\> 和比较函数 equal_to\<Key\>。

2) 当然，在创建 unordered_set 容器的同时，可以完成初始化操作。比如：
```
std::unordered_set<std::string> uset{ " http://c.biancheng.net/c/" ,
" http://c.biancheng.net/java/" ,
" http://c.biancheng.net/linux/" };
```
通过此方法创建的 uset 容器中，就包含有 3 个 string 类型元素。

3) 还可以调用 unordered_set 模板中提供的**复制（拷贝）构造函数**，将现有 unordered_set 容器中存储的元素全部用于为新建 unordered_set 容器初始化。

例如，在第二种方式创建好 uset 容器的基础上，再创建并初始化一个 uset 2 容器：
```
std::unordered_set<std::string> uset 2 (uset);
```
由此，umap 2 容器中就包含有 umap 容器中所有的元素。

除此之外，C++ 11 标准中还向 unordered_set 模板类**增加了移动构造函数，即以右值引用的方式**，利用临时 unordered_set 容器中存储的所有元素，给新建容器初始化。例如：
```
//返回临时 unordered_set 容器的函数
std:: unordered_set <std::string> retuset () {
    std::unordered_set<std::string> tempuset{ " http://c.biancheng.net/c/" ,
                                              " http://c.biancheng.net/java/" ,
                                              " http://c.biancheng.net/linux/" };
    return tempuset;
}
//调用移动构造函数，创建 uset 容器
std::unordered_set<std::string> uset (retuset ());
```

> 注意，无论是调用复制构造函数还是拷贝构造函数，必须保证 2 个容器的类型完全相同。

4) 当然，如果不想全部拷贝，可以使用 unordered_set 类模板提供的迭代器，在现有 unordered_set 容器中选择部分区域内的元素，为新建 unordered_set 容器初始化。例如：
```
//传入 2 个迭代器，
std::unordered_set<std::string> uset 2 (++uset.begin (), uset.end ());
```
通过此方式创建的 uset 2 容器，其内部就包含 uset 容器中除第 1 个元素外的所有其它元素。

## unordered_set 的成员方法

unordered_set 类模板中，提供了如下表所示的成员方法。

| 成员方法                | 功能                                                                                                                                      |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| begin ()            | 返回指向容器中第一个元素的正向迭代器。                                                                                                                     |
| end ();             | 返回指向容器中最后一个元素之后位置的正向迭代器。                                                                                                                |
| cbegin ()           | 和 begin () 功能相同，只不过其返回的是 const 类型的正向迭代器。                                                                                                |
| cend ()             | 和 end () 功能相同，只不过其返回的是 const 类型的正向迭代器。                                                                                                  |
| empty ()            | 若容器为空，则返回 true；否则 false。                                                                                                                |
| size ()             | 返回当前容器中存有元素的个数。                                                                                                                         |
| max_size ()         | 返回容器所能容纳元素的最大个数，不同的操作系统，其返回值亦不相同。                                                                                                       |
| find (key)          | 查找以值为 key 的元素，如果找到，则返回一个指向该元素的正向迭代器；反之，则返回一个指向容器中最后一个元素之后位置的迭代器（如果 end () 方法返回的迭代器）。                                                    |
| count (key)         | 在容器中查找值为 key 的元素的个数。                                                                                                                    |
| equal_range (key)   | 返回一个 pair 对象，其包含 2 个迭代器，用于表明当前容器中值为 key 的元素所在的范围。                                                                                       |
| emplace ()          | 向容器中添加新元素，效率比 insert () 方法高。                                                                                                            |
| emplace_hint ()     | 向容器中添加新元素，效率比 insert () 方法高。                                                                                                            |
| insert ()           | 向容器中添加新元素。                                                                                                                              |
| erase ()            | 删除指定元素。                                                                                                                                 |
| clear ()            | 清空容器，即删除容器中存储的所有元素。                                                                                                                     |
| swap ()             | 交换 2 个 unordered_set&nbsp; 容器存储的元素，前提是必须保证这 2 个容器的类型完全相等。                                                                               |
| bucket_count ()     | 返回当前容器底层存储元素时，使用桶（一个线性链表代表一个桶）的数量。                                                                                                      |
| max_bucket_count () | 返回当前系统中，unordered_set&nbsp; 容器底层最多可以使用多少桶。                                                                                              |
| bucket_size (n)     | 返回第 n 个桶中存储元素的数量。                                                                                                                       |
| bucket (key)        | 返回值为 key 的元素所在桶的编号。                                                                                                                     |
| load_factor ()      | 返回 unordered_set&nbsp; 容器中当前的负载因子。负载因子，指的是的当前容器中存储元素的数量（size ()）和使用桶数（bucket_count ()）的比值，即 load_factor () = size () / bucket_count ()。 |
| max_load_factor ()  | 返回或者设置当前 unordered_set&nbsp; 容器的负载因子。                                                                                                   |
| rehash (n)          | 将当前容器底层使用桶的数量设置为 n。                                                                                                                     |
| reserve ()          | 将存储桶的数量（也就是 bucket_count () 方法的返回值）设置为至少容纳 count 个元（不超过最大负载因子）所需的数量，并重新整理容器。                                                            |
| hash_function ()    | 返回当前容器使用的哈希函数对象。                                                                                                                        |

注意，此容器模板类中没有重载 \[\] 运算符，也没有提供 at () 成员方法。不仅如此，由于 unordered_set 容器内部存储的元素值不能被修改，因此无论使用那个迭代器方法获得的迭代器，都不能用于修改容器中元素的值。

另外，对于实现互换 2 个相同类型 unordered_set 容器的所有元素，除了调用表 2 中的 swap () 成员方法外，还可以使用 STL 标准库提供的 swap () 非成员函数，它们具有相同的名称，用法也相同（都只需要传入 2 个参数即可），仅是调用方式上有差别。

下面的样例演示了表 2 中部分成员方法的用法：

```
#include <iostream>
#include <string>
#include <unordered_set>
using namespace std;

int main ()
{
    //创建一个空的 unordered_set 容器
    std::unordered_set<std::string> uset;
    //给 uset 容器添加数据
    uset.emplace (" http://c.biancheng.net/java/" );
    uset.emplace (" http://c.biancheng.net/c/" );
    uset.emplace (" http://c.biancheng.net/python/" );
    //查看当前 uset 容器存储元素的个数
    cout << "uset size = " << uset.size () << endl;
    //遍历输出 uset 容器存储的所有元素
    for (auto iter = uset.begin (); iter != uset.end (); ++iter) {
        cout << *iter << endl;
    }
    return 0;
}
```

程序执行结果为：
```
uset size = 3  
http://c.biancheng.net/java/  
http://c.biancheng.net/c/  
http://c.biancheng.net/python/
```
