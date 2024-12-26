
## Overloading operators

Classes, essentially, define new types to be used in C++ code. And types in C++ not only interact with code by means of constructions and assignments. They also interact by means of operators.（本质上类定义了 Cpp 代码中使用的新类型，而类型不止通过构造和复制与代码进行交互，也通过运算符交互）

For example, take the following operation on fundamental types:

```
int a, b, c;
a = b + c;
```

Here, different variables of a fundamental type (`int`) are applied the addition operator, and then the assignment operator. For a fundamental arithmetic type, the meaning of such operations is generally obvious and unambiguous, but it may not be so for certain class types. For example:

```
struct myclass {
  string product;
  float price;
} a, b, c;
a = b + c;
```

Here, it is not obvious what the result of the addition operation on `b` and `c` does. In fact, this code alone would cause a compilation error, since the type `myclass` has no defined behavior for additions. However, C++ allows most operators to be overloaded so that their behavior can be defined for just about any type, including classes. Here is a list of all the operators that can be overloaded:

![[overridable-operator-list.png]]

Operators are overloaded by means of `operator` functions, which are regular functions with special names: their name begins by the `operator` keyword followed by the _operator sign_ that is overloaded. （运算符通过 `operator` 函数重载，函数通常是具有特殊名称的常规函数——名称以 `operator` 关键字开头，后面跟要重载的运算符）

The syntax is:

`type operator sign (parameters) { /*... body ...*/ }`

For example, _cartesian vectors_ are sets of two coordinates: `x` and `y`. The addition operation of two _cartesian vectors_ is defined as the addition both `x` coordinates together, and both `y` coordinates together. For example, adding the _cartesian vectors_ `(3,1)` and `(1,2)` together would result in `(3+1,1+2) = (4,3)`. This could be implemented in C++ with the following code:

```cpp
// overloading operators example
#include <iostream> 
using namespace std;

class CVector {
  public:
    int x, y;
    CVector () {};
    CVector (int a, int b) : x (a), y (b) {}
    CVector operator + (const CVector&);
};

CVector CVector::operator+ (const CVector& param) {
  CVector temp;
  temp.x = x + param.x;
  temp.y = y + param.y;
  return temp;
}

int main () {
  CVector foo (3,1);
  CVector bar (1,2);
  CVector result;
  result = foo + bar;
  cout << result.x << ',' << result.y << '\n';
  return 0;
}
```

If confused about so many appearances of `CVector`, consider that some of them refer to the class name (i.e., the type) `CVector` and some others are functions with that name (i.e., constructors, which must have the same name as the class). For example:

```
CVector (int, int) : x(a), y(b) {}  // function name CVector (constructor)
CVector operator+ (const CVector&); // function that returns a CVector
```

The function `operator+` of class `CVector` overloads the addition operator (`+`) for that type. Once declared, this function can be called either implicitly using the operator, or explicitly using its functional name:

```
c = a + b;
c = a.operator+ (b);
```

Both expressions are equivalent.

The operator overloads are just regular functions which can have any behavior; there is actually no requirement that the operation performed by that overload bears a relation to the mathematical or usual meaning of the operator, although it is strongly recommended. For example, a class that overloads `operator+` to actually subtract or that overloads `operator==` to fill the object with zeros, is perfectly valid, although using such a class could be challenging. （尽管重载运算符可以赋予它们任意行为，但为了可读性，建议重载后的运算符仍在数学意义上与之相关）

The parameter expected for a member function overload for operations such as `operator+` is naturally the operand to the right hand side of the operator. This is common to all binary operators (those with an operand to its left and one operand to its right). But operators can come in diverse forms. （运算符有一元、二元、三元等之分，各自的重载方式也不同，例如二元运算符预期的参数是原算符右侧的操作数，因此重新的样例如下表：）

Here you have a table with a summary of the parameters needed for each of the different operators than can be overloaded (please, replace `@` by the operator in each case):

![[operator.png]]

Where `a` is an object of class `A`, `b` is an object of class `B` and `c` is an object of class `C`. `TYPE` is just any type (that operators overloads the conversion to type `TYPE`).

Notice that some operators may be overloaded in two forms: either as a member function or as a non-member function: The first case has been used in the example above for `operator+`. But some operators can also be overloaded as non-member functions; In this case, the operator function takes an object of the proper class as first argument.（作为成员函数重载的运算符类似上文的 `operator+`，作为非成员函数重载的运算符将对应的类对象作为第一个参数）

For example:

```cpp
// non-member operator overloads
#include <iostream> 
using namespace std;

class CVector {
  public:
    int x, y;
    CVector () {}
    CVector (int a, int b) : x (a), y (b) {}
};


CVector operator+ (const CVector& lhs, const CVector& rhs) {
  CVector temp;
  temp.x = lhs.x + rhs.x;
  temp.y = lhs.y + rhs.y;
  return temp;
}

int main () {
  CVector foo (3,1);
  CVector bar (1,2);
  CVector result;
  result = foo + bar;
  cout << result.x << ',' << result.y << '\n';
  return 0;
}
```


## The keyword `this`

The keyword `this` represents a pointer to the object whose member function is being executed. It is used within a class's member function to refer to the object itself. （`this` 代表指向正在执行其成员函数的对象的指针，它在类的成员函数中用于引用对象本身）

One of its uses can be to check if a parameter passed to a member function is the object itself. For example:

```cpp
// example on this
#include <iostream> 
using namespace std;

class Dummy {
  public:
    bool isitme (Dummy& param);
};

bool Dummy:: isitme (Dummy& param)
{
  if (&m == this) return true;
  else return false;
}

int main () {
  Dummy a;
  Dummy* b = & a;
  if ( b-> isitme (a) )
    cout << "yes, & a is b\n";
  return 0;
}
```

It is also frequently used in `operator=` member functions that return objects by reference. （除了用于检查传递给成员函数的参数是否是对象本身，还经常用于引用返回对象的成员函数中，如下）

Following with the examples on _cartesian vector_ seen before, its `operator=` function could have been defined as:

```
CVector& CVector::operator= (const CVector& param)
{
  x=param.x;
  y=param.y;
  return *this;
}
```

In fact, this function is very similar to the code that the compiler generates implicitly for this class for `operator=`.  

## Static members

A class can contain static members, either data or functions.

A static data member of a class is also known as a "class variable", because there is only one common variable for all the objects of that same class, sharing the same value: i.e., its value is not different from one object of this class to another.（静态数据成员又称“类变量”，因为同一类的所有对象只有一个公共变量，共享相同的值）

For example, it may be used for a variable within a class that can contain a counter with the number of objects of that class that are currently allocated, as in the following example:

```cpp
// static members in classes
#include <iostream> 
using namespace std;

class Dummy {
  public:
    static int n;
    Dummy () { n++; };
};

int Dummy::n=0;

int main () {
  Dummy a;
  Dummy b[5];
  cout << a.n << '\n';
  Dummy * c = new Dummy;
  cout << Dummy:: n << '\n';
  delete c;
  return 0;
}
```

In fact, static members have the same properties as non-member variables but they enjoy class scope. For that reason, and to avoid them to be declared several times, they cannot be initialized directly in the class, but need to be initialized somewhere outside it. （静态成员与非成员变量有许多相似点，但不同之处在于静态成员享有类的范围，因此为了避免静态成员被多次声明，它们不能在类中直接初始化，而是在类之外的地方初始化）

As in the previous example:

```cpp
int Dummy::n = 0;
```

Because it is a common variable value for all the objects of the same class, it can be referred to as a member of any object of that class or even directly by the class name (of course this is only valid for static members):（由于静态成员是同一类的所有对象共用的变量，所以可以被该类的任何对象成员进行引用，甚至直接通过类名引用，当然也只有静态成员可以如此）

```
cout << a.n;
cout << Dummy::n;
```

These two calls above are referring to the same variable: the static variable `n` within class `Dummy` shared by all objects of this class.

Again, it is just like a non-member variable, but with a name that requires to be accessed like a member of a class (or an object).

Classes can also have static member functions. These represent the same: members of a class that are common to all object of that class, acting exactly as non-member functions but being accessed like members of the class. Because they are like non-member functions, they cannot access non-static members of the class (neither member variables nor member functions). They neither can use the keyword `this`.（同理还有静态成员函数，它们由该类的所有对象的成员共用，静态成员函数充当非成员函数，但是像类的成员一样被访问。静态成员函数无法访问类的非静态成员，也不能使用关键字 `this`）

## Const member functions

When an object of a class is qualified as a `const` object:

```
const MyClass myobject;
```

The access to its data members from outside the class is restricted to read-only, as if all its data members were `const` for those accessing them from outside the class. Note though, that the constructor is still called and is allowed to initialize and modify these data members:（const 修饰的对象，其数据成员从外部都仅可读，但构造函数仍能调用并初始化、修改这些数据成员）

```cpp
// constructor on const object
#include <iostream> 
using namespace std;

class MyClass {
  public:
    int x;
    MyClass (int val) : x (val) {}
    int get () {return x;}
};

int main () {
  const MyClass foo (10);
// foo.x = 20;            // not valid: x cannot be modified
  cout << foo.x << '\n';  // ok: data member x can be read
  return 0;
}
```

The member functions of a `const` object can only be called if they are themselves specified as `const` members; in the example above, member `get` (which is not specified as `const`) cannot be called from `foo`. （const 对象的成员函数只有被声明为 const 类型时才可以调用，要声明为 const 成员，需要将 const 关键字放在目标成员函数的参数之后）

To specify that a member is a `const` member, the `const` keyword shall follow the function prototype, after the closing parenthesis for its parameters:

```
int get() const {return x;}
```

Note that `const` can be used to qualify the type returned by a member function. This `const` is not the same as the one which specifies a member as `const`. （const 也可以用于修饰成员函数返回值，注意不要与前文修饰成员函数混淆）

Both are independent and are located at different places in the function prototype:

```
int get() const {return x;}        // const member function
const int& get() {return x;}       // member function returning a const&
const int& get() const {return x;} // const member function returning a const&
```

Member functions specified to be `const` cannot modify non-static data members nor call other non- `const` member functions. In essence, `const` members shall not modify the state of an object.（const 成员函数不能修改非静态数据成员、不得调用其它非 const 成员函数。本质上，const 成员不得修改对象的状态）

`const` objects are limited to access only member functions marked as `const`, but non- `const` objects are not restricted and thus can access both `const` and non- `const` member functions alike.（非 const 对象访问成员没有限制，无论 const 成员还是非 const 成员都可以）

You may think that anyway you are seldom going to declare `const` objects, and thus marking all members that don't modify the object as const is not worth the effort, but const objects are actually very common. Most functions taking classes as parameters actually take them by `const` reference, and thus, these functions can only access their `const` members:（大多数引入类作为参数的函数都会通过 const 引用它们，这样可以只访问它们的 const 成员，进而避免不确定的行为导致对象的改变）

```cpp
// const objects
#include <iostream> 
using namespace std;

class MyClass {
    int x;
  public:
    MyClass (int val) : x (val) {}
    const int& get () const {return x;}
};

void print (const MyClass& arg) {
  cout << arg.get () << '\n';
}

int main () {
  MyClass foo (10);
  print (foo);

  return 0;
}
```

If in this example, `get` was not specified as a `const` member, the call to `arg.get()` in the `print` function would not be possible, because `const` objects only have access to `const` member functions.

Member functions can be overloaded on their constness: i.e., a class may have two member functions with identical signatures except that one is `const` and the other is not: in this case, the `const` version is called only when the object is itself const, and the non- `const` version is called when the object is itself non- `const`.（成员函数可以在其恒常性上重载，即，一个类可以有两个同名成员函数，一个限定为 const 只允许 const 对象使用，一个不限定 const 允许非 const 对象使用）

```cpp
// overloading members on constness
#include <iostream> 
using namespace std;

class MyClass {
    int x;
  public:
    MyClass (int val) : x (val) {}
    const int& get () const {return x;}
    int& get () {return x;}
};

int main () {
  MyClass foo (10);
  const MyClass bar (20);
  foo.get () = 15;         // ok: get () returns int&amp;
// bar.get () = 25;        // not valid: get () returns const int&amp;
  cout << foo.get () << '\n';
  cout << bar.get () << '\n';

  return 0;
}
```

## Class templates

Just like we can create function templates, we can also create class templates, allowing classes to have members that use template parameters as types. For example:

```
template <class T>
class mypair {
    T values [2];
  public:
    mypair (T first, T second)
    {
      values[0]=first;  
      values[1]=second;
    }
};
```

The class that we have just defined serves to store two elements of any valid type. For example, if we wanted to declare an object of this class to store two integer values of type `int` with the values 115 and 36 we would write:

```
mypair<int> myobject (115, 36);
```

This same class could also be used to create an object to store any other type, such as:

```
mypair<double> myfloats (3.0, 2.18);
```

The constructor is the only member function in the previous class template and it has been defined inline within the class definition itself. In case that a member function is defined outside the defintion of the class template, it shall be preceded with the `template <...>` prefix:

```cpp
// class templates
#include <iostream> 
using namespace std;

template <class T>
class mypair {
    T a, b;
  public:
    mypair (T first, T second)
      {a=first; b=second;}
    T getmax ();
};

template <class T>
T mypair<T> :: getmax ()
{
  T retval;
  retval = a> b? a : b;
  return retval;
}

int main () {
  mypair < int> myobject (100, 75);
  cout << myobject.getmax ();
  return 0;
}
```

Notice the syntax of the definition of member function `getmax`:

```
template <class T>
T mypair<T>::getmax ()
```

Confused by so many `T` 's? There are three `T` 's in this declaration: The first one is the template parameter. The second `T` refers to the type returned by the function. And the third `T` (the one between angle brackets) is also a requirement: It specifies that this function's template parameter is also the class template parameter.  

## Template specialization

It is possible to define a different implementation for a template when a specific type is passed as template argument. This is called a _template specialization_. （当特定类型作为模板参数传递时，可以为模板定义不同的实现，这称为*模板专用化*）

For example, let's suppose that we have a very simple class called `mycontainer` that can store one element of any type and that has just one member function called `increase`, which increases its value. But we find that when it stores an element of type `char` it would be more convenient to have a completely different implementation with a function member `uppercase`, so we decide to declare a class template specialization for that type:

```cpp
// template specialization
#include <iostream> 
using namespace std;

// class template:
template <class T>
class mycontainer {
    T element;
  public:
    mycontainer (T arg) {element=arg;}
    T increase () {return ++element;}
};

// class template specialization:
template <>
class mycontainer <char> {
    char element;
  public:
    mycontainer (char arg) {element=arg;}
    char uppercase ()
    {
      if ((element>='a')&&(element<='z'))
      element+='A'-'a';
      return element;
    }
};

int main () {
  mycontainer< int> myint (7);
  mycontainer< char> mychar ('j');
  cout << myint.increase () << endl;
  cout << mychar.uppercase () << endl;
  return 0;
}
```

This is the syntax used for the class template specialization:

```
template <> class mycontainer <char> { ... };
```

First of all, notice that we precede the class name with `template<>` , including an empty parameter list. This is because all types are known and no template arguments are required for this specialization, but still, it is the specialization of a class template, and thus it requires to be noted as such. （注意，模板专用化在定义时需要 `template<>` ——包含一个空参数列表，这是因为在专用化时所有类型都是已知的，因此专用化不需要模板参数，但它仍然是类模板的专用化，因此需要保留 `<>`）

But more important than this prefix, is the `<char>` specialization parameter after the class template name. This specialization parameter itself identifies the type for which the template class is being specialized (`char`). Notice the differences between the generic class template and the specialization:

```
template <class T> class mycontainer { ... };
template <> class mycontainer <char> { ... };
```

The first line is the generic template, and the second one is the specialization.

When we declare specializations for a template class, we must also define all its members, even those identical to the generic template class, because there is no "inheritance" of members from the generic template to the specialization.

>[! warning] Define all members of specialized template class
>为模板类声明专用化时，必须定义其所有成员，即使是与通用模板类相同的成员，这是因为没有成员从泛型模板继承到专用化中。