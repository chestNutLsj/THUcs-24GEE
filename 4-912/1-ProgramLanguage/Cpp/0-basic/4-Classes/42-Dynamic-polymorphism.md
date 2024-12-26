Dynamic polymorphism is a programming concept in object-oriented languages like C++ where a derived class can override or redefine methods of its base class. This means that a single method call can have different implementations based on the type of object it is called on.

Dynamic polymorphism is achieved through **virtual functions**, which are member functions of a base class marked with the `virtual` keyword. When you specify a virtual function in a base class, it can be overridden in any derived class to provide a different implementation.

## Example

Here’s an example in C++ demonstrating dynamic polymorphism.

```
#include <iostream>

// Base class
class Shape {
public:
    virtual void draw() {
        std::cout << "Drawing a shape" << std::endl; 
    }
};

// Derived class 1
class Circle : public Shape {
public:
    void draw() override {
        std::cout << "Drawing a circle" << std::endl; 
    }
};

// Derived class 2
class Rectangle : public Shape {
public:
    void draw() override {
        std::cout << "Drawing a rectangle" << std::endl;
    }
};

int main() {
    Shape* shape;
    Circle circle;
    Rectangle rectangle;

    // Storing the address of circle
    shape = &circle;

    // Call circle draw function
    shape->draw();

    // Storing the address of rectangle
    shape = &rectangle;

    // Call rectangle draw function
    shape->draw();

    return 0;
}
```

This code defines a base class `Shape` with a virtual function `draw`. Two derived classes `Circle` and `Rectangle` both override the `draw` function to provide their own implementations. Then in the `main` function, a pointer of type `Shape` is used to call the respective `draw` functions of `Circle` and `Rectangle` objects. The output of this program will be:

```
Drawing a circle
Drawing a rectangle
```

As you can see, using dynamic polymorphism, we can determine at runtime which `draw` method should be called based on the type of object being used.

## Virtual Methods

Virtual methods are a key aspect of dynamic polymorphism in C++. They allow subclass methods to override the methods of their base class, so the appropriate method is called depending on the actual type of an object at runtime.

To declare a method as virtual, simply use the `virtual` keyword in the method’s declaration in the base class. This tells the compiler that the method should be treated as a virtual method, allowing it to be overridden by derived classes.

## Code Example

Here’s an example demonstrating virtual methods:

```
#include <iostream>

// Base class
class Shape {
public:
    virtual double area() const {
        return 0;
    }
};

// Derived class
class Circle : public Shape {
public:
    Circle(double r) : radius(r) {}

    // Override the base class method
    double area() const override {
        return 3.14 * radius * radius;
    }

private:
    double radius;
};

// Derived class
class Rectangle : public Shape {
public:
    Rectangle(double w, double h) : width(w), height(h) {}

    // Override the base class method
    double area() const override {
        return width * height;
    }

private:
    double width;
    double height;
};

int main() {
    Circle c(5);
    Rectangle r(4, 6);

    Shape* shape = &c;
    std::cout << "Circle's area: " << shape->area() << std::endl;

    shape = &r;
    std::cout << "Rectangle's area: " << shape->area() << std::endl;

    return 0;
}
```

In this example, we define a base class `Shape` that has a virtual method `area`. This method is then overridden by the derived classes `Circle` and `Rectangle`. By using a virtual method and a base class pointer to the derived objects, we can invoke the appropriate `area` method based on the actual object type at runtime.

## Virtual Tables

Virtual Tables (or Vtable) are a mechanism used by C++ compilers to support dynamic polymorphism. In dynamic polymorphism, the appropriate function is called at runtime, depending on the actual object type.

When a class contains a virtual function, the compiler creates a virtual table for that class. This table contains function pointers to the virtual functions defined in the class. Each object of that class has a pointer to its virtual table (_vptr_, virtual pointer), which is automatically initialized by the compiler during object construction.

### Example

Let’s consider the following example:

```
class Base {
public:
    virtual void function1() {
        std::cout << "Base::function1" << std::endl;
    }

    virtual void function2() {
        std::cout << "Base::function2" << std::endl;
    }
};

class Derived : public Base {
public:
    void function1() override {
        std::cout << "Derived::function1" << std::endl;
    }

    void function3() {
        std::cout << "Derived::function3" << std::endl;
    }
};

int main() {
    Base* obj = new Derived(); // create a Derived object and assign a pointer of type Base*
    obj->function1(); // calls Derived::function1, due to dynamic polymorphism
    obj->function2(); // calls Base::function2

    delete obj;
    return 0;
}
```

In this example, when a `Derived` object is created, the compiler generates a Vtable for `Derived` class, containing pointers to its virtual functions:

- `Derived::function1` (overridden from `Base`)
- `Base::function2` (inherits from Base)

The `_vptr_` pointer in the `Derived` object points to this Vtable. When the `function1` is called on the `Base` pointer pointing to the `Derived` object, the function pointer in the Vtable is used to call the correct function (in this case, `Derived::function1`). Similarly, the call to `function2` calls `Base::function2`, since it’s the function pointer stored in the Vtable for `Derived` class.

Note that `function3` is not part of the Vtable, as it is not a virtual function.