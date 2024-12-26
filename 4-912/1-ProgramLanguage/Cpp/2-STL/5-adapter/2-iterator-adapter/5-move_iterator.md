
C++ 11 还为 STL 标准库增添了一种迭代器适配器，即本节要讲的 move_iterator 移动迭代器适配器。

move_iterator 迭代器适配器，又可简称为移动迭代器，其可以实现以移动而非复制的方式，将某个区域空间中的元素移动至另一个指定的空间。

举个例子，前面讲了 vector 容器，该类型容器支持如下初始化的方式（程序一）：

```
#include <iostream>
#include <vector>
#include <list>
#include <string>
using namespace std;
int main()
{
    //创建并初始化一个 vector 容器
    vector<string> myvec{ "STL","Python","Java" };
    //再次创建一个 vector 容器，利用 myvec 为其初始化
    vector<string>othvec(myvec.begin(), myvec.end());
  
    cout << "myvec:" << endl;
    //输出 myvec 容器中的元素
    for (auto ch : myvec) {
        cout << ch << " ";
    }
    cout << endl << "othvec:" << endl;
    //输出 othvec 容器中的元素
    for (auto ch : othvec) {
        cout << ch << " ";
    }
    return 0;
}
```

程序执行结果为：
```
myvec:  
STL Python Java  
othvec:  
STL Python Java
```

注意程序第 11 行，初始化 othvec 容器是*通过复制 myvec 容器中的元素实现的*。也就是说，othvec 容器从 myvec 容器中复制了一份 "STL"、"Python"、"Java" 并存储起来，此过程不会影响 myvec 容器。

那么，如果不想采用复制的方式，而就是想 myvec 容器中存储的元素全部移动到 othvec 容器中，该怎么办呢？没错，就是采用移动迭代器。

值得一提的是，实现移动迭代器的模板类定义在 `<iterator>` 头文件，并位于 std 命名空间中。因此，在使用该类型迭代器时，程序中应包含如下代码：

```
#include <iterator>
using namespace std;
```

实现 move_iterator 移动迭代器的模板类定义如下：
```
template <class Iterator>
    class move_iterator;
```

可以看到，在使用此迭代器时，需要传入一个基础迭代器 Iterator。

注意，此基础迭代器的类型虽然没有明确要求，但该模板类中某些成员方法的底层实现，**需要此基础迭代器为双向迭代器**或者**随机访问迭代器**。也就是说，如果指定的 Iterator 类型仅仅是输入迭代器，则某些成员方法将无法使用。

> 实际上，在 move_iterator 模板类中就包含有指定 Iterator 类型的基础迭代器，整个模板类也是借助此基础迭代器实现的。关于 move_iterator 的底层实现，[C++ STL move_iterator 手册](http://www32.cplusplus.com/reference/iterator/move_iterator/)给出了详细的参考代码，有兴趣的读者可自行研究。

## move_iterator 的创建
-------------------------

move_iterator 模板类中，提供了 4 种创建 move_iterator 迭代器的方法。

1) 通过调用该模板类的默认构造函数，可以创建一个不指向任何对象的移动迭代器。比如：
```
//将 vector 容器的随机访问迭代器作为新建移动迭代器底层使用的基础迭代器
typedef std::vector<std::string>:: iterator Iter;
//调用默认构造函数，创建移动迭代器
std::move_iterator<Iter>mIter;
```
由此，我们就创建好了一个 mIter 移动迭代器，该迭代器底层使用的是 vector 容器的随机访问迭代器，但这里没有为此基础迭代器明确指向，所以 mIter 迭代器也不知向任何对象。

2) 当然，在创建 move_iterator 迭代器的同时，也可以为其初始化。比如：
```
//创建一个 vector 容器
std::vector<std::string> myvec{ "one","two","three" };
//将 vector 容器的随机访问迭代器作为新建移动迭代器底层使用的基础迭代器
typedef std::vector<std::string>:: iterator Iter;
//创建并初始化移动迭代器
std::move_iterator<Iter>mIter (myvec.begin ());
```
这里，我们创建了一个 mIter 移动迭代器，同时还为底层使用的随机访问迭代器做了初始化，即令其指向 myvec 容器的第一个元素。

3) move_iterator 模板类还支持用已有的移动迭代器初始化新建的同类型迭代器，比如，在上面创建好 mIter 迭代器的基础上，还可以向如下这样为新建的移动迭代器初始化：
```
std::move_iterator<Iter>mIter 2 (mIter);
//还可以使用 = 运算符，它们是等价的
//std::move_iterator<Iter>mIter 2 = mIter;
```
这样创建的 mIter 2 迭代器和 mIter 迭代器完全一样。也就是说，mIter 2 底层会复制 mIter 迭代器底层使用的基础迭代器。

4) 以上 3 种创建 move_iterator 迭代器的方式，其本质都是直接调用 move_iterator 模板类中的构造方法实现的。除此之外，C++ STL 标准库还提供了一个 make_move_iterator () 函数，通过调用此函数可以快速创建一个 move_iterator 迭代器。
C++ STL 标准库中，make_move_iterator () 是以函数模板的形式提供的，其语法格式如下：
```
template <class Iterator>  
  move_iterator<Iterator> make_move_iterator (const Iterator& it);
```
其中，参数 it 为基础迭代器，用于初始化新建迭代器。同时，该函数会返回一个创建好的移动迭代器。

举个例子：
```
typedef std::vector<std::string>:: iterator Iter;
std::vector<std::string> myvec{ "one","two","three" };
//将 make_move_iterator () 的返回值赋值给同类型的 mIter 迭代器
std::move_iterator<Iter>mIter = make_move_iterator (myvec.begin ());
```


## 应用
下面程序对程序一做了修改，即运用移动迭代器为 othvec 容器初始化：

```
#include <iostream>
#include <vector>
#include <list>
#include <string>
using namespace std;
int main ()
{
    //创建并初始化一个 vector 容器
    vector<string> myvec{ "STL","Python","Java" };
    //再次创建一个 vector 容器，利用 myvec 为其初始化
    vector<string>othvec (make_move_iterator (myvec.begin ()), make_move_iterator (myvec.end ()));
   
    cout << "myvec: " << endl;
    //输出 myvec 容器中的元素
    for (auto ch : myvec) {
        cout << ch << " ";
    }
    cout << endl << "othvec: " << endl;
    //输出 othvec 容器中的元素
    for (auto ch : othvec) {
        cout << ch << " ";
    }
    return 0;
}
```

程序执行结果为：
```
myvec:

      othvec:  
STL Python Java
```

通过和程序一做对比不难看出它们的区别，由于程序第 11 行为 othvec 容器初始化时，使用的是移动迭代器，其会将 myvec 容器中的元素直接移动到 othvec 容器中。

> 注意，即便通过移动迭代器将容器中某区域的元素移动到了其他容器中，该区域内仍可能**残留有之前存储的元素，但这些元素是不能再被使用的**，否则极有可能使程序产生各种其他错误。

和其他迭代器适配器一样，move_iterator 模板类中也提供有 base () 成员方法，通过该方法，我们可以获取到当前移动迭代器底层所使用的基础迭代器。

举个例子：

```
#include <iostream>
#include <vector>
#include <list>
#include <string>
using namespace std;
int main ()
{
    typedef std::vector<std::string>:: iterator Iter;

    //创建并初始化一个 vector 容器
    vector<std::string> myvec{ "STL","Java","Python" };
    //创建 2 个移动迭代器
    std::move_iterator<Iter>begin = make_move_iterator (myvec.begin ());
    std::move_iterator<Iter>end = make_move_iterator (myvec.end ());
    //以复制的方式初始化 othvec 容器
    vector <std::string> othvec (begin.base (), end.base ());
   
    cout << "myvec: " << endl;
    //输出 myvec 容器中的元素
    for (auto ch : myvec) {
        cout << ch << " ";
    }
    cout << endl << "othvec: " << endl;
    //输出 othvec 容器中的元素
    for (auto ch : othvec) {
        cout << ch << " ";
    }
    return 0;
}
```

程序执行结果为：
```
myvec:  
STL Java Python  
othvec:  
STL Java Python
```
显然，通过调用 base () 成员方法，初始化 othvec 容器的方式转变为以复制而非移动的方式，因此 myvec 容器不会受到影响。