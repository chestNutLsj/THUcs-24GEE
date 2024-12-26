
C++ STL 标准库中，除了提供有 unordered_map 无序关联容器，还提供有和 unordered_map 容器非常相似的 unordered_multimap 无序关联容器。

和 unordered_map 容器一样，unordered_multimap 容器也以键值对的形式存储数据，且底层也采用哈希表结构存储各个键值对。两者唯一的不同之处在于，unordered_multimap 容器可以存储多个键相等的键值对，而 unordered_map 容器不行。

> 《[[1-unordered_map#深度剖析无序容器的底层实现机制|深度剖析Cpp无序容器的底层实现机制]]》一文提到，无序容器中存储的各个键值对，都会哈希存到各个桶（本质为链表）中。而对于 unordered_multimap 容器来说，其存储的所有键值对中，键相等的键值对会被哈希到同一个桶中存储。

STL 标准库中实现 unordered_multimap 容器的模板类并没有定义在以自己名称命名的头文件中，而是和 unordered_map 容器一样，定义在 `<unordered_map>` 头文件，且位于 std 命名空间中。因此，在使用 unordered_multimap 容器之前，程序中应包含如下 2 行代码：

```
#include <unordered_map>
using namespace std;
```

unordered_multimap 容器模板的定义如下所示：

```
template < class Key,      //键（key）的类型
           class T,        //值（value）的类型
           class Hash = hash<Key>,  //底层存储键值对时采用的哈希函数
           class Pred = equal_to<Key>,  //判断各个键值对的键相等的规则
           class Alloc = allocator< pair<const Key,T> > // 指定分配器对象的类型
           > class unordered_multimap;
```

以上 5 个参数中，必须显式给前 2 个参数传值，且除极个别的情况外，最多只使用前 4 个参数，它们各自的含义和功能如表 1 所示。

| 参数                          | 含义                                                                                                                           |
|-----------------------------|------------------------------------------------------------------------------------------------------------------------------|
| &lt; key, T&gt;             | 前 2 个参数分别用于确定键值对中键和值的类型，也就是存储键值对的类型。                                                                                         |
| Hash = hash&lt; Key&gt;     | 用于指明容器在存储各个键值对时要使用的哈希函数，默认使用 STL 标准库提供的 hash&lt; key&gt; 哈希函数。注意，默认哈希函数只适用于基本数据类型（包括 string 类型），而不适用于自定义的结构体或者类。             |
| Pred = equal_to&lt; Key&gt; | unordered_multimap 容器可以存储多个键相等的键值对，而判断是否相等的规则，由此参数指定。默认情况下，使用 STL 标准库中提供的 equal_to&lt; key&gt; 规则，该规则仅支持可直接用 == 运算符做比较的数据类型。 |

注意，当 unordered_multimap 容器中存储键值对的键为自定义类型时，默认的哈希函数 hash\<key\> 以及比较函数 equal_to\<key\> 将不再适用，这种情况下，需要我们自定义适用的哈希函数和比较函数，并分别显式传递给 Hash 参数和 Pred 参数。

> 关于给 unordered_multimap 容器自定义哈希函数和比较函数的方法，后续章节会做详细讲解。

## 创建 unordered_multimap 容器

常见的创建 unordered_map 容器的方法有以下几种。

1) 利用 unordered_multimap 容器类模板中的默认构造函数，可以创建空的 unordered_multimap 容器。比如：
```
std::unordered_multimap<std::string, std::string>myummap;
```
由此，就创建好了一个可存储 <string, string> 类型键值对的 unordered_multimap 容器，只不过当前容器是空的，即没有存储任何键值对。

2) 当然，在创建空 unordered_multimap 容器的基础上，可以完成初始化操作。比如：
```
unordered_multimap<string, string>myummap{
    {"Python 教程"," http://c.biancheng.net/python/" },
    {"Java 教程"," http://c.biancheng.net/java/" },
    {"Linux 教程"," http://c.biancheng.net/linux/" } };
```
通过此方法创建的 myummap 容器中，就包含有 3 个键值对。

3) 另外，unordered_multimap 模板中还提供有**复制（拷贝）构造函数**，可以实现在创建 unordered_multimap 容器的基础上，用另一 unordered_multimap 容器中的键值对为其初始化。

例如，在第二种方式创建好 myummap 容器的基础上，再创建并初始化一个 myummap 2 容器：
```
unordered_multimap<string, string>myummap 2 (myummap);
```
由此，刚刚创建好的 myummap 2 容器中，就包含有 myummap 容器中所有的键值对。

除此之外，C++ 11 标准中还向 **unordered_multimap 模板类增加了移动构造函数**，即以右值引用的方式将临时 unordered_multimap 容器中存储的所有键值对，全部复制给新建容器。例如：
```
//返回临时 unordered_multimap 容器的函数
std:: unordered_multimap <std::string, std::string > retUmmap () {
    std::unordered_multimap<std::string, std::string>tempummap{
        {"Python 教程"," http://c.biancheng.net/python/" },
        {"Java 教程"," http://c.biancheng.net/java/" },
        {"Linux 教程"," http://c.biancheng.net/linux/" } };
    return tempummap;
}
//创建并初始化 myummap 容器
std::unordered_multimap<std::string, std::string> myummap (retummap ());
```
注意，无论是调用复制构造函数还是拷贝构造函数，必须保证 2 个容器的类型完全相同。

4) 当然，如果不想全部拷贝，可以使用 unordered_multimap 类模板提供的迭代器，**在现有 unordered_multimap 容器中选择部分区域内的键值对**，为新建 unordered_multimap 容器初始化。例如：
```
//传入 2 个迭代器，
std::unordered_multimap<std::string, std::string> myummap 2 (++myummap.begin (), myummap.end ());
```
通过此方式创建的 myummap 2 容器，其内部就包含 myummap 容器中除第 1 个键值对外的所有其它键值对。

## unordered_multimap 的成员方法

和 unordered_map 容器相比，unordered_multimap 容器的类模板中没有重载 \[\] 运算符，也没有提供 at () 成员方法，除此之外它们完全一致。

> 没有提供 \[\] 运算符和 at () 成员方法，意味着 unordered_multimap 容器无法通过指定键获取该键对应的值，因为该容器允许存储多个键相等的键值对，每个指定的键可能对应多个不同的值。

unordered_multimap 类模板提供的成员方法如表 2 所示。  

| 成员方法                | 功能                                                                                                                                      |
|---------------------|-----------------------------------------------------------------------------------------------------------------------------------------|
| begin ()            | 返回指向容器中第一个键值对的正向迭代器。                                                                                                                    |
| end ()&nbsp;        | 返回指向容器中最后一个键值对之后位置的正向迭代器。                                                                                                               |
| cbegin ()           | 和 begin () 功能相同，只不过在其基础上增加了 const 属性，即该方法返回的迭代器不能用于修改容器内存储的键值对。                                                                         |
| cend ()             | 和 end () 功能相同，只不过在其基础上，增加了 const 属性，即该方法返回的迭代器不能用于修改容器内存储的键值对。                                                                          |
| empty ()            | 若容器为空，则返回 true；否则 false。                                                                                                                |
| size ()             | 返回当前容器中存有键值对的个数。                                                                                                                        |
| max_size ()         | 返回容器所能容纳键值对的最大个数，不同的操作系统，其返回值亦不相同。                                                                                                      |
| find (key)          | 查找以 key 为键的键值对，如果找到，则返回一个指向该键值对的正向迭代器；反之，则返回一个指向容器中最后一个键值对之后位置的迭代器（如果 end () 方法返回的迭代器）。                                                 |
| count (key)         | 在容器中查找以 key 键的键值对的个数。                                                                                                                   |
| equal_range (key)   | 返回一个 pair 对象，其包含 2 个迭代器，用于表明当前容器中键为 key 的键值对所在的范围。                                                                                      |
| emplace ()          | 向容器中添加新键值对，效率比 insert () 方法高。                                                                                                           |
| emplace_hint ()     | 向容器中添加新键值对，效率比 insert () 方法高。                                                                                                           |
| insert ()&nbsp;     | 向容器中添加新键值对。                                                                                                                             |
| erase ()            | 删除指定键值对。                                                                                                                                |
| clear ()&nbsp;      | 清空容器，即删除容器中存储的所有键值对。                                                                                                                    |
| swap ()             | 交换 2 个 unordered_multimap 容器存储的键值对，前提是必须保证这 2 个容器的类型完全相等。                                                                               |
| bucket_count ()     | 返回当前容器底层存储键值对时，使用桶（一个线性链表代表一个桶）的数量。                                                                                                     |
| max_bucket_count () | 返回当前系统中，unordered_multimap 容器底层最多可以使用多少桶。                                                                                               |
| bucket_size (n)     | 返回第 n 个桶中存储键值对的数量。                                                                                                                      |
| bucket (key)        | 返回以 key 为键的键值对所在桶的编号。                                                                                                                   |
| load_factor ()      | 返回 unordered_multimap 容器中当前的负载因子。负载因子，指的是的当前容器中存储键值对的数量（size ()）和使用桶数（bucket_count ()）的比值，即 load_factor () = size () / bucket_count ()。 |
| max_load_factor ()  | 返回或者设置当前 unordered_multimap 容器的负载因子。                                                                                                    |
| rehash (n)          | 将当前容器底层使用桶的数量设置为 n。                                                                                                                     |
| reserve ()          | 将存储桶的数量（也就是 bucket_count () 方法的返回值）设置为至少容纳 count 个元（不超过最大负载因子）所需的数量，并重新整理容器。                                                            |
| hash_function ()    | 返回当前容器使用的哈希函数对象。                                                                                                                        |

> 注意，对于实现互换 2 个相同类型 unordered_multimap 容器的键值对，除了可以调用该容器模板类中提供的 swap () 成员方法外，STL 标准库还提供了同名的 swap () 非成员函数。

下面的样例演示了表中部分成员方法的用法：

```
#include <iostream>
#include <string>
#include <unordered_map>
using namespace std;

int main ()
{
    //创建空容器
    std::unordered_multimap<std::string, std::string> myummap;
    //向空容器中连续添加 5 个键值对
    myummap.emplace ("Python 教程", " http://c.biancheng.net/python/" );
    myummap.emplace ("STL 教程", " http://c.biancheng.net/stl/" );
    myummap.emplace ("Java 教程", " http://c.biancheng.net/java/" );
    myummap.emplace ("C 教程", " http://c.biancheng.net" );
    myummap.emplace ("C 教程", " http://c.biancheng.net/c/" );
    //输出 muummap 容器存储键值对的个数
    cout << "myummmap size = " << myummap.size () << endl;
    //利用迭代器输出容器中存储的所有键值对
    for (auto iter = myummap.begin (); iter != myummap.end (); ++iter) {
        cout << iter->first << " " << iter->second << endl;
    }
    return 0;
}
```

程序执行结果为：
```
myummmap size = 5  
Python 教程 http://c.biancheng.net/python/  
C 教程 http://c.biancheng.net  
C 教程 http://c.biancheng.net/c/  
STL 教程 http://c.biancheng.net/stl/  
Java 教程 http://c.biancheng.net/java/
```

> 值得一提的是，unordered_multimap 模板提供的所有成员方法的用法，都和 unordered_map 提供的同名成员方法的用法完全相同（仅是调用者发生了改变），由于在讲解 unordered_map 容器时，已经对大部分成员方法的用法做了详细的讲解，后续不再做重复性地赘述。