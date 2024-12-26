>[!warning] Prerequisite knowledge
>You should properly understand of [[33-Dynamic-memory]]

Special member functions are member functions that are implicitly defined as member of classes under certain circumstances. There are six:

| Member function     | typical form for class C:        |
|---------------------|----------------------------------|
| Default constructor | C:: C ();                        |
| Destructor          | C::~C ();                        |
| Copy constructor    | C:: C (const C&amp;);            |
| Copy assignment     | C&amp; operator= (const C&amp;); |
| Move constructor    | C:: C (C&amp;&amp;);             |
| Move assignment     | C&amp; operator= (C&amp;&amp;);  |

Let's examine each of these:  

## Default constructor

The default constructor is the constructor called when objects of a class are declared, but are not initialized with any arguments.

If a class definition has no constructors, the compiler assumes the class to have an implicitly defined _default constructor_. Therefore, after declaring a class like this:

```
class Example {
  public:
    int total;
    void accumulate (int x) { total += x; }
};
```

The compiler assumes that `Example` has a _default constructor_. Therefore, objects of this class can be constructed by simply declaring them without any arguments:  

```cpp
Example exp;
```

But as soon as a class has some constructor taking any number of parameters explicitly declared, the compiler no longer provides an implicit default constructor, and no longer allows the declaration of new objects of that class without arguments. For example, the following class:

```
class Example2 {
  public:
    int total;
    Example2 (int initial_value) : total(initial_value) { };
    void accumulate (int x) { total += x; };
};
```

Here, we have declared a constructor with a parameter of type `int`. Therefore the following object declaration would be correct:

```
Example2 ex (100);   // ok: calls constructor
```

But the following is not valid:  

```
Example2 ex;         // not valid: no default constructor
```

Would not be valid, since the class has been declared with an explicit constructor taking one argument and that replaces the implicit _default constructor_ taking none.

Therefore, if objects of this class need to be constructed without arguments, the proper _default constructor_ shall also be declared in the class.（一旦类中显式定义了任何一种构造函数，那么编译器就不再隐式添加一个无参构造函数，此时声明该类的对象必须填写参数，如果想要无参的对象，则要相应显式地编写一个无参构造函数）

For example:

```cpp
// classes and default constructors
#include <iostream>
#include <string>

using namespace std;

class Example3 {
    string data;
public:
    Example3(const string &str) : data(str) {}

    Example3() {}

    const string &content() const { return data; }
};

int main() {
    Example3 foo;
    Example3 bar("Example");

    cout << "bar's content: " << bar.content() << '\n';
    return 0;
}
```

Here, `Example3` has a _default constructor_ (i.e., a constructor without parameters) defined as an empty block:  

```cpp
Example3() {}
```

This allows objects of class `Example3` to be constructed without arguments (like `foo` was declared in this example). Normally, a default constructor like this is implicitly defined for all classes that have no other constructors and thus no explicit definition is required. But in this case, `Example3` has another constructor:

```
Example3 (const string& str);
```

And when any constructor is explicitly declared in a class, no implicit _default constructors_ is automatically provided.  

## Destructor

Destructors fulfill the opposite functionality of _constructors_: They are responsible for the necessary cleanup needed by a class when its lifetime ends. The classes we have defined in previous chapters did not allocate any resource and thus did not really require any clean up. （析构函数负责类在其生存期结束时所需的必要清理）

But now, let's imagine that the class in the last example allocates dynamic memory to store the string it had as data member; in this case, it would be very useful to have a function called automatically at the end of the object's life in charge of releasing this memory.

To do this, we use a _destructor_. A destructor is a member function very similar to a _default constructor_: it takes no arguments and returns nothing, not even `void`. It also uses the class name as its own name, but preceded with a tilde sign (`~`):

```cpp
// destructors
#include "iostream"
#include "string"
using namespace std;

class Example4 {
    string* ptr;
  public:
    // constructors:
    Example4 () : ptr (new string) {}
    Example4 (const string& str) : ptr (new string (str)) {}
    // destructor:
    ~Example4 () {delete ptr;}
    // access content:
    const string& content () const {return *ptr;}
};

int main () {
  Example4 foo;
  Example4 bar ("Example");

  cout << "bar's content: " << bar.content () << '\n';
  return 0;
}
```

On construction, `Example4` allocates storage for a `string`. Storage that is later released by the destructor.

The destructor for an object is called at the end of its lifetime; in the case of `foo` and `bar` this happens at the end of function `main`.

## Copy constructor

When an object is passed a named object of its own type as argument, its _copy constructor_ is invoked in order to construct a copy. （当对象作为参数传递其自身类型的命名对象时，将调用*复制构造函数*以构造其副本）

A _copy constructor_ is a constructor whose first parameter is of type _reference to the class_ itself (possibly `const` qualified) and which can be invoked with a single argument of this type.（复制构造函数是一个构造函数，其第一个参数的类型引用是类本身，并且可以使用此类型的单个参数调用） 

For example, for a class `MyClass`, the _copy constructor_ may have the following signature:

```
MyClass::MyClass (const MyClass&);
```

If a class has no custom _copy_ nor _move_ constructors (or assignments) defined, an implicit _copy constructor_ is provided. This copy constructor simply performs a copy of its own members.（如果类没有自定义复制或移动构造函数，则提供隐式复制构造函数，这个复制构造函数只是执行其自己成员的副本）

For example, for a class such as:

```
class MyClass {
  public:
    int a, b; string c;
};
```

An implicit _copy constructor_ is automatically defined. The definition assumed for this function performs a _shallow copy_, roughly equivalent to（自动隐式定义的复制构造函数，假定执行*浅拷贝*，大致相当于：）

```
MyClass::MyClass(const MyClass& x) : a(x.a), b(x.b), c(x.c) {}
```

This default _copy constructor_ may suit the needs of many classes. But _shallow copies_ only copy the members of the class themselves, and this is probably not what we expect for classes like class `Example4` we defined above, because it contains pointers of which it handles its storage. For that class, performing a _shallow copy_ means that the pointer value is copied, but not the content itself; This means that both objects (the copy and the original) would be sharing a single `string` object (they would both be pointing to the same object), and at some point (on destruction) both objects would try to delete the same block of memory, probably causing the program to crash on runtime. This can be solved by defining the following custom _copy constructor_ that performs a _deep copy_: （默认的*复制构造函数*可能满足很多类的需求，而*浅拷贝*只复制类本身的成员，在 Example4 类中浅拷贝不能满足需求，因为其中包含处理动态存储的指针，对于该类执行浅拷贝意味着复制指针值而不是内容本身，这将导致副本和原始对象共享同一对象，并在销毁时两个对象会尝试删除相同的内存块，这将导致程序在运行时崩溃。因此可以通过自定义的复制构造函数来执行深层复制）

```cpp
// copy constructor: deep copy
#include <iostream> 
#include <string> 
using namespace std;

class Example5 {
    string* ptr;
  public:
    Example5 (const string& str) : ptr (new string (str)) {}
    ~Example5 () {delete ptr;}
    // copy constructor:
    Example5 (const Example5& x) : ptr (new string (x.content ())) {}
    // access content:
    const string& content () const {return *ptr;}
};

int main () {
  Example5 foo ("Example");
  Example5 bar = foo;

  cout << "bar's content: " << bar.content () << '\n';
  return 0;
}
```

The _deep copy_ performed by this _copy constructor_ allocates storage for a new string, which is initialized to contain a copy of the original object. In this way, both objects (copy and original) have distinct copies of the content stored in different locations.（通过复制构造函数执行的*深拷贝*为新字符串分配存储，该字符串初始化为原始对象的副本，这样副本和原始对象都有存储在不同位置的相同内容）

## Copy assignment

Objects are not only copied on construction, when they are initialized: They can also be copied on any assignment operation. （对象不仅在初始化时通过构造复制，也可以在赋值操作中进行复制）

See the difference:

```
MyClass foo;
MyClass bar (foo);       // object initialization: copy constructor called
MyClass baz = foo;       // object initialization: copy constructor called
foo = bar;               // object already initialized: copy assignment called
```

Note that `baz` is initialized on construction using an _equal sign_, but this is not an assignment operation! (although it may look like one): The declaration of an object is not an assignment operation, it is just another of the syntaxes to call single-argument constructors.（注意 `baz` 在构造时使用等号初始化，但这并不是赋值操作，对象的声明不属于赋值操作，它只是调用单参数构造函数的另一种语法）

The assignment on `foo` is an assignment operation. No object is being declared here, but an operation is being performed on an existing object; `foo`.（`foo` 上是对现有对象的赋值操作）

The _copy assignment operator_ is an overload of `operator=` which takes a _value_ or _reference_ of the class itself as parameter. The return value is generally a reference to `*this` (although this is not required). （复制赋值运算符是对 `=` 的重载，其以类本身的值或引用作为参数，返回值通常是对 `*this` 的引用）

For example, for a class `MyClass`, the _copy assignment_ may have the following signature:

```
MyClass& operator= (const MyClass&);
```

The _copy assignment operator_ is also a _special function_ and is also defined implicitly if a class has no custom _copy_ nor _move_ assignments (nor move constructor) defined.（复制赋值运算符也是一个特殊函数，如果类没有自定义*复制*或*移动*赋值，则会隐式定义）

But again, the _implicit_ version performs a _shallow copy_ which is suitable for many classes, but not for classes with pointers to objects they handle its storage, as is the case in `Example5`.（同样的，隐式版本的复制赋值运算符执行了一种浅拷贝，这对一些类是合适的，但不适用于具有指向它们处理其存储对象的指针的类，如 `Example5` 这样的类）

In this case, not only the class incurs the risk of deleting the pointed object twice, but the assignment creates memory leaks by not deleting the object pointed by the object before the assignment. These issues could be solved with a _copy assignment_ that deletes the previous object and performs a _deep copy_（在这种情况下，不止类会产生两次删除指向对象的风险，而且这种赋值还会在不删除对象指向的对象时执行，这将会导致内存泄漏。这些问题可以通过*复制赋值*来解决，该赋值删除从前的对象并执行深层拷贝）:

```
Example5& operator= (const Example5& x) {
  delete ptr;                      // delete currently pointed string
  ptr = new string (x.content());  // allocate space for new string, and copy
  return *this;
}
```

Or even better, since its `string` member is not constant, it could re-utilize the same `string` object:

```
Example5& operator= (const Example5& x) {
  *ptr = x.content();
  return *this;
}
```

## Move constructor and assignment

Similar to copying, moving also uses the value of an object to set the value to another object. But, unlike copying, the content is actually transferred from one object (the source) to the other (the destination): the source loses that content, which is taken over by the destination. This moving only happens when the source of the value is an _unnamed object_.（移动也是将对象的值设置为另一个对象的方法，其与复制的区别在于内容从源对象传输到目标对象，而源丢失内容，内容被目标接管。仅当值的源是*未命名对象*时才会发生移动）

_Unnamed objects_ are objects that are temporary in nature, and thus haven't even been given a name. Typical examples of _unnamed objects_ are return values of functions or type-casts. （未命名对象本质上是临时的，甚至没有赋予名称，典型例子就是函数或类型转换的返回值）

Using the value of a temporary object such as these to initialize another object or to assign its value, does not really require a copy: the object is never going to be used for anything else, and thus, its value can be _moved into_ the destination object. These cases trigger the _move constructor_ and _move assignments_:（使用这类临时对象的值来初始化另一个对象或赋值时，实际上不需要副本，临时对象永远不会用于其他任何内容，其值直接移动到目标对象中。这些情形会触发移动构造函数和移动赋值）

The _move constructor_ is called when an object is initialized on construction using an unnamed temporary. Likewise, the _move assignment_ is called when an object is assigned the value of an unnamed temporary:（当使用未命名的临时对象在构造初始化对象时，将会调用*移动构造函数*；当为对象分配了未命名临时对象的值是，也会调用*移动赋值*）

```
MyClass fn();            // function returning a MyClass object
MyClass foo;             // default constructor
MyClass bar = foo;       // copy constructor
MyClass baz = fn();      // move constructor
foo = bar;               // copy assignment
baz = MyClass();         // move assignment
```

Both the value returned by `fn` and the value constructed with `MyClass` are unnamed temporaries. In these cases, there is no need to make a copy, because the unnamed object is very short-lived and can be acquired by the other object when this is a more efficient operation.（`fn` 和 `MyClass` 返回的值都是未命名临时值，由于其生存期非常短，不需要创建副本而直接移动构造、赋值给另一个对象）

The move constructor and move assignment are members that take a parameter of type _rvalue reference to the class_ itself:（移动构造和移动赋值都是成员，其参数是*类的右值引用*这一类型）

```
MyClass (MyClass&&);             // move-constructor
MyClass& operator= (MyClass&&);  // move-assignment
```

An _rvalue reference_ is specified by following the type with two ampersands (`&&`). As a parameter, an _rvalue reference_ matches arguments of temporaries of this type.（右值引用通过在类型后添加 `&&` 来指定，作为参数，右值引用使用这一类型的临时变量的作为参数） ^d293e1

The concept of moving is most useful for objects that manage the storage they use, such as objects that allocate storage with new and delete. In such objects, copying and moving are really different operations:  
- Copying from A to B means that new memory is allocated to B and then the entire content of A is copied to this new memory allocated for B.  
- Moving from A to B means that the memory already allocated to A is transferred to B without allocating any new storage. It involves simply copying the pointer.
（移动对于管理其使用存储的对象最有用，例如涉及 new 和 delete 的对象，这类对象中复制和移动在物理上实现是不同的：
- 从 A 复制到 B 意味着新的内存分配给 B，然后将 A 的全部内容复制到新的内存中；
- 从 A 移动到 B 意味着已分配给 A 的内存将传输到 B，而不涉及新的存储，只是简单地复制指针）

For example:

```cpp
// move constructor/assignment
#include <iostream> 
#include <string> 
using namespace std;

class Example6 {
    string* ptr;
  public:
    Example6 (const string& str) : ptr (new string (str)) {}
    ~Example6 () {delete ptr;}
    // move constructor
    Example6 (Example6&& x) : ptr (x.ptr) {x.ptr=nullptr;}
    // move assignment
    Example6& operator= (Example6&& x) {
      delete ptr; 
      ptr = x.ptr;
      x.ptr=nullptr;
      return *this;
    }
    // access content:
    const string& content () const {return *ptr;}
    // addition:
    Example6 operator+(const Example6& rhs) {
      return Example6 (content ()+rhs.content ());
    }
};


int main () {
  Example6 foo ("Exam");
  Example6 bar = Example6 ("ple");   // move-construction
  
  foo = foo + bar;                  // move-assignment

  cout << "foo's content: " << foo.content () << '\n';
  return 0;
}
```

Compilers already optimize many cases that formally require a move-construction call in what is known as _Return Value Optimization_. Most notably, when the value returned by a function is used to initialize an object. In these cases, the _move constructor_ may actually never get called. （编译器已经优化了许多正式需要移动构造调用的情况，即*返回值优化*，特别是在函数返回的值用于初始化对象时。这些情况下，移动构造函数实际上可能永远不会被调用）

Note that even though _rvalue references_ can be used for the type of any function parameter, it is seldom useful for uses other than the _move constructor_. Rvalue references are tricky, and unnecessary uses may be the source of errors quite difficult to track.（值得注意，即使右值引用可用于任何函数参数的类型，但仍很少用于移动构造函数以外的用途。使用它非常棘手，造成的程序错误可能很难排查）

## Implicit members

The six _special members functions_ described above are members implicitly declared on classes under certain circumstances:

| Member function     | implicitly defined:                                                   | default definition: |
| ------------------- | --------------------------------------------------------------------- | ------------------- |
| Default constructor | if no other constructors                                              | does nothing        |
| Destructor          | if no destructor                                                      | does nothing        |
| Copy constructor    | if no move constructor and no move assignment                         | copies all members  |
| Copy assignment     | if no move constructor and no move assignment                         | copies all members  |
| Move constructor    | if no destructor, no copy constructor and no copy nor move assignment | moves all members   |
| Move assignment     | if no destructor, no copy constructor and no copy nor move assignment | moves all members   |

Notice how not all _special member functions_ are implicitly defined in the same cases. This is mostly due to backwards compatibility with C structures and earlier C++ versions, and in fact some include deprecated cases. Fortunately, each class can select explicitly which of these members exist with their default definition or which are deleted by using the keywords `default` and `delete`, respectively. （这些特殊成员函数并不是相同情况下隐式定义，这与 C 和早期 Cpp 的向后兼容有关，实际上有些用法已经弃用。每个类都可以显式选择这些成员中哪些成员可以默认定义存在，或使用 `default` 和 `delete` 关键字删除哪些成员）

The syntax is either one of:

```
function_declaration = default;
function_declaration = delete;
```

For example:

```cpp
// default and delete implicit members
#include <iostream> 
using namespace std;

class Rectangle {
    int width, height;
  public:
    Rectangle (int x, int y) : width (x), height (y) {}
    Rectangle () = default;
    Rectangle (const Rectangle& other) = delete;
    int area () {return width*height;}
};

int main () {
  Rectangle foo;
  Rectangle bar (10,20);

  cout << "bar's area: " << bar.area () << '\n';
  return 0;
}
```

Here, `Rectangle` can be constructed either with two `int` arguments or be _default-constructed_ (with no arguments). It cannot however be _copy-constructed_ from another `Rectangle` object, because this function has been deleted. （Rectangle 类的对象可以通过两个 int 参数进行构造，也可以使用无参数的默认构造，但不能使用从另一个对象进行复制构造）

Therefore, assuming the objects of the last example, the following statement would not be valid:  

```cpp
Rectangle baz (foo);
```

It could, however, be made explicitly valid by defining its copy constructor as:

```
Rectangle:: Rectangle (const Rectangle& other) = default;
```

Which would be essentially equivalent to:

```
Rectangle:: Rectangle (const Rectangle& other) : width (other. width), height (other. height) {}
```

Note that, the keyword `default` does not define a member function equal to the _default constructor_ (i.e., where _default constructor_ means constructor with no parameters), but equal to the constructor that would be implicitly defined if not deleted. （`default` 关键字不是定义等于默认构造函数的成员函数，而是等于如果不删除将会隐式定义的构造函数）

In general, and for future compatibility, classes that explicitly define one copy/move constructor or one copy/move assignment but not both, are encouraged to specify either `delete` or `default` on the other special member functions they don't explicitly define.（通常为了将来的兼容性考虑，建议显式定义一个复制/移动构造函数或复制/移动赋值而不是二者同时定义的类，以此指定 `delete` 或 `default` 修饰的其它未显式定义的成员函数）