
## Implicit conversion

Implicit conversions are automatically performed when a value is copied to a compatible type. For example:

```
short a=2000;
int b;
b=a;
```

Here, the value of `a` is promoted from `short` to `int` without the need of any explicit operator. This is known as a _standard conversion_. Standard conversions affect fundamental data types, and allow the conversions between numerical types (`short` to `int`, `int` to `float`, `double` to `int`...), to or from `bool`, and some pointer conversions. （从 `short` 到 `int` 的类型提升不需要显式运算符，这种涉及基本数据类型的转换称为*标准转换*）

Converting to `int` from some smaller integer type, or to `double` from `float` is known as _promotion_, and is guaranteed to produce the exact same value in the destination type. （从 `short` 等小整数到 `int`、或从 `float` 到 `double` 的类型转换称为提升，这能够保证在目标类型中生成与原值完全相同的值）

Other conversions between arithmetic types may not always be able to represent the same value exactly:（其它算术类型之间的转换不总是能准确地表示相同的值）

*   If a negative integer value is converted to an unsigned type, the resulting value corresponds to its 2's complement bitwise representation (i.e., `-1` becomes the largest value representable by the type, `-2` the second largest, ...).（如果将负整数转换为无符号类型，其结果对应于其 2 进制补码按位表示的形式）
*   The conversions from/to `bool` consider `false` equivalent to _zero_ (for numeric types) and to _null pointer_ (for pointer types); `true` is equivalent to all other values and is converted to the equivalent of `1`.（在转自或转向 `bool` 类型中，false 等价于数字类型的 0 或指针类型的 nullptr；true 等价于所有其它值，并最终转换为等效值 1）
*   If the conversion is from a floating-point type to an integer type, the value is truncated (the decimal part is removed). If the result lies outside the range of representable values by the type, the conversion causes _undefined behavior_.（从浮点数转换为整数，则小数部分将会截断。如果结果超出类型可表示值的范围，该转换会导致未定义行为）
*   Otherwise, if the conversion is between numeric types of the same kind (integer-to-integer or floating-to-floating), the conversion is valid, but the value is _implementation-specific_ (and may not be portable).（同类型间转换是有效的，但基于特定实现，并且可能不可移植）

Some of these conversions may imply a loss of precision, which the compiler can signal with a warning. This warning can be avoided with an explicit conversion.（其中一些转换可能存在精度损失，编译器可以通过警告发出信号，显式转换可以避免这些警告）

For non-fundamental types, arrays and functions implicitly convert to pointers, and pointers in general allow the following conversions:

*   _Null pointers_ can be converted to pointers of any type（空指针可以转换为任何类型的指针）
*   Pointers to any type can be converted to `void` pointers.（指向任何类型的指针都可以转换为 `void` 指针）
*   Pointer _upcast_: pointers to a derived class can be converted to a pointer of an _accessible_ and _unambiguous_ base class, without modifying its `const` or `volatile` qualification.（指针向上转换——指向派生类的指针可以转换为可访问且明确的基类的指针，而无需以 `const` 或 `volatile` 限定）  

## Implicit conversions with classes

In the world of classes, implicit conversions can be controlled by means of three member functions:  

*   **Single-argument constructors:** allow implicit conversion from a particular type to initialize an object. （单参数构造函数——允许从特定类型隐式转换以初始化对象）
*   **Assignment operator:** allow implicit conversion from a particular type on assignments.（赋值运算符——允许从赋值上的特定类型隐式转换）
*   **Type-cast operator:** allow implicit conversion to a particular type.（类型转换运算符——允许隐式转换为特定类型）

For example:

```cpp
// implicit conversion of classes:
#include <iostream> 
using namespace std;

class A {};

class B {
public:
  // conversion from A (constructor):
  B (const A& x) {}
  // conversion from A (assignment):
  B& operator= (const A x) {return *this;}
  // conversion to A (type-cast operator)
  operator A () {return A ();}
};

int main ()
{
  A foo;
  B bar = foo;    // calls constructor
  bar = foo;      // calls assignment
  foo = bar;      // calls type-cast operator
  return 0;
}
```

The type-cast operator uses a particular syntax: it uses the `operator` keyword followed by the destination type and an empty set of parentheses. Notice that the return type is the destination type and thus is not specified before the `operator` keyword.  （类型转换运算符使用特定的语法——使用 `operator` 关键字，后跟目标类型和空括号。需要注意返回类型是目标类型，因此未在 `operator` 关键字之前指定）

## Keyword `explicit`

On a function call, C++ allows one implicit conversion to happen for each argument. This may be somewhat problematic for classes, because it is not always what is intended. For example, if we add the following function to the last example: 

```cpp
void fn(B arg){}
```

This function takes an argument of type `B`, but it could as well be called with an object of type `A` as argument:  

```cpp
fn (foo);
```

This may or may not be what was intended. But, in any case, it can be prevented by marking the affected constructor with the `explicit` keyword:

（函数调用中 Cpp 会对每个参数进行一次隐式转换，但在类中并不总是可行，可能导致不同类型的参数都可使用。通过关键字 `explicit` 限定受影响的构造函数可以防止它）

```cpp
// explicit:
#include <iostream>
using namespace std;

class A {};

class B {
public:
  explicit B (const A& x) {}
  B& operator= (const A& x) {return *this;}
  operator A() {return A();}
};

void fn (B x) {}

int main ()
{
  A foo;
  B bar (foo);
  bar = foo;
  foo = bar;
  
//  fn (foo);  // not allowed for explicit ctor.
  fn (bar);  

  return 0;
}
```

Additionally, constructors marked with `explicit` cannot be called with the assignment-like syntax; In the above example, `bar` could not have been constructed with:  （标记为 `explicit` 的构造函数不能使用类似赋值的语法调用）

```cpp
B bar = foo;
```

Type-cast member functions (those described in the previous section) can also be specified as `explicit`. This prevents implicit conversions in the same way as `explicit` -specified constructors do for the destination type.  

## Type casting

C++ is a strong-typed language. Many conversions, specially those that imply a different interpretation of the value, require an explicit conversion, known in C++ as _type-casting_.（Cpp 是强类型语言，许多转换需要显式转换，尤其是涉及对值不同解释的转换，这些称为*类型转换*）

There exist two main syntaxes for generic type-casting: _functional_ and _c-like_:

```
double x = 10.3;
int y;
y = int (x);    // functional notation
y = (int) x;    // c-like cast notation
```

The functionality of these generic forms of type-casting is enough for most needs with fundamental data types. However, these operators can be applied indiscriminately on classes and pointers to classes, which can lead to code that -while being syntactically correct- can cause runtime errors. （泛型的类型转换足以满足大多数基本数据类型的需求，但是若不加选择地应用于类和指向类的指针，则可能导致代码虽然语法正确，但是却运行时错误）

For example, the following code compiles without errors:

```cpp
// class type-casting
#include <iostream> 
using namespace std;

class Dummy {
    double i, j;
};

class Addition {
    int x, y;
  public:
    Addition (int a, int b) { x=a; y=b; }
    int result () { return x+y;}
};

int main () {
  Dummy d;
  Addition * padd;
  padd = (Addition*) &d;
  cout << padd-> result ();
  return 0;
}
```

The program declares a pointer to `Addition`, but then it assigns to it a reference to an object of another unrelated type using explicit type-casting:  

```cpp
padd = (Addition*) &d;
```

Unrestricted explicit type-casting allows to convert any pointer into any other pointer type, independently of the types they point to. The subsequent call to member `result` will produce either a run-time error or some other unexpected results. （上述程序中直接使用显式类型转换为其分配另一个不相关类型的对象和引用，这种不受限制的显式类型转换允许将任何指针转换为任何其他指针类型，而与指向的类型无关，这对成员的后续调用将产生运行时错误和未定义的结果）

In order to control these types of conversions between classes, we have four specific casting operators: `dynamic_cast`, `reinterpret_cast`, `static_cast` and `const_cast`. Their format is to follow the new type enclosed between angle-brackets (`<>`) and immediately after, the expression to be converted between parentheses.（为了控制类之间的类型转换，有四个特定的转换运算符。使用语法是 `转换运算符+<转换目标类型>+(表达式) `）

```
dynamic_cast <new_type> (expression)  
reinterpret_cast <new_type> (expression)  
static_cast <new_type> (expression)  
const_cast <new_type> (expression)  
```

The traditional type-casting equivalents to these expressions would be:

```
(new_type) expression  
new_type (expression)  
```  

but each one with its own special characteristics:

### dynamic_cast

`dynamic_cast` can only be used with pointers and references to classes (or with `void*`). Its purpose is to ensure that the result of the type conversion points to a valid complete object of the destination pointer type. （`dynamic_cast` 转换运算符只能与指针或对类的引用一起使用，目的是确保类型转换的结果是指向目标指针类型的有效完整对象）

This naturally includes _pointer upcast_ (converting from pointer-to-derived to pointer-to-base), in the same way as allowed as an _implicit conversion_.（指针存在向上转换，即从指向派生类转换为指向基类，其方式与隐式转换相同）

But `dynamic_cast` can also _downcast_ (convert from pointer-to-base to pointer-to-derived) polymorphic classes (those with virtual members) if -and only if- the pointed object is a valid complete object of the target type.（`dynamic_cast` 可以向下转换多态类，即从指向基类转换为指向派生类，当且仅当指向的对象是目标类型的有效完整对象）

For example:

```cpp
// dynamic_cast
#include <iostream> 
#include <exception> 
using namespace std;

class Base { virtual void dummy () {} };
class Derived: public Base { int a; };

int main () {
  try {
    Base * pba = new Derived;
    Base * pbb = new Base;
    Derived * pd;

    pd = dynamic_cast <Derived*>(pba);
    if (pd==0) cout << "Null pointer on first type-cast.\n";

    pd = dynamic_cast <Derived*>(pbb);
    if (pd==0) cout << "Null pointer on second type-cast.\n";

  } catch (exception& e) {cout << "Exception: " << e.what ();}
  return 0;
}
```

>[! tldr] **Compatibility note:** 
>This type of `dynamic_cast` requires _Run-Time Type Information (RTTI)_ to keep track of dynamic types. Some compilers support this feature as an option which is disabled by default. This needs to be enabled for runtime type checking using `dynamic_cast` to work properly with these types.
>（`dynamic_cast` 类型需要运行时类型信息来跟踪动态类型，一些编译器支持此功能作为默认禁用的选项，需要启用此功能才能使用 `dynamic_cast` 类型进行运行时类型检查）  

The code above tries to perform two dynamic casts from pointer objects of type `Base*` (`pba` and `pbb`) to a pointer object of type `Derived*`, but only the first one is successful. Notice their respective initializations:

```
Base * pba = new Derived;
Base * pbb = new Base;
```

Even though both are pointers of type `Base*`, `pba` actually points to an object of type `Derived`, while `pbb` points to an object of type `Base`. Therefore, when their respective type-casts are performed using `dynamic_cast`, `pba` is pointing to a full object of class `Derived`, whereas `pbb` is pointing to an object of class `Base`, which is an incomplete object of class `Derived`.（即使二指针都是 `Base` 类型，但实际上 `pba` 指向 `Derived` 类型对象，`pbb` 指向 `Base` 类型对象，各自通过 `dynamic_cast` 进行类型转换使用时，`pba` 指向 `Derived` 类的完整对象，然而 `pbb` 指向 `Base` 类的对象是 `Derived` 类的不完整对象）

When `dynamic_cast` cannot cast a pointer because it is not a complete object of the required class -as in the second conversion in the previous example- it returns a _null pointer_ to indicate the failure. If `dynamic_cast` is used to convert to a reference type and the conversion is not possible, an exception of type `bad_cast` is thrown instead.（当 `dynamic_cast` 无法强制转换指针时，是因为它不是所需类的完整对象，这将返回一个 nullptr 指针表示失败。如果 `dynamic_cast` 用于转换引用类型，却无法进行转换，会抛出 `bad_cast` 异常）

`dynamic_cast` can also perform the other implicit casts allowed on pointers: casting null pointers between pointers types (even between unrelated classes), and casting any pointer of any type to a `void*` pointer.（`dynamic_cast` 还可以执行指针上允许的其它隐式强制转换——在指针类型之间强制转换 nullptr，以及将任何类型的指针强制转换为 `void*` 指针）

### static_cast

`static_cast` can perform conversions between pointers to related classes, not only _upcasts_ (from pointer-to-derived to pointer-to-base), but also _downcasts_ (from pointer-to-base to pointer-to-derived). No checks are performed during runtime to guarantee that the object being converted is in fact a full object of the destination type. Therefore, it is up to the programmer to ensure that the conversion is safe. On the other side, it does not incur the overhead of the type-safety checks of `dynamic_cast`.（`static_cast` 与 `dynamic_cast` 相似，其可以在指向相关类的指针之间执行转换，不仅可以向上转换也可以向下转换。但 `static_cast` 运行时不进行任何检查来确保要转换的对象实际是目标类型的完整对象，这非常考验程序员的功底，程序员有责任确保转换安全，因此静态转换不会产生 `dynamic_cast` 的类型安全检查开销）

```
class Base {};
class Derived: public Base {};
Base * a = new Base;
Derived * b = static_cast<Derived*>(a);
```

This would be valid code, although `b` would point to an incomplete object of the class and could lead to runtime errors if dereferenced.（尽管 `b` 指向类的不完整对象，但这仍是有效代码。如果解引用，可能导致运行时错误）

Therefore, `static_cast` is able to perform with pointers to classes not only the conversions allowed implicitly, but also their opposite conversions.

`static_cast` is also able to perform all conversions allowed implicitly (not only those with pointers to classes), and is also able to perform the opposite of these. （`static_cast` 能够执行所有允许的隐式转换，不止是带有指向类的指针的转换，还能够执行与这些相反的转换）

It can:
* Convert from `void*` to any pointer type. In this case, it guarantees that if the `void*` value was obtained by converting from that same pointer type, the resulting pointer value is the same.（从 `void*` 转换为任何指针类型，此时能够保证如果 `void*` 值是通过从同一指针类型转换获得的，则生成的指针值相同）
* Convert integers, floating-point values and enum types to enum types.（将整数、浮点值、枚举类型转换为枚举类型）

Additionally, `static_cast` can also perform the following:  
* Explicitly call a single-argument constructor or a conversion operator.（显式调用单参数构造函数或转换运算符）
* Convert to _rvalue references_.（转换为右值引用）
* Convert `enum class` values into integers or floating-point values.（将 `enum class` 值转换为整数或浮点值）
* Convert any type to `void`, evaluating and discarding the value.（将任何类型转换为 `void`，计算并丢弃该值）

### reinterpret_cast

`reinterpret_cast` converts any pointer type to any other pointer type, even of unrelated classes. The operation result is a simple binary copy of the value from one pointer to the other. All pointer conversions are allowed: neither the content pointed nor the pointer type itself is checked.（`reinterpret_cast` 可以将任何指针类型转换为其它任何的指针类型，甚至是不相关的类。操作结果的值是从一个指针到另一个指针的简单二进制副本。这一操作允许所有指针的转换，既不检查指向的内容，也不检查指针类型本身）

It can also cast pointers to or from integer types. The format in which this integer value represents a pointer is platform-specific. The only guarantee is that a pointer cast to an integer type large enough to fully contain it (such as [intptr_t](https://cplusplus.com/intptr_t) ), is guaranteed to be able to be cast back to a valid pointer.（`reinterpret_cast` 还可以将指针强制转换为整数类型或从整数类型转成指针，这种整数值表示指针的格式是特定于平台的。唯一可以保证的是，强制转换为足够大的整数以完全包含它的指针，保障能够强制转换为有效指针）

The conversions that can be performed by `reinterpret_cast` but not by `static_cast` are low-level operations based on reinterpreting the binary representations of the types, which on most cases results in code which is system-specific, and thus non-portable. （可以由 `reinterpret_cast` 转换而不能由 `static_cast` 的转换是基于重新解释类型的二进制表示的低级操作，大多数情况下会导致代码特定于系统，因此不可移植）

For example:

```
class A { /* ... */ };
class B { /* ... */ };
A * a = new A;
B * b = reinterpret_cast<B*>(a);
```

This code compiles, although it does not make much sense, since now `b` points to an object of a totally unrelated and likely incompatible class. Dereferencing `b` is unsafe.  

### const_cast

This type of casting manipulates the constness of the object pointed by a pointer, either to be set or to be removed. For example, in order to pass a const pointer to a function that expects a non-const argument:（`const_type` 类型的强制转换操纵指针所指对象的恒定性，无论要设置还是删除。例如为了将 const 指针传递给需要非 const 参数的函数）

```cpp
// const_cast
#include <iostream> 
using namespace std;

void print (char * str)
{
  cout << str << '\n';
}

int main () {
  const char * c = "sample text";
  print ( const_cast<char *> (c) );
  return 0;
}
```

The example above is guaranteed to work because function `print` does not write to the pointed object. Note though, that removing the constness of a pointed object to actually write to it causes _undefined behavior_.  （`print` 函数不会写入指向的对象，需要注意删除指向对象的恒常性以实现实际写入它，将会导致未定义行为）

## typeid

`typeid` allows to check the type of an expression:

`typeid (expression)`

This operator returns a reference to a constant object of type [type_info](https://cplusplus.com/type_info)  that is defined in the standard header  [\<typeinfo\>](https://cplusplus.com/%3Ctypeinfo%3E) . A value returned by `typeid` can be compared with another value returned by `typeid` using operators `==` and `!=` or can serve to obtain a null-terminated character sequence representing the data type or class name by using its `name ()` member.（此运算符返回对标准库文件中定义的类型的常量对象的引用，可以将 `typeid` 的返回值与另一个使用 `==` 和 `!=` 的 `typeid` 运算返回值进行比较，或者可以使用其 `name()` 成员获取表示数据类型或类名的以 null 结尾的字符序列）

```cpp
// typeid
#include <iostream> 
#include <typeinfo> 
using namespace std;

int main () {
  int * a, b;
  a=0; b=0;
  if (typeid (a) != typeid (b))
  {
    cout << "a and b are of different types:\n";
    cout << "a is: " << typeid (a).name () << '\n';
    cout << "b is: " << typeid (b).name () << '\n';
  }
  return 0;
}
```

When `typeid` is applied to classes, `typeid` uses the RTTI to keep track of the type of dynamic objects. When `typeid` is applied to an expression whose type is a polymorphic class, the result is the type of the most derived complete object: （当 `typeid` 应用于类时，它使用 RTTI 跟踪动态对象的类型。当 `typeid` 应用于多态类表达式时，结果是派生最多的完整对象的类型）

```cpp
// typeid, polymorphic class
#include <iostream> 
#include <typeinfo> 
#include <exception> 
using namespace std;

class Base { virtual void f (){} };
class Derived : public Base {};

int main () {
  try {
    Base* a = new Base;
    Base* b = new Derived;
    cout << "a is: " << typeid (a). name () << '\n';
    cout << "b is: " << typeid (b). name () << '\n';
    cout << "*a is: " << typeid (*a). name () << '\n';
    cout << "*b is: " << typeid (*b). name () << '\n';
  } catch (exception& e) { cout << "Exception: " << e.what () << '\n'; }
  return 0;
}
```

>[! note] 
>The string returned by member `name` of  [type_info](https://cplusplus.com/type_info)  depends on the specific implementation of your compiler and library. It is not necessarily a simple string with its typical type name, like in the compiler used to produce this output.
>（通过 `typeinfo` 的 `name` 成员函数返回的字符串取决于编译器和库的具体实现，它不一定是具有典型类型名称的简单字符串，就像用于生成此处输出的编译器一样）

Notice how the type that `typeid` considers for pointers is the pointer type itself (both `a` and `b` are of type `class Base *`). However, when `typeid` is applied to objects (like `*a` and `*b`) `typeid` yields their dynamic type (i.e. the type of their most derived complete object).（`typeid` 考虑指针的类型是指针本身的类型，但若 `typeid` 应用于 `*a` 这样的对象时，会产生它们的动态类型——即它们派生的最完整对象的类型）

If the type `typeid` evaluates is a pointer preceded by the dereference operator (`*`), and this pointer has a null value, `typeid` throws a  [bad_typeid](https://cplusplus.com/bad_typeid)  exception.（如果 `typeid` 类型是一个指针，前面有解引用运算符，并且该指针是 null 值，则 `typeid` 会抛出 bad_typeid 异常）