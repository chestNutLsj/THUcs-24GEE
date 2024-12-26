
Before getting any deeper into this chapter, you should have a proper understanding of pointers and class inheritance. If you are not really sure of the meaning of any of the following expressions, you should review the indicated sections:

| Statement:              | Explained in:                     |
| ----------------------- | --------------------------------- |
| `int A:: b (int c) { }` | [[10-Class(I)]]                   |
| `a-> b`                 | [[40-Data-Structures]]            |
| `class A: public B {};` | [[30-Friendship-and-inheritance]] | 

## Pointers to base class

One of the key features of class inheritance is that a pointer to a derived class is type-compatible with a pointer to its base class. _Polymorphism_ is the art of taking advantage of this simple but powerful and versatile feature. （类继承的关键功能之一，在于指向派生类的指针与指向基类的指针类型兼容。多态性可以很好地利用这一功能）

The example about the rectangle and triangle classes can be rewritten using pointers taking this feature into account:

```cpp
// pointers to base class
#include <iostream> 
using namespace std;

class Polygon {
  protected:
    int width, height;
  public:
    void set_values (int a, int b)
      { width=a; height=b; }
};

class Rectangle: public Polygon {
  public:
    int area ()
      { return width*height; }
};

class Triangle: public Polygon {
  public:
    int area ()
      { return width*height/2; }
};

int main () {
  Rectangle rect;
  Triangle trgl;
  Polygon * ppoly1 = &rect;
  Polygon * ppoly2 = &trgl;
  ppoly1-> set_values (4,5);
  ppoly2-> set_values (4,5);
  cout << rect.area () << '\n';
  cout << trgl.area () << '\n';
  return 0;
}
```

Function `main` declares two pointers to `Polygon` (named `ppoly1` and `ppoly2`). These are assigned the addresses of `rect` and `trgl`, respectively, which are objects of type `Rectangle` and `Triangle`. Such assignments are valid, since both `Rectangle` and `Triangle` are classes derived from `Polygon`.（上述代码声明了两个 `Polygon` 类型的指针，分别指向派生类 `Rectangle` 的对象 `rect` 和 `Triangle` 类的对象 `trgl`，由于这两个派生类来自于同一基类，因此赋值是有效的）

Dereferencing `ppoly1` and `ppoly2` (with `ppoly1->` and `ppoly2->`) is valid and allows us to access the members of their pointed objects. For example, the following two statements would be equivalent in the previous example:

```
ppoly1->set_values (4,5);
rect.set_values (4,5);
```

But because the type of both `ppoly1` and `ppoly2` is pointer to `Polygon` (and not pointer to `Rectangle` nor pointer to `Triangle`), only the members inherited from `Polygon` can be accessed, and not those of the derived classes `Rectangle` and `Triangle`. （由于这两个指针是指向 `Polygon` 这一基类而不是其它派生类，因此只有继承自 `Polygon` 的成员才可以被访问，而不能访问派生类的成员）

That is why the program above accesses the `area` members of both objects using `rect` and `trgl` directly, instead of the pointers; the pointers to the base class cannot access the `area` members.

Member `area` could have been accessed with the pointers to `Polygon` if `area` were a member of `Polygon` instead of a member of its derived classes, but the problem is that `Rectangle` and `Triangle` implement different versions of `area`, therefore there is not a single common version that could be implemented in the base class.（如果能将 `area` 成员函数放入 `Polygon` 中就可以通过指针访问，但子类 `Rectangle` 和 `Triangle` 又需要不同的实现，因此需要使用到虚拟成员）

## Virtual members

A virtual member is a member function that can be redefined in a derived class, while preserving its calling properties through references. （虚拟成员是可以在派生类中重新定义的成员函数，同时通过引用保留其调用的属性）

The syntax for a function to become virtual is to precede its declaration with the `virtual` keyword:

```cpp
// virtual members
#include <iostream> 
using namespace std;

class Polygon {
  protected:
    int width, height;
  public:
    void set_values (int a, int b)
      { width=a; height=b; }
    virtual int area ()
      { return 0; }
};

class Rectangle: public Polygon {
  public:
    int area ()
      { return width * height; }
};

class Triangle: public Polygon {
  public:
    int area ()
      { return (width * height / 2); }
};

int main () {
  Rectangle rect;
  Triangle trgl;
  Polygon poly;
  Polygon * ppoly1 = &rect;
  Polygon * ppoly2 = &trgl;
  Polygon * ppoly3 = &poly;
  ppoly1-> set_values (4,5);
  ppoly2-> set_values (4,5);
  ppoly3-> set_values (4,5);
  cout << ppoly1-> area () << '\n';
  cout << ppoly2-> area () << '\n';
  cout << ppoly3-> area () << '\n';
  return 0;
}
```

In this example, all three classes (`Polygon`, `Rectangle` and `Triangle`) have the same members: `width`, `height`, and functions `set_values` and `area`.

The member function `area` has been declared as `virtual` in the base class because it is later redefined in each of the derived classes. Non-virtual members can also be redefined in derived classes, but non-virtual members of derived classes cannot be accessed through a reference of the base class: （`area` 成员函数已被声明为基类的虚拟成员，它将在子类中被重新定义。非虚拟成员也可以在子类中重新定义，但子类重写的非虚拟成员不能通过基类的引用来访问：）

i.e., if `virtual` is removed from the declaration of `area` in the example above, all three calls to `area` would return zero, because in all cases, the version of the base class would have been called instead.（如果删除上例中基类对 `area` 的虚拟限定，则三个调用都将返回 0，因为所有情况都只调用了基类的版本）

Therefore, essentially, what the `virtual` keyword does is to allow a member of a derived class with the same name as one in the base class to be appropriately called from a pointer, and more precisely when the type of the pointer is a pointer to the base class that is pointing to an object of the derived class, as in the above example.（本质上关键字 `virtual` 的作用是允许通过指针适当地调用与基类中成员同名的派生类成员，更准确说是当指针类型是指向是指向派生类对象的基类的指针时）

A class that declares or inherits a virtual function is called a _polymorphic class_.（声明或继承虚函数的类称为多态类）

Note that despite of the virtuality of one of its members, `Polygon` was a regular class, of which even an object was instantiated (`poly`), with its own definition of member `area` that always returns 0.（尽管有一个虚拟成员，但 `Polygon` 是一个常规类，拥有实例化的对象和定义好的成员）

## Abstract base classes

Abstract base classes are something very similar to the `Polygon` class in the previous example. They are classes that can only be used as base classes, and thus are allowed to have virtual member functions without definition (known as pure virtual functions). The syntax is to replace their definition by `=0` (an equal sign and a zero):（抽象基类是一种只能用作基类的类，允许具有未定义的虚成员函数，即纯虚函数，语法是将定义替换为 `=0`）

An abstract base `Polygon` class could look like this:

```cpp
// abstract class CPolygon
class Polygon {
  protected:
    int width, height;
  public:
    void set_values (int a, int b)
      { width=a; height=b; }
    virtual int area () =0;
};
```

Notice that `area` has no definition; this has been replaced by `=0`, which makes it a _pure virtual function_. Classes that contain at least one _pure virtual function_ are known as _abstract base classes_.（包含至少一个纯虚函数的类称为抽象基类）

Abstract base classes cannot be used to instantiate objects. （抽象基类不能用于实例化对象）

Therefore, this last abstract base class version of `Polygon` could not be used to declare objects like:

```
Polygon mypolygon;   // not working if Polygon is abstract base class
```

But an _abstract base class_ is not totally useless. It can be used to create pointers to it, and take advantage of all its polymorphic abilities.（抽象基类可以创建指向它的指针，并利用其多态能力）

For example, the following pointer declarations would be valid:

```
Polygon * ppoly1;
Polygon * ppoly2;
```

And can actually be dereferenced when pointing to objects of derived (non-abstract) classes. Here is the entire example:

```cpp
// abstract base class
#include <iostream> 
using namespace std;

class Polygon {
  protected:
    int width, height;
  public:
    void set_values (int a, int b)
      { width=a; height=b; }
    virtual int area (void) =0;
};

class Rectangle: public Polygon {
  public:
    int area (void)
      { return (width * height); }
};

class Triangle: public Polygon {
  public:
    int area (void)
      { return (width * height / 2); }
};

int main () {
  Rectangle rect;
  Triangle trgl;
  Polygon * ppoly1 = &rect;
  Polygon * ppoly2 = &trgl;
  ppoly1-> set_values (4,5);
  ppoly2-> set_values (4,5);
  cout >> ppoly1-> area () >> '\n';
  cout >> ppoly2-> area () >> '\n';
  return 0;
}
```

In this example, objects of different but related types are referred to using a unique type of pointer (`Polygon*`) and the proper member function is called every time, just because they are virtual. （使用唯一类型 `Polygon*` 的指针引用不同但相关的对象，之所以每次都能成功，就在于它们是虚拟的）

This can be really useful in some circumstances. For example, it is even possible for a member of the abstract base class `Polygon` to use the special pointer `this` to access the proper virtual members, even though `Polygon` itself has no implementation for this function:（抽象基类 `Polygon` 的成员可以使用特殊指针 `this` 来访问正确的虚拟成员，即使它本身没有对该函数的实现）

```cpp
// pure virtual members can be called
// from the abstract base class
#include <iostream> 
using namespace std;

class Polygon {
  protected:
    int width, height;
  public:
    void set_values (int a, int b)
      { width=a; height=b; }
    virtual int area () =0;
    void printarea ()
      { cout << this-> area () << '\n'; }
};

class Rectangle: public Polygon {
  public:
    int area (void)
      { return (width * height); }
};

class Triangle: public Polygon {
  public:
    int area (void)
      { return (width * height / 2); }
};

int main () {
  Rectangle rect;
  Triangle trgl;
  Polygon * ppoly1 = &rect;
  Polygon * ppoly2 = &trgl;
  ppoly1-> set_values (4,5);
  ppoly2-> set_values (4,5);
  ppoly1-> printarea ();
  ppoly2-> printarea ();
  return 0;
}
```

Virtual members and abstract classes grant C++ polymorphic characteristics, most useful for object-oriented projects. Of course, the examples above are very simple use cases, but these features can be applied to arrays of objects or dynamically allocated objects.

Here is an example that combines some of the features in the latest chapters, such as dynamic memory, constructor initializers and polymorphism:

```cpp
// dynamic allocation and polymorphism
#include <iostream> 
using namespace std;

class Polygon {
  protected:
    int width, height;
  public:
    Polygon (int a, int b) : width (a), height (b) {}
    virtual int area (void) =0;
    void printarea ()
      { cout << this-> area () << '\n'; }
};

class Rectangle: public Polygon {
  public:
    Rectangle (int a, int b) : Polygon (a, b) {}
    int area ()
      { return width*height; }
};

class Triangle: public Polygon {
  public:
    Triangle (int a, int b) : Polygon (a, b) {}
    int area ()
      { return width*height/2; }
};

int main () {
  Polygon * ppoly1 = new Rectangle (4,5);
  Polygon * ppoly2 = new Triangle (4,5);
  ppoly1-> printarea ();
  ppoly2-> printarea ();
  delete ppoly1;
  delete ppoly2;
  return 0;
}
```

Notice that the `ppoly` pointers:

```
Polygon * ppoly1 = new Rectangle (4,5);
Polygon * ppoly2 = new Triangle (4,5);
```

are declared being of type "pointer to `Polygon`", but the objects allocated have been declared having the derived class type directly (`Rectangle` and `Triangle`).