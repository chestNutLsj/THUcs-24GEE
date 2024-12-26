
## Scopes

Named entities, such as variables, functions, and compound types need to be declared before being used in C++.

**Scope** refers to the visibility and accessibility of variables, functions, classes, and other identifiers in a C++ program. It determines the lifetime and extent of these identifiers. In C++, there are four types of scope:

### global scope
An entity declared outside any block has _global scope_, meaning that its name is valid anywhere in the code. While an entity declared within a block, such as a function or a selective statement, has _block scope_, and is only visible within the specific block in which it is declared, but not outside it.（在块之外声明的命名实体具有全局范围，可以再代码的任何位置有效使用；在块内声明的实体则只有局部范围，只能在块内使用，无论是函数块、选择分支块）

The lifetime of a global identifier is the entire duration of the program.

### local scope
Variables with block scope are known as _local variables_.They can be accessed only within the function or the block they were declared in. Their lifetime is limited to the duration of the function/block execution.

For example, a variable declared in the body of a function is a _local variable_ that extends until the end of the the function (i.e., until the brace `}` that closes the function definition), but not outside it:

```cpp
int foo;        // global variable

int some_function ()
{
  int bar;      // local variable
  bar = 0;
}

int other_function ()
{
  foo = 1;  // ok: foo is a global variable
  bar = 2;  // wrong: bar is not visible from this function
}
```

In each scope, a name can only represent one entity. For example, there cannot be two variables with the same name in the same scope:

```
int some_function ()
{
  int x;
  x = 0;
  double x;   // wrong: name already used in this scope
  x = 0.0;
}
```

The visibility of an entity with _block scope_ extends until the end of the block, including inner blocks. Nevertheless, an inner block, because it is a different block, can re-utilize a name existing in an outer scope to refer to a different entity; in this case, the name will refer to a different entity only within the inner block, hiding the entity it names outside. While outside it, it will still refer to the original entity. (内部块中可以命名外部块已存在的实体名称，达到重写的目的，即内部块中该实体的值以内部块定义为准，而不影响外部块中对应名字的实体。但若内部块中没有定义，而直接使用外部块的变量，则会引用该外部变量，做出的操作同样会改变外部变量。) 

For example:

```cpp
// inner block scopes
#include <iostream>
using namespace std;

int main () {
  int x = 10;
  int y = 20;
  {
    int x;   // ok, inner scope.
    x = 50;  // sets value to inner x
    y = 50;  // sets value to (outer) y
    cout << "inner block:\n";
    cout << "x: " << x << '\n';
    cout << "y: " << y << '\n';
  }
  cout << "outer block:\n";
  cout << "x: " << x << '\n';
  cout << "y: " << y << '\n';
  return 0;
}
```

![[inner-block.png]]

Note that `y` is not hidden in the inner block, and thus accessing `y` still accesses the outer variable.

Variables declared in declarations that introduce a block, such as function parameters and variables declared in loops and conditions (such as those declared on a `for` or an `if`) are local to the block they introduce.

### namespace scope
A namespace is a named scope that groups related identifiers together. Identifiers declared within a namespace have the namespace scope. They can be accessed using the namespace name and the scope resolution operator `::`.

```
#include <iostream>

namespace MyNamespace {
    int namespaceVar = 42;
}

int main() {
    std::cout << "Namespace variable: " << MyNamespace::namespaceVar << std::endl;
}
```

### class scope
Identifiers declared within a class have a class scope. They can be accessed using the class name and the scope resolution operator `::` or, for non-static members, an object of the class and the dot `.` or arrow `->` operator.

```
#include <iostream>

class MyClass {
public:
    static int staticMember;
    int nonStaticMember;

    MyClass(int value) : nonStaticMember(value) {}
};

int MyClass::staticMember = 7;

int main() {
    MyClass obj(10);
    std::cout << "Static member: " << MyClass::staticMember << std::endl;
    std::cout << "Non-static member: " << obj.nonStaticMember << std::endl;
}
```

Understanding various types of scope in C++ is essential for effective code structuring and management of resources in a codebase.

## Namespaces

Only one entity can exist with a particular name in a particular scope. This is seldom a problem for local names, since blocks tend to be relatively short, and names have particular purposes within them, such as naming a counter variable, an argument, etc...（本地名称中特定实体只有一个特定名称，由于块常常相对较短，很少产生问题）

But non-local names bring more possibilities for name collision, especially considering that libraries may declare many functions, types, and variables, neither of them local in nature, and some of them very generic.（但非本地的命名可能导致名称的冲突，尤其是在库中可能声明许多函数、类型和变量）

Namespaces allow us to group named entities that otherwise would have _global scope_ into narrower scopes, giving them _namespace scope_. This allows organizing the elements of programs into different logical scopes referred to by names.（命名空间允许将命名实体分组，将它们从全局范围分组到更窄的作用域中，这就是命名空间范围，这将允许程序元素根据名称引用的不同组织到不同的逻辑作用域中）

The syntax to declare a namespaces is:

```cpp
namespace identifier
{
  named_entities
}
```

Where `identifier` is any valid identifier and `named_entities` is the set of variables, types and functions that are included within the namespace. For example:

```
namespace myNamespace
{
  int a, b;
}
```

In this case, the variables `a` and `b` are normal variables declared within a namespace called `myNamespace`.

These variables can be accessed from within their namespace normally, with their identifier (either `a` or `b`), but if accessed from outside the `myNamespace` namespace they have to be properly qualified with the scope operator `::`. For example, to access the previous variables from outside `myNamespace` they should be qualified like:

```
myNamespace::a
myNamespace::b
```

Namespaces are particularly useful to avoid name collisions. For example:

```cpp
// namespaces
#include <iostream>
using namespace std;

namespace foo
{
  int value () { return 5; }
}

namespace bar
{
  const double pi = 3.1416;
  double value () { return 2*pi; }
}

int main () {
  cout << foo:: value () << '\n';
  cout << bar:: value () << '\n';
  cout << bar:: pi << '\n';
  return 0;
}
```

In this case, there are two functions with the same name: `value`. One is defined within the namespace `foo`, and the other one in `bar`. No redefinition errors happen thanks to namespaces. 

Notice also how `pi` is accessed in an unqualified manner from within namespace `bar` (just as `pi`), while it is again accessed in `main`, but here it needs to be qualified as `bar::pi`.

Namespaces can be split: Two segments of a code can be declared in the same namespace:

```
namespace foo { int a; }
namespace bar { int b; }
namespace foo { int c; }
```

This declares three variables: `a` and `c` are in namespace `foo`, while `b` is in namespace `bar`. Namespaces can even extend across different translation units (i.e., across different files of source code).  (命名空间可以拆分，甚至可以跨越不同的翻译单元，即不同的源文件)

### nesing namespace
Namespaces can be nested within other namespaces:

```
#include <iostream>

namespace outer {
    int x = 10;

    namespace inner {
        int y = 20;
    }
}

int main() {
    std::cout << "Outer x: " << outer::x << std::endl;
    std::cout << "Inner y: " << outer::inner::y << std::endl;

    return 0;
}
```

## using

The keyword `using` introduces a name into the current declarative region (such as a block), thus avoiding the need to qualify the name. For example:

```cpp
// using
#include <iostream>
using namespace std;

namespace first
{
  int x = 5;
  int y = 10;
}

namespace second
{
  double x = 3.1416;
  double y = 2.7183;
}

int main () {
  using first:: x;
  using second:: y;
  cout << x << '\n';
  cout << y << '\n';
  cout << first:: y << '\n';
  cout << second:: x << '\n';
  return 0;
}
```

Notice how in `main`, the variable `x` (without any name qualifier) refers to `first::x`, whereas `y` refers to `second::y`, just as specified by the `using` declarations. The variables `first::y` and `second::x` can still be accessed, but require fully qualified names.

The keyword `using` can also be used as a directive to introduce an entire namespace:

```cpp
// using
#include <iostream>
using namespace std;

namespace first
{
  int x = 5;
  int y = 10;
}

namespace second
{
  double x = 3.1416;
  double y = 2.7183;
}

int main () {
  using namespace first;
  cout << x << '\n';
  cout << y << '\n';
  cout << second:: x << '\n';
  cout << second:: y << '\n';
  return 0;
}
```

In this case, by declaring that we were using namespace `first`, all direct uses of `x` and `y` without name qualifiers were also looked up in namespace `first`.

`using` and `using namespace` have validity only in the same block in which they are stated or in the entire source code file if they are used directly in the global scope. 

For example, it would be possible to first use the objects of one namespace and then those of another one by splitting the code in different blocks:

```cpp
// using namespace example
#include <iostream> 
using namespace std;

namespace first
{
  int x = 5;
}

namespace second
{
  double x = 3.1416;
}

int main () {
  {
    using namespace first;
    cout << x << '\n';
  }
  {
    using namespace second;
    cout << x << '\n';
  }
  return 0;
}
```

## Namespace aliasing

Existing namespaces can be aliased with new names, with the following syntax:

```cpp
namespace new_name = current_name;
```

## The std namespace

All the entities (variables, types, constants, and functions) of the standard C++ library are declared within the `std` namespace. Most examples in these tutorials, in fact, include the following line:  

```cpp
using namespace std;
```

This introduces direct visibility of all the names of the `std` namespace into the code. This is done in these tutorials to facilitate comprehension and shorten the length of the examples, but many programmers prefer to qualify each of the elements of the standard library used in their programs. For example, instead of:

```
cout << "Hello world!";
```

It is common to instead see:

```
std::cout << "Hello world!";
```

Whether the elements in the `std` namespace are introduced with `using` declarations or are fully qualified on every use does not change the behavior or efficiency of the resulting program in any way. It is mostly a matter of style preference, although for projects mixing libraries, explicit qualification tends to be preferred.  (使用混合库的项目，明确的命名空间限定会大大提高代码可读性)

## Storage classes

The storage for variables with _global_ or _namespace scope_ is allocated for the entire duration of the program. This is known as _static storage_, and it contrasts with the storage for _local variables_ (those declared within a block). These use what is known as automatic storage. （全局和命名空间范围的变量所占用的存储空间，会在程序整个持续时间内分配，这称为**静态存储**）

The storage for local variables is only available during the block in which they are declared; after that, that same storage may be used for a local variable of some other function, or used otherwise.（局部变量所占用的存储空间，仅在声明它们的块运行期间可用，事后这些存储可能被用于其他功能的分配，因此称为**自动存储**）

But there is another substantial (实质性的) difference between variables with _static storage_ and variables with _automatic storage_:  
- Variables with _static storage_ (such as global variables) that are not explicitly initialized are automatically initialized to zeroes.  （未显式初始化的静态存储变量，将默认初始化为 0）
- Variables with _automatic storage_ (such as local variables) that are not explicitly initialized are left uninitialized, and thus have an undetermined value.（未显式初始化的自动存储变量，将保持未初始化状态，其值是不确定的）

For example:

```cpp
// static vs automatic storage
#include <iostream> 
using namespace std;

int x;

int main ()
{
  int y;
  cout << x << '\n';
  cout << y << '\n';
  return 0;
}
```

![[static-vs-automatic-storage.png]]

The actual output may vary, but only the value of `x` is guaranteed to be zero. `y` can actually contain just about any value (including zero).