
## C++ STL 关联式容器是什么
----------------

通过学习所有的序列式容器不难发现，无论是哪种序列式容器，其存储的都是 C++ 基本数据类型（诸如 int、double、float、string 等）或使用结构体自定义类型的元素。例如，如下是一个存储 int 类型元素的 vector 容器：
```
std::vector<int> primes {2, 3, 5, 7, 11, 13, 17, 19};
```

关联式容器则大不一样，此类容器在存储元素值的同时，还会为各元素额外再配备一个值（又称为 “键”，其本质也是一个 C++ 基础数据类型或自定义类型的元素），它的功能是在使用关联式容器的过程中，如果已知目标元素的键的值，则直接通过该键就可以找到目标元素，而无需再通过遍历整个容器的方式。

> 弃用序列式容器，转而选用关联式容器存储元素，往往就是看中了关联式容器可以快速查找、读取或者删除所存储的元素，同时该类型容器插入元素的效率也比序列式容器高。

也就是说，使用关联式容器存储的元素，都是一个一个的 “键值对”（ <key,value> ），这是和序列式容器最大的不同。除此之外，序列式容器中存储的元素默认都是未经过排序的，而使用关联式容器存储的元素，**默认会根据各元素的键值的大小做升序排序**。

注意，关联式容器所具备的这些特性，归咎于 STL 标准库在实现该类型容器时，底层选用了 「红黑树」这种数据结构来组织和存储各个键值对。

## C++ STL 关联式容器种类
---------------

C++ STL 标准库提供了 4 种关联式容器，分别为 map、set、multimap、multiset，其各自的特点如表 1 所示。

| 关联式容器名称  | 特点                                                                                                                               |
|----------|----------------------------------------------------------------------------------------------------------------------------------|
| map      | 定义在 &lt; map&gt; 头文件中，使用该容器存储的数据，其各个元素的键必须是唯一的（即不能重复），该容器会根据各元素键的大小，默认进行升序排序（调用 std::less&lt; T&gt;）。                            |
| set      | 定义在 &lt; set&gt; 头文件中，使用该容器存储的数据，各个元素键和值完全相同，且各个元素的值不能重复（保证了各元素键的唯一性）。该容器会自动根据各个元素的键（其实也就是元素值）的大小进行升序排序（调用 std::less&lt; T&gt;）。 |
| multimap | 定义在 &lt; map&gt; 头文件中，和 map 容器唯一的不同在于，multimap 容器中存储元素的键可以重复。                                                                    |
| multiset | 定义在 &lt; set&gt; 头文件中，和 set 容器唯一的不同在于，multiset 容器中存储元素的值可以重复（一旦值重复，则意味着键也是重复的）。                                                  |

> 除此之外，C++ 11 还新增了 4 种哈希容器，即 unordered_map、unordered_multimap 以及 unordered_set、unordered_multiset。严格来说，它们也属于关联式容器，但由于哈希容器底层采用的是哈希表，而不是红黑树，因此本教程将它们分开进行讲解（有关哈希容器，将放在后续章节做详细讲解）。

为了让读者直观地认识到关联式容器的特性，这里为 map 容器为例，编写了一个样例（如下所示）。对于该程序，读者只需体会关联式容器的特性即可，无需纠结 map 容器的具体用法。

```
#include <iostream>
#include <map> //使用 map 容器，必须引入该头文件
#include <string>
using namespace std;
int main ()
{
    //创建一个空的 map 关联式容器，该容器中存储的键值对，其中键为 string 字符串，值也为 string 字符串类型
    map<string, string> mymap;
    //向 mymap 容器中添加数据
    mymap[" http://c.biancheng.net/c/" ] = "C 语言教程";
    mymap[" http://c.biancheng.net/python/" ] = "Python 教程";
    mymap[" http://c.biancheng.net/java/" ] = "Java 教程";

    //使用 map 容器的迭代器，遍历 mymap 容器，并输出其中存储的各个键值对
    for (map<string, string>:: iterator it = mymap.begin (); it != mymap.end (); ++it) {
        //输出各个元素中的键和值
        cout << it->first << " => " << it->second << '\n';
    }
    return 0;
}
```

程序输出结果为：
```
http://c.biancheng.net/c/ => C 语言教程  
http://c.biancheng.net/java/ => Java 教程  
http://c.biancheng.net/python/ => Python 教程
```

通过分析该程序的执行过程不难看出，mymap 关联式容器中的存储了以下 3 个键值对：
```
<"http://c.biancheng.net/c/", "C 语言教程">
<"http://c.biancheng.net/python/", "Python 教程">
<"http://c.biancheng.net/java/", "Java 教程">
```
但需要注意的一点是，由于 map 容器在存储元素时，会根据各个元素键的大小自动调整元素的顺序（默认按照升序排序），因此该容器最终存储的元素顺序为：
```
<"http://c.biancheng.net/c/", "C 语言教程">  
<"http://c.biancheng.net/java/", "Java 教程">  
<"http://c.biancheng.net/python/", "Python 教程">
```

## C++ STL pair 用法

我们知道，关联式容器存储的是 “键值对” 形式的数据，比如：
```
<"C 语言教程", "http://c.biancheng.net/c/">  
<"[Python](http://c.biancheng.net/python/) 教程 "," http://c.biancheng.net/python/">  
<"[Java](http://c.biancheng.net/java/) 教程 "," http://c.biancheng.net/java/">
```
如上所示，每行都表示一个键值对，其中第一个元素作为键（key），第二个元素作为值（value）。

> 注意，基于各个关联式容器存储数据的特点，只有各个键值对中的键和值全部对应相等时，才能使用 set 和 multiset 关联式容器存储，否则就要选用 map 或者 multimap 关联式容器。

考虑到 “键值对” 并不是普通类型数据，C++ STL标准库提供了 pair 类模板，其专门用来将 2 个普通元素 first 和 second（可以是 C++ 基本数据类型、结构体、类自定的类型）创建成一个新元素 `<first, second>`。通过其构成的元素格式不难看出，使用 pair 类模板来创建 “键值对” 形式的元素，再合适不过。

注意，pair 类模板定义在 `<utility>` 头文件中，所以在使用该类模板之前，需引入此头文件。另外值得一提的是，在 C++ 11 标准之前，pair 类模板中提供了以下 3 种构造函数：

```
#1) 默认构造函数，即创建空的 pair 对象
pair();
#2) 直接使用 2 个元素初始化成 pair 对象
pair (const first_type& a, const second_type& b);
#3) 拷贝（复制）构造函数，即借助另一个 pair 对象，创建新的 pair 对象
template<class U, class V> pair (const pair<U,V>& pr);
```

在 C++ 11 标准中，在引入 [[20-Special-members#^d293e1|右值引用]] 的基础上，pair 类模板中又增添了如下 2 个构造函数：

```
#4) 移动构造函数
template<class U, class V> pair (pair<U,V>&& pr);
#5) 使用右值引用参数，创建 pair 对象
template<class U, class V> pair (U&& a, V&& b);
```

> 除此之外，C++ 11 标准中 pair 类模板还新增加了如下一种构造函数：`pair (piecewise_construct_t pwc, tuple<Args1...> first_args, tuple<Args2...> second_args);`
> 
> ，该构造 pair 类模板的方式很少用到，因此本节不再对其进行详细介绍，感兴趣的读者可自行查阅资料。

下面程序演示了以上几种创建 pair 对象的方法：

```
#include <iostream>
#include <utility>      // pair
#include <string>       // string
using namespace std;
int main() {
    // 调用构造函数 1，也就是默认构造函数
    pair <string, double> pair1;
    // 调用第 2 种构造函数
    pair <string, string> pair2("STL教程","http://c.biancheng.net/stl/");  
    // 调用拷贝构造函数
    pair <string, string> pair3(pair2);
    //调用移动构造函数
    pair <string, string> pair4(make_pair("C++教程", "http://c.biancheng.net/cplus/"));
    // 调用第 5 种构造函数
    pair <string, string> pair5(string("Python教程"), string("http://c.biancheng.net/python/"));  
   
    cout << "pair1: " << pair1.first << " " << pair1.second << endl;
    cout << "pair2: "<< pair2.first << " " << pair2.second << endl;
    cout << "pair3: " << pair3.first << " " << pair3.second << endl;
    cout << "pair4: " << pair4.first << " " << pair4.second << endl;
    cout << "pair5: " << pair5.first << " " << pair5.second << endl;
    return 0;
}
```

程序输出结果为：
```
pair 1:  0  
pair 2: STL 教程 http://c.biancheng.net/stl/  
pair 3: STL 教程 http://c.biancheng.net/stl/  
pair 4: C++ 教程 http://c.biancheng.net/cplus/  
pair 5: Python 教程 http://c.biancheng.net/python/
```

上面程序在创建 pair 4 对象时，调用了 make_pair () 函数，它也是 \<utility\> 头文件提供的，其功能是生成一个 pair 对象。因此，当我们将 make_pair () 函数的返回值（是一个临时对象）作为参数传递给 pair () 构造函数时，其调用的是移动构造函数，而不是拷贝构造函数。

在上面程序的基础上，C++ 11 还允许我们手动为 pair1 对象赋值，比如：

```
pair1.first = "Java 教程";
pair1.second = " http://c.biancheng.net/java/" ;
cout << "new pair 1: " << pair1.first << " " << pair1.second << endl;
```

执行结果为：
```
new pair 1: Java 教程 http://c.biancheng.net/java/
```

同时，上面程序中 pair4 对象的创建过程，还可以写入如下形式，它们是完全等价的：

```
pair <string, string> pair4 = make_pair ("C++教程", " http://c.biancheng.net/cplus/" );
cout << "pair4: " << pair4.first << " " << pair4.second << endl;
```

`<utility>`头文件中除了提供创建 pair 对象的方法之外，还为 pair 对象重载了 <、<=、>、>=、\=\=、!= 这 6 的运算符，其运算规则是：对于进行比较的 2 个 pair 对象，先比较 pair. first 元素的大小，如果相等则继续比较 pair. second 元素的大小。

> 注意，对于进行比较的 2 个 pair 对象，其对应的键和值的类型必须相同，否则将没有可比性，同时编译器提示没有相匹配的运算符，即找不到合适的重载运算符。

举个例子：

```
#include <iostream>
#include <utility>      // pair
#include <string>       // string
using namespace std;
int main () {
    pair <string, int> pair 1 ("STL 教程", 20);
    pair <string, int> pair 2 ("C++教程", 20);
    pair <string, int> pair 3 ("C++教程", 30);
    //pair 1 和 pair 2 的 key 不同，value 相同
    if (pair 1 != pair 2) {
        cout << "pair != pair 2" << endl;
    }
    //pair 2 和 pair 3 的 key 相同，value 不同
    if (pair 2 != pair 3) {
        cout << "pair 2 != pair 3" << endl;
    }
    return 0;
}
```

程序执行结果为：
```
pair != pair 2  
pair 2 != pair 3
```
最后需要指出的是，pair 类模板还提供有一个 swap () 成员函数，能够互换 2 个 pair 对象的键值对，其操作成功的前提是这 2 个 pair 对象的键和值的类型要相同。例如：

```
#include <iostream>
#include <utility>      // pair
#include <string>       // string
using namespace std;
int main () {
    pair <string, int> pair 1 ("pair", 10);                   
    pair <string, int> pair 2 ("pair 2", 20);
    //交换 pair 1 和 pair 2 的键值对
    pair 1.swap (pair 2);
    cout << "pair 1: " << pair 1. first << " " << pair 1. second << endl;
    cout << "pair 2: " << pair 2. first << " " << pair 2. second << endl;
    return 0;
}
```

程序执行结果为：
```
pair 1: pair2 20  
pair 2: pair 10
```