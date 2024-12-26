## copy_n ()

copy_n () 算法可以从源容器复制指定个数的元素到目的容器中。第一个参数是指向第一个源元素的输入迭代器，第二个参数是需要复制的元素的个数，第三个参数是指向目的容器的第一个位置的迭代器。这个算法会返回一个指向最后一个被复制元素的后一个位置的迭代器，或者只是第三个参数——输出迭代器——如果第二个参数为 0。

下面是一个使用它的示例：
```
std::vector<string> names {"A1","Beth", "Carol", "Dan", "Eve","Fred","George" ,"Harry", "Iain", "Joe"};
std::unordered_set<string> more_names {"Janet", "John"};
std::copy_n(std:rbegin(names)+1, 3, std::inserter(more_names, std::begin(more_names)));
```
这个 copy_n () 操作会从 names 的第二个元素开始复制 3 个元素到关联容器 more_names 中。目的容器是由一个 unordered_set 容器的 insert_iterator 对象指定的，它是由 inserter () 函数模板生成的。insert_iterator 对象会调用容器的成员函数 insert () 来向容器中添加元素。

当然，copy_n () 的目的地址也可是以流迭代器：
```
std::copy_n(std::begin(more_names), more_names.size()-1,std::ostream_iterator<string> {std::cout, " "});
```
这样会输出 more_names 中除了最后一个元素之外的全部元素。注意，如果被复制元素的个数超过了实际元素的个数，程序会因此崩溃。如果元素的个数为 0 或负数，copy_n () 算法什么也不做。

## copy_if ()

copy_if () 算法可以**从源序列复制使谓词返回 true 的元素，所以可以把它看作一个过滤器**。前两个参数定义源序列的输入迭代器，第三个参数是指向目的序列的第一个位置的输出迭代器，第 4 个参数是一个谓词。会返回一个输出迭代器，它指向最后一个被复制元素的下一个位置。

下面是一个使用 copy_if () 的示例：
```
std::vector<string> names {"A1", "Beth", "Carol", "Dan", "Eve","Fred", "George", "Harry", "Iain", "Joe"};
std::unordered_set<string> more_names {"Jean", "John"};
size_t max_length{4};
std::copy_if(std::begin(names), std::end(names), std::inserter(more_names, std::begin(more_names)), [max_length](const string& s) { return s.length() <= max_length;});
```
因为作为第 4 个参数的 lambda 表达式所添加的条件，这里的 copy_if () 操作只会复制 names 中的 4 个字符串或更少。目的容器是一个 unordered_set 容器 more_names，它已经包含两个含有 4 个字符的名称。和前面的章节一样，insert_itemtor 会将元素添加到限定的关联容器中。如果想要展示它是如何工作的，可以用 copy () 算法列出 more_names 的内容：

```
std::copy(std::begin(more_names), std::end(more_names), std::ostream iterator <string>{std::cout, " "});
std::cout << std::endl;
```

当然，copy_if () 的目的容器也可以是一个流迭代器：
```
std::vector<string> names { "Al", "Beth", "Carol", "Dan", "Eve","Fred", "George", "Harry", "Iain", "Joe"};
size_t max_length{4};
std::copy_if(std::begin(names), std::end(names), std::ostream iterator< string> {std::cout," "}, [max_length](const string& s) { return s.length() > max_length; });
std::cout << std::endl;
```
这里会将 names 容器中包含的含有 4 个以上字符的名称写到标准输出流中。这段代码会输出如下内容：
```
Carol George Harry
```

输入流迭代器可以作为 copy_if () 算法的源，也可以将它用在其他需要输入迭代器的算法上。例如：
```
std::unordered_set<string> names;
size_t max_length {4};
std::cout << "Enter names of less than 5 letters. Enter Ctrl+Z on a separate line to end:\n";
std::copy_if(std::istream_iterator<string>{std::cin},std:: istream iterator<string>{}, std::inserter(names, std::begin (names)),[max_length](const string& s) { return s.length() <= max_length; });
std::copy(std::begin(names), std::end(names), std::ostream_iterator <string>{std::cout," "});
std::cout << std::endl;
```
容器 names 最初是一个空的 unordered_set。只有当从标准输入流读取的姓名的长度小于或等于 4 个字符时，copy_if () 算法才会复制它们。执行这段代码可能会产生如下输出：
```
Enter names of less than 5 letters. Enter Ctrl+Z on a separate line to end:  
Jim Bethany Jean Al Algernon Bill Adwina Ella Frederick Don ^Z  
Ella Jim Jean Al Bill Don
```
超过 5 个字母的姓名可以从 cin 读入，但是被忽略掉，因为在这种情况下第 4 个参数 的判定会返回 false。因此，输入的 10 个姓名里面只有 6 个会被存储在容器中。

## copy_backward ()

不要被 copy_backward () 算法的名称所误导，它不会逆转元素的顺序。它只会像 copy () 那样复制元素，但是从**最后一个元素开始直到第一个元素**。

copy_backward () 会复制前两个迭代器参数指定的序列。**第三个参数是目的序列的结束迭代器，通过将源序列中的最后一个元素复制到目的序列的结束迭代器之前，源序列会被复制到目的序列中**，如图所示。copy_backward () 的 3 个参数都必须是可以自增或自减的双向迭代器，这意味着这个算法只能应用到序列容器的序列上。

![[copy_backward.png]]

上图说明了源序列 from 的最后一个元素是如何先被复制到目的序列 to 的最后一个元素的。从源序列的反向，将每一个元素依次复制到目的序列的前一个元素之前的位置。在进行这个操作之前，目的序列中的元素必须存在，因此目的序列至少要有和源序列一样多的元素，但也可以有更多。**copy_backward () 算法会返回一个指向最后一个被复制元素的迭代器，在目的序列的新位置，它是一个开始迭代器**。

我们可能会好奇，相对于普通的从第一个元素开始复制的 copy () 算法，copy_backward () 提供了哪些优势。

一个回答是，在序列重叠时，可以用 copy () 将元素复制到重叠的目的序列剩下的位置——也就是目的序列第一个元素之前的位置。如果想尝试用 copy () 算法将元素复制到同一个序列的右边，这个操作不会成功，因为被复制的元素在复制之前会被重写。如果想将它们复制到右边，可以使用 copy_backward ()，只要目的序列的结束迭代器在源序列的结束迭代器的右边。

下图说明了在将元素复制到重叠的序列的右边时，这两个算法的不同。
![[copy-vs-copy_backward.png]]
上图展示了在序列右边的前三个位置运用 copy () 和 copy_backward () 算法的结果。在想将元素复制到右边时，copy () 算法显然不能如我们所愿，因为一些元素在复制之前会被重写。在这种情况下，copy_backward () 可以做到我们想做的事。相反在需要将元素复制到 序列的左边时，copy () 可以做到，但 copy_backward () 做不到。

下面是一个说明 copy_backward () 用法的示例：
```
std::deque<string> song{ "jingle", "bells"，"jingle", "all", "the", "way"};
song.resize(song.size()+2); // Add 2 elements
std::copy_backward(std::begin(song), std::begin(song)+6, std::end(song));
std::copy(std::begin(song), std::end(song), std::ostream iterator <string> {std::cout, " "});
std::cout << std::endl;
```
为了能够在右边进行序列的反向复制操作，需要添加一些额外的元素，可以通过使用 deque 的成员函数 resize () 来增加 deque 容器的元素个数。copy_backward () 算法会将原有的元素复制到向右的两个位置，保持前两个元素不变，所以这段代码的输出如下：
```
jingle bells jingle bells jingle all the way
```

## reverse_copy ()

reverse_copy () 算法可以**将源序列复制到目的序列中，目的序列中的元素是逆序的**。定义源序列的前两个迭代器参数必须是双向迭代器。目的序列由第三个参数指定，它是目的序列的开始迭代器，也是一个输出迭代器。如果序列是重叠的，函数的行为是未定义的。这个算法会返回一个输出迭代器，它指向目的序列最后一个元素的下一个位置。

下面是一个使用 reverse_copy () 和 copy_if () 的示例：
```
// Testing for palindromes using reverse_copy()
#include <iostream>                                      // For standard streams
#include <iterator>                                      // For stream iterators and begin() and end()
#include <algorithm>                                     // For reverse_copy() and copy_if()
#include <cctype>                                        // For toupper() and isalpha()
#include <string>
using std::string;

int main()
{
    while(true)
    {
        string sentence;
        std::cout << "Enter a sentence or Ctrl+Z to end: ";
        std::getline(std::cin, sentence);
        if(std::cin.eof()) break;

        // Copy as long as the characters are alphabetic & convert to upper case
        string only_letters;
        std::copy_if(std::begin(sentence), std::end(sentence), std::back_inserter(only_letters),[](char ch) { return std::isalpha(ch); });
        std::for_each(std::begin(only_letters), std::end(only_letters), [](char& ch) { ch = toupper(ch); });

        // Make a reversed copy
        string reversed;
        std::reverse_copy(std::begin(only_letters), std::end(only_letters), std::back_inserter(reversed));
        std::cout << '"' << sentence << '"'<< (only_letters == reversed ? " is" : " is not") << " a palindrome." << std::endl;
    }
}
```
这个程序会检查一条语句 (也可以是很多条语句) 是否是回文的。回文语句是指正着读或反着读都相同的句子，前提是忽略一些像空格或标点这样的细节。while 使我们可以检查尽可能多的句子。用 getline () 读一条句子到 sentence 中。如果读到 `Ctrl+Z`，输入流中会设置 1 个 EOF 标志，它会结束循环。用 copy_if () 将 sentence 中的字母复制到 only_letters。这个 lambda 表达式只在参数是学母时返回 true，所以其他的任何字符都会被忽略。然后用 back_inserter () 生成的 back_insert_iterator 对象将这些字符追加到 only_letter。

for_each () 算法会将三个参数指定的函数对象应用到前两个参数定义的序列的元素上，因此这里会将 only_letters 中的字符全部转换为大写。然后用 reverse_copy () 算法生成和 only_letters 的内容相反的内容。比较 only_letters 和 reversed 来判断输入的语句是否为回文。

该程序的输出结果为：
```
Enter a sentence or Ctrl+Z to end: Lid off a daffodil.  
"Lid off a daffodil." is a palindrome.  
Enter a sentence or Ctrl+Z to end: Engaga le jeu que je le gagne.  
"Engaga le jeu que je le gagne." is not a palindrome.  
Enter a sentence or Ctrl+Z to end: ^Z
```

reverse () 算法可以在原地逆序它的两个双向迭代器参数所指定序列的元素。可以如下 所示用它来代替上述程序中的 reverse_copy ():
```
string reversed {only_letters};
std::reverse(std::begin(reversed), std::end(reversed));
```

这两条语句会替换上述程序中 reversed 的定义和 reverse_copy () 调用。它们生成一个 only_letters 的副本 reversed，然后调用 reverse () 原地逆序 reversed 中的字符序列。