> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [changkun.de](https://changkun.de/modern-cpp/zh-cn/10-cpp20/)

> Modern C++ Tutorial | C++ 11/14/17/20 On the Fly | 现代 C++ 教程 | 高速上手 C++11/14/17/20

C++20 如同 C++11 一样，似乎能够成为一个振奋人心的更新。例如，早在 C++11 时期就跃跃欲试呼声极高却最终落选的 `Concept`，如今已经箭在弦上。 C++ 组委会在讨论投票最终确定 C++20 有很多提案，诸如 **Concepts**/**Module**/**Coroutine**/**Ranges**/ 等等。 本章我们就来一览 C++20 即将引入的那些重要特性。

[](#概念与约束)[]( #概念与约束 "概念与约束")概念与约束
---------------------------------

概念（Concepts）是对 C++ 模板编程的进一步增强扩展。简单来说，概念是一种编译期的特性， 它能够让编译器在编译期时对模板参数进行判断，从而大幅度增强我们在 C++ 中模板编程的体验。 使用模板进行编程时候我们经常会遇到各种令人发指的错误， 这是因为到目前为止我们始终不能够对模板参数进行检查与限制。 举例而言，下面简单的两行代码会造成大量的几乎不可读的编译错误：

```
#include <list>
#include <algorithm>
int main() {
    std::list<int> l = {1, 2, 3};
    std::sort(l.begin(), l.end());
    return 0;
}
```

而这段代码出现错误的根本原因在于，`std::sort` 对排序容器必须提供随机迭代器，否则就不能使用，而我们知道 `std::list` 是不支持随机访问的。 用概念的语言来说就是：`std::list` 中的迭代器不满足 `std::sort` 中随机迭代器这个概念的约束（Constraint）。 在引入概念后，我们就可以这样对模板参数进行约束：

```
template <typename T>
requires Sortable<T> // Sortable 是一个概念
void sort(T& c);
```

缩写为：

```
template<Sortable T> // T 是一个 Sortable 的类型名
void sort(T& c)
```

甚至于直接将其作为类型来使用：

```
void sort(Sortable& c); // c 是一个 Sortable 类型的对象
```

我们现在来看一个实际的例子。

TODO: [https://godbolt.org/z/9liFPD](https://godbolt.org/z/9liFPD)

[](#模块)[]( #模块 "模块")模块
---------------------

TODO:

[](#合约)[]( #合约 "合约")合约
---------------------

TODO:

[](#范围)[]( #范围 "范围")范围
---------------------

TODO:

[](#协程)[]( #协程 "协程")协程
---------------------

TODO:

[](#事务内存)[]( #事务内存 "事务内存")事务内存
-----------------------------

TODO:

[](#总结)[]( #总结 "总结")总结
---------------------

总的来说，终于在 C++20 中看到 Concepts/Ranges/Modules 这些令人兴奋的特性， 这对于一门已经三十多岁『高龄』的编程语言，依然是充满魅力的。

[](#进一步阅读的参考资料)[]( #进一步阅读的参考资料 "进一步阅读的参考资料")进一步阅读的参考资料
-----------------------------------------------------

*   [Why Concepts didn't make C++17？](http://honermann.net/blog/2016/03/06/why-concepts-didnt-make-cxx17/)
*   [C++11/14/17/20 编译器支持情况](https://en.cppreference.com/w/cpp/compiler_support)
*   [C++ 历史](https://en.cppreference.com/w/cpp/language/history)