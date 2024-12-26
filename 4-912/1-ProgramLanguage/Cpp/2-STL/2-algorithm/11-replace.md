> replace () 算法会用新的值来替换和给定值相匹配的元素。

replace () 算法会用新的值来替换和给定值相匹配的元素。它的前两个参数是被处理序列的正向迭代器，第 3 个参数是被替换的值，第 4 个参数是新的值。下面展示了它的用法:

```
std::deque<int> data {10, -5, 12, -6, 10, 8, -7, 10, 11};
std::replace(std::begin(data), std::end(data), 10, 99);
// Result: 99 -5 12 -6 99 8 -7 99 11
```

这里，data 容器中和 10 匹配的全部元素都会被 99 替代。

replace_if () 会将使谓词返回 true 的元素替换为新的值。它的第 3 个参数是一个谓词，第 4 个参数是新的值。参数的类型一般是元素类型的 const 引用；const 不是强制性的，但谓词不应该改变元素。下面是一个使用 replace_if () 的示例：

```
string password { "This is a good choice !"};
std::replace_if(std::begin(password), std::end(password),[](char ch){return std::isspace(ch);}, '_');
//Result:This_is_a_good_choice!
```

这个谓词会为任何是空格字符的元素返回 true，因此这里的空格都会被下划线代替。

replace_copy () 算法和 replace () 做的事是一样的，但它的结果会被保存到另一个序列中，而不会改变原始序列。它的前两个参数是输入序列的正向迭代器，第 3 个参数是输入序列的开始迭代器，最后两个参数分别是要被替换的值和替换值。例如：

```
std::vector<string> words { "one","none", "two", "three", "none", "four"};
std::vector<string> new_words;
std::replace_copy (std::begin (words), std::end(words), std::back_inserter (new_words), string{"none"}, string{"0"});
// Result:"one", "0", "two","three","0","four"
```

在执行这段代码后，new_words 会包含注释中的 string 元素。

可以在序列中有选择地替换元素的最后一个算法是 replace_copy_if ()，它和 replace_if () 算法是相同的，但它的结果会被保存到另一个序列中。它的前两个参数是输入序列的迭代器，第 3 个参数是输出序列的开始迭代器，最后两个参数分别是谓词和替换值。例如：

```
std::deque<int> data {10, -5, 12, -6, 10, 8, -7, 10，11}; std::vector<int> data_copy;
std::replace_copy_if(std::begin(data), std::end(data),std::back_inserter(data_copy),[](int value) {return value == 10;}, 99);
// Result:99 -5 12 -6 99 8 -7 99 11
```

data_copy 是一个 vector 容器，这里使用它只是为了说明输出容器可以和输入容器不同。这段代码执行后，它会包含注释中所示的元素。