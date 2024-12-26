An array is a series of elements of the same type placed in contiguous memory locations that can be individually referenced by adding an index to a unique identifier. (数组要求元素是相同类型)

That means that, for example, five values of type `int` can be declared as an array without having to declare 5 different variables (each with its own identifier). Instead, using an array, the five `int` values are stored in contiguous（连续的） memory locations, and all five can be accessed using the same identifier, with the proper index.（用不同的索引，通过相同的标识符，访问不同的数组元素）

For example, an array containing 5 integer values of type `int` called `foo` could be represented as:

![[array-blank.png]]

where each blank panel represents an element of the array. In this case, these are values of type `int`. These elements are numbered from 0 to 4, being 0 the first and 4 the last; In C++, the first element in an array is always numbered with a zero (not a one), no matter its length.

Like a regular variable, an array must be declared before it is used. A typical declaration for an array in C++ is:

`type name [elements];`

where `type` is a valid type (such as `int`, `float`...), `name` is a valid identifier and the `elements` field (which is always enclosed in square brackets `[]`), specifies the length of the array in terms of the number of elements.

Therefore, the `foo` array, with five elements of type `int`, can be declared as:

```cpp
int foo [5];
```

NOTE: The `elements` field within square brackets `[]`, representing the number of elements in the array, must be a _constant expression_, since arrays are blocks of static memory whose size must be determined at compile time, before the program runs.  (数组是一种静态存储块，因此 `[]` 内代表数组元素数量的内容，必须是一个常量表达式，在编译时大小必须确定)

## Initializing arrays

By default, regular arrays of _local scope_ (for example, those declared within a function) are left uninitialized. This means that none of its elements are set to any particular value; their contents are undetermined at the point the array is declared.（本地作用域的常规数组，会保持未初始化的状态，在声明时内容不能确定）

But the elements in an array can be explicitly initialized to specific values when it is declared, by enclosing those initial values in braces `{}`. For example:

```cpp
int foo [5] = { 16, 2, 77, 40, 12071 };
```

This statement declares an array that can be represented like this:

![[array-with-int.png]]

The number of values between braces `{}` shall not be greater than the number of elements in the array. For example, in the example above, `foo` was declared having 5 elements (as specified by the number enclosed in square brackets, `[]`), and the braces `{}` contained exactly 5 values, one for each element. If declared with less, the remaining elements are set to their default values (which for fundamental types, means they are filled with zeroes). （显式初始化的数据数量不能超过声明时数组的元素数量，当少于声明时数量时，会将剩余元素设定为对应类型的默认值，如 int 对应 0，char 对应 空，float 对应 0，bool 对应false）

For example:

```
int bar [5] = { 10, 20, 30 };
```

Will create an array like this:

![[array-init-less.png]]

The initializer can even have no values, just the braces:

```cpp
int bar [5] = {};
```

This creates an array of five `int` values, each initialized with a value of zero:

![[array-init-empty.png]]

When an initialization of values is provided for an array, C++ allows the possibility of leaving the square brackets empty `[]`. In this case, the compiler will assume automatically a size for the array that matches the number of values included between the braces `{}`:

```
int foo [] = { 16, 2, 77, 40, 12071 };
```

After this declaration, array `foo` would be 5 `int` long, since we have provided 5 initialization values. (如果不声明数组内元素的数量 `[]`，那么编译器会根据 `{}` 内初始化的数量进行自动匹配数组大小)

Finally, the evolution of C++ has led to the adoption of _universal initialization_ also for arrays. Therefore, there is no longer need for the equal sign between the declaration and the initializer. （Cpp 通用初始化可以使数组声明和初始化之间不必使用等号，例子如下）Both these statements are equivalent:

```cpp
int foo[] = { 10, 20, 30 };
int foo[] { 10, 20, 30 };
```

Static arrays, and those declared directly in a namespace (outside any function), are always initialized. If no explicit initializer is specified, all the elements are default-initialized (with zeroes, for fundamental types). （静态数组及直接在命名空间声明的数组始终处于初始化状态，未指定值时，对于基本类型使用 0 默认初始化所有元素） 

## Accessing the values of an array

The values of any of the elements in an array can be accessed just like the value of a regular variable of the same type. The syntax is:

`name[index]`

Following the previous examples in which `foo` had 5 elements and each of those elements was of type `int`, the name which can be used to refer to each element is the following:

![[array-index.png]]

For example, the following statement stores the value 75 in the third element of `foo`:

```cpp
foo [2] = 75;
```

and, for example, the following copies the value of the third element of `foo` to a variable called `x`:  

```cpp
x = foo [2];
```

Therefore, the expression `foo[2]` is itself a variable of type `int`.

Notice that the third element of `foo` is specified `foo[2]`, since the first one is `foo[0]`, the second one is `foo[1]`, and therefore, the third one is `foo[2]`. By this same reason, its last element is `foo[4]`. Therefore, if we write `foo[5]`, we would be accessing the sixth element of `foo`, and therefore actually exceeding the size of the array. (数组越界异常)

In C++, it is syntactically correct（语法上正确） to exceed the valid range of indices for an array. This can create problems, since accessing out-of-range elements do not cause errors on compilation, but can cause errors on runtime. The reason for this being allowed will be seen in a later chapter when pointers are introduced.（数组访问越界不会导致编译错误，但会导致运行时错误。这样做的原因在后文指针中进行讲解）

At this point, it is important to be able to clearly distinguish between the two uses that brackets `[]` have related to arrays. They perform two different tasks: one is to specify the size of arrays when they are declared; and the second one is to specify indices for concrete array elements when they are accessed. Do not confuse these two possible uses of brackets `[]` with arrays.

```cpp
int foo[5];         // declaration of a new array
foo[2] = 75;        // access to an element of the array.
```

The main difference is that the declaration is preceded by the type of the elements, while the access is not.

Some other valid operations with arrays:

```cpp
foo[0] = a;
foo[a] = 75;
b = foo [a+2];
foo[foo[a]] = foo[2] + 5;
```

For example:

```cpp
// arrays example
#include <iostream>
using namespace std;

int foo [] = {16, 2, 77, 40, 12071};
int n, result=0;

int main ()
{
  for ( n=0 ; n< 5 ; ++n )
  {
    result += foo[n];
  }
  cout << result;
  return 0;
}
```

## Multidimensional arrays

Multidimensional arrays can be described as "arrays of arrays". For example, a bidimensional array can be imagined as a two-dimensional table made of elements, all of them of a same uniform data type.

![[bidimensional-array.png]]

`jimmy` represents a bidimensional array of 3 per 5 elements of type `int`. The C++ syntax for this is:

```cpp
int jimmy [3][5];
```

and, for example, the way to reference the second element vertically and fourth horizontally in an expression would be:  

```cpp
jimmy[1][3]
```

![[bidimensional-accsess-element.png]]

(remember that array indices always begin with zero).

Multidimensional arrays are not limited to two indices (i.e., two dimensions). They can contain as many indices as needed. Although be careful: the amount of memory needed for an array increases exponentially with each dimension. 

>[! warning] Multi-dimentional Array Memory
>⚠：数组所需的内存量随维度数量呈指数级增长！

For example:

```
char century [100][365][24][60][60]; // 100年，365天，24小时，60分，60秒
```

declares an array with an element of type `char` for each second in a century. This amounts to more than 3 billion `char`! So this declaration would consume more than 3 gigabytes of memory!

At the end, multidimensional arrays are just an abstraction for programmers, since the same results can be achieved with a simple array, by multiplying its indices:（直接调用多维数组只是对实际问题的浅层抽象，可以通过优化抽象的方式降维，以获得更低的内存占用。对于简单的数组，可以通过乘其索引来获得相同的结果）

```
int jimmy [3][5];   // is equivalent to
int jimmy [15];     // (3 * 5 = 15)
```

With the only difference that with multidimensional arrays, the compiler automatically remembers the depth of each imaginary dimension.（多维数组和降维后数组的唯一区别，在于编译器会自动记住每个虚构维度的深度）

The following two pieces of code produce the exact same result, but one uses a bidimensional array while the other uses a simple array:

![[multidimentional-vs-imaginary-dimension.png]]

  
None of the two code snippets above produce any output on the screen, but both assign values to the memory block called jimmy in the following way:

![[jimmy.png]]

Note that the code uses defined constants for the width and height, instead of using directly their numerical values. This gives the code a **better readability**, and allows changes in the code to be made easily in one place.

## Arrays as parameters

At some point, we may need to pass an array to a function as a parameter. In C++, it is not possible to pass the entire block of memory represented by an array to a function directly as an argument. But what can be passed instead is its address. In practice, this has almost the same effect, and it is a much faster and more efficient operation.（数组表示的整个内存块不会作为参数直接传递给函数，可以传递的是它的地址）

To accept an array as parameter for a function, the parameters can be declared as the array type, but with empty brackets, omitting the actual size of the array.（接收数组作为函数参数，需要将参数声明为数组类型，且不必声明实际大小） For example:

```
void procedure (int arg[])
```

This function accepts a parameter of type "array of `int`" called `arg`. In order to pass to this function an array declared as:  

```cpp
int myarray [40];
```

it would be enough to write a call like this:  

```cpp
procedure(myarray);
```

Here you have a complete example:

```cpp
// arrays as parameters
#include <iostream> 
using namespace std;

void printarray (int arg[], int length) {
  for (int n=0; n< length; ++n)
    cout << arg[n] << ' ';
  cout << '\n';
}

int main ()
{
  int firstarray[] = {5, 10, 15};
  int secondarray[] = {2, 4, 6, 8, 10};
  printarray (firstarray, 3);
  printarray (secondarray, 5);
}
```

In the code above, the first parameter (`int arg[]`) accepts any array whose elements are of type `int`, whatever its length. For that reason, we have included a second parameter that tells the function the length of each array that we pass to it as its first parameter. This allows the for loop that prints out the array to know the range to iterate in the array passed, without going out of range.

In a function declaration, it is also possible to include multidimensional arrays. The format for a tridimensional array parameter is:

```
base_type[][depth][depth]
```

For example, a function with a multidimensional array as argument could be:

```
void procedure (int myarray[][3][4])
```

Notice that the first brackets `[]` are left empty, while the following ones specify sizes for their respective dimensions. This is necessary in order for the compiler to be able to determine the depth of each additional dimension. (对于多维数组作为参数，第一个 `[]` 要留空，剩下的括号要指定各自尺寸的大小，这样编译器能够确定每个附加维度的深度)

In a way, passing an array as argument always loses a dimension. The reason behind is that, for historical reasons, arrays cannot be directly copied, and thus what is really passed is a pointer. （数组不能直接复制，真正传递的是一个指针，因此数组作为参数传递时总会丢失一个维度）This is a common source of errors for novice programmers. Although a clear understanding of pointers, explained in a coming chapter, helps a lot.

## Library arrays

The arrays explained above are directly implemented as a language feature, inherited from the C language. They are a great feature, but by restricting its copy and easily decay into pointers, they probably suffer from an excess of optimization.（前文的数组直接实现为语言功能，这继承自 C 语言，但是在使用时常常需要限制副本而衰减为使用指针，这可能会遭受过度优化的影响）

To overcome some of these issues with language built-in arrays, C++ provides an alternative array type as a standard container. It is a type template (a class template, in fact) defined in header [<array\>](https://cplusplus.com/%3Carray%3E).(Cpp 使用另一种数组类型作为标准容器，在头文件 array 中定义的模板类)

Containers are a library feature that falls out of the scope of this tutorial, and thus the class will not be explained in detail here. Suffice it to say that they operate in a similar way to built-in arrays, except that they allow being copied (an actually expensive operation that copies the entire block of memory, and thus to use with care) and decay into pointers only when explicitly told to do so (by means of its member `data`).（容器是一个库功能，以类似于内置数组的方式运行，允许被复制，并且只在明确告知时才衰减为指针，由于复制操作开销极大，可以复制整个内存块，需要小心使用）

Just as an example, these are two versions of the same example using the language built-in array described in this chapter, and the container in the library:

![[container-array.png]]

![[array-container.png]]

As you can see, both kinds of arrays use the same syntax to access its elements: `myarray[i]`. Other than that, the main differences lay on the declaration of the array, and the inclusion of an additional header for the _library array_. Notice also how it is easy to access the size of the _library array_.（主要区别在于引入 array 库文件、数组声明，并且访问库数组大小非常容易，仅需要 `.size()`）