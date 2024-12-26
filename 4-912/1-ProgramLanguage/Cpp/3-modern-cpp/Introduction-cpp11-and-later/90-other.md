> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [changkun.de](https://changkun.de/modern-cpp/zh-cn/09-others/)

> Modern C++ Tutorial | C++ 11/14/17/20 On the Fly | 现代 C++ 教程 | 高速上手 C++11/14/17/20

[](#9-1-新类型)[]( #9 -1-新类型 "9.1 新类型") 9.1 新类型
-----------------------------------------

### [](#long-long-int)[]( #long -long-int "long long int") `long long int`

`long long int` 并不是 C++11 最先引入的，其实早在 C 99， `long long int` 就已经被纳入 C 标准中，所以大部分的编译器早已支持。 C++11 的工作则是正式把它纳入标准库， 规定了一个 `long long int` 类型至少具备 64 位的比特数。

[](#9-2-noexcept-的修饰和操作)[]( #9 -2-noexcept-的修饰和操作 "9.2 noexcept 的修饰和操作") 9.2 noexcept 的修饰和操作
-----------------------------------------------------------------------------------------

C++ 相比于 C 的一大优势就在于 C++ 本身就定义了一套完整的异常处理机制。 然而在 C++11 之前，几乎没有人去使用在函数名后书写异常声明表达式， 从 C++11 开始，这套机制被弃用，所以我们不去讨论也不去介绍以前这套机制是如何工作如何使用， 你更不应该主动去了解它。

C++11 将异常的声明简化为以下两种情况：

1.  函数可能抛出任何异常
2.  函数不能抛出任何异常

并使用 `noexcept` 对这两种行为进行限制，例如：

```
void may_throw(); // 可能抛出异常
void no_throw() noexcept; // 不可能抛出异常
```

使用 `noexcept` 修饰过的函数如果抛出异常，编译器会使用 `std::terminate()` 来立即终止程序运行。

`noexcept` 还能够做操作符，用于操作一个表达式，当表达式无异常时，返回 `true`，否则返回 `false`。

```
#include <iostream>
void may_throw() {
    throw true;
}
auto non_block_throw = []{
    may_throw();
};
void no_throw() noexcept {
    return;
}

auto block_throw = []() noexcept {
    no_throw();
};
int main()
{
    std::cout << std::boolalpha
        << "may_throw() noexcept? " << noexcept(may_throw()) << std::endl
        << "no_throw() noexcept? " << noexcept(no_throw()) << std::endl
        << "lmay_throw() noexcept? " << noexcept(non_block_throw()) << std::endl
        << "lno_throw() noexcept? " << noexcept(block_throw()) << std::endl;
    return 0;
}
```

`noexcept` 修饰完一个函数之后能够起到封锁异常扩散的功效，如果内部产生异常，外部也不会触发。例如：

```
try {
    may_throw();
} catch (...) {
    std::cout << "捕获异常, 来自 may_throw()" << std::endl;
}
try {
    non_block_throw();
} catch (...) {
    std::cout << "捕获异常, 来自 non_block_throw()" << std::endl;
}
try {
    block_throw();
} catch (...) {
    std::cout << "捕获异常, 来自 block_throw()" << std::endl;
}
```

最终输出为：

```
捕获异常, 来自 may_throw()
捕获异常, 来自 non_block_throw()
```

[](#9-3-字面量)[]( #9 -3-字面量 "9.3 字面量") 9.3 字面量
-----------------------------------------

### [](#原始字符串字面量)[]( #原始字符串字面量 "原始字符串字面量")原始字符串字面量

传统 C++ 里面要编写一个充满特殊字符的字符串其实是非常痛苦的一件事情， 比如一个包含 HTML 本体的字符串需要添加大量的转义符， 例如一个 Windows 上的文件路径经常会：`C:\\File\\To\\Path`。

C++11 提供了原始字符串字面量的写法，可以在一个字符串前方使用 `R` 来修饰这个字符串， 同时，将原始字符串使用括号包裹，例如：

```
#include <iostream>
#include <string>

int main() {
    std::string str = R"(C:\File\To\Path)";
    std::cout << str << std::endl;
    return 0;
}
```

### [](#自定义字面量)[]( #自定义字面量 "自定义字面量")自定义字面量

C++11 引进了自定义字面量的能力，通过重载双引号后缀运算符实现：

```
// 字符串字面量自定义必须设置如下的参数列表
std::string operator"" _wow1(const char *wow1, size_t len) {
    return std::string(wow1)+"woooooooooow, amazing";
}

std::string operator"" _wow2 (unsigned long long i) {
    return std::to_string(i)+"woooooooooow, amazing";
}

int main() {
    auto str = "abc"_wow1;
    auto num = 1_wow2;
    std::cout << str << std::endl;
    std::cout << num << std::endl;
    return 0;
}
```

自定义字面量支持四种字面量：

1.  整型字面量：重载时必须使用 `unsigned long long`、`const char *`、模板字面量算符参数，在上面的代码中使用的是前者；
2.  浮点型字面量：重载时必须使用 `long double`、`const char *`、模板字面量算符；
3.  字符串字面量：必须使用 `(const char *, size_t)` 形式的参数表；
4.  字符字面量：参数只能是 `char`, `wchar_t`, `char16_t`, `char32_t` 这几种类型。

[](#9-4-内存对齐)[]( #9 -4-内存对齐 "9.4 内存对齐") 9.4 内存对齐
---------------------------------------------

C++ 11 引入了两个新的关键字 `alignof` 和 `alignas` 来支持对内存对齐进行控制。 `alignof` 关键字能够获得一个与平台相关的 `std::size_t` 类型的值，用于查询该平台的对齐方式。 当然我们有时候并不满足于此，甚至希望自定定义结构的对齐方式，同样，C++ 11 还引入了 `alignas` 来重新修饰某个结构的对齐方式。我们来看两个例子：

```
#include <iostream>

struct Storage {
    char      a;
    int       b;
    double    c;
    long long d;
};

struct alignas(std::max_align_t) AlignasStorage {
    char      a;
    int       b;
    double    c;
    long long d;
};

int main() {
    std::cout << alignof(Storage) << std::endl;
    std::cout << alignof(AlignasStorage) << std::endl;
    return 0;
}
```

其中 `std::max_align_t` 要求每个标量类型的对齐方式严格一样，因此它几乎是最大标量没有差异， 进而大部分平台上得到的结果为 `long double`，因此我们这里得到的 `AlignasStorage` 的对齐要求是 8 或 16。

[](#总结)[]( #总结 "总结")总结
---------------------

本节介绍的几个特性是从仍未介绍的现代 C++ 新特性里使用频次较靠前的特性了，`noexcept` 是最为重要的特性，它的一个功能在于能够阻止异常的扩散传播，有效的让编译器最大限度的优化我们的代码。