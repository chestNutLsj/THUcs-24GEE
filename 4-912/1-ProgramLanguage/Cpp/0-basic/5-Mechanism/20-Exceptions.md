
Exceptions provide a way to react to exceptional circumstances (like runtime errors) in programs by transferring control to special functions called _handlers_. （通过将控制权转移到特殊的处理函数以应对程序运行中的异常状况）

To catch exceptions, a portion of code is placed under exception inspection. This is done by enclosing that portion of code in a _try-block_. When an exceptional circumstance arises within that block, an exception is thrown that transfers the control to the exception handler. If no exception is thrown, the code continues normally and all handlers are ignored.（为了捕获异常，将一部分代码置于 `try-block` 异常检查之下，当程序运行至此发生异常时，将引发异常转移控制权到异常处理程序。若无异常，代码将继续运行，并忽略所有处理程序）

An exception is thrown by using the `throw` keyword from inside the `try` block. Exception handlers are declared with the keyword `catch`, which must be placed immediately after the `try` block:（*try-block*中使用 `throw` 关键字抛出异常，异常处理程序通过 `catch` 关键字声明，该声明必须经摁在 *try-block* 块之后）

```cpp
// exceptions
#include <iostream> 
using namespace std;

int main () {
  try
  {
    throw 20;
  }
  catch (int e)
  {
    cout << "An exception occurred. Exception Nr. " << e << '\n';
  }
  return 0;
}
```

The code under exception handling is enclosed in a `try` block. In this example this code simply throws an exception:  

```cpp
throw 20;
```

A `throw` expression accepts one parameter (in this case the integer value `20`), which is passed as an argument to the exception handler.

The exception handler is declared with the `catch` keyword immediately after the closing brace of the `try` block. The syntax for `catch` is similar to a regular function with one parameter. The type of this parameter is very important, since the type of the argument passed by the `throw` expression is checked against it, and only in the case they match, the exception is caught by that handler. （异常处理程序的参数非常重要，因为这个参数的类型要与 `throw` 检查、抛出的异常类型一致才可以正确地捕获）

Multiple handlers (i.e., `catch` expressions) can be chained; each one with a different parameter type. Only the handler whose argument type matches the type of the exception specified in the `throw` statement is executed.（可以链接多个 `catch` 表达式作为复合异常处理程序，每个参数都有不同的参数类型，仅执行参数类型与抛出异常类型一致的处理程序）

If an ellipsis (`...`) is used as the parameter of `catch`, that handler will catch any exception no matter what the type of the exception thrown. This can be used as a default handler that catches all exceptions not caught by other handlers:（如果使用 `...` 作为参数，则无论引发的异常类型如何，该处理程序都将捕获，这可以用作处理程序链的最后一位——作为默认异常处理程序）

```
try {
  // code here
}
catch (int param) { cout << "int exception"; }
catch (char param) { cout << "char exception"; }
catch (...) { cout << "default exception"; }
```

In this case, the last handler would catch any exception thrown of a type that is neither `int` nor `char`.

After an exception has been handled the program, execution resumes after the _try-catch_ block, not after the `throw` statement!.（处理程序解决异常后，在 *try-catch*块之后恢复运行，而不是 `throw` 之后）

It is also possible to nest `try-catch` blocks within more external `try` blocks. In these cases, we have the possibility that an internal `catch` block forwards the exception to its external level. This is done with the expression `throw;` with no arguments. For example:（嵌套的异常块可以将内部块的异常转发到外层处理程序中）

```
try {
  try {
      // code here
  }
  catch (int n) {
      throw;
  }
}
catch (...) {
  cout << "Exception occurred";
}
```

## Exception specification

Older code may contain _dynamic exception specifications_. They are now deprecated in C++, but still supported. A _dynamic exception specification_ follows the declaration of a function, appending a `throw` specifier to it. （动态异常规范在 Cpp 中已经弃用，但仍受支持，动态异常规范跟随在函数声明之后，通过追加 `throw` 说明符）

For example:

```
double myfunction (char param) throw (int);
```

This declares a function called `myfunction`, which takes one argument of type `char` and returns a value of type `double`. If this function throws an exception of some type other than `int`, the function calls [std::unexpected](https://cplusplus.com/unexpected) instead of looking for a handler or calling [std::terminate](https://cplusplus.com/terminate).（如果此函数引发除 `int` 之外的异常，则该程序将调用 `std::unexpected` 而不是查找处理程序或调用 `std::terminate`）

If this `throw` specifier is left empty with no type, this means that [std::unexpected](https://cplusplus.com/unexpected) is called for any exception. Functions with no `throw` specifier (regular functions) never call [std::unexpected](https://cplusplus.com/unexpected), but follow the normal path of looking for their exception handler.（`throw` 中留空表示任何异常都将抛出；没有 `throw` 说明符的函数则从不调用 `std::unexpected`，而是遵循查找其异常处理程序的正常路径）

```
int myfunction (int param) throw(); // all exceptions call unexpected
int myfunction (int param);         // normal exception handling
```

## Standard exceptions

The C++ Standard library provides a base class specifically designed to declare objects to be thrown as exceptions. It is called [std::exception](https://cplusplus.com/exception) and is defined in the [\<exception\>](https://cplusplus.com/%3Cexception%3E) header. 

This class has a virtual member function called `what` that returns a null-terminated character sequence (of type `char *`) and that can be overwritten in derived classes to contain some sort of description of the exception. （`exception` 类有一个虚拟成员函数 `what`，调用 ` what ` 成员返回以 null 结尾的字符序列，并且可以在派生类中覆盖该序列以包含某种异常描述）

```cpp
// using standard exceptions
#include <iostream> 
#include <exception> 
using namespace std;

class myexception: public exception
{
  virtual const char* what () const throw ()
  {
    return "My exception happened";
  }
} myex;

int main () {
  try
  {
    throw myex;
  }
  catch (exception& e)
  {
    cout << e.what () << '\n';
  }
  return 0;
}
```

We have placed a handler that catches exception objects by reference (notice the ampersand `&` after the type), therefore this catches also classes derived from `exception`, like our `myex` object of type `myexception`.

All exceptions thrown by components of the C++ Standard library throw exceptions derived from this `exception` class. These are:

| exception         | description                                            |
|-------------------|--------------------------------------------------------|
| bad_alloc         | thrown by new on allocation failure                    |
| bad_cast          | thrown by dynamic_cast when it fails in a dynamic cast |
| bad_exception     | thrown by certain dynamic exception specifiers         |
| bad_typeid        | thrown by typeid                                       |
| bad_function_call | thrown by empty function objects                       |
| bad_weak_ptr      | thrown by shared_ptr when passed a bad weak_ptr        |

Also deriving from `exception`, header  [\<exception\>]( https://cplusplus.com/%3Cexception%3E )  defines two generic exception types that can be inherited by custom exceptions to report errors: （同样派生自 `exception` 头文件，定义了两个泛型异常类型，可由自定义异常继承以报告错误）

| exception     | description                                        |
|---------------|----------------------------------------------------|
| logic_error   | error related to the internal logic of the program |
| runtime_error | error detected during runtime                      |

A typical example where standard exceptions need to be checked for is on memory allocation:

```cpp
// bad_alloc standard exception
#include <iostream> 
#include <exception> 
using namespace std;

int main () {
  try
  {
    int* myarray= new int[1000];
  }
  catch (exception& e)
  {
    cout << "Standard exception: " << e.what () << endl;
  }
  return 0;
}
```

The exception that may be caught by the exception handler in this example is a `bad_alloc`. Because `bad_alloc` is derived from the standard base class `exception`, it can be caught (capturing by reference, captures all related classes（通过引用捕获，捕获所有相关类）).