The usefulness of the "Hello World" programs shown in the previous chapter is rather questionable. We had to write several lines of code, compile them, and then execute the resulting program, just to obtain the result of a simple sentence written on the screen. It certainly would have been much faster to type the output sentence ourselves.

However, programming is not limited only to printing simple texts on the screen. In order to go a little further on and to become able to write programs that perform useful tasks that really save us work, we need to introduce the concept of _variables_.

Let's imagine that I ask you to remember the number 5, and then I ask you to also memorize the number 2 at the same time. You have just stored two different values in your memory (5 and 2). Now, if I ask you to add 1 to the first number I said, you should be retaining the numbers 6 (that is 5+1) and 2 in your memory. Then we could, for example, subtract these values and obtain 4 as result.

The whole process described above is a simile of what a computer can do with two variables. The same process can be expressed in C++ with the following set of statements:

```cpp
a = 5;
b = 2;
a = a + 1;
result = a - b;
```

Obviously, this is a very simple example, since we have only used two small integer values, but consider that your computer can store millions of numbers like these at the same time and conduct sophisticated mathematical operations with them.

We can now define `variable` as a portion of memory to store a value.

Each variable needs a name that identifies it and distinguishes it from the others. For example, in the previous code the variable names were `a`, `b`, and `result`, but we could have called the variables any names we could have come up with, as long as they were valid C++ identifiers.

### Identifiers

A valid identifier is a sequence of one or more letters, digits, or underscore characters (`_`). Spaces, punctuation marks, and symbols cannot be part of an identifier. In addition, identifiers shall always begin with a letter. They can also begin with an underline character (), but such identifiers are -on most cases- considered reserved for compiler-specific keywords or external identifiers, as well as identifiers containing two successive underscore characters anywhere. In no case can they begin with a digit.

C++ uses a number of keywords to identify operations and data descriptions; therefore, identifiers created by a programmer cannot match these keywords. The standard reserved keywords that cannot be used for programmer created identifiers are:

`__alignas`, `alignof`, `and`, `and_eq`, `asm`, `auto`, `bitand`, `bitor`, `bool`, `break`, `case`, `catch`, char, char16_t, char32_t, class, compl, const, constexpr, const_cast, continue, decltype, default, delete, do, double, dynamic_cast, else, enum, explicit, export, extern, false, float, for, friend, goto, if, inline, int, long, mutable, namespace, new, noexcept, not, not_eq, nullptr, operator, or, or_eq, private, protected, public, register, reinterpret_cast, return, short, signed, sizeof, static, static_assert, static_cast, struct, switch, template, this, thread_local, throw, true, try, typedef, typeid, typename, union, unsigned, using, virtual, void, volatile, wchar_t, while, xor, xor_eq ``RESULT`` result ``Result`  

>[! note] Reserved by compilers.
> Specific compilers may also have additional specific reserved keywords.

>[! warning] Case Sensitive.
>The C++ language is a "case sensitive" language. That means that an identifier written in capital letters is not equivalent to another one with the same name but written in small letters. Thus, for example, the variable is not the same as the variable or the variable. These are three different identifiers identifiying three different variables.
  

### Fundamental data types

The values of variables are stored somewhere in an unspecified location in the computer memory as zeros and ones. Our program does not need to know the exact location where a variable is stored; it can simply refer to it by its name. What the program needs to be aware of is the kind of data stored in the variable. It's not the same to store a simple integer as it is to store a letter or a large floating-point number; even though they are all represented using zeros and ones, they are not interpreted in the same way, and in many cases, they don't occupy the same amount of memory.

Fundamental data types are basic types implemented directly by the language that represent the basic storage units supported natively by most systems. They can mainly be classified into:

* **Character types:** They can represent a single character, such as `A` or `$`. The most basic type is `char`, which is a one-byte character. Other types are also provided for wider characters. 
* **Numerical integer types:** They can store a whole number value, such as `7` or `1024`. They exist in a variety of sizes, and can either be _signed_ or _unsigned_, depending on whether they support negative values or not. 
* **Floating-point types:** They can represent real values, such as `3.14` or `0.01`, with different levels of precision, depending on which of the three floating-point types is used.
* **Boolean type:** The boolean type, known in C++ as `bool`, can only represent one of two states, ` true ` or ` false `.

Here is the complete list of fundamental types in C++:  
<table><tbody><tr><th>Group</th><th>Type names*</th><th>Notes on size / precision</th></tr><tr><td rowspan="4">Character types</td><td><code><b>char</b></code></td><td>Exactly one byte in size. At least 8 bits.</td></tr><tr><td><code><b>char 16_t</b></code></td><td>Not smaller than <code>char</code>. At least 16 bits.</td></tr><tr><td><code><b>char 32_t</b></code></td><td>Not smaller than <code>char 16_t</code>. At least 32 bits.</td></tr><tr><td><code><b>wchar_t</b></code></td><td>Can represent the largest supported character set.</td></tr><tr><td rowspan="5">Integer types (signed)</td><td><code><b>signed char</b></code></td><td>Same size as <code>char</code>. At least 8 bits.</td></tr><tr><td><code><i>signed</i> <b>short</b> <i>int</i></code></td><td>Not smaller than <code>char</code>. At least 16 bits.</td></tr><tr><td><code><i>signed</i> <b>int</b></code></td><td>Not smaller than <code>short</code>. At least 16 bits.</td></tr><tr><td><code><i>signed</i> <b>long</b> <i>int</i></code></td><td>Not smaller than <code>int</code>. At least 32 bits.</td></tr><tr><td><code><i>signed</i> <b>long long</b> <i>int</i></code></td><td>Not smaller than <code>long</code>. At least 64 bits.</td></tr><tr><td rowspan="5">Integer types (unsigned)</td><td><code><b>unsigned char</b></code></td><td rowspan="5">(same size as their signed counterparts)</td></tr><tr><td><code><b>unsigned short</b> <i>int</i></code></td></tr><tr><td><code><b>unsigned</b> <i>int</i></code></td></tr><tr><td><code><b>unsigned long</b> <i>int</i></code></td></tr><tr><td><code><b>unsigned long long</b> <i>int</i></code></td></tr><tr><td rowspan="3">Floating-point types</td><td><code><b>float</b></code></td><td></td></tr><tr><td><code><b>double</b></code></td><td>Precision not less than <code>float</code></td></tr><tr><td><code><b>long double</b></code></td><td>Precision not less than <code>double</code></td></tr><tr><td>Boolean type</td><td><code><b>bool</b></code></td><td></td></tr><tr><td>Void type</td><td><code><b>void</b></code></td><td>no storage</td></tr><tr><td>Null pointer</td><td><code><b>decltype (nullptr)</b></code></td><td></td></tr></tbody></table>  
* The names of certain integer types can be abbreviated without their `signed` and `int` components - only the part not in italics is required to identify the type, the part in italics is optional. I.e., _signed_ short _int_ can be abbreviated as `signed short`, `short int`, or simply `short`; they all identify the same fundamental type.

Within each of the groups above, the difference between types is only their size (i.e., how much they occupy in memory): the first type in each group is the smallest, and the last is the largest, with each type being at least as large as the one preceding it in the same group. Other than that, the types in a group have the same properties.

Note in the panel above that other than `char` (which has a size of exactly one byte), none of the fundamental types has a standard size specified (but a minimum size, at most). Therefore, the type is not required (and in many cases is not) exactly this minimum size. This does not mean that these types are of an undetermined size, but that there is no standard size across all compilers and machines; each compiler implementation may specify the sizes for these types that fit the best the architecture where the program is going to run. This rather generic size specification for types gives the C++ language a lot of flexibility to be adapted to work optimally in all kinds of platforms, both present and future.

Type sizes above are expressed in bits; the more bits a type has, the more distinct values it can represent, but at the same time, also consumes more space in memory:

<table><tbody><tr><th>Size</th><th>Unique representable values</th><th>Notes</th></tr><tr><td>8-bit</td><td><code>256</code></td><td>= 2<sup>8</sup></td></tr><tr><td>16-bit</td><td><code>65 536</code></td><td>= 2<sup>16</sup></td></tr><tr><td>32-bit</td><td><code>4 294 967 296</code></td><td>= 2<sup>32</sup> (~4 billion)</td></tr><tr><td>64-bit</td><td><code>18 446 744 073 709 551 616</code></td><td>= 2<sup>64</sup> (~18 billion billion)</td></tr></tbody></table>  

For integer types, having more representable values means that the range of values they can represent is greater; for example, a 16-bit unsigned integer would be able to represent 65536 distinct values in the range 0 to 65535, while its signed counterpart would be able to represent, on most cases, values between -32768 and 32767. Note that the range of positive values is approximately halved in signed types compared to unsigned types, due to the fact that one of the 16 bits is used for the sign; this is a relatively modest difference in range, and seldom justifies the use of unsigned types based purely on the range of positive values they can represent.

For floating-point types, the size affects their precision, by having more or less bits for their significant and exponent.

If the size or precision of the type is not a concern, then `char`, `int`, and `double` are typically selected to represent characters, integers, and floating-point values, respectively. The other types in their respective groups are only used in very particular cases.

The properties of fundamental types in a particular system and compiler implementation can be obtained by using the [numeric_limits](https://cplusplus.com/numeric_limits) classes (see standard header `[<limits>](https://cplusplus.com/%3Climits%3E)`). If for some reason, types of specific sizes are needed, the library defines certain fixed-size type aliases in header `[<cstdint>](https://cplusplus.com/%3Ccstdint%3E)`.

The types described above (characters, integers, floating-point, and boolean) are collectively known as arithmetic types. But two additional fundamental types exist: `void`, which identifies the lack of type; and the type `nullptr`, which is a special type of pointer. Both types will be discussed further in a coming chapter about pointers.

C++ supports a wide variety of types based on the fundamental types discussed above; these other types are known as _compound data types_, and are one of the main strengths of the C++ language. We will also see them in more detail in future chapters.

### Declaration of variables

C++ is a strongly-typed language, and requires every variable to be declared with its type before its first use. This informs the compiler the size to reserve in memory for the variable and how to interpret its value. The syntax to declare a new variable in C++ is straightforward: we simply write the type followed by the variable name (i.e., its identifier). For example:

```cpp
int a;
float mynumber;
```

These are two valid declarations of variables. The first one declares a variable of type `int` with the identifier `a`. The second one declares a variable of type `float` with the identifier `mynumber`. Once declared, the variables `a` and `mynumber` can be used within the rest of their scope in the program.  
If declaring more than one variable of the same type, they can all be declared in a single statement by separating their identifiers with commas. For example:  

```cpp
int a;
int b;
int c;
```

This declares three variables (`a`, `b` and `c`), all of them of type `int`, and has exactly the same meaning as:  
To see what variable declarations look like in action within a program, let's have a look at the entire C++ code of the example about your mental memory proposed at the beginning of this chapter:
```cpp
// operating with variables

#include <iostream>
using namespace std;

int main ()
{
  // declaring variables:
  int a, b;
  int result;

  // process:
  a = 5;
  b = 2;
  a = a + 1;
  result = a - b;

  // print out the result:
  cout << result;

  // terminate the program:
  return 0;
}
```

Don't be worried if something else than the variable declarations themselves look a bit strange to you. Most of it will be explained in more detail in coming chapters.  

### Initialization of variables

When the variables in the example above are declared, they have an undetermined value until they are assigned a value for the first time. But it is possible for a variable to have a specific value from the moment it is declared. This is called the _initialization_ of the variable.

In C++, there are three ways to initialize variables. They are all equivalent and are reminiscent of the evolution of the language over the years:

1. The first one, known as _c-like initialization_ (because it is inherited from the C language), consists of appending an equal sign followed by the value to which the variable is initialized:
	
	`type identifier = initial_value;` 
	
	For example, to declare a variable of type `int` called `x` and initialize it to a value of zero from the same moment it is declared, we can write:
	`int x=0;`

2. A second method, known as _constructor initialization_ (introduced by the C++ language), encloses the initial value between parentheses (`()`):
	
	`type identifier (initial_value);`  
	
	For example: `int x(0);`

3. Finally, a third method, known as _uniform initialization_, similar to the above, but using curly braces (`{}`) instead of parentheses (this was introduced by the revision of the C++ standard, in 2011):
	
	`type identifier {initial_value};`  
	
	For example: `int x {0};`

All three ways of initializing variables are valid and equivalent in C++.

```cpp
// initialization of variables

#include <iostream>;
using namespace std;

int main ()
{
  int a=5;               // initial value: 5
  int b (3);              // initial value: 3
  int c{2};              // initial value: 2
  int result;            // initial value undetermined

  a = a + b;
  result = a - c;
  cout << result;

  return 0;
}
```

### Type deduction: auto and decltype

When a new variable is initialized, the compiler can figure out what the type of the variable is automatically by the initializer. For this, it suffices to use `auto` as the type specifier for the variable:

```
int foo = 0;
auto bar = foo;  // the same as: int bar = foo;
```

Here, `bar` is declared as having an `auto` type; therefore, the type of `bar` is the type of the value used to initialize it: in this case it uses the type of `foo`, which is `int`.

Variables that are not initialized can also make use of type deduction with the `decltype` specifier:

```
int foo = 0;
decltype(foo) bar;  // the same as: int bar;
```

Here, `bar` is declared as having the same type as `foo`.

`auto` and `decltype` are powerful features recently added to the language. But the type deduction features they introduce are meant to be used either when the type cannot be obtained by other means or when using it improves code readability. The two examples above were likely neither of these use cases. In fact they probably decreased readability, since, when reading the code, one has to search for the type of `foo` to actually know the type of `bar`.

### Introduction to strings

Fundamental types represent the most basic types handled by the machines where the code may run. But one of the major strengths of the C++ language is its rich set of compound types, of which the fundamental types are mere building blocks.

An example of compound type is the `string` class. Variables of this type are able to store sequences of characters, such as words or sentences. A very useful feature!

A first difference with fundamental data types is that in order to declare and use objects (variables) of this type, the program needs to include the header where the type is defined within the standard library (header `<string>`):

```cpp
// my first string
#include <iostream>
#include <string>
using namespace std;

int main ()
{
  string mystring;
  mystring = "This is a string";
  cout << mystring;
  return 0;
}

```

As you can see in the previous example, strings can be initialized with any valid string literal, just like numerical type variables can be initialized to any valid numerical literal. As with fundamental types, all initialization formats are valid with strings:

```
string mystring = "This is a string";
string mystring ("This is a string");
string mystring {"This is a string"};
```

Strings can also perform all the other basic operations that fundamental data types can, like being declared without an initial value and change its value during execution:

```cpp
// my first string
#include <iostream>
#include <string>
using namespace std;

int main ()
{
  string mystring;
  mystring = "This is the initial string content";
  cout &lt;&lt; mystring &lt;&lt; endl;
  mystring = "This is a different string content";
  cout &lt;&lt; mystring &lt;&lt; endl;
  return 0;
}

```

>[! note] std::endl
>**Note:** inserting the `endl` manipulator **end**s the **l**ine (printing a newline character and flushing the stream).

The [string](https://cplusplus.com/string) class is a _compound type_. As you can see in the example above, _compound types_ are used in the same way as _fundamental types_: the same syntax is used to declare variables and to initialize them.

For more details on standard C++ strings, see the [string](https://cplusplus.com/string) class reference.