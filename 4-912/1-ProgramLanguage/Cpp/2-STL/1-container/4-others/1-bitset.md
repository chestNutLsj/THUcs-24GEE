A bitset is an array of bools but each boolean value is not stored in a separate byte instead, bitset optimizes the space such that ***each boolean value takes 1-bit space only***, so space taken by bitset is less than that of an array of bool or vector of bool. 

A limitation of the bitset is that ***size must be known at compile time i.e. size of the bitset is fixed.***

> Cpp 中 bool 变量存放在一个字节中，尽管它只表示 0/1 。bitset 是专用于优化 bool 变量存储和操作的数组，其中的每个 bool 变量只占 1 位。
> 
> bitset 的一个限制就是其大小必须在编译时确定，因此导致 bitset 的大小也是固定的。

`std::bitset` is the class template for bitset that is defined inside `<bitset>` header file so we need to include the header file before using bitset in our program.

## *Init Syntax*

```
bitset<size> variable_name (initialization);
```

We can initialize bitset in three ways :

1. ***Uninitialized***: All the bits will be set to zero.

```cpp
bitset<size> variable_name;
```

> 所有位都设置为 0 。

2. ***Initialization with decimal integer***: Bitset will represent the given decimal number in binary form.

```cpp
bitset<size> variable_name (DECIMAL_NUMBER);
```

> 将十进制数字的二进制表示，存放在 bitset 中。

3. ***Initialization with binary string***: Bitset will represent the given binary string.

```cpp
bitset<size> variable_name (string ("BINARY_STRING"));
bitset<size> variable_name ("BINARY_STRING");
```

> 按给定的二进制字符串进行初始化。

### *Example*

```cpp
// C++ program to demonstrate the bitset 
#include <bitset>
#include <iostream>

using namespace std;

int main()
{
	// declaring an uninitialized bitset object
	bitset<8> uninitializedBitset;

	// initialization with decimal number
	bitset<8> decimalBitset(15);

	// initialization with binary string
	bitset<8> stringBitset(string("1111"));

	cout << "Uninitialized bitset: " << uninitializedBitset
		<< endl;
	cout << "Initialized with decimal: " << decimalBitset
		<< endl;
	cout << "Initialized with string: " << stringBitset
		<< endl;

	return 0;
}

```

***Output***：

```output
Uninitialized bitset: 00000000
Initialized with decimal: 00001111
Initialized with string: 00001111
```

## *Member Functions*

`std::bitset` class contains some useful member functions to work on the bitset objects. Here is the list of some member functions of `std::bitset`:

|                                Function Name                                | Function Description                          |
| :-------------------------------------------------------------------------: | --------------------------------------------- |
| [set()](https://www.geeksforgeeks.org/bitset-set-function-in-c-stl/?ref=rp) | Set the bit value at the given index to 1.    |
|                                   reset()                                   | Set the bit value at a given index to 0*.     |
|        [flip()](https://www.geeksforgeeks.org/bitset-any-in-c-stl/)         | Flip (翻转) the bit value at the given index.   |
|       [count()](https://www.geeksforgeeks.org/bitsetcount-in-c-stl/)        | Count the number of set bits.                 |
|                                   test()                                    | Returns the boolean value at the given index. |
|         [any()](https://www.geeksforgeeks.org/bitset-any-in-c-stl/)         | Checks if any bit is set.                     |
|        [none()](https://www.geeksforgeeks.org/bitsetnone-in-c-stl/)         | Checks if none bit is set.                    |
|         [all()](https://www.geeksforgeeks.org/bitsetall-in-c-stl/)          | Check if all bit is set.                      |
|                                   size()                                    | Returns the size of the bitset.               |
|                                 to_string()                                 | Converts bitset to std:: string.              |
|                                 to_ulong()                                  | Converts bitset to unsigned long.             |
|                                 to_ullong()                                 | Converts bitset to unsigned long long.        |

### *Example*

>[!Note] boolalpha
>boolalpha is used to print “true” and “false” instead of 1 or 0 for boolean values and noboolalpha for opposite.

```cpp
// C++ program to demonstrate the
// use of std::bitset member
// functions
#include <bitset>
#include <iostream>
 
using namespace std;
 
int main ()
{
    // declaring index variable
    int index = 0;
 
    // declaring few bitset objects
    bitset<4> allSet ("1111"), allUnset;
 
    cout << "any () value: " << boolalpha << allSet.any ()
         << endl;
 
    cout << "all () value: " << allSet.all () << endl;
 
    cout << "none () value: " << allSet.none () << endl;
 
    cout << "test () at index 0: " << noboolalpha
         << allSet.test (index) << endl;
 
    cout << "size () value: " << allSet.size () << endl;
 
    cout << "Value of allUnset on before using set (): "
         << allUnset << endl;
    allUnset.set (index);
    cout << "Value of allUnset on after using set (): "
         << allUnset << endl;
 
    cout << "Value of allSet on before using reset (): "
         << allSet << endl;
    allSet.reset (index);
    cout << "Value of allSet on after using reset (): "
         << allSet << endl;
 
    // declaring an empty string
    string bitString;
    // using to_string () method to assign value to empty
    // string
    bitString = allSet. to_string ();
    cout << "bitString: " << bitString << endl;
 
    cout << "Unsigned Long value: " << allSet. to_ulong ();
 
    return 0;
}
```

***Output***：

```output
any () value: true
all () value: true
none () value: false
test () at index 0: 1
size () value: 4
Value of allUnset on before using set (): 0000
Value of allUnset on after using set (): 0001
Value of allSet on before using reset (): 1111
Value of allSet on after using reset (): 1110
bitString: 1110
Unsigned Long value: 14
```

## std:: bitset Operators

Some of the basic operators are overloaded to work with bitset objects. Following is the list of those operators:

| Operator | Operation                                                |
| -------- | -------------------------------------------------------- |
| []       | Access operator                                          |
| &        | Bitwise AND                                              |
| \|       | Bitwise OR                                               |
| !        | **Bitwise XOR**                                          |
| >>=      | Binary *Right shift and assign*                          |
| <<=      | Binary *Left shift and assign*                           |
| &=       | Assign the value of bitwise **AND** to the first bitset. |
| \|=      | Assign the value of bitwise **OR** to the first bitset.  |
| ^=       | Assign the value of bitwise **XOR** to the first bitset. |
| ~        | Bitwise **NOT**                                          |

### *Example*

```cpp
// C++ program to show the different operator functions on
// bitset
#include <bitset>
#include <iostream>

using namespace std;

int main()
{

	bitset<4> bitset1("1001"), bitset2("1010");
	bitset<4> result;

	cout << "Bitset1: " << bitset1
		<< "\nBitset2: " << bitset2 << endl;

	cout << "Accessing bit value at index 1 of bitset1: "
		<< bitset1[1] << endl;

	// bitwise AND
	cout << "Bitwise AND using &: "
		<< (result = bitset1 & bitset2) << endl;
	cout << "Bitwise AND using &=: " << (bitset1 &= bitset2)
		<< endl;

	// bitwise OR
	bitset1 = 9; // 9 = 1001
	cout << "Bitwise OR using |: "
		<< (result = bitset1 | bitset2) << endl;
	cout << "Bitwise OR using |=: " << (bitset1 |= bitset2)
		<< endl;

	// bitwise NOT
	cout << "Bitwise NOT: " << (result = ~bitset1) << endl;

	// bitwise XOR
	bitset1 = 9;
	cout << "Bitwise XOR: " << (bitset1 ^= bitset2) << endl;

	bitset1 = 9;
	cout << "Binary leftshift on bitwise1: "
		<< (bitset1 <<= 1) << endl;
	bitset1 = 9;
	cout << "Binary rightshift on bitwise1: "
		<< (bitset1 >>= 1) << endl;

	return 0;
}

```

***Output***：

```
Bitset1: 1001
Bitset2: 1010
Accessing bit value at index 1 of bitset1: 0
Bitwise AND using &: 1000
Bitwise AND using &=: 1000
Bitwise OR using |: 1011
Bitwise OR using |=: 1011
Bitwise NOT: 0100
Bitwise XOR: 0011
Binary leftshift on bitwise1: 0010
Binary rightshift on bitwise1: 0100
```

## Difference between `std::bitset` and `std::vector<bool>` and an array of bool

Vector of bool and array of bool can also be implemented to store a sequence of boolean values like bitset but there are some differences between each implementation: 

| Parameter      | bitset                                                                                                | vector of bool                                                                                | array of bool                                             |
| -------------- | ----------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- | --------------------------------------------------------- |
| **Definition** | A class template consisting of a sequence of bits stored such that each bit occupies 1 bit of memory. | A variation of vectors of C++ STL in which each element is of size 1 bit and is of type bool. | A fixed size contiguous collection of bool data elements. |
| **Size**       | Fixed Size.                                                                                           | Dynamic Size.                                                                                 | Fixed Size.                                               |
| **Memory**     | A single element occupies 1 bit of memory.                                                            | A single element occupies 1 bit of memory.                                                    | A single element occupies 1 byte of memory.               |
| **Speed**      | Same                                                                                                  | Same                                                                                          | Faster                                                    |

> - bitset 是包含一串 bit 的类模板，每个 bit 实际占用 1b 的空间，整体上是定长的；
> - `vector<bool>` 是 STL 中每个元素都占用 1 位空间的 vector 向量，每个元素的类型都是 bool ，并且有 vector 的变长性质，运行速度与 bitset 相当；
> - `bool arr[N]` 是包含 N 个 bool 变量的数组，是定长的，每个元素实际占用 1 个字节，不过运行速度比前二者都快。

>[!warning] 为什么不建议使用 `std::vector<bool>` ？
>在 C++中，`vector<bool>` 是一个特化版本的 `vector`，用于存储布尔值。然而，由于尝试提高空间效率，`vector<bool>` 的行为与其他 `vector` 类型有显著不同，这导致了一些不建议使用它的原因：
>1. **空间优化**：`vector<bool>` 会对布尔值进行压缩，每个布尔值只占用一个比特而不是一个完整的字节。这种压缩策略虽然节省了空间，但也带来了性能开销，因为每次访问元素时都需要进行位操作。
>2. **接口和行为差异**：由于这种压缩表示，`vector<bool>` 的引用和迭代器的行为与标准容器不同。例如，它不能返回布尔值的实际引用（因为位不能被直接引用），而是返回一个临时对象。这意味着你不能像使用其他类型的 `vector` 那样使用 `vector<bool>` 的引用和迭代器。
>3. **不兼容通用代码**：上述差异导致 `vector<bool>` 不能无缝地用于需要模板或泛型编程的代码中，因为期望容器元素可以被引用和修改的代码可能无法正确工作。
>4. **性能问题**：尽管节省了空间，但在许多情况下，访问和修改压缩存储的布尔值的开销可能超过了节省空间的好处。特别是在需要频繁访问元素的应用中，这种性能损失尤为明显。
>5. **替代方案**：对于需要存储布尔值序列的情况，可以考虑使用其他数据结构，如 `std::deque<bool>` 或 `std::vector<char>`，或者使用专门的位操作库如 `std::bitset`，这些替代方案提供了更一致的接口和可能更优的性能表现。
>
>综上所述，尽管 `vector<bool>` 在特定情况下可以节省空间，但由于其非标准行为和潜在的性能问题，通常不推荐使用它，特别是在性能敏感或需要容器行为一致性的场景中。