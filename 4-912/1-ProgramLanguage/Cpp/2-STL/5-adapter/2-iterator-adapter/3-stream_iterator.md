
流迭代器也是一种迭代器适配器，不过和之前讲的迭代器适配器有所差别，它的操作对象不再是某个容器，而是流对象。即通过流迭代器，我们可以读取指定流对象中的数据，也可以将数据写入到流对象中。

> 通常情况下，我们经常使用的 cin、cout 就属于流对象，其中 cin 可以获取键盘输入的数据，cout 可以将指定数据输出到屏幕上。除此之外，更常见的还有文件 I/O 流等。关于什么流，更详细的介绍可阅读《[C++ 流类和流对象](http://www.cdsy.xyz/computer/programme/vc/20210307/cd161510709011592.html)》一文。

介于流对象又可细分为输入流对象（istream）和输出流对象（ostream），C++ STL 标准库中，也对应的提供了 2 类流迭代器：
* 将绑定到输入流对象的迭代器称为输入流迭代器（istream_iterator），其可以用来读取输入流中的数据；
* 将绑定到输出流对象的迭代器称为输出流迭代器（ostream_iterator），其用来将数据写入到输出流中。

接下来，就分别讲解这 2 个流迭代器的用法。

## 输入流迭代器 （istream_iterator）
--------------------------------

输入流迭代器用于直接从指定的输入流中读取元素，该类型迭代器本质上就是一个输入迭代器，这意味着假设 p 是一个输入流迭代器，则其只能进行 `++p`、`p++`、`*p ` 操作，同时输入迭代器之间也只能使用 == 和 != 运算符。

实际上，输入流迭代器的底层是通过重载 ++ 运算符实现的，该运算符内部会调用 operator >> 读取数据。也就是说，假设 iit 为输入流迭代器，则只需要执行 ++iit 或者 iit++，即可读取一个指定类型的元素。

值得一提的是，istream_iterator 定义在 `<iterator>` 头文件，并位于 std 命名空间中，因此使用此迭代器之前，程序中应包含如下语句：

```
#include <iterator>
using namespace std;
```

### 创建方式
创建输入流迭代器的方式有 3 种，分别为：

1) 调用 istream_iterator 模板类的默认构造函数，可以创建出一个具有结束标志的输入流迭代器。要知道，当我们从输入流中不断提取数据时，总有将流中数据全部提取完的那一时刻，这一时刻就可以用此方式构建的输入流迭代器表示。
```
std::istream_iterator<double> eos;
```
由此，即创建了一个可读取 double 类型元素，并代表结束标志的输入流迭代器。

2) 除此之外，还可以创建一个可用来读取数据的输入流迭代器，比如：
```
std::istream_iterator<double> iit (std::cin);
```
这里创建了一个可从标准输入流 cin 读取数据的输入流迭代器。值得注意的一点是，通过此方式创建的输入流迭代器，其调用的构造函数中，会自行尝试去指定流中读取一个指定类型的元素。

3) istream_iterator 模板类还支持用已创建好的 istream_iterator 迭代器为新建 istream_iterator 迭代器初始化，例如，在上面 iit 的基础上，再创建一个相同的 iit 2 迭代器：
```
std::istream_iterator<double> iit2 (iit1);
```
由此，就创建好了一个和 iit 1 完全相同的输入流迭代器。

### 用法
下面程序演示了输入流迭代器的用法：
```
#include <iostream>
#include <iterator>
using namespace std;
int main () {
    //用于接收输入流中的数据
    double value 1, value 2;
    cout << "请输入 2 个小数: ";
    //创建表示结束的输入流迭代器
    istream_iterator<double> eos;
    //创建一个可逐个读取输入流中数据的迭代器，同时这里会让用户输入数据
    istream_iterator<double> iit (cin);
    //判断输入流中是否有数据
    if (iit != eos) {
        //读取一个元素，并赋值给 value 1
        value 1 = *iit;
    }
    //如果输入流中此时没有数据，则用户要输入一个；反之，如果流中有数据，iit 迭代器后移一位，做读取下一个元素做准备
    iit++;
    if (iit != eos) {
        //读取第二个元素，赋值给 value 2
        value 2 = *iit;
    }
    //输出读取到的 2 个元素
    cout << "value 1 = " << value 1 << endl;
    cout << "value 2 = " << value 2 << endl;
    return 0;
}
```

程序执行结果为：
```
请输入 2 个小数: 1.2 2.3  
value 1 = 1.2  
value 2 = 2.3
```
> 注意，只有读取到 EOF 流结束符时，程序中的 iit 才会和 eos 相等。另外，Windows 平台上使用 Ctrl+Z 组合键输入 ^Z 表示 EOF 流结束符，此结束符需要单独输入，或者输入换行符之后再输入才有效。

## 输出流迭代器 （ostream_iterator）
--------------------------------

和输入流迭代器恰好相反，输出流迭代器用于将数据写到指定的输出流（如 cout）中。另外，该类型迭代器本质上属于输出迭代器，假设 p 为一个输出迭代器，则它能执行 `++p`、`p++`、`*p=t` 以及 `*p++=t` 等类似操作。

其次，输出迭代器底层是通过重载赋值（=）运算符实现的，即借助该运算符，每个赋值给输出流迭代器的元素都会被写入到指定的输出流中。

值得一提的是，实现 ostream_iterator 迭代器的模板类也定义在 `<iterator>` 头文件，并位于 std 命名空间中，因此在使用此类型迭代器时，程序也应该包含以下 2 行代码：

```
#include <iterator>
using namespace std;
```

### 创建
ostream_iterator 模板类中也提供了 3 种创建 ostream_iterator 迭代器的方法。

1) 通过调用该模板类的默认构造函数，可以创建了一个指定输出流的迭代器：
```
std::ostream_iterator<int> out_it (std::cout);
```
由此，我们就创建了一个可将 int 类型元素写入到输出流（屏幕）中的迭代器。

2) 在第一种方式的基础上，还可以为写入的元素之间指定一个分隔符，例如：
```
std::ostream_iterator<int> out_it (std:: cout，",");
```
和第一种写入方式不同之处在于，此方式在向输出流写入 int 类型元素的同时，还会附带写入一个逗号（,）。

3) 另外，在创建输出流迭代器时，可以用已有的同类型的迭代器，为其初始化。例如，利用上面已创建的 out_it，再创建一个完全相同的 out_it 1：
```
std::ostream_iterator<int> out_it 1 (out_it);
```

### 用法

下面程序演示了 ostream_iterator 输出流迭代器的功能：
```
#include <iostream>
#include <iterator>
#include <string>
using namespace std;
int main () {
    //创建一个输出流迭代器
    ostream_iterator<string> out_it (cout);
    //向 cout 输出流写入 string 字符串
    *out_it = " http://www.cdsy.xyz/computer/programme/stl/" ;
    cout << endl;

    //创建一个输出流迭代器，设置分隔符 ,
    ostream_iterator<int> out_it 1 (cout, ",");
    //向 cout 输出流依次写入 1、2、3
    *out_it 1 = 1;
    *out_it 1 = 2;
    *out_it 1 = 3;
    return 0;
}
```

程序输出结果为：
```
http://www.cdsy.xyz/computer/programme/stl/  
1,2,3,
```

在实际场景中，输出流迭代器常和 copy () 函数连用，即作为该函数第 3 个参数。比如：
```
#include <iostream>
#include <iterator>
#include <vector>
#include <algorithm>    // std::copy
using namespace std;
int main () {
    //创建一个 vector 容器
    vector<int> myvector;
    //初始化 myvector 容器
    for (int i = 1; i < 10; ++i) {
        myvector. push_back (i);
    }
    //创建输出流迭代器
    std::ostream_iterator<int> out_it (std:: cout, ", ");
    //将 myvector 容器中存储的元素写入到 cout 输出流中
    std:: copy (myvector.begin (), myvector.end (), out_it);
    return 0;
}
```

程序执行结果为：
```
1, 2, 3, 4, 5, 6, 7, 8, 9,
```
