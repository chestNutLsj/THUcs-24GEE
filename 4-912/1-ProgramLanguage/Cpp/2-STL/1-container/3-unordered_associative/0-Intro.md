
继 map、multimap、set、multiset 关联式容器之后，从本节开始，再讲解一类 “特殊” 的关联式容器，它们常被称为“无序容器”、“哈希容器” 或者 “无序关联容器”。

> 注意，无序容器是 C++ 11 标准才正式引入到 STL 标准库中的，这意味着如果要使用该类容器，则必须选择支持 C++ 11 标准的编译器。

和关联式容器一样，无序容器也使用键值对（pair 类型）的方式存储数据。不过，本教程将二者分开进行讲解，因为它们有本质上的不同：
* **关联式容器的底层实现采用的树存储结构**，更确切的说是**红黑树结构**；
* **无序容器的底层实现采用的是哈希表**的存储结构。

> C++ STL 底层采用哈希表实现无序容器时，会将所有数据存储到一整块连续的内存空间中，并且当数据存储位置发生冲突时，解决方法选用的是 “链地址法”（又称 “开链法”）。有关哈希表存储结构，读者可阅读《[哈希表 (散列表) 详解](http://c.biancheng.net/view/3437.html)》一文做详细了解。

基于底层实现采用了不同的数据结构，因此和关联式容器相比，无序容器具有以下 2 个特点：
1. *无序容器内部存储的键值对是无序*的，各键值对的存储位置取决于该键值对中的键，
2. 和关联式容器相比，*无序容器擅长通过指定键查找对应的值*（平均时间复杂度为 $O (1)$ ）；但对于使用迭代器遍历容器中存储的元素，无序容器的执行效率则不如关联式容器。

## STL 无序容器种类
--------------

和关联式容器一样，无序容器只是一类容器的统称，其包含有 4 个具体容器，分别为 unordered_map、unordered_multimap、unordered_set 以及 unordered_multiset。

下表对这 4 种无序容器的功能做了详细的介绍。

| 无序容器                | 功能                                                                                                                           |
|---------------------|------------------------------------------------------------------------------------------------------------------------------|
| unordered_map&nbsp; | 存储键值对 &lt; key, value&gt; 类型的元素，其中各个键值对键的值不允许重复，且该容器中存储的键值对是无序的。                                                             |
| unordered_multimap  | 和 unordered_map 唯一的区别在于，该容器允许存储多个键相同的键值对。                                                                                    |
| unordered_set       | 不再以键值对的形式存储数据，而是直接存储数据元素本身（当然也可以理解为，该容器存储的全部都是键 key 和值 value 相等的键值对，正因为它们相等，因此只存储 value 即可）。另外，该容器存储的元素不能重复，且容器内部存储的元素也是无序的。 |
| unordered_multiset  | 和 unordered_set 唯一的区别在于，该容器允许存储值相同的元素。                                                                                       |

可能读者已经发现，以上 4 种无序容器的名称，仅是在前面所学的 4 种关联式容器名称的基础上，添加了 "unordered_"。如果读者已经学完了 map、multimap、set 和 multiset 容器不难发现，以 map 和 unordered_map 为例，其实它们仅有一个区别，即 map 容器内存会对存储的键值对进行排序，而 unordered_map 不会。

> 也就是说，C++ 11 标准的 STL 中，在已提供有 4 种关联式容器的基础上，又新增了各自的 “unordered” 版本（无序版本、哈希版本），提高了查找指定元素的效率。

有读者可能会问，既然无序容器和之前所学的关联式容器类似，那么在实际使用中应该选哪种容器呢？总的来说，实际场景中如果涉及大量遍历容器的操作，建议首选关联式容器；反之，如果更多的操作是通过键获取对应的值，则应首选无序容器。

为了加深读者对无序容器的认识，这里以 unordered_map 容器为例，举个例子（不必深究该容器的具体用法）：

```
#include <iostream>
#include <string>
#include <unordered_map>
using namespace std;

int main()
{
    //创建并初始化一个 unordered_map 容器，其存储的 <string,string> 类型的键值对
    std::unordered_map<std::string, std::string> my_uMap{
        {"C语言教程","http://c.biancheng.net/c/"},
        {"Python教程","http://c.biancheng.net/python/"},
        {"Java教程","http://c.biancheng.net/java/"} };
    //查找指定键对应的值，效率比关联式容器高
    string str = my_uMap.at("C语言教程");
    cout << "str = " << str << endl;

    //使用迭代器遍历哈希容器，效率不如关联式容器
    for (auto iter = my_uMap.begin(); iter != my_uMap.end(); ++iter)
    {
        //pair 类型键值对分为 2 部分
        cout << iter->first << " " << iter->second << endl;
    }
    return 0;
}
```

程序执行结果为：
```
str = http://c.biancheng.net/c/  
C 语言教程 http://c.biancheng.net/c/  
Python 教程 http://c.biancheng.net/python/  
Java 教程 http://c.biancheng.net/java/
```
