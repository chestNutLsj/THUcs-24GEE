In the programs seen in previous chapters, all memory needs were determined before program execution by defining the variables needed. But there may be cases where the memory needs of a program can only be determined during runtime. For example, when the memory needed depends on user input. On these cases, programs need to dynamically allocate memory, for which the C++ language integrates the operators `new` and `delete`.  （为了应对某些运行时才能确定的内存需求，Cpp 继承了 new 和 delete 运算符）

## Operators `new` and `new[]`

Dynamic memory is allocated using operator `new`. `new` is followed by a data type specifier and, if a sequence of more than one element is required, the number of these within brackets `[]`. It returns a pointer to the beginning of the new block of memory allocated.（动态内存使用 `new` 分配，后接类型说明符，如有多个元素则用 `[nums]` 声明数量。`new` 返回的是指向分配的新内存块的开头的指针）

Its syntax is:

`pointer = new type`

`pointer = new type [number_of_elements]`  

The first expression is used to allocate memory to contain one single element of type `type`. The second one is used to allocate a block (an array) of elements of type `type`, where `number_of_elements` is an integer value representing the amount of these. For example:

```
int * foo;
foo = new int [5];
```

In this case, the system dynamically allocates space for five elements of type `int` and returns a pointer to the first element of the sequence, which is assigned to `foo` (a pointer). Therefore, `foo` now points to a valid block of memory with space for five elements of type `int`.(系统为 int 类型的五个元素动态分配空间，并返回指向序列的第一个元素的指针。foo 指针指向一个有效的内存块)

![[dynamic-pointer.png]]

Here, `foo` is a pointer, and thus, the first element pointed to by `foo` can be accessed either with the expression `foo[0]` or the expression `*foo` (both are equivalent). The second element can be accessed either with `foo[1]` or `*(foo+1)`, and so on...

There is a substantial difference between declaring a normal array and allocating dynamic memory for a block of memory using `new`. The most important difference is that the size of a regular array needs to be a _constant expression_, and thus its size has to be determined at the moment of designing the program, before it is run, whereas the dynamic memory allocation performed by `new` allows to assign memory during runtime using any variable value as size.

The dynamic memory requested by our program is allocated by the system from the memory heap. However, computer memory is a limited resource, and it can be exhausted. Therefore, there are no guarantees that all requests to allocate memory using operator `new` are going to be granted by the system.（程序请求的动态内存分配来自于系统的内存堆，但计算机内存是有限的资源，可能会耗尽，因此不能保证系统允许所有的分配请求）

C++ provides two standard mechanisms to check if the allocation was successful:

### allocation failed: `exceptions`
One is by handling exceptions. Using this method, an exception of type `bad_alloc` is thrown when the allocation fails. (分配空间失败时，将抛出 `bad_alloc` 异常)Exceptions are a powerful C++ feature explained later in these tutorials. But for now, you should know that if this exception is thrown and it is not handled by a specific handler, the program execution is terminated.

This exception method is the method used by default by `new`, and is the one used in a declaration like:

```
foo = new int [5];  // if allocation fails, an exception is thrown
```

### allocation failed:  `nothrow`
The other method is known as `nothrow`, and what happens when it is used is that when a memory allocation fails, instead of throwing a `bad_alloc` exception or terminating the program, the pointer returned by `new` is a _null pointer_, and the program continues its execution normally.（分配失败时，返回 nullptr，而不是抛出异常或终止，因此通过检测返回的指针是否为 nullptr 可以判断是否分配成功）

This method can be specified by using a special object called `nothrow`, declared in header [\<new\>](https://cplusplus.com/%3Cnew%3E), as argument for `new`:

```
foo = new (nothrow) int [5];
```

In this case, if the allocation of this block of memory fails, the failure can be detected by checking if `foo` is a null pointer:

```cpp
int * foo;
foo = new (nothrow) int [5];
if (foo == nullptr) {
  // error assigning memory. Take measures.
}
```

This `nothrow` method is likely to produce less efficient code than exceptions, since it implies explicitly checking the pointer value returned after each and every allocation. Therefore, the exception mechanism is generally preferred, at least for critical allocations. Still, most of the coming examples will use the ` nothrow ` mechanism due to its simplicity.  （`nothrow` 机制使用起来简单，但运行时比异常更低效，因为它会显式检查每次分配后返回的检查值，因此异常机制是首选，尤其对关键的内存分配而言）

## Operators `delete` and `delete[]`

In most cases, memory allocated dynamically is only needed during specific periods of time within a program; once it is no longer needed, it can be freed so that the memory becomes available again for other requests of dynamic memory.（申请的动态内存常常在程序运行的特定时间段需要，一旦不再需要可以释放它，以便降低内存消耗，满足其他动态内存的申请）

This is the purpose of operator `delete`, whose syntax is:

```
delete pointer;
delete[] pointer;
```

The first statement releases the memory of a single element allocated using `new`, and the second one releases the memory allocated for arrays of elements using new and a size in brackets (`[]`).

The value passed as argument to `delete` shall be either a pointer to a memory block previously allocated with `new`, or a _null pointer_ (in the case of a _null pointer_, `delete` produces no effect).

```cpp
// rememb-o-matic
#include <iostream> 
#include <new> 
using namespace std;

int main ()
{
  int i, n;
  int * p;
  cout << "How many numbers would you like to type? ";
  cin >> i;
  p= new (nothrow) int[i];
  if (p == nullptr)
    cout << "Error: memory could not be allocated";
  else
  {
    for (n=0; n< i; n++)
    {
      cout << "Enter number: ";
      cin >> p[n];
    }
    cout << "You have entered: ";
    for (n=0; n< i; n++)
      cout << p[n] << ", ";
    delete[] p;
  }
  return 0;
}
```

Notice how the value within brackets in the new statement is a variable value entered by the user (`i`), not a constant expression:

```
p= new (nothrow) int[i];
```

There always exists the possibility that the user introduces a value for `i` so big that the system cannot allocate enough memory for it. For example, when I tried to give a value of 1 billion to the "How many numbers" question, my system could not allocate that much memory for the program, and I got the text message we prepared for this case (`Error: memory could not be allocated`).

It is considered good practice for programs to always be able to handle failures to allocate memory, either by checking the pointer value (if `nothrow`) or by catching the proper exception.

## Dynamic memory in C

C++ integrates the operators `new` and `delete` for allocating dynamic memory. But these were not available in the C language; instead, it used a library solution, with the functions [malloc](https://cplusplus.com/malloc), [calloc](https://cplusplus.com/calloc), [realloc](https://cplusplus.com/realloc) and [free](https://cplusplus.com/free), defined in the header [\<cstdlib\>](https://cplusplus.com/%3Ccstdlib%3E) (known as `<stdlib.h>` in C). The functions are also available in C++ and can also be used to allocate and deallocate dynamic memory.

Note, though, that the memory blocks allocated by these functions are not necessarily compatible with those returned by `new`, so they should not be mixed; each one should be handled with its own set of functions or operators. (C 中的 malloc 等内存分配与 Cpp 中的 new 等机制不一定兼容，因此不建议混合使用)

## Memory Leakage
Memory leakage occurs when a program allocates memory in the heap but does not release the memory back to the operating system when it is no longer needed. Over time, this leads to exhaustion of available memory, resulting in low system performance or crashes.

In C++, when you use raw pointers, you need to manage the memory allocation and deallocation manually. In many cases, you will use the `new` keyword to allocate memory for an object in the heap and use `delete` keyword to deallocate that memory when it’s no longer needed. Forgetting to do this can cause memory leaks.

Here’s an example:

```
void create_memory_leak() {
    int* ptr = new int[100]; // Allocating memory in the heap for an array of integers
    // Some code...
    // Code to deallocate the memory is missing: delete[] ptr;
} // ptr goes out of scope, memory block allocated is not deallocated, causing a memory leak.
```

To avoid memory leaks, you should always ensure that memory is deallocated before a pointer goes out of scope or is reassigned. Some ways to achieve this include using the C++ smart pointers (`std::unique_ptr`, `std::shared_ptr`), RAII (Resource Acquisition Is Initialization) techniques, and containers from the C++ standard library that manage memory allocation internally (e.g., `std::vector`, `std::string`).

For example, this code will not have a memory leak:

```
#include <memory>

void no_memory_leak() {
    std::shared_ptr<int> ptr = std::make_shared<int[]>(100); // Allocating memory in the heap for an array of integers using shared_ptr
    // Some code...
} // shared_ptr goes out of scope and it will automatically deallocate the memory block assigned to it.
```