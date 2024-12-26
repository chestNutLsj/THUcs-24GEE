In earlier chapters, variables have been explained as locations in the computer's memory which can be accessed by their identifier (their name). This way, the program does not need to care about the physical address of the data in memory; it simply uses the identifier whenever it needs to refer to the variable.（前文中变量被解释成计算机主存的位置，可以通过标识符访问，程序不必关心主存中数据的物理地址，只要在需要时引用变量标识符即可）

For a C++ program, the memory of a computer is like a succession of memory cells, each one byte in size, and each with a unique address. These single-byte memory cells are ordered in a way that allows data representations larger than one byte to occupy memory cells that have consecutive addresses.（这些单字节主存单元的排序方式允许大于一个字节的数据表示占据具有连续地址的主存单元）

This way, each cell can be easily located in the memory by means of its unique address. For example, the memory cell with the address `1776` always follows immediately after the cell with address `1775` and precedes the one with `1777`, and is exactly one thousand cells after `776` and exactly one thousand cells before `2776`.

When a variable is declared, the memory needed to store its value is assigned a specific location in memory (its memory address). Generally, C++ programs do not actively decide the exact memory addresses where its variables are stored. Fortunately, that task is left to the environment where the program is run - generally, an operating system that decides the particular memory locations on runtime.（声明变量时，需要存储其值的主存空间被分配到主存整体的特定位置，这一过程并不由 Cpp 程序确定，而是 Cpp 程序运行的环境——操作系统在程序运行时确定）

However, it may be useful for a program to be able to obtain the address of a variable during runtime in order to access data cells that are at a certain position relative to it.（程序能够在运行时访问变量的地址，以获取相对于变量处于特定位置的数据单元）

## Address-of operator (&)

The address of a variable can be obtained by preceding the name of a variable with an ampersand sign (`&`), known as _address-of operator_. For example:

```cpp
foo = &myvar;
```

This would assign the address of variable `myvar` to `foo`; by preceding the name of the variable `myvar` with the _address-of operator_ (`&`), we are no longer assigning the content of the variable itself to `foo`, but its address.

The actual address of a variable in memory cannot be known before runtime, (变量在主存中的地址，在运行前时无法预知的) but let's assume, in order to help clarify some concepts, that `myvar` is placed during runtime in the memory address `1776`.

In this case, consider the following code fragment:

```
myvar = 25;
foo = &myvar;
bar = myvar;
```

The values contained in each variable after the execution of this are shown in the following diagram:

![[var-address-value.png]]

First, we have assigned the value `25` to `myvar` (a variable whose address in memory we assumed to be `1776`).

The second statement assigns `foo` the address of `myvar`, which we have assumed to be `1776`.

Finally, the third statement, assigns the value contained in `myvar` to `bar`. This is a standard assignment operation, as already done many times in earlier chapters.

The main difference between the second and third statements is the appearance of the _address-of operator_ (`&`).

The variable that stores the address of another variable (like `foo` in the previous example) is what in C++ is called a _pointer_. (Cpp 中将存储了其他变量地址的变量称作“指针”) Pointers are a very powerful feature of the language that has many uses in lower level programming. A bit later, we will see how to declare and use pointers.

## Dereference operator (`*`)

As just seen, a variable which stores the address of another variable is called a _pointer_. Pointers are said to "point to" the variable whose address they store.（指针被称为“指向”它们存储地址的变量）

An interesting property of pointers is that they can be used to access the variable they point to directly. This is done by preceding the pointer name with the _dereference operator_ (`*`). The operator itself can be read as "value pointed to by".（指针的一个属性是可以用于直接访问它们所指向的变量，只需前缀 `*` 这一接触引用运算符。运算符本身可以理解为“指向的值”）

Therefore, following with the values of the previous example, the following statement:

```cpp
baz = *foo；
```

This could be read as: "`baz` equal to value pointed to by `foo`", and the statement would actually assign the value `25` to `baz`, since `foo` is `1776`, and the value pointed to by `1776` (following the example above) would be `25`.

![[address-pointer-value.png]]

It is important to clearly differentiate that `foo` refers to the value `1776`, while `*foo` (with an asterisk `*` preceding the identifier) refers to the value stored at address `1776`, which in this case is `25`. Notice the difference of including or not including the _dereference operator_ (I have added an explanatory comment of how each of these two expressions could be read):

```
baz = foo;   // baz equal to foo (1776)
baz = *foo;  // baz equal to value pointed to by foo (25)
```

The reference and dereference operators are thus complementary:  

*   `&` is the _address-of operator_, and can be read simply as "address of"
*   `*` is the _dereference operator_, and can be read as "value pointed to by"

Thus, they have sort of opposite meanings: An address obtained with `&` can be dereferenced with `*`.

Earlier, we performed the following two assignment operations:

```
myvar = 25;
foo = &myvar;
```

Right after these two statements, all of the following expressions would give true as result:

```
myvar == 25
&myvar == 1776
foo == 1776
*foo == 25
```

The first expression is quite clear, considering that the assignment operation performed on `myvar` was `myvar=25`. The second one uses the address-of operator (`&`), which returns the address of `myvar`, which we assumed it to have a value of `1776`. The third one is somewhat obvious, since the second expression was true and the assignment operation performed on `foo` was `foo=&myvar`. The fourth expression uses the _dereference operator_ (`*`) that can be read as "value pointed to by", and the value pointed to by `foo` is indeed `25`.

So, after all that, you may also infer that for as long as the address pointed to by `foo` remains unchanged, the following expression will also be true:

```cpp
*foo == myvar
```

## Declaring pointers

Due to the ability of a pointer to directly refer to the value that it points to, a pointer has different properties when it points to a `char` than when it points to an `int` or a `float`. Once dereferenced, the type needs to be known. And for that, the declaration of a pointer needs to include the data type the pointer is going to point to. (指针能够直接引用所指向的值，即指向 char 类型、int 类型和 float 类型的指针具有不同的属性，那么取消引用时就需要知道对应类型，因此指针的声明需要包括指针将要指向的数据类型)

The declaration of pointers follows this syntax:

`type * name;`

where `type` is the data type pointed to by the pointer. This type is not the type of the pointer itself, but the type of the data the pointer points to. For example:

```
int * number;
char * character;
double * decimals;
```

These are three declarations of pointers. Each one is intended to point to a different data type, but, in fact, all of them are pointers and all of them are likely going to occupy the same amount of space in memory (the size in memory of a pointer depends on the platform where the program runs). （指针本身占用相同的主存空间，这取决于程序运行的平台）

Nevertheless, the data to which they point to do not occupy the same amount of space nor are of the same type: the first one points to an `int`, the second one to a `char`, and the last one to a `double`. Therefore, although these three example variables are all of them pointers, they actually have different types: `int*`, `char*`, and `double*` respectively, depending on the type they point to.（指针所指向的变量占用主存空间大小不同，这与各自类型有关）

Note that the asterisk (`*`) used when declaring a pointer only means that it is a pointer (it is part of its type compound specifier), and should not be confused with the _dereference operator_ seen a bit earlier, but which is also written with an asterisk (`*`). They are simply two different things represented with the same sign.（声明指针时的 `*` 只表示当前变量是一个指针，而与前文的解除引用运算符无关，注意不要混淆）

Let's see an example on pointers:

```cpp
// my first pointer
#include <iostream> 
using namespace std;

int main ()
{
  int firstvalue, secondvalue;
  int * mypointer;

  mypointer = & firstvalue;
  *mypointer = 10;
  mypointer = & secondvalue;
  *mypointer = 20;
  cout << "firstvalue is " << firstvalue << '\n';
  cout << "secondvalue is " << secondvalue << '\n';
  return 0;
}
```

![[pointer-declare.png]]

Notice that even though neither `firstvalue` nor `secondvalue` are directly set any value in the program, both end up with a value set indirectly through the use of `mypointer`. This is how it happens:

First, `mypointer` is assigned the address of firstvalue using the address-of operator (`&`). Then, the value pointed to by `mypointer` is assigned a value of `10`. Because, at this moment, `mypointer` is pointing to the memory location of `firstvalue`, this in fact modifies the value of `firstvalue`.

In order to demonstrate that a pointer may point to different variables during its lifetime in a program, the example repeats the process with `secondvalue` and that same pointer, `mypointer`.

Here is an example a little bit more elaborated:

```cpp
// more pointers
#include <iostream> 
using namespace std;

int main ()
{
  int firstvalue = 5, secondvalue = 15;
  int * p1, * p2;

  p1 = & firstvalue;  // p 1 = address of firstvalue
  p2 = & secondvalue; // p 2 = address of secondvalue
  *p1 = 10;          // value pointed to by p 1 = 10
  *p2 = *p1;         // value pointed to by p 2 = value pointed to by p 1
  p1 = p2;           // p 1 = p 2 (value of pointer is copied)
  *p1 = 20;          // value pointed to by p 1 = 20
  
  cout << "firstvalue is " << firstvalue << '\n';
  cout << "secondvalue is " << secondvalue << '\n';
  return 0;
}
```

运行结果：
1. 注释 `p1=p2` 和 `*p1=20`：
	![[pointer-result1.png]]
2. 取消注释 `p1=p2`：
	![[Pasted image 20230707172743.png]]
3. 取消注释 `*p1=20`：
	![[Pasted image 20230707172807.png]]

Each assignment operation includes a comment on how each line could be read: i.e., replacing ampersands (`&`) by "address of", and asterisks (`*`) by "value pointed to by".

Notice that there are expressions with pointers `p1` and `p2`, both with and without the _dereference operator_ (`*`). The meaning of an expression using the _dereference operator_ (`*`) is very different from one that does not. (当两个指针 p1 和 p2 同时出现时，解除引用运算符 `*` 也总是同时出现或消失。这意味着解除引用运算符在使用与不使用时含义大不相同)

When this operator precedes the pointer name, the expression refers to the value being pointed, while when a pointer name appears without this operator, it refers to the value of the pointer itself (i.e., the address of what the pointer is pointing to).（当 `*` 位于指针之前时，表示指针所指向的变量；当指针前没有 `*` 时，表示指针本身的值）

Another thing that may call your attention is the line:

```cpp
int *p1, *p2;
```

This declares the two pointers used in the previous example. But notice that there is an asterisk (`*`) for each pointer, in order for both to have type `int*` (pointer to `int`). This is required due to the precedence rules. Note that if, instead, the code was:  

```cpp
int *p1, p2;
```

`p1` would indeed be of type `int*`, but `p2` would be of type `int`. Spaces do not matter at all for this purpose. But anyway, simply remembering to put one asterisk per pointer is enough for most pointer users interested in declaring multiple pointers per statement. Or even better: use a different statement for each variable.  

>[! warning] Becare of the number of `*`
>注意看上面两个例子，对每个标识符前添加 `*`，将会获得多个指针；如果只有最开始的标识符添加了 `*`，那么也只会有一个指针，其余的是对应类型的变量。
>
>更好的办法是对每一个变量都独自声明；

## Pointers and arrays

The concept of arrays is related to that of pointers. In fact, arrays work very much like pointers to their first elements, and, actually, an array can always be implicitly converted to the pointer of the proper type. （数组始终可以隐式转换为对应类型的指针）

For example, consider these two declarations:

```
int myarray [20];
int * mypointer;
```

The following assignment operation would be valid: 

```cpp
mypointer = myarray;
```

After that, `mypointer` and `myarray` would be equivalent and would have very similar properties. The main difference being that `mypointer` can be assigned a different address, whereas `myarray` can never be assigned anything, and will always represent the same block of 20 elements of type `int`.(指针和数组的首元素基本等效，主要区别在于指针可以分配不同地址，而数组一经确定就不能再直接赋值，并且始终表示相同类型的 20 个元素块) 

Therefore, the following assignment would not be valid:  

```cpp
myarray = mypointer;
```

Let's see an example that mixes arrays and pointers:

（五种使用指针和数组首元素的方法）
```cpp
// more pointers
#include <iostream> 
using namespace std;

int main ()
{
  int numbers[5];
  int * p;
  p = numbers;  *p = 10;
  p++;  *p = 20;
  p = & numbers[2];  *p = 30;
  p = numbers + 3;  *p = 40;
  p = numbers;  *(p+4) = 50;
  for (int n=0; n< 5; n++)
    cout << numbers[n] << ", ";
  return 0;
}
```

![[pointer-array.png]]

Pointers and arrays support the same set of operations, with the same meaning for both. The main difference being that pointers can be assigned new addresses, while arrays cannot.

In the chapter about arrays, brackets (`[]`) were explained as specifying the index of an element of the array. Well, in fact these brackets are a dereferencing operator known as _offset operator_. They dereference the variable they follow just as `*` does, but they also add the number between brackets to the address being dereferenced. (数组的 `[]` 实际上也是一种解除引用运算符，称作偏移运算符。`[]` 像 `*` 一样解除它们所跟随的变量的引用，但 `[]` 中的数字将被添加到要取消引用的地址中)

For example:

```
a[5] = 0;       // a [offset of 5] = 0
*(a+5) = 0;     // pointed to by (a+5) = 0
```

These two expressions are equivalent and valid, not only if `a` is a pointer, but also if `a` is an array. Remember that if an array, its name can be used just like a pointer to its first element.  

## Pointer initialization

Pointers can be initialized to point to specific locations at the very moment they are defined:

```
int myvar;
int * myptr = &myvar;
```

The resulting state of variables after this code is the same as after:

```
int myvar;
int * myptr;
myptr = &myvar;
```

When pointers are initialized, what is initialized is the address they point to (i.e., `myptr`), never the value being pointed (i.e., `*myptr`). Therefore, the code above shall not be confused with:

```
int myvar;
int * myptr;
*myptr = &myvar;
```

Which anyway would not make much sense (and is not valid code).

The asterisk (`*`) in the pointer declaration (line 2) only indicates that it is a pointer, it is not the dereference operator (as in line 3). Both things just happen to use the same sign: `*`. As always, spaces are not relevant, and never change the meaning of an expression.

Pointers can be initialized either to the address of a variable (such as in the case above), or to the value of another pointer (or array)（指针可以初始化另一个指针或数组）:

```
int myvar;
int *foo = &myvar;
int *bar = foo;
```

  

## Pointer arithmetics

To conduct arithmetical operations on pointers is a little different than to conduct them on regular integer types. To begin with, only addition and subtraction operations are allowed; the others make no sense in the world of pointers. But both addition and subtraction have a slightly different behavior with pointers, according to the size of the data type to which they point.（指针运算只允许加法和减法，并且运算行为与所指向数据类型的大小有关）

When fundamental data types were introduced, we saw that types have different sizes. For example: `char` always has a size of 1 byte, `short` is generally larger than that, and `int` and `long` are even larger; the exact size of these being dependent on the system. For example, let's imagine that in a given system, `char` takes 1 byte, `short` takes 2 bytes, and `long` takes 4.

Suppose now that we define three pointers in this compiler:

```
char *mychar;
short *myshort;
long *mylong;
```

and that we know that they point to the memory locations `1000`, `2000`, and `3000`, respectively.

Therefore, if we write:

```
++mychar;
++myshort;
++mylong;
```

`mychar`, as one would expect, would contain the value 1001. But not so obviously, `myshort` would contain the value 2002, and `mylong` would contain 3004, even though they have each been incremented only once. The reason is that, when adding one to a pointer, the pointer is made to point to the following element of the same type, and, therefore, the size in bytes of the type it points to is added to the pointer.（向指针添加一个指针时，指针指向相同类型的下一个元素，这将以字节为单位、类型的实际大小增长）

![[pointer-increment.png]]

This is applicable both when adding and subtracting any number to a pointer. It would happen exactly the same if we wrote:

```
mychar = mychar + 1;
myshort = myshort + 1;
mylong = mylong + 1;
```

Regarding the increment (`++`) and decrement (`--`) operators, they both can be used as either prefix or suffix of an expression, with a slight difference in behavior: 
- as a prefix, the increment happens before the expression is evaluated, and 
- as a suffix, the increment happens after the expression is evaluated. 

This also applies to expressions incrementing and decrementing pointers, which can become part of more complicated expressions that also include dereference operators (`*`). Remembering operator precedence rules, we can recall that postfix operators, such as increment `++` and decrement `--`, have higher precedence than prefix operators, such as the dereference operator (`*`). Therefore, the following expression:  

```cpp
*p++;
```

is equivalent to `*(p++)`. And what it does is to increase the value of `p` (so it now points to the next element), but because `++` is used as postfix, the whole expression is evaluated as the value pointed originally by the pointer (the address it pointed to before being incremented).

Essentially, these are the four possible combinations of the dereference operator with both the prefix and suffix versions of the increment operator (the same being applicable also to the decrement operator):

```
*p++   // same as *(p++): increment pointer, and dereference unincremented address
*++p   // same as *(++p): increment pointer, and dereference incremented address
++*p   // same as ++(*p): dereference pointer, and increment the value it points to
(*p)++ // dereference pointer, and post-increment the value it points to
```

A typical -but not so simple- statement involving these operators is:  

```cpp
*p++ = *q++;
```

Because `++` has a higher precedence than `*`, both `p` and `q` are incremented, but because both increment operators (`++`) are used as postfix and not prefix, the value assigned to `*p` is `*q` before both `p` and `q` are incremented. And then both are incremented. It would be roughly equivalent to:

```cpp
*p = *q;
++p;
++q;
```

Like always, parentheses reduce confusion by adding legibility to expressions.  

## Pointers and const

Pointers can be used to access a variable by its address, and this access may include modifying the value pointed. But it is also possible to declare pointers that can access the pointed value to read it, but not to modify it. For this, it is enough with qualifying the type pointed to by the pointer as `const`. (使用 const 修饰，获得可以访问变量的指针，但不能修改变量的值)

For example:

```
int x;
int y = 10;
const int * p = &y;
x = *p;          // ok: reading p
*p = x;          // error: modifying p, which is const-qualified
```

Here `p` points to a variable, but points to it in a `const` -qualified manner, meaning that it can read the value pointed, but it cannot modify it. Note also, that the expression `&y` is of type `int*`, but this is assigned to a pointer of type `const int*`. This is allowed: a pointer to non-const can be implicitly converted to a pointer to const. But not the other way around!（const 限定的指针只能访问而不能修改。同时，非限定的指针可以隐式转换为 const 限定的指针，但反之不然！） As a safety feature, pointers to `const` are not implicitly convertible to pointers to non- `const`.

One of the use cases of pointers to `const` elements is as function parameters: a function that takes a pointer to non-`const` as parameter can modify the value passed as argument, while a function that takes a pointer to `const` as parameter cannot.

```cpp
// pointers as arguments:
#include <iostream> 
using namespace std;

void increment_all (int* start, int* stop)
{
  int * current = start;
  while (current != stop) {
    ++(*current);  // increment value pointed
    ++current;     // increment pointer
  }
}

void print_all (const int* start, const int* stop)
{
  const int * current = start;
  while (current != stop) {
    cout << *current << '\n';
    ++current;     // increment pointer
  }
}

int main ()
{
  int numbers[] = {10,20,30};
  increment_all (numbers, numbers+3);
  print_all (numbers, numbers+3);
  return 0;
}
```

Note that `print_all` uses pointers that point to constant elements. These pointers point to constant content they cannot modify, but they are not constant themselves: i.e., the pointers can still be incremented or assigned different addresses, although they cannot modify the content they point to. (const 修饰的指针只是无法修改指向的变量的内容，而它本身并不是常量，仍然可以改变)

And this is where a second dimension to constness is added to pointers: Pointers can also be themselves const. And this is specified by appending const to the pointed type (after the asterisk):

```
int x;
      int *       p1 = &x;  // non-const pointer to non-const int
const int *       p2 = &x;  // non-const pointer to const int
      int * const p3 = &x;  // const pointer to non-const int
const int * const p4 = &x;  // const pointer to const int
```

> [! warning] const pointer and variable
> ⚠️：指针本身也可以是 const 的，这可以通过在解除引用运算符之后添加 const 来实现，将表示指针的常量性

The syntax with `const` and pointers is definitely tricky, and recognizing the cases that best suit each use tends to require some experience. In any case, it is important to get constness with pointers (and references) right sooner rather than later, but you should not worry too much about grasping everything if this is the first time you are exposed to the mix of `const` and pointers. More use cases will show up in coming chapters.

To add a little bit more confusion to the syntax of `const` with pointers, the `const` qualifier can either precede or follow the pointed type, with the exact same meaning:

```
const int * p2a = &x;  //      non-const pointer to const int
int const * p2b = &x;  // also non-const pointer to const int
```

As with the spaces surrounding the asterisk, the order of const in this case is simply a matter of style. This chapter uses a prefix `const`, as for historical reasons this seems to be more extended, but both are exactly equivalent. The merits of each style are still intensely debated on the internet.  （在指向变量的类型前后都可以添加 const，这二者完全等价，都表示非常量的指针指向常量的变量内容，只是代码风格的差异）

## Pointers and string literals

As pointed earlier, _string literals_ are arrays containing null-terminated character sequences. In earlier sections, string literals have been used to be directly inserted into `cout`, to initialize strings and to initialize arrays of characters.

But they can also be accessed directly. String literals are arrays of the proper array type to contain all its characters plus the terminating null-character, with each of the elements being of type `const char` (as literals, they can never be modified). （字符串本质是每个元素都以 const 修饰的字符序列以及一个 null 终止符）For example:

```
const char * foo = "hello";
```

> [! tip] 使用指针初始化字符串
> 上面的代码使用指针初始化了一个字符串。其底层逻辑是：字符串本身是一个字符序列数组，其首元素作为整个数组的指针，因此使用 `"hello"` 字符串直接创建了一个字符序列数组，并且将其首元素赋值给 char 类型的 foo 指针，再添加 const 修饰符。

This declares an array with the literal representation for `"hello"`, and then a pointer to its first element is assigned to `foo`. If we imagine that `"hello"` is stored at the memory locations that start at address 1702, we can represent the previous declaration as:

![[pointer-assignment.png]]

Note that here `foo` is a pointer and contains the value 1702, and not `'h'`, nor `"hello"`, although 1702 indeed is the address of both of these.

The pointer `foo` points to a sequence of characters. And because pointers and arrays behave essentially in the same way in expressions, `foo` can be used to access the characters in the same way arrays of null-terminated character sequences are. For example:

```cpp
*(foo+4)
foo[4]
```

Both expressions have a value of `'o'` (the fifth element of the array).  

## Pointers to pointers

C++ allows the use of pointers that point to pointers, that these, in its turn, point to data (or even to other pointers). The syntax simply requires an asterisk (`*`) for each level of indirection in the declaration of the pointer:

```
char a;
char * b;
char ** c;
a = 'z';
b = &a;
c = &b;
```

This, assuming the randomly chosen memory locations for each variable of `7230`, `8092`, and `10502`, could be represented as:

![[pointer-to-pointer.png]]

With the value of each variable represented inside its corresponding cell, and their respective addresses in memory represented by the value under them.

The new thing in this example is variable `c`, which is a pointer to a pointer, and can be used in three different levels of indirection, each one of them would correspond to a different value:

*   `c` is of type `char**` and a value of `8092`
*   `*c` is of type `char*` and a value of `7230`
*   `**c` is of type `char` and a value of `'z'`

  

## void pointers

The `void` type of pointer is a special type of pointer. In C++, `void` represents the absence of type. Therefore, `void` pointers are pointers that point to a value that has no type (and thus also an undetermined length and undetermined dereferencing properties). (Cpp 中 `void` 代表缺少类型，推之 void pointer表示指向的值没有类型，即不确定的存储长度和解引用属性)

This gives `void` pointers a great flexibility, by being able to point to any data type, from an integer value or a float to a string of characters. （一方面，无类型指针能够指向任意类型的数据，这提供了极大的灵活性）

In exchange, they have a great limitation: the data pointed to by them cannot be directly dereferenced (which is logical, since we have no type to dereference to), and for that reason, any address in a `void` pointer needs to be transformed into some other pointer type that points to a concrete data type before being dereferenced.（另一方面，作为代价，无类型指针不能直接解引用，因为不能确定其类型，但可以通过转换为指定类型的指针再进行解引用）

One of its possible uses may be to pass generic parameters to a function. For example:

```cpp
// increaser
#include <iostream> 
using namespace std;

void increase (void* data, int psize)
{
  if ( psize == sizeof (char) )
  { char* pchar; pchar=(char*) data; ++(*pchar); }
  else if (psize == sizeof (int) )
  { int* pint; pint=(int*) data; ++(*pint); }
}

int main ()
{
  char a = 'x';
  int b = 1602;
  increase (& a,sizeof (a));
  increase (& b,sizeof (b));
  cout << a << ", " << b << '\n';
  return 0;
}
```

`sizeof` is an operator integrated in the C++ language that returns the size in bytes of its argument. For non-dynamic data types, this value is a constant. Therefore, for example, `sizeof(char)` is 1, because `char` has always a size of one byte.  

## Invalid pointers and null pointers

In principle, pointers are meant to point to valid addresses, such as the address of a variable or the address of an element in an array. But pointers can actually point to any address, including addresses that do not refer to any valid element. Typical examples of this are _uninitialized pointers_ and pointers to nonexistent elements of an array（指针实际上可以指向任何地址，包括不引用任何有效元素的地址，典型例子是未初始化的指针和指向数组中不存在元素的指针）:

```
int * p;               // uninitialized pointer (local variable)

int myarray[10];
int * q = myarray+20;  // element out of bounds
```

Neither `p` nor `q` point to addresses known to contain a value, but none of the above statements causes an error. In C++, pointers are allowed to take any address value, no matter whether there actually is something at that address or not. （Cpp 中指针可以指向任何地址，即使那里没有任何东西）

What can cause an error is to dereference such a pointer (i.e., actually accessing the value they point to). Accessing such a pointer causes undefined behavior, ranging from an error during runtime to accessing some random value.（实际上引起错误的是对未定义指针进行解引用，即实际访问一个不存在的值，这将导致未定义的行为，范围从运行时的错误到访问某个随机值）

But, sometimes, a pointer really needs to explicitly point to nowhere, and not just an invalid address. For such cases, there exists a special value that any pointer type can take: the _null pointer value_.（有时指针确实需要显式地指向“无处”，而不是一个无效的地址。这时指针可以指向 “空指针值” ） This value can be expressed in C++ in two ways: either with an integer value of zero, or with the `nullptr` keyword:

```
int * p = 0;
int * q = nullptr;
```

Here, both `p` and `q` are _null pointers_, meaning that they explicitly point to nowhere, and they both actually compare equal: all _null pointers_ compare equal to other _null pointers_. (所有空指针之间的比较都相等) It is also quite usual to see the defined constant `NULL` be used in older code to refer to the _null pointer_ value:  

```cpp
int *r = NULL;
```

`NULL` is defined in several headers of the standard library, and is defined as an alias of some _null pointer_ constant value (such as `0` or `nullptr`).

Do not confuse _null pointers_ with `void` pointers! A _null pointer_ is a value that any pointer can take to represent that it is pointing to "nowhere", while a `void` pointer is a type of pointer that can point to somewhere without a specific type. One refers to the value stored in the pointer, and the other to the type of data it points to. (空指针是任何指针都可以采用的值，用于表示指向“无处”，无类型指针是一种指针类型，可以指向没有特定类型的某个位置。前者是存储在指针中的值，后者是指代它所指向的数据的类型)

>[! note] Void pointer vs. Null pointer
>在 C++中，`void*` 指针和空指针（`nullptr`）有一些区别。
>1. 类型信息：`void*` 是一个指向未知类型的指针，可以指向任何类型的数据，而空指针（`nullptr`）是一个特殊的指针值，表示指针不指向任何有效的对象。
>2. 内存操作：`void*` 指针可以进行指针运算，如加法、减法等，但是在使用之前必须先将其转换为具体的类型指针。空指针（`nullptr`）不能进行指针运算，因为它不指向任何有效的内存位置。
>3. 类型安全：`void*` 指针是一种非类型安全的指针，因为它可以指向任何类型的对象，而在对其进行解引用操作时，需要手动进行类型转换。空指针（`nullptr`）是一种类型安全的指针，因为它不能被解引用。
>4. 用途：`void*` 指针通常用于在不知道具体类型的情况下传递指针或在某些情况下实现通用算法。空指针（`nullptr`）通常用于表示指针不指向有效的对象，或者作为函数的默认参数。
>
>总的来说，`void*` 指针是一种通用的指针类型，可以指向任何类型的对象，但使用时需要小心处理类型转换。空指针（`nullptr`）是一个特殊的指针值，表示指针不指向任何有效的对象。

## Pointers to functions

C++ allows operations with pointers to functions. The typical use of this is for passing a function as an argument to another function. （指针可以指向函数，用途是将一个函数作为参数传递给另一个函数）Pointers to functions are declared with the same syntax as a regular function declaration, except that the name of the function is enclosed between parentheses () and an asterisk (`*`) is inserted before the name:

```cpp
// pointer to functions
#include <iostream> 
using namespace std;

int addition (int a, int b)
{ return (a+b); }

int subtraction (int a, int b)
{ return (a-b); }

int operation (int x, int y, int (*functocall)(int, int))
{
  int g;
  g = (*functocall)(x, y);
  return (g);
}

int main ()
{
  int m, n;
  int (*minus)(int, int) = subtraction;

  m = operation (7, 5, addition);
  n = operation (20, m, minus);
  cout << n;
  return 0;
}
```

In the example above, `minus` is a pointer to a function that has two parameters of type `int`. It is directly initialized to point to the function `subtraction`:

```
int (* minus)(int,int) = subtraction;
```