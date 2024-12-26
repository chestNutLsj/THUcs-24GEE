
在掌握 Cpp STL map 容器的基础上，本节再讲一个和 map 相似的关联式容器，即 multimap 容器。

所谓 “相似”，指的是 multimap 容器具有和 map 相同的特性，即 multimap 容器也用于存储 pair<const K, T> 类型的键值对（其中 K 表示键的类型，T 表示值的类型），其中各个键值对的键的值不能做修改；并且，该容器也会自行根据键的大小对存储的所有键值对做排序操作。和 map 容器的区别在于，multimap 容器中可以同时存储多（≥2）个键相同的键值对。

和 map 容器一样，实现 multimap 容器的类模板也定义在 `<map>` 头文件，并位于 std 命名空间中。因此，在使用 multimap 容器前，程序应包含如下代码：
```
#include <map>  
using namespace std;
```

multimap 容器类模板的定义如下：

```
template < class Key,                                   // 指定键（key）的类型
           class T,                                     // 指定值（value）的类型
           class Compare = less<Key>,                   // 指定排序规则
           class Alloc = allocator<pair<const Key,T> >  // 指定分配器对象的类型
           > class multimap;
```

可以看到，multimap 容器模板有 4 个参数，其中后 2 个参数都设有默认值。

## 创建 C++ multimap 容器
---------------------

multimap 类模板内部提供有多个构造函数，总的来说，创建 multimap 容器的方式可归为以下 5 种。

1) 通过调用 multimap 类模板的默认构造函数，可以创建一个空的 multimap 容器：
```
std::multimap<std::string, std::string>mymultimap;
```

2) 当然，在创建 multimap 容器的同时，还可以进行初始化操作。比如：
```
//创建并初始化 multimap 容器
multimap<string, string>mymultimap{ {"C 语言教程", " http://c.biancheng.net/c/" },
                                    {"Python 教程", " http://c.biancheng.net/python/" },
                                    {"STL 教程", " http://c.biancheng.net/stl/" } };
```
注意，使用此方式初始化 multimap 容器时，其底层会先将每一个`{key, value}`创建成 pair 类型的键值对，然后再用已建好的各个键值对初始化 multimap 容器。

实际上，我们完全可以先手动创建好键值对，然后再用其初始化 multimap 容器。下面程序使用了 2 种方式创建 pair 类型键值对，再用其初始化 multimap 容器，它们是完全等价的：
```
//借助 pair 类模板的构造函数来生成各个 pair 类型的键值对
multimap<string, string>mymultimap{
    pair<string,string>{"C 语言教程", " http://c.biancheng.net/c/" },
    pair<string,string>{ "Python 教程", " http://c.biancheng.net/python/" },
    pair<string,string>{ "STL 教程", " http://c.biancheng.net/stl/" }
};
//调用 make_pair () 函数，生成键值对元素
//创建并初始化 multimap 容器
multimap<string, string>mymultimap{
    make_pair ("C 语言教程", " http://c.biancheng.net/c/" ),
    make_pair ("Python 教程", " http://c.biancheng.net/python/" ),
    make_pair ("STL 教程", " http://c.biancheng.net/stl/" )
};
```

3) 除此之外，通过调用 multimap 类模板的拷贝（复制）构造函数，也可以初始化新的 multimap 容器。例如：
```
multimap<string, string>newmultimap (mymultimap);
```
由此，就成功创建一个和 mymultimap 完全一样的 newmultimap 容器。

在 C++ 11 标准中，还为 multimap 类增添了移动构造函数。即当有临时的 multimap 容器作为参数初始化新 multimap 容器时，其底层就会调用移动构造函数来实现初始化操作。举个例子：
```
//创建一个会返回临时 multimap 对象的函数
multimap<string, string> dismultimap () {
    multimap<string, string>tempmultimap{ {"C 语言教程", " http://c.biancheng.net/c/" },{"Python 教程", " http://c.biancheng.net/python/" } };
    return tempmultimap;
}  
//调用 multimap 类模板的移动构造函数创建 newMultimap 容器
multimap<string, string>newmultimap (dismultimap ());
```
上面程序中，由于 dismultimap () 函数返回的 tempmultimap 容器是一个临时对象，因此在实现初始化 newmultimap 容器时，底层调用的是 multimap 容器的移动构造函数，而不再是拷贝构造函数。

> 注意，无论是调用复制构造函数还是调用拷贝构造函数，都必须保证这 2 个容器的类型完全一致。

4) multimap 类模板还支持从已有 multimap 容器中，选定某块区域内的所有键值对，用作初始化新 multimap 容器时使用。例如：
```
//创建并初始化 multimap 容器
multimap<string, string>mymultimap{ {"C 语言教程", " http://c.biancheng.net/c/" },
                                    {"Python 教程", " http://c.biancheng.net/python/" },
                                    {"STL 教程", " http://c.biancheng.net/stl/" } };
multimap<string, string>newmultimap (++mymultimap.begin (), mymultimap.end ());
```
这里使用了 multimap 容器的迭代器，选取了 mymultimap 容器中的最后 2 个键值对，用于初始化 newmultimap 容器。

> multimap 容器迭代器，和 map 容器迭代器的用法完全相同，这里不再赘述。

5) 前面讲到，multimap 类模板共可以接收 4 个参数，其中第 3 个参数可用来修改 multimap 容器内部的排序规则。默认情况下，此参数的值为`std::less<T>`，这意味着以下 2 种创建 multimap 容器的方式是等价的：
```
multimap<char, int>mymultimap{ {'a', 1},{'b', 2} };
multimap<char, int, std::less<char>>mymultimap{ {'a', 1},{'b', 2} };
```

mymultimap 容器中键值对的存储顺序为：
```
<a,1>  
<b,2>
```

下面程序利用了 STL 模板库提供的`std::greater<T>`排序函数，实现令 multimap 容器对存储的键值对做降序排序：
```
multimap<char, int, std::greater<char>>mymultimap{ {'a', 1},{'b', 2} };
```

其内部键值对的存储顺序为：
```
<b,2>  
<a,1>
```

> 在某些特定场景中，我们还可以为 multimap 容器自定义排序规则，此部分知识后续将利用整整一节做重点讲解。

## C ++ multimap 容器包含的成员方法
----------------------

下表列出了 multimap 类模板提供的常用成员方法及各自的功能。

| 成员方法              | 功能                                                                                                                                               |
|-------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| begin ()          | 返回指向容器中第一个（注意，是已排好序的第一个）键值对的双向迭代器。如果 multimap 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                              |
| end ()            | 返回指向容器最后一个元素（注意，是已排好序的最后一个）所在位置后一个位置的双向迭代器，通常和 begin () 结合使用。如果 multimap 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                   |
| rbegin ()         | 返回指向最后一个（注意，是已排好序的最后一个）元素的反向双向迭代器。如果 multimap 容器用 const 限定，则该方法返回的是 const 类型的反向双向迭代器。                                                            |
| rend ()           | 返回指向第一个（注意，是已排好序的第一个）元素所在位置前一个位置的反向双向迭代器。如果 multimap 容器用 const 限定，则该方法返回的是 const 类型的反向双向迭代器。                                                     |
| cbegin ()         | 和 begin () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的键值对。                                                                                           |
| cend ()           | 和 end () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的键值对。                                                                                             |
| crbegin ()        | 和 rbegin () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的键值对。                                                                                          |
| crend ()          | 和 rend () 功能相同，只不过在其基础上，增加了 const 属性，不能用于修改容器内存储的键值对。                                                                                            |
| find (key)        | 在 multimap 容器中查找首个键为 key 的键值对，如果成功找到，则返回指向该键值对的双向迭代器；反之，则返回和 end () 方法一样的迭代器。另外，如果 multimap 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                |
| lower_bound (key) | 返回一个指向当前 multimap 容器中第一个大于或等于 key 的键值对的双向迭代器。如果 multimap 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                   |
| upper_bound (key) | 返回一个指向当前 multimap 容器中第一个大于 key 的键值对的迭代器。如果 multimap 容器用 const 限定，则该方法返回的是 const 类型的双向迭代器。                                                        |
| equal_range (key) | 该方法返回一个 pair 对象（包含 2 个双向迭代器），其中 pair. first 和 lower_bound () 方法的返回值等价，pair. second 和 upper_bound () 方法的返回值等价。也就是说，该方法将返回一个范围，该范围中包含的键为 key 的键值对。 |
| empty ()&nbsp;    | 若容器为空，则返回 true；否则 false。                                                                                                                         |
| size ()           | 返回当前 multimap&nbsp; 容器中存有键值对的个数。                                                                                                                 |
| max_size ()       | 返回 multimap 容器所能容纳键值对的最大个数，不同的操作系统，其返回值亦不相同。                                                                                                     |
| insert ()         | 向 multimap 容器中插入键值对。                                                                                                                             |
| erase ()          | 删除 multimap 容器指定位置、指定键（key）值或者指定区域内的键值对。                                                                                                         |
| swap ()           | 交换 2 个 multimap 容器中存储的键值对，这意味着，操作的 2 个键值对的类型必须相同。                                                                                                |
| clear ()          | 清空 multimap 容器中所有的键值对，使 multimap 容器的 size () 为 0。                                                                                                |
| emplace ()        | 在当前 multimap 容器中的指定位置处构造新键值对。其效果和插入键值对一样，但效率更高。                                                                                                  |
| emplace_hint ()   | 在本质上和 emplace () 在 multimap 容器中构造新键值对的方式是一样的，不同之处在于，使用者必须为该方法提供一个指示键值对生成位置的迭代器，并作为该方法的第一个参数。                                                     |
| count (key)       | 在当前 multimap 容器中，查找键为 key 的键值对的个数并返回。                                                                                                            |

和 map 容器相比，multimap 未提供 at () 成员方法，也没有重载 \[\] 运算符。这意味着，map 容器中通过指定键获取指定指定键值对的方式，将不再适用于 multimap 容器。其实这很好理解，因为 multimap 容器中指定的键可能对应多个键值对，而不再是 1 个。

> 另外值的一提的是，由于 multimap 容器可存储多个具有相同键的键值对，因此表中的 lower_bound ()、upper_bound ()、equal_range () 以及 count () 成员方法会经常用到。

下面例子演示了部分成员方法的用法：

```
#include <iostream>
#include <map>  //map
using namespace std;   

int main ()
{
    //创建并初始化 multimap 容器
    multimap<char, int>mymultimap{ {'a', 10},{'b', 20},{'b', 15}, {'c', 30} };
    //输出 mymultimap 容器存储键值对的数量
    cout << mymultimap.size () << endl;
    //输出 mymultimap 容器中存储键为 'b' 的键值对的数量
    cout << mymultimap.count ('b') << endl;

    for (auto iter = mymultimap.begin (); iter != mymultimap.end (); ++iter) {
        cout << iter->first << " " << iter->second << endl;
    }
    return 0;
}
```

程序执行结果为：
```
4  
2  
a 10  
b 20  
b 15  
c 30
```
