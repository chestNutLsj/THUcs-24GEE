
## Friend functions

In principle, private and protected members of a class cannot be accessed from outside the same class in which they are declared. However, this rule does not apply to _"friends"_.

Friends are functions or classes declared with the `friend` keyword. （友元是用 `friend` 声明的函数或类）

A non-member function can access the private and protected members of a class if it is declared a _friend_ of that class. （如果被声明为类的友元，非成员函数可以访问该类的私有成员和受保护成员） That is done by including a declaration of this external function within the class, and preceding it with the keyword `friend`:

```cpp
// friend functions
#include <iostream> 
using namespace std;

class Rectangle {
    int width, height;
  public:
    Rectangle () {}
    Rectangle (int x, int y) : width (x), height (y) {}
    int area () {return width * height;}
    friend Rectangle duplicate (const Rectangle&);
};

Rectangle duplicate (const Rectangle& param)
{
  Rectangle res;
  res.width = param.width*2;
  res.height = param.height*2;
  return res;
}

int main () {
  Rectangle foo;
  Rectangle bar (2,3);
  foo = duplicate (bar);
  cout << foo.area () << '\n';
  return 0;
}
```

The `duplicate` function is a _friend_ of class `Rectangle`. Therefore, function `duplicate` is able to access the members `width` and `height` (which are private) of different objects of type `Rectangle`. Notice though that neither in the declaration of `duplicate` nor in its later use in `main`, function `duplicate` is considered a member of class `Rectangle`. It isn't! It simply has access to its private and protected members without being a member. （需要注意的是，无论 `duplicate` 的声明还是在接下来的使用中，都不应视作 `Rectangle` 类的成员函数，它仅是有权访问私有成员，而不是作为成员）

Typical use cases of friend functions are operations that are conducted between two different classes accessing private or protected members of both.

## Friend classes

Similar to friend functions, a friend class is a class whose members have access to the private or protected members of another class:

```cpp
// friend class
#include <iostream> 
using namespace std;

class Square;

class Rectangle {
    int width, height;
  public:
    int area ()
      {return (width * height);}
    void convert (Square a);
};

class Square {
  friend class Rectangle;
  private:
    int side;
  public:
    Square (int a) : side (a) {}
};

void Rectangle:: convert (Square a) {
  width = a.side;
  height = a.side;
}
  
int main () {
  Rectangle rect;
  Square sqr (4);
  rect.convert (sqr);
  cout << rect.area ();
  return 0;
}
```

In this example, class `Rectangle` is a friend of class `Square` allowing `Rectangle` 's member functions to access private and protected members of `Square`. More concretely, `Rectangle` accesses the member variable `Square::side`, which describes the side of the square.

There is something else new in this example: at the beginning of the program, there is an empty declaration of class `Square`. This is necessary because class `Rectangle` uses `Square` (as a parameter in member `convert`), and `Square` uses `Rectangle` (declaring it a friend). （在程序开头有一个空的 `Square` 类声明，这是有必要的，因为 `Rectangle` 类使用 `Square` 作为其成员函数的参数）

Friendships are never corresponded unless specified: In our example, `Rectangle` is considered a friend class by `Square`, but Square is not considered a friend by `Rectangle`. Therefore, the member functions of `Rectangle` can access the protected and private members of `Square` but not the other way around. Of course, `Square` could also be declared friend of `Rectangle`, if needed, granting such an access.

Another property of friendships is that they are not transitive: The friend of a friend is not considered a friend unless explicitly specified.

>[!warning] Friendship don't transmit
 友元只有指定才生效，而不会因为指定 A 是 B 的友元，而同时推断出 B 也是 A 的友元；
 同样友谊不会传递，朋友的朋友不是我的朋友

## Inheritance between classes

Classes in C++ can be extended, creating new classes which retain characteristics of the base class. This process, known as inheritance, involves a _base class_ and a _derived class_: The _derived class_ inherits the members of the _base class_, on top of which it can add its own members. （Cpp 中类可以扩展，从而创建保留基类特征的新类，该过程称为继承。派生类继承基类的成员，可以选择在基类上添加自己的成员）

For example, let's imagine a series of classes to describe two kinds of polygons: rectangles and triangles. These two polygons have certain common properties, such as the values needed to calculate their areas: they both can be described simply with a height and a width (or base).

This could be represented in the world of classes with a class `Polygon` from which we would derive the two other ones: `Rectangle` and `Triangle`:

![[inheritance.png]]

The `Polygon` class would contain members that are common for both types of polygon. In our case: `width` and `height`. And `Rectangle` and `Triangle` would be its derived classes, with specific features that are different from one type of polygon to the other.

Classes that are derived from others inherit all the accessible members of the base class.（派生类继承基类的所有可访问成员）

That means that if a base class includes a member `A` and we derive a class from it with another member called `B`, the derived class will contain both member `A` and member `B`.

The inheritance relationship of two classes is declared in the derived class. Derived classes definitions use the following syntax:

```
class derived_class_name: public base_class_name  
{ /*...*/ };  
```

Where `derived_class_name` is the name of the derived class and `base_class_name` is the name of the class on which it is based. The `public` access specifier may be replaced by any one of the other access specifiers (`protected` or `private`). This access specifier limits the most accessible level for the members inherited from the base class: The members with a more accessible level are inherited with this level instead, while the members with an equal or more restrictive access level keep their restrictive level in the derived class. （需要注意的是 public 处可以使用 private、protected 替换，这将限制从基类继承成员的最高可访问级别——具有更易于访问级别的成员将继承自该机别，而具有相同或更严格访问级别的成员保留原来的限制级别。一句话就是较低访问级别优先）

```cpp
// derived classes
#include <iostream> 
using namespace std;

class Polygon {
  protected:
    int width, height;
  public:
    void set_values (int a, int b)
      { width=a; height=b;}
 };

class Rectangle: public Polygon {
  public:
    int area ()
      { return width * height; }
 };

class Triangle: public Polygon {
  public:
    int area ()
      { return width * height / 2; }
  };
  
int main () {
  Rectangle rect;
  Triangle trgl;
  rect.set_values (4,5);
  trgl.set_values (4,5);
  cout << rect.area () << '\n';
  cout << trgl.area () << '\n';
  return 0;
}
```

The objects of the classes `Rectangle` and `Triangle` each contain members inherited from `Polygon`. These are: `width`, `height` and `set_values`.

The `protected` access specifier used in class `Polygon` is similar to `private`. Its only difference occurs in fact with inheritance: When a class inherits another one, the members of the derived class can access the protected members inherited from the base class, but not its private members. （`Polygon` 类中使用的 protected 限定符与 private 相似，唯一区别在于继承上，派生类可以访问 protected 成员，而不能访问 private 成员）

By declaring `width` and `height` as `protected` instead of `private`, these members are also accessible from the derived classes `Rectangle` and `Triangle`, instead of just from members of `Polygon`. If they were public, they could be accessed just from anywhere.

We can summarize the different access types according to which functions can access them in the following way:

| Access                    | public | protected | private |
|---------------------------|--------|-----------|---------|
| members of the same class | yes    | yes       | yes     |
| members of derived class  | yes    | yes       | no      |
| not members               | yes    | no        | no      |

Where "not members" represents any access from outside the class, such as from `main`, from another class or from a function.

In the example above, the members inherited by `Rectangle` and `Triangle` have the same access permissions as they had in their base class `Polygon`:

```
Polygon::width           // protected access
Rectangle::width         // protected access

Polygon::set_values()    // public access
Rectangle::set_values()  // public access
```

This is because the inheritance relation has been declared using the `public` keyword on each of the derived classes:

```
class Rectangle: public Polygon { /* ... */ }
```

This `public` keyword after the colon (`:`) denotes the most accessible level the members inherited from the class that follows it (in this case `Polygon`) will have from the derived class (in this case `Rectangle`). Since `public` is the most accessible level, by specifying this keyword the derived class will inherit all the members with the same levels they had in the base class.

With `protected`, all public members of the base class are inherited as `protected` in the derived class. Conversely, if the most restricting access level is specified (`private`), all the base class members are inherited as `private`.

For example, if daughter were a class derived from mother that we defined as:

```
class Daughter: protected Mother;
```

This would set `protected` as the less restrictive access level for the members of `Daughter` that it inherited from mother. That is, all members that were `public` in `Mother` would become `protected` in `Daughter`. Of course, this would not restrict `Daughter` from declaring its own public members. That _less restrictive access level_ is only set for the members inherited from `Mother`.

If no access level is specified for the inheritance, the compiler assumes private for classes declared with keyword `class` and public for those declared with `struct`.（如果继承时未指定访问级别，则编译器假定使用 private 声明 class，而使用 public 声明 struct ）

Actually, most use cases of inheritance in C++ should use public inheritance. When other access levels are needed for base classes, they can usually be better represented as member variables instead.（通常 Cpp 中继承的大多数用例都使用 public 继承，当基类需要其它访问级别时，通常可以更好地将它们表示为成员变量）

## What is inherited from the base class?

In principle, a publicly derived class inherits access to every member of a base class except:

*   its constructors and its destructor
*   its assignment operator members (operator=)
*   its friends
*   its private members
（public 派生类继承基类大部分成员的访问权限，除了基类的构造与析构函数、赋值运算符成员、友元、私有成员）

Even though access to the constructors and destructor of the base class is not inherited as such, they are automatically called by the constructors and destructor of the derived class. Unless otherwise specified, the constructors of a derived class calls the default constructor of its base classes (i.e., the constructor taking no arguments). （即使派生类对基类的构造和析构函数不会直接继承，但也会在派生类的构造函数中自动调用基类的默认构造函数（不带参数的构造函数））

Calling a different constructor of a base class is possible, using the same syntax used to initialize member variables in the initialization list: （可以使用初始化列表中用于初始化成员变量的相同语法，调用基类的不同构造函数，如下：）

`derived_constructor_name (parameters) : base_constructor_name (parameters) {...}`

For example:

```cpp
// constructors and derived classes
#include <iostream> 
using namespace std;

class Mother {
  public:
    Mother ()
      { cout << "Mother: no parameters\n"; }
    Mother (int a)
      { cout << "Mother: int parameter\n"; }
};

class Daughter : public Mother {
  public:
    Daughter (int a)
      { cout << "Daughter: int parameter\n\n"; }
};

class Son : public Mother {
  public:
    Son (int a) : Mother (a)
      { cout << "Son: int parameter\n\n"; }
};

int main () {
  Daughter kelly (0);
  Son bud (0);
  
  return 0;
}
```

![[derived_class_constructor.png]]

Notice the difference between which `Mother`'s constructor is called when a new `Daughter` object is created and which when it is a `Son` object. The difference is due to the different constructor declarations of `Daughter` and `Son`:

```
Daughter (int a)          // nothing specified: call default constructor
Son (int a) : Mother (a)  // constructor specified: call this specific constructor
```

## Multiple inheritance

A class may inherit from more than one class by simply specifying more base classes, separated by commas, in the list of a class's base classes (i.e., after the colon). For example, if the program had a specific class to print on screen called `Output`, and we wanted our classes `Rectangle` and `Triangle` to also inherit its members in addition to those of `Polygon` we could write:

```
class Rectangle: public Polygon, public Output;
class Triangle: public Polygon, public Output;
```

Here is the complete example:

```cpp
// multiple inheritance
#include <iostream> 
using namespace std;

class Polygon {
  protected:
    int width, height;
  public:
    Polygon (int a, int b) : width (a), height (b) {}
};

class Output {
  public:
    static void print (int i);
};

void Output:: print (int i) {
  cout << i << '\n';
}

class Rectangle: public Polygon, public Output {
  public:
    Rectangle (int a, int b) : Polygon (a, b) {}
    int area ()
      { return width*height; }
};

class Triangle: public Polygon, public Output {
  public:
    Triangle (int a, int b) : Polygon (a, b) {}
    int area ()
      { return width*height/2; }
};
  
int main () {
  Rectangle rect (4,5);
  Triangle trgl (4,5);
  rect.print (rect.area ());
  Triangle:: print (trgl.area ());
  return 0;
}
```