## Weak Pointer

A `weak_ptr` is a type of smart pointer in C++ that adds a level of indirection and safety to a raw pointer. It is mainly used to break reference cycles in cases where two objects have shared pointers to each other, or when you need a non-owning reference to an object that is managed by a `shared_ptr`.

A `weak_ptr` doesn’t increase the reference count of the object it points to, which is a crucial distinction between `weak_ptr` and `shared_ptr`. This ensures that the object will be deleted once the last `shared_ptr` that owns it goes out of scope, even if there are `weak_ptr`s still referencing it.

To use a `weak_ptr`, you must convert it to a `shared_ptr` using the `lock()` function, which tries to create a new `shared_ptr` that shares ownership of the object. If successful, the object’s reference count is increased and you can use the returned `shared_ptr` to safely access the object.

Here’s an example of using `weak_ptr`:

```
#include <iostream>
#include <memory>

class MyClass {
public:
    void DoSomething() {
        std::cout << "Doing something...\n";
    }
};

int main() {
    std::weak_ptr<MyClass> weak;

    {
        std::shared_ptr<MyClass> shared = std::make_shared<MyClass>();
        weak = shared;

        if(auto sharedFromWeak = weak.lock()) {
            sharedFromWeak->DoSomething(); // Safely use the object
            std::cout << "Shared uses count: " << sharedFromWeak.use_count() << '\n'; // 2
        }
    }

    // shared goes out of scope and the MyClass object is destroyed

    if(auto sharedFromWeak = weak.lock()) {
        // This block will not be executed because the object is destroyed
    }
    else {
        std::cout << "Object has been destroyed\n";
    }

    return 0;
}
```

In this example, we create a `shared_ptr` named `shared` that manages a `MyClass` object. By assigning it to a `weak_ptr` named `weak`, we store a non-owning reference to the object. Inside the inner scope, we create a new `shared_ptr` named `sharedFromWeak` using `weak.lock()` to safely use the object. After the inner scope, the `MyClass` object is destroyed since `shared` goes out of scope, and any further attempt to create a `shared_ptr` from `weak` will fail as the object is already destroyed.

- [CPP Reference](https://en.cppreference.com/w/cpp/memory/weak_ptr)

## Shared Pointer

A `shared_ptr` is a type of smart pointer in C++ that allows multiple pointers to share ownership of a dynamically allocated object. The object will be automatically deallocated only when the last `shared_ptr` that points to it is destroyed.

When using a `shared_ptr`, the reference counter is automatically incremented every time a new pointer is created, and decremented when each pointer goes out of scope. Once the reference counter reaches zero, the system will clean up the memory.

### Code Example

Here’s an example of how to use `shared_ptr`:

```
#include <iostream>
#include <memory>

class MyClass {
public:
    MyClass() { std::cout << "Constructor is called." << std::endl; }
    ~MyClass() { std::cout << "Destructor is called." << std::endl; }
};

int main() {
    // create a shared pointer to manage the MyClass object
    std::shared_ptr<MyClass> ptr1(new MyClass());
    
    {
        // create another shared pointer and initialize it with the previously created pointer
        std::shared_ptr<MyClass> ptr2 = ptr1;

        std::cout << "Inside the inner scope." << std::endl;
        // both pointers share the same object, and the reference counter has been increased to 2
    }

    std::cout << "Outside the inner scope." << std::endl;
    // leaving the inner scope will destroy ptr2, and the reference counter is decremented to 1
    
    // the main function returns, ptr1 goes out of scope, and the reference counter becomes 0
    // this causes the MyClass object to be deleted and the destructor is called
}
```

Output:

```
Constructor is called.
Inside the inner scope.
Outside the inner scope.
Destructor is called.
```

In this example, `ptr1` and `ptr2` share ownership of the same object. The object is only destroyed when both pointers go out of scope and the reference counter becomes zero.

## Unique Pointer (`unique_ptr`)

`std::unique_ptr` is a smart pointer provided by the C++ Standard Library. It is a template class that is used for managing single objects or arrays.

`unique_ptr` works on the concept of _exclusive ownership_ - meaning only one `unique_ptr` is allowed to own an object at a time. This ownership can be transferred or moved, but it cannot be shared or copied.

This concept helps to prevent issues like dangling pointers, reduce memory leaks, and eliminates the need for manual memory management. When the `unique_ptr` goes out of scope, it automatically deletes the object it owns.

Let’s take a look at some basic examples of using `unique_ptr`:

### Creating a unique_ptr

```
#include <iostream>
#include <memory>

int main() {
    std::unique_ptr<int> p1(new int(5)); // Initialize with pointer to a new integer
    std::unique_ptr<int> p2 = std::make_unique<int>(10); // Preferred method (C++14 onwards)

    std::cout << *p1 << ", " << *p2 << std::endl;
    return 0;
}
```

### Transferring Ownership

```
#include <iostream>
#include <memory>

int main() {
    std::unique_ptr<int> p1(new int(5));

    std::unique_ptr<int> p2 = std::move(p1); // Ownership is transferred from p1 to p2

    if (p1) {
        std::cout << "p1 owns the object" << std::endl;
    } else if (p2) {
        std::cout << "p2 owns the object" << std::endl;
    }

    return 0;
}
```

### Using unique_ptr with Custom Deleters

```
#include <iostream>
#include <memory>

struct MyDeleter {
    void operator()(int* ptr) {
        std::cout << "Custom Deleter: Deleting pointer" << std::endl;
        delete ptr;
    }
};

int main() {
    std::unique_ptr<int, MyDeleter> p1(new int(5), MyDeleter());
    return 0; // Custom Deleter will be called when p1 goes out of scope
}
```

Remember that since unique_ptr has exclusive ownership, you cannot use it when you need shared access to an object. For such cases, you can use `std::shared_ptr`.