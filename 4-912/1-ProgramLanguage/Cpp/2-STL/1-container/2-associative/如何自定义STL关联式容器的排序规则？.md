
前面在讲解如何创建 map、multimap、set 以及 multiset 容器时，遗留了一个问题，即如何自定义关联式容器中的排序规则？

> 实际上，为关联式容器自定义排序规则的方法，已经在 [STL priority_queue 自定义排序方法](http://www.cdsy.xyz/computer/programme/stl/20210307/cd161510785212026.html)一节中做了详细的讲解。换句话说，为 Priority_queue 容器适配器自定义排序规则的方法，同样适用于所有关联式容器。

总的来说，为关联式容器自定义排序规则，有以下 2 种方法。

## 使用函数对象自定义排序规则
----------------

无论关联式容器中存储的是基础类型（如 int、double、float 等）数据，还是自定义的结构体变量或类对象（包括 string 类），都可以使用函数对象的方式为该容器自定义排序规则。

下面样例以 set 容器为例，演示了如何用函数对象的方式自定义排序规则：

```
#include <iostream>
#include <set>      // set
#include <string>       // string
using namespace std;
//定义函数对象类
class cmp {
public:
    //重载 () 运算符
    bool operator ()(const string &a,const string &b) {
        //按照字符串的长度，做升序排序(即存储的字符串从短到长)
        return  (a.length() < b.length());
    }
};

int main() {
    //创建 set 容器，并使用自定义的 cmp 排序规则
    std::set<string, cmp>myset{"http://www.cdsy.xyz/computer/programme/stl/",
                               "http://www.cdsy.xyz/computer/programme/Python/",
                               "http://www.cdsy.xyz/computer/programme/java/"};
    //输出容器中存储的元素
    for (auto iter = myset.begin(); iter != myset.end(); ++iter) {
            cout << *iter << endl;
    }
    return 0;
}
```

程序执行结果为：
```
http://www.cdsy.xyz/computer/programme/stl/  
http://www.cdsy.xyz/computer/programme/java/  
http://www.cdsy.xyz/computer/programme/Python/
```

重点分析一下 6~13 行代码，其定义了一个函数对象类，并在其重载 () 运算符的方法中自定义了新的排序规则，即按照字符串的长度做升序排序。在此基础上，程序第 17 行代码中，通过将函数对象类的类名 cmp 通过 set 类模板的第 2 个参数传递给 myset 容器，该容器内部排序数据的规则，就改为了以字符串的长度为标准做升序排序。

> 需要注意的是，此程序中创建的 myset 容器，*由于是以字符串的长度为准进行排序，因此其无法存储相同长度的多个字符串*。

另外，C++ 中的 struct 和 class 非常类似，因此上面程序中，函数对象类 cmp 也可以使用 struct 关键字创建：

```
//定义函数对象类
struct cmp {
    //重载 () 运算符
    bool operator ()(const string &a, const string &b) {
        //按照字符串的长度，做升序排序(即存储的字符串从短到长)
        return  (a.length() < b.length());
    }
};
```

值得一提的是，在定义函数对象类时，也可以将其定义为模板类。比如：

```
//定义函数对象模板类
template <typename T>
class cmp {
public:
    //重载 () 运算符
    bool operator ()(const T &a, const T &b) {
        //按照值的大小，做升序排序
        return  a < b;
    }
};
```

> 注意，此方式必须保证 T 类型元素可以直接使用关系运算符（比如这里的 < 运算符）做比较。

## 重载关系运算符实现自定义排序
-----------------

其实在 STL 标准库中，本就包含几个可供关联式容器使用的排序规则，如表表示。 

| 排序规则                             | 功能                                               |
|----------------------------------|--------------------------------------------------|
| std::less&lt; T&gt;&nbsp; &nbsp; | 底层采用 &lt; 运算符实现升序排序，各关联式容器默认采用的排序规则。             |
| std::greater&lt; T&gt;           | 底层采用 &gt; 运算符实现降序排序，同样适用于各个关联式容器。                |
| std::less_equal&lt; T&gt;        | 底层采用 &lt;= 运算符实现升序排序，多用于 multimap 和 multiset 容器。 |
| std::greater_equal&lt; T&gt;     | 底层采用 &gt;= 运算符实现降序排序，多用于 multimap 和 multiset 容器。 |

值得一提的是，表 1 中的这些排序规则，其底层也是采用函数对象的方式实现的。以 std::less\<T\> 为例，其底层实现为：

```
template <typename T>
struct less {
    //定义新的排序规则
    bool operator ()(const T &_lhs, const T &_rhs) const {
        return _lhs < _rhs;
    }
}
```

在此基础上，当关联式容器中存储的数据类型为自定义的结构体变量或者类对象时，通过对现有排序规则中所用的关系运算符进行重载，也能实现自定义排序规则的目的。

> 注意，当关联式容器中存储的元素类型为结构体指针变量或者类的指针对象时，只能使用函数对象的方式自定义排序规则，此方法不再适用。

举个例子：

```
#include <iostream>
#include <set>      // set
#include <string>       // string
using namespace std;
//自定义类
class myString {
public:
    //定义构造函数，向 myset 容器中添加元素时会用到
    myString (string tempStr) : str (tempStr) {};
    //获取 str 私有对象，由于会被私有对象调用，因此该成员方法也必须为 const 类型
    string getStr () const;
private:
    string str;
};
string myString:: getStr () const{
    return this->str;
}
//重载 < 运算符，参数必须都为 const 类型
bool operator <(const myString &stra, const myString & strb) {
    //以字符串的长度为标准比较大小
    return stra.getStr (). length () < strb.getStr (). length ();
}

int main () {
    //创建空 set 容器，仍使用默认的 less<T> 排序规则
    std::set<myString>myset;
    //向 set 容器添加元素，这里会调用 myString 类的构造函数
    myset.emplace (" http://www.cdsy.xyz/computer/programme/stl/" );
    myset.emplace (" http://www.cdsy.xyz/computer/programme/C_language/" );
    myset.emplace (" http://www.cdsy.xyz/computer/programme/Python/" );
    //
    for (auto iter = myset.begin (); iter != myset.end (); ++iter) {
        myString mystr = *iter;
        cout << mystr.getStr () << endl;
    }
    return 0;
}
```

程序执行结果为：
```
http://www.cdsy.xyz/computer/programme/C_language/ 
http://www.cdsy.xyz/computer/programme/stl/  
http://www.cdsy.xyz/computer/programme/Python/
```

在这个程序中，虽然 myset 容器表面仍采用默认的 std::less\<T\> 排序规则，但由于我们对其所用的 < 运算符进行了重载，使得 myset 容器内部实则是以字符串的长度为基准，对各个 mystring 类对象进行排序。

另外，上面程序以全局函数的形式实现对 < 运算符的重载，还可以使用成员函数或者友元函数的形式实现。其中，**当以成员函数的方式重载 < 运算符时，该成员函数必须声明为 const 类型，且参数也必须为 const 类型**：

```
bool operator <(const myString & tempStr) const {
    //以字符串的长度为标准比较大小
    return this->str.length () < tempStr.str.length ();
}
```

> 至于参数的传值方式是采用按引用传递还是按值传递，都可以（建议采用*按引用传递，效率更高*）。

同样，*如果以友元函数的方式重载 < 运算符时，要求参数必须使用 const 修饰*：

```
//类中友元函数的定义
friend bool operator <(const myString &a, const myString &b);

//类外部友元函数的具体实现
bool operator <(const myString &stra, const myString &strb) {
    //以字符串的长度为标准比较大小
    return stra.str.length () < strb.str.length ();
}
```

> 当然，本节所讲自定义排序规则的方法并不仅仅适用于 set 容器，其它关联式容器（map、multimap、multiset）也同样适用，有兴趣的读者可自行编写代码验证。