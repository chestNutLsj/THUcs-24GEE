Once introduced to variables and constants, we can begin to operate with them by using _operators_. What follows is a complete list of operators. At this point, it is likely not necessary to know all of them, but they are all listed here to also serve as reference.  


## Assignment operator (`=`)

The assignment operator assigns a value to a variable.  

```cpp
x = 5;
```

This statement assigns the integer value `5` to the variable `x`. The assignment operation always takes place from right to left, and never the other way around:

```cpp
x = y;
```

This statement assigns to variable `x` the value contained in variable `y`. The value of `x` at the moment this statement is executed is lost and replaced by the value of `y`.

Consider also that we are only assigning the value of `y` to `x` at the moment of the assignment operation. Therefore, if `y` changes at a later moment, it will not affect the new value taken by `x`.

For example, let's have a look at the following code - I have included the evolution of the content stored in the variables as comments:

```cpp
// assignment operator
#include <iostream>
using namespace std;

int main ()
{
  int a, b;         // a:?,  b:?
  a = 10;           // a: 10, b:?
  b = 4;            // a: 10, b:4
  a = b;            // a: 4,  b:4
  b = 7;            // a: 4,  b:7

  cout << "a: " << a;
  cout << " b: " << b;

}
```

This program prints on screen the final values of `a` and `b` (4 and 7, respectively). Notice how `a` was not affected by the final modification of `b`, even though we declared `a = b` earlier.

Assignment operations are expressions that can be evaluated. That means that the assignment itself has a value, and -for fundamental types- this value is the one assigned in the operation. For example:

```cpp
y = 2 + (x = 5);
```

In this expression, `y` is assigned the result of adding 2 and the value of another assignment expression (which has itself a value of 5). It is roughly equivalent to:  

```cpp
x = 5;
y = 2 + x;
```

With the final result of assigning 7 to `y`.

The following expression is also valid in C++:

```cpp
x = y = z = 5;
```

It assigns 5 to the all three variables: `x`, `y` and `z`; always from right-to-left.  

## Arithmetic operators (`+`, `-`, `*`, `/`, `%`)

The five arithmetical operations supported by C++ are:

| operator | description    |
|----------|----------------|
| +        | addition       |
| -        | subtraction    |
| *        | multiplication |
| /        | division       |
| %        | modulo         |

Operations of addition, subtraction, multiplication and division correspond literally to their respective mathematical operators. The last one, _modulo operator_, represented by a percentage sign (`%`), gives the remainder of a division of two values. For example:

```cpp
x = 11 % 3  //x = 2
```

results in variable `x` containing the value 2, since dividing 11 by 3 results in 3, with a remainder of 2. 

## Compound assignment (`+=`, `-=`, `*=`, `/=`, `%=`, `>>=`, `<<=`, `&=`, `^=`, `|=`)

Compound assignment operators modify the current value of a variable by performing an operation on it. They are equivalent to assigning the result of an operation to the first operand:

| expression          | equivalent to...           |
|---------------------|----------------------------|
| y += x;             | y = y + x;                 |
| x -= 5;             | x = x - 5;                 |
| x /= y;             | x = x / y;                 |
| price \*= units + 1; | price = price \* (units+1); |

and the same for all other compound assignment operators. For example:

```cpp
// compound assignment operators
#include <iostream>
using namespace std;

int main ()
{
  int a, b=3;
  a = b;
  a+=2;             // equivalent to a=a+2
  cout << a;        // a = 5
}

```

## Increment and decrement (`++`, `--`)

Some expression can be shortened even more: the increase operator (`++`) and the decrease operator (`--`) increase or reduce by one the value stored in a variable. They are equivalent to `+=1` and to `-=1`, respectively. Thus:  

```cpp
++x;
x+=1;
x=x+1;
```

are all equivalent in its functionality; the three of them increase by one the value of `x`.

In the early C compilers, the three previous expressions may have produced different executable code depending on which one was used. Nowadays, this type of code optimization is generally performed automatically by the compiler, thus the three expressions should produce **exactly the same executable code**.

A peculiarity of this operator is that it can be used both as a prefix and as a suffix. That means that it can be written either before the variable name (`++x`) or after it (`x++`). Although in simple expressions like `x++` or `++x`, both have exactly the same meaning; in other expressions in which the result of the increment or decrement operation is evaluated, they may have an important difference in their meaning: 
- In the case that the increase operator is used as a prefix (`++x`) of the value, the expression evaluates to the final value of `x`, once it is already increased. 
- On the other hand, in case that it is used as a suffix (`x++`), the value is also increased, but the expression evaluates to the value that x had before being increased. Notice the difference:

<table><tbody><tr><th>Example 1</th><th>Example 2</th></tr><tr><td><code>x = 3;<br>y = ++x;<br>// x contains 4, y contains 4<br></code></td><td><code>x = 3;<br>y = x++;<br>// x contains 4, y contains 3<br></code></td></tr></tbody></table>  

In _Example 1_, the value assigned to `y` is the value of `x` after being increased. While in _Example 2_, it is the value `x` had before being increased.  

## Relational and comparison operators (`==`, `!=`, `>`, `<`, `>=`, `<=` )

Two expressions can be compared using relational and equality operators. For example, to know if two values are equal or if one is greater than the other.

The result of such an operation is either true or false (i.e., a Boolean value).

The relational operators in C++ are:

| operator | description              |
|----------|--------------------------|
| ==       | Equal to                 |
| !=       | Not equal to             |
| &lt;     | Less than                |
| &gt;     | Greater than             |
| &lt;=    | Less than or equal to    |
| &gt;=    | Greater than or equal to |

Here there are some examples:

```
(7 == 5)     // evaluates to false
(5 > 4)      // evaluates to true
(3 != 2)     // evaluates to true
(6 >= 6)     // evaluates to true
(5 < 5)      // evaluates to false
```

Of course, it's not just numeric constants that can be compared, but just any value, including, of course, variables. Suppose that `a=2`, `b=3` and `c=6`, then:

```
(a == 5)     // evaluates to false, since a is not equal to 5
(a*b >= c)   // evaluates to true, since (2*3 >= 6) is true
(b+4 > a*c)  // evaluates to false, since (3+4 > 2*6) is false
((b=2) == a) // evaluates to true
```

Be careful! The assignment operator (operator `=`, with one equal sign) is not the same as the equality comparison operator (operator `==`, with two equal signs); the first one (`=`) assigns the value on the right-hand to the variable on its left, while the other (`==`) compares whether the values on both sides of the operator are equal. 

Therefore, in the last expression (`(b=2) == a`), we first assigned the value `2` to `b` and then we compared it to `a` (that also stores the value 2), yielding `true`.  

## Logical operators (`!`, `&&`, `||`)

The operator `!` is the C++ operator for the Boolean operation NOT. It has only one operand, to its right, and inverts it, producing `false` if its operand is `true`, and `true` if its operand is `false`. Basically, it returns the opposite Boolean value of evaluating its operand. For example:

```cpp
!(5 == 5)   // evaluates to false because the expression at its right (5 == 5) is true
!(6 <= 4)   // evaluates to true because (6 <= 4) would be false
!true       // evaluates to false
!false      // evaluates to true
```

The logical operators `&&` and `||` are used when evaluating two expressions to obtain a single relational result. The operator `&&` corresponds to the Boolean logical operation AND, which yields `true` if both its operands are `true`, and `false` otherwise. The following panel shows the result of operator `&&` evaluating the expression `a&&b`:

| a     | b     | a && b |
| ----- | ----- | ------ |
| true  | true  | true   |
| true  | false | false  |
| false | true  | false  |
| false | false | false  |
<center>Table: `&&` OPERATOR (and) </center>

The operator `||` corresponds to the Boolean logical operation OR, which yields `true` if either of its operands is `true`, thus being false only when both operands are false. Here are the possible results of `a||b`:

| a     | b     | a \|\| b     |
| ----- | ----- | ----- |
| true  | true  | true  |
| true  | false | true  |
| false | true  | true  |
| false | false | false |

For example:  

```
( (5 == 5) && (3 > 6) )  // evaluates to false ( true && false )
( (5 == 5) || (3 > 6) )  // evaluates to true ( true || false )
```

When using the logical operators, C++ only evaluates what is necessary from left to right to come up with the combined relational result, ignoring the rest. Therefore, in the last example (`(5==5)||(3>6)`), C++ evaluates first whether `5==5` is `true`, and if so, it never checks whether `3>6` is `true` or not. This is known as _short-circuit evaluation_, and works like this for these operators:

| operator   | short-circuit                                                                                                                |
|------------|------------------------------------------------------------------------------------------------------------------------------|
| &amp;&amp; | if the left-hand side expression is false, the combined result is false (the right-hand side expression is never evaluated). |
| \|\|         | if the left-hand side expression is true, the combined result is true (the right-hand side expression is never evaluated).   |

This is mostly important when the right-hand expression has side effects, such as altering values:

```
if ( (i<10) && (++i<n) ) { /*...*/ }   // note that the condition increments i
```

Here, the combined conditional expression would increase `i` by one, but only if the condition on the left of `&&` is `true`, because otherwise, the condition on the right-hand side (`++i<n`) is never evaluated.  

## Conditional ternary operator (` ? : `)

The conditional operator evaluates an expression, returning one value if that expression evaluates to `true`, and a different one if the expression evaluates as `false`. Its syntax is:

`condition ? result 1 : result 2`

If `condition` is `true`, the entire expression evaluates to `result 1`, and otherwise to `result 2`.

```
7==5 ? 4 : 3     // evaluates to 3, since 7 is not equal to 5.
7==5+2 ? 4 : 3   // evaluates to 4, since 7 is equal to 5+2.
5>3 ? a : b      // evaluates to the value of a, since 5 is greater than 3.
a>b ? a : b      // evaluates to whichever is greater, a or b.
```

For example:

```cpp
// conditional operator
#include <iostream>
using namespace std;

int main ()
{
  int a, b, c;

  a=2;
  b=7;
  c = (a > b) ? a : b;

  cout << c << '\n';
}

```

In this example, `a` was 2, and `b` was 7, so the expression being evaluated (`a>b`) was not `true`, thus the first value specified after the question mark was discarded in favor of the second value (the one after the colon) which was `b` (with a value of 7).  

## Comma operator (`,`)

The comma operator (`,`) is used to separate two or more expressions that are included where only one expression is expected. When the set of expressions has to be evaluated for a value, only the right-most expression is considered.

For example, the following code:

```cpp
#include "iostream"  
  
int main() {  
	int b = 0;  
	int a;  
	a = (b = 3, a = 2, b + 2); 
	std::cout << "a = " << a; // a = 5  
	  
	return 0;  
}
```

would first assign the value 3 to `b`, and then assign `b+2` to variable `a`. So, at the end, variable `a` would contain the value 5 while variable `b` would contain value 3.

## Bitwise operators (`&`, `|`, `^`, `~`, `<<`, `>>` )

Bitwise operators modify variables considering the bit patterns that represent the values they store.

| operator | asm equivalent | description                      |
|----------|----------------|----------------------------------|
| &amp;    | AND            | Bitwise AND                      |
| \|        | OR             | Bitwise inclusive OR             |
| ^        | XOR            | Bitwise exclusive OR             |
| \~        | NOT            | Unary complement (bit inversion) |
| &lt;&lt; | SHL            | Shift bits left                  |
| &gt;&gt; | SHR            | Shift bits right                 |

## Explicit type casting operator

Type casting operators allow to convert a value of a given type to another type. There are several ways to do this in C++. The simplest one, which has been inherited from the C language, is to precede the expression to be converted by the new type enclosed between parentheses (`(type)`):

```
int i;
float f = 3.14;
i = (int) f; // i = 3
```

The previous code converts the floating-point number `3.14` to an integer value (`3`); the remainder is lost. Here, the typecasting operator was `(int)`. 

Another way to do the same thing in C++ is to use the functional notation preceding the expression to be converted by the type and enclosing the expression between parentheses:

```cpp
i = int (f)
```

Both ways of casting types are valid in C++.  

## sizeof

This operator accepts one parameter, which can be either a type or a variable, and returns the size in bytes of that type or object:  

```cpp
x = sizeof (char)
```

Here, `x` is assigned the value `1`, because `char` is a type with a size of one byte.

The value returned by `sizeof` is a compile-time constant, so it is always determined before program execution.

## Other operators

Later in these tutorials, we will see a few more operators, like the ones referring to pointers or the specifics for object-oriented programming.  

## Precedence of operators

A single expression may have multiple operators. For example:  

```cpp
x = 5 + 7 % 2;
```

In C++, the above expression always assigns 6 to variable `x`, because the `%` operator has a higher precedence than the `+` operator, and is always evaluated before. Parts of the expressions can be enclosed in parenthesis to override this precedence order, or to make explicitly clear the intended effect. Notice the difference:

```
x = 5 + (7 % 2);    // x = 6 (same as without parenthesis)
x = (5 + 7) % 2;    // x = 0
```

From greatest to smallest priority, C++ operators are evaluated in the following order:  

<table><tbody><tr><th>Level</th><th>Precedence group</th><th>Operator</th><th>Description</th><th>Grouping</th></tr><tr><td>1</td><td>Scope</td><td><code>::</code></td><td>scope qualifier</td><td>Left-to-right</td></tr><tr><td rowspan="4">2</td><td rowspan="4">Postfix (unary)</td><td><code>++ --</code></td><td>postfix increment / decrement</td><td rowspan="4">Left-to-right</td></tr><tr><td><code>()</code></td><td>functional forms</td></tr><tr><td><code>[]</code></td><td>subscript</td></tr><tr><td><code>. -&gt;</code></td><td>member access</td></tr><tr><td rowspan="7">3</td><td rowspan="7">Prefix (unary)</td><td><code>++ --</code></td><td>prefix increment / decrement</td><td rowspan="7">Right-to-left</td></tr><tr><td><code>~ !</code></td><td>bitwise NOT / logical NOT</td></tr><tr><td><code>+ -</code></td><td>unary prefix</td></tr><tr><td><code>&amp; *</code></td><td>reference / dereference</td></tr><tr><td><code>new delete</code></td><td>allocation / deallocation</td></tr><tr><td><code>sizeof</code></td><td>parameter pack</td></tr><tr><td><code>(<i>type</i>)</code></td><td>C-style type-casting</td></tr><tr><td>4</td><td>Pointer-to-member</td><td><code>.* -&gt;*</code></td><td>access pointer</td><td>Left-to-right</td></tr><tr><td>5</td><td>Arithmetic: scaling</td><td><code>* / %</code></td><td>multiply, divide, modulo</td><td>Left-to-right</td></tr><tr><td>6</td><td>Arithmetic: addition</td><td><code>+ -</code></td><td>addition, subtraction</td><td>Left-to-right</td></tr><tr><td>7</td><td>Bitwise shift</td><td><code>&lt;&lt; &gt;&gt;</code></td><td>shift left, shift right</td><td>Left-to-right</td></tr><tr><td>8</td><td>Relational</td><td><code>&lt; &gt; &lt;= &gt;=</code></td><td>comparison operators</td><td>Left-to-right</td></tr><tr><td>9</td><td>Equality</td><td><code>== !=</code></td><td>equality / inequality</td><td>Left-to-right</td></tr><tr><td>10</td><td>And</td><td><code>&amp;</code></td><td>bitwise AND</td><td>Left-to-right</td></tr><tr><td>11</td><td>Exclusive or</td><td><code>^</code></td><td>bitwise XOR</td><td>Left-to-right</td></tr><tr><td>12</td><td>Inclusive or</td><td><code>|</code></td><td>bitwise OR</td><td>Left-to-right</td></tr><tr><td>13</td><td>Conjunction</td><td><code>&amp;&amp;</code></td><td>logical AND</td><td>Left-to-right</td></tr><tr><td>14</td><td>Disjunction</td><td><code>||</code></td><td>logical OR</td><td>Left-to-right</td></tr><tr><td rowspan="2">15</td><td rowspan="2">Assignment-level expressions</td><td><code>= *= /= %= += -=<br>&gt;&gt;= &lt;&lt;= &amp;= ^= |=</code></td><td>assignment / compound assignment</td><td rowspan="2">Right-to-left</td></tr><tr><td><code>?:</code></td><td>conditional operator</td></tr><tr><td>16</td><td>Sequencing</td><td><code>,</code></td><td>comma separator</td><td>Left-to-right</td></tr></tbody></table>  


When an expression has two operators with the same precedence level, _grouping_ determines which one is evaluated first: either left-to-right or right-to-left.

Enclosing all sub-statements in parentheses (even those unnecessary because of their precedence) improves code readability.