
stack 栈适配器是一种单端开口的容器，实际上该容器模拟的就是栈存储结构，即无论是向里存数据还是从中取数据，都只能从这一个开口实现操作。

![[stl-stack.png]]

如图所示，stack 适配器的开头端通常称为栈顶。由于数据的存和取只能从栈顶处进行操作，因此对于存取数据，stack 适配器有这样的特性，即每次只能访问适配器中位于最顶端的元素，也只有移除 stack 顶部的元素之后，才能访问位于栈中的元素。

> 栈中存储的元素满足 “后进先出（简称 LIFO）” 的准则，stack 适配器也同样遵循这一准则。

## stack 容器适配器的创建
--------------

由于 stack 适配器以模板类 stack<T,Container=deque\<T\>>（其中 T 为存储元素的类型，Container 表示底层容器的类型）的形式位于 \<stack\> 头文件中，并定义在 std 命名空间里。因此，在创建该容器之前，程序中应包含以下 2 行代码：

```
#include <stack>
using namespace std;
```

> std 命名空间也可以在使用 stack 适配器时额外注明。

创建 stack 适配器，大致分为如下几种方式。

1) 创建一个不包含任何元素的 stack 适配器，并采用默认的 deque 基础容器：
```
std::stack<int> values;
```
上面这行代码，就成功创建了一个可存储 int 类型元素，底层采用 deque 基础容器的 stack 适配器。

2) 上面提到，stack<T,Container=deque\<T\>> 模板类提供了 2 个参数，通过指定第二个模板类型参数，我们可以使用出 deque 容器外的其它序列式容器，只要该容器支持 empty ()、size ()、back ()、push_back ()、pop_back () 这 5 个成员函数即可。

在介绍适配器时提到，序列式容器中同时包含这 5 个成员函数的，有 vector、deque 和 list 这 3 个容器。因此，stack 适配器的基础容器可以是它们 3 个中任何一个。例如，下面展示了如何定义一个使用 list 基础容器的 stack 适配器：
```
std::stack<std::string, std::list<int>> values;
```

3) 可以用一个基础容器来初始化 stack 适配器，只要该容器的类型和 stack 底层使用的基础容器类型相同即可。例如：
```
std::list<int> values {1, 2, 3};
std::stack<int,std::list<int>> my_stack (values);
```
注意，初始化后的 my_stack 适配器中，栈顶元素为 3，而不是 1。另外在第 2 行代码中，stack 第 2 个模板参数必须显式指定为 list\<int\>（必须为 int 类型，和存储类型保持一致），否则 stack 底层将默认使用 deque 容器，也就无法用 lsit 容器的内容来初始化 stack 适配器。

4) 还可以用一个 stack 适配器来初始化另一个 stack 适配器，只要它们存储的元素类型以及底层采用的基础容器类型相同即可。例如：
```
std::list<int> values{1,2,3};
std::stack<int, std::list<int>> mystack1(values);
std::stack<int, std::list<int>> mystack=mystack1;
//std::stack<int,std::list<int>> mystack(mystack1)
```

可以看到，和使用基础容器不同，使用 stack 适配器给另一个 stack 进行初始化时，有 2 种方式，使用哪一种都可以。

> 注意，第 3、4 种初始化方法中，my_stack 适配器的数据是经过拷贝得来的，也就是说，操作 my_stack 适配器，并不会对 values 容器以及 my_stack 1 适配器有任何影响；反过来也是如此。

## stack 的成员函数
------------------

和其他序列容器相比，stack 是一类存储机制简单、提供成员函数较少的容器。下表列出了 stack 容器支持的全部成员函数。

| 成员函数                                     | 功能                                                                           |
|------------------------------------------|------------------------------------------------------------------------------|
| empty ()                                 | 当 stack 栈中没有元素时，该成员函数返回 true；反之，返回 false。                                    |
| size ()                                  | 返回 stack 栈中存储元素的个数。                                                          |
| top ()                                   | 返回一个栈顶元素的引用，类型为 T&amp;。如果栈为空，程序会报错。                                          |
| push (const T&amp; val)                  | 先复制 val，再将 val 副本压入栈顶。这是通过调用底层容器的 push_back () 函数完成的。                        |
| push (T&amp;&amp; obj)                   | 以移动元素的方式将其压入栈顶。这是通过调用底层容器的有右值引用参数的 push_back () 函数完成的。                       |
| pop ()                                   | 弹出栈顶元素。                                                                      |
| emplace (arg...)                         | arg... 可以是一个参数，也可以是多个参数，但它们都只用于构造一个对象，并在栈顶直接生成该对象，作为新的栈顶元素。                  |
| swap (stack&lt; T&gt; &amp; other_stack) | 将两个 stack 适配器中的元素进行互换，需要注意的是，进行互换的 2 个 stack 适配器中存储的元素类型以及底层采用的基础容器类型，都必须相同。 |

下面这个例子中演示了表中部分成员函数的用法：

```
std::list<int> values{ 1, 2, 3 };
std::stack<int, std::list<int>> my_stack 1 (values);
std::stack<int, std::list<int>> my_stack=my_stack 1;
//std::stack<int, std::list<int>> my_stack (my_stack 1);
```

运行结果为：
```
size of my_stack: 3  
3  
2  
1
```

## stack 应用——实现一个计算器

前面章节中，已经对 stack 容器适配器及其用法做了详细的讲解。本节将利用 stack 适配器实现一个简单的计算机程序，此计算机支持基本的加（`+`）、 减（` - `）、乘（`*`）、除（`/`）、幂（`^`）运算。

这里，先给大家展示出完整的实现代码，读者可先自行思考该程序的实现流程。当然，后续也会详细的讲解：

```cpp
#include <iostream>
#include <cmath>       // pow()
#include <stack>       // stack<T>
#include <algorithm>   // remove()
#include <stdexcept>   // runtime_error
#include <string>      // string

using std::string;

inline size_t precedence(const char op) {
    if (op == '+' || op == '-')
        return 1;
    if (op == '*' || op == '/')
        return 2;
    if (op == '^')
        return 3;
    throw std::runtime_error{string{"表达中包含无效的运算符"} + op};
}


double execute(std::stack<char> &ops, std::stack<double> &operands) {
    double result{};
    double rhs{operands.top()};
    operands.pop();
    double lhs{operands.top()};
    operands.pop();
    switch (ops.top()) {
        case '+':
            result = lhs + rhs;
            break;
        case '-':
            result = lhs - rhs;
            break;
        case '*':
            result = lhs * rhs;
            break;
        case '/':
            result = lhs / rhs;
            break;
        case '^':
            result = std::pow(lhs, rhs);
            break;
        default:
            throw std::runtime_error{string{"invalid operator: "} + ops.top()};
    }
    ops.pop();
    operands.push(result);
    return result;
}

int main() {
    std::stack<double> operands;
    std::stack<char> operators;
    string exp;

    try {
        while (true) {
            std::cout << "输入表达式(按Enter结束):" << std::endl;
            std::getline(std::cin, exp, '\n');
            if (exp.empty()) break;
            exp.erase(std::remove(std::begin(exp), std::end(exp), ' '), std::end(exp));
            size_t index{};
            operands.push(std::stod(exp, &index));
            std::cout << index << std::endl;
            while (true) {
                operators.push(exp[index++]);
                size_t i{};
                operands.push(std::stod(exp.substr(index), &i));
                index += i;
                if (index == exp.length()) {
                    while (!operators.empty())
                        execute(operators, operands);
                    break;
                }
                while (!operators.empty() && precedence(exp[index]) <= precedence(operators.top()))
                    execute(operators, operands);
            }
            std::cout << "result = " << operands.top() << std::endl;
        }
    }
    catch (const std::exception &e) {
        std::cerr << e.what() << std::endl;
    }

    std::cout << "计算结束" << std::endl;
    return 0;
}
```

下面是一些示例输出：
```
输入表达式 (按 Enter 结束):  
5*2-3  
result = 7  
输入表达式 (按 Enter 结束):  
4+4*2  
result = 12  
输入表达式 (按 Enter 结束):↙   <-- 键入 Enter

计算结束
```

### 计算器程序的实现流程
----------

了解一个程序的功能，通常是从 main () 函数开始。因此，下面从 main () 函数开始，给大家讲解程序的整个实现过程。

首先，我们创建 2 个 stack 适配器，operands 负责将表达式中的运算符逐个压栈，operators 负责将表达式的数值逐个压栈，同时还需要一个 string 类型的 exp，用于接收用户输入的表达式。

正如上面代码中所有看到的，所有的实现代码都包含在一个由 try 代码块包裹着的 while 循环中，这样既可以实现用户可以多次输入表达式的功能（当输入的表达式为一个空字符串时，循环结束），还可以捕获程序运行过程中抛出的任何异常（在 catch 代码块中，调用异常对象的成员函数 what () 会将错误信息输出到标准错误流中）。

当用户输入完要计算的表达式之后，由于整个表达式是以字符串的形式接收的，考虑到字符串中可能掺杂空格，影响后续对字符串的处理，因此又必须借助 remove () 函数来移除输入表达式中的多余空格（第 70 行代码处）。

得到统一格式的表达式之后，接下来才是实现计算功能的核心，其实现思路为：
1) 因为所有的运算符都需要两个操作数，所以有效的输入表达式格式为 “操作数 运算符 操作数 运算符 操作数...”，即序列的第一个和最后一个元素肯定都是操作数，每对操作数之间有一个运算符。由于有效表达式总是以操作数开头，所以第一个操作数在分析表达式的嵌套循环之前被提取出来。
2) 在循环中，输入字符串的运算符会被压入 operators 栈。在确认没有到达字符串末尾后，再从 exp 提取第二个操作数。这时 stod () 的第一个参数是从 index 开始的 exp 字符串，它是被压入 operators 栈的运算符后的所有字符。此时字符串中第一个运算符的索引为 i，因为 i 是相对于 index 的，所以我们会将 index 加上 i 的值，使它指向操作数后的一个运算符（如果是 exp 中的最后一个操作数，它会指向字符串末尾的下一个位置）。
3) 当 index 的值超过 exp 的最后一个字符时，会执行 operators 容器中剩下的运算符。如果没有到达字符串末尾，operators 容器也不为空，我们会比较 operators 栈顶运算符和 exp 中下一个运算符的优先级。如果栈顶运算符的优先级高于下一个运算符，就先执行栈顶的运算符。否则，就不执行栈顶运算符，在下一次循环开始时，将下一个运算符压入 operators 栈。通过这种方式，就可以正确计算出带优先级的表达式的值。

以 `5-2*3+1` 为例，以上程序的计算过程如下：
1) 取  5 和 2 进 operands 栈容器，同时它们之间的 - 运算符进 operators 栈容器，判断后续是否还有表达式，显然还有`*3+1`，这种情况下，取 operators 栈顶运算符 - 和后续的 * 运算符做优先级比较，由于 * 的优先级更高，此时继续将后续的 * 和 3 分别进栈；

> 此时，operands 中从栈顶依次存储的是 3、2、5，operators 容器中从栈顶依次存储的是 `*`、`-`。

2) 继续判断后续是否还有表达式，由于还有 “+1”，则取 operators 栈顶运算符 * 和 + 运算符做优先级比较，显然前者的优先级更高，此时将 operands 栈顶的 2 个元素（2 和 3）取出并弹栈，同时将 operators 栈顶元素（`*`）取出并弹栈，计算它们组成的表达式 `2*3`，并将计算结果再入 operands 栈。

> 计算到这里，operands 中从栈顶依次存储的是 6、5，operators 中从栈顶依次存储的是 -。

3) 由于 operator 容器不空，因此继续取新的栈顶运算符 “-” 和“+”做优先级比较，由于它们的优先级是相同的，因为继续将 operands 栈顶的 2 个元素（5 和 6）取出并弹栈，同时将 operators 栈顶元素（-） 取出并弹栈，计算它们组成的表达式“5-6”，并将计算结果 -1 再入 operands 栈。

> 此时，operands 中从栈顶依次存储的是 -1，operator 为空。

4）由于此时 operator 栈为空，因此将后续 “+1” 表达式中的 1 和 + 分别进栈。由于后续再无其他表达式，此时就可以直接取 operands 位于栈顶的 2 个元素（-1 和 1），和 operator 的栈顶运算符（+），执行 -1+1 运算，并将计算结果再入 operands 栈。

通过以上几步，最终 `5-2*3+1` 的计算结果 0 位于 operands 的栈顶。