Multiple inheritance is a feature in C++ where a class can inherit characteristics (data members and member functions) from more than one parent class. The concept is similar to single inheritance (where a class inherits from a single base class), but in multiple inheritance, a class can have multiple base classes.

When a class inherits multiple base classes, it becomes a mixture of their properties and behaviors, and can override or extend them as needed.

## Syntax

Here is the syntax to declare a class with multiple inheritance:

```
class DerivedClass : access-specifier BaseClass1, access-specifier BaseClass2, ...
{
    // class body
};
```

The `DerivedClass` will inherit members from both `BaseClass1` and `BaseClass2`. The `access-specifier` (like `public`, `protected`, or `private`) determines the accessibility of the inherited members.

## Example

Here is an example of multiple inheritance in action:

```
#include <iostream>

// Base class 1
class Animal
{
public:
    void eat()
    {
        std::cout << "I can eat!" << std::endl;
    }
};

// Base class 2
class Mammal
{
public:
    void breath()
    {
        std::cout << "I can breathe!" << std::endl;
    }
};

// Derived class inheriting from both Animal and Mammal
class Dog : public Animal, public Mammal
{
public:
    void bark()
    {
        std::cout << "I can bark! Woof woof!" << std::endl;
    }
};

int main()
{
    Dog myDog;

    // Calling members from both base classes
    myDog.eat();
    myDog.breath();
    
    // Calling a member from the derived class
    myDog.bark();

    return 0;
}
```

## Note

In some cases, multiple inheritance can lead to complications such as ambiguity and the “diamond problem”. Ensure that you use multiple inheritance judiciously and maintain well-structured and modular classes to prevent issues.

For more information on C++ multiple inheritance and related topics, refer to C++ documentation or a comprehensive C++ programming guide.

## Diamond Inheritance

Diamond inheritance is a specific scenario in multiple inheritance where a class is derived from two or more classes, which in turn, are derived from a common base class. It creates an ambiguity that arises from duplicating the common base class, which leads to an ambiguous behavior while calling the duplicate members.

To resolve this ambiguity, you can use virtual inheritance. A virtual base class is a class that is shared by multiple classes using `virtual` keyword in C++. This ensures that only one copy of the base class is inherited in the final derived class, and thus, resolves the diamond inheritance problem.

_Example:_

```
#include<iostream>
using namespace std;

class Base {
public:
    void print() {
        cout << "Base class" << endl;
    }
};

class Derived1 : virtual public Base {
public:
    void derived1Print() {
        cout << "Derived1 class" << endl;
    }
};

class Derived2 : virtual public Base {
public:
    void derived2Print() {
        cout << "Derived2 class" << endl;
    }
};

class Derived3 : public Derived1, public Derived2 {
public:
    void derived3Print() {
        cout << "Derived3 class" << endl;
    }
};

int main()
{
    Derived3 d3;
    d3.print(); // Now, there is no ambiguity in calling the base class function
    d3.derived1Print();
    d3.derived2Print();
    d3.derived3Print();

    return 0;
}
```

In the code above, `Derived1` and `Derived2` are derived from the `Base` class using virtual inheritance. So, when we create an object of `Derived3` and call the `print()` function from the `Base` class, there is no ambiguity, and the code executes without any issues.