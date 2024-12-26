在 C++11 之前，对于非 const 的静态数据成员，是必须在类外进行定义和初始化的。静态数据成员仅仅是在类内声明，并不像非静态数据成员那样可以在类内直接初始化。

## 为什么非要在类外定义和初始化？
要理解为什么要在类外定义和初始化非 const 的静态数据成员，我们需要考虑一些基本概念： 
1. 静态数据成员是类的所有实例共享的成员，不依赖于类的对象。
2. 静态数据成员的存储是在全局数据区（静态存储区）。
因此静态数据成员不属于类的对象，而是属于类本身，所以必须在类外进行定义和初始化，以便分配存储空间。类外定义也让编译器知道在哪里存储静态数据成员，否则编译器就无法正确处理这些成员的内存分配和访问。

在 C++11 之前，通常的做法是在类的实现文件中进行类外定义和初始化，例如：

```cpp
// MyClass.h 头文件
class MyClass {
public:
    static int myStaticVariable;
};

// MyClass.cpp 实现文件
int MyClass::myStaticVariable = 0; // 在类外定义和初始化静态数据成员
```

## Cpp 11 修改了什么？
在 C++11 引入了 "in-class 初始值" (in-class initializer) 特性后，你可以在类内直接初始化静态数据成员，无论是 const 还是非 const 的都可以。示例如下：

```cpp
// MyClass.h 头文件
class MyClass {
public:
    static int myStaticVariable;
    static const int myConstStaticVariable = 42; // C++11 允许在类内初始化 const 静态数据成员
};

// MyClass.cpp 实现文件（通常不再需要定义和初始化静态数据成员，除非它是模板类）
```

总结起来，在 C++11 之前，非 const 的静态数据成员必须在类外定义和初始化，而 C++11 引入的 "in-class 初始值" 特性则简化了 const 静态数据成员的初始化。
