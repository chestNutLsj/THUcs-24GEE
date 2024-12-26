
> 在学习本节之前，读者有必要先了解什么是缓冲区，可阅读《[进入缓冲区（缓存）的世界](http://www.cdsy.xyz/computer/programme/vc/20201226/cd16089893876300.html)》一节做详细了解。

我们知道在 C++ STL 标准库中，流迭代器又细分为输入流迭代器和输出流迭代器，流缓冲区迭代器也是如此，其又被细分为输入流缓冲区迭代器和输出流缓冲区迭代器：
* 输入流缓冲区迭代器（istreambuf_iterator）：从输入流缓冲区中读取字符元素；
* 输出流缓冲区迭代器（ostreambuf_iterator）：将连续的字符元素写入到输出缓冲区中。

流缓冲区迭代器和流迭代器最大的区别在于，前者仅仅会将元素以字符的形式（包括 char、wchar_t、char 16_t 及 char 32_t 等）读或者写到流缓冲区中，由于不会涉及数据类型的转换，读写数据的速度比后者要快。

接下来，将一一对它们的功能和用法做讲解。

## 输入流缓冲区迭代器 istreambuf_iterator
--------------------------------------

istreambuf_iterator 输入流缓冲区迭代器的功能是从指定的流缓冲区中读取字符元素。

值得一提的是，该类型迭代器本质是一个输入迭代器，即假设 p 是一个输入流迭代器，则其只能进行 `++p`、`p++`、`*p` 操作，同时迭代器之间也只能使用 == 和 != 运算符。

另外，实现该类型迭代器的模板类也定义在 `<iterator>` 头文件，并位于 std 命名空间中。因此，在创建并使用该类型迭代器之前，程序中应包含如下代码：

```
#include <iterator>
using namespace std;
```

### 创建
创建输入流缓冲区迭代器的常用方式，有以下 2 种：

1) 通过调用 istreambuf_iterator 模板类中的默认构造函数，可以创建一个表示结尾的输入流缓冲区迭代器。要知道，当我们从流缓冲区中不断读取数据时，总有读取完成的那一刻，这一刻就可以用此方式构建的流缓冲区迭代器表示。
```
std::istreambuf_iterator<char> end_in;
```
其中，<> 尖括号中用于指定从流缓冲区中读取的字符类型。

2) 当然，我们还可以指定要读取的流缓冲区，比如：
```
std::istreambuf_iterator<char> in{ std:: cin };
```

除此之外，还可以传入流缓冲区的地址，比如：
```
std::istreambuf_iterator<char> in {std:: cin.rdbuf ()};
```
其中，rdbuf () 函数的功能是获取指定流缓冲区的地址。
> 无论是传入流缓冲区，还是传入其地址，它们最终构造的输入流缓冲区迭代器是一样的。

### 用法

下面程序演示了输入流缓冲区迭代器的用法：
```
#include <iostream>     // std:: cin, std::cout
#include <iterator>     // std::istreambuf_iterator
#include <string>       // std::string
using namespace std;
int main () {
    //创建结束流缓冲区迭代器
    istreambuf_iterator<char> eos;
    //创建一个从输入缓冲区读取字符元素的迭代器
    istreambuf_iterator<char> iit (cin);
    string mystring;

    cout << "向缓冲区输入元素：\n";
    //不断从缓冲区读取数据，直到读取到 EOF 流结束符
    while (iit != eos) {
        mystring += *iit++;
    }
    cout << "string：" << mystring;
    return 0;
}
```

程序执行结果为：
```
向缓冲区输入元素：  
abc ↙  
^Z ↙  
string：  
abc
```

注意，只有读取到 EOF 流结束符时，程序中的 iit 才会和 eos 相等。在 Windows 平台上，使用 Ctrl+Z 组合键输入 ^Z 表示 EOF 流结束符，此结束符需要单独输入，或者输入换行符之后再输入才有效。

## 输出流缓冲区迭代器 ostreambuf_iterator
--------------------------------------

和 istreambuf_iterator 输入流缓冲区迭代器恰恰相反，ostreambuf_iterator 输出流缓冲区迭代器用于将字符元素写入到指定的流缓冲区中。

实际上，该类型迭代器本质上是一个输出迭代器，这意味着假设 p 为一个输出迭代器，则它仅能执行 `++p`、`p++`、`*p=t` 以及 `*p++=t` 操作。

另外，和 ostream_iterator 输出流迭代器一样，istreambuf_iterator 迭代器底层也是通过重载赋值（=）运算符实现的。换句话说，即通过赋值运算符，每个赋值给输出流缓冲区迭代器的字符元素，都会被写入到指定的流缓冲区中。

需要指出的是，istreambuf_iterator 类模板也定义在 `<iterator>` 头文件，并位于 std 命名空间中，因此使用该类型迭代器，程序中需要包含以下代码：

```
#include <iterator>
using namespace std;
```

### 创建
在此基础上，创建输出流缓冲区迭代器的常用方式有以下 2 种：

1) 通过传递一个流缓冲区对象，即可创建一个输出流缓冲区迭代器，比如：
```
std::ostreambuf_iterator<char> out_it (std::cout);
```
同样，尖括号 <> 中用于指定要写入字符的类型，可以是 char、wchar_t、char 16_t 以及 char 32_t 等。

2) 还可以借助 rdbuf ()，传递一个流缓冲区的地址，也可以成功创建输出流缓冲区迭代器：
```
std::ostreambuf_iterator<char> out_it (std:: cout.rdbuf ());
```

### 用法
下面程序演示了输出流缓冲区迭代器的用法：

```
#include <iostream>     // std:: cin, std::cout
#include <iterator>     // std::ostreambuf_iterator
#include <string>       // std::string
#include <algorithm>    // std::copy

int main () {
    //创建一个和输出流缓冲区相关联的迭代器
    std::ostreambuf_iterator<char> out_it (std::cout); // stdout iterator
    //向输出流缓冲区中写入字符元素
    *out_it = 'S';
    *out_it = 'T';
    *out_it = 'L';

    //和 copy () 函数连用
    std:: string mystring ("\n http://www.cdsy.xyz/stl/" );
    //将 mystring 中的字符串全部写入到输出流缓冲区中
    std:: copy (mystring.begin (), mystring.end (), out_it);
    return 0;
}
```

程序执行结果为：
```
STL  
http://www.cdsy.xyz/computer/programme/stl/
```
