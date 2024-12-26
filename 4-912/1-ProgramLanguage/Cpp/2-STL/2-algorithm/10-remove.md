
如果不知道具体的场景，即元素保存在什么样的容器中，是不能从序列中移除元素的。因此，“移除元素的”算法也无法做到这一点，它们只会重写被选择的元素或者忽略复制的元素。移除操作不会改变被 “移除” 元素的序列的元素个数。

有 4 种移除算法：
* remove () 可以从它的前两个正向迭代器参数指定的序列中移除和第三个参数相等的对象。基本上每个元素都是通过用它后面的元素覆盖它来实现移除的。它会返回一个指向新的最后一个元素之后的位置的迭代器。
* remove_copy () 可以将前两个正向迭代器参数指定的序列中的元素复制到第三个参数指定的目的序列中，并忽略和第 4 个参数相等的元素。它返回一个指向最后一个被复制到目的序列的元素的后一个位置的迭代器。序列不能是重叠的。
* remove_if () 可以从前两个正向迭代器指定的序列中移除能够使作为第三个参数的谓词返回 true 的元素。
* remove_copy_if () 可以将前两个正向迭代器参数指定的序列中，能够使作为第 4 个参数的谓词返回 true 的元素，复制到第三个参数指定的目的序列中。它返回一个指向最后一个被复制到目的序列的元素的后一个位置的迭代器。序列不能是重叠的。

## remove ()
可以按如下方式使用 remove ():
```
std::deque<double> samples {1.5, 2.6, 0.0, 3.1, 0.0, 0.0, 4.1, 0.0, 6.7, 0.0};
samples.erase(std::remove(std::begin(samples), std::end(samples), 0.0), std::end(samples));
std::copy(std::begin(samples),std::end(samples), std::ostream iterator <double> {std::cout," "});
std::cout << std::endl;
// 1.5 2.6 3.1 4.1 6.7
```

sample 中不应包含为 0 的物理测量值。remove () 算法会通过左移其他元素来覆盖它们，通过这种方式就可以消除杂乱分布的 0。remove () 返回的迭代器指向通过这个操作得到的新序列的尾部，所以可以用它作为被删除序列的开始迭代器来调用 samples 的成员函数 erase ()。注释说明容器中的元素没有被改变。

## remove_copy ()
如果想保留原始序列，并生成一个移除选定元素之后的副本，可以使用 remove_copy ()。 例如：
```
std::deque<double> samples {1.5, 2.6, 0.0, 3.1, 0.0, 0.0, 4.1, 0.0, 6.7, 0.0}; std::vector<double> edited_samples;
std::remove_copy(std::begin(samples), std::end(samples), std::back_inserter(edited_samples), 0.0);
```

samples 容器中的非零元素会被复制到 edited_samples 容器中，edited_samples 正好是一个 vector 容器。通过 back_insert_iterator 对象将这些元素添加到 edited_samples，因此这个容器只包含从 sample 中复制的元素。

## remove_if ()
remove_if () 提供了更强大的能力，它能够从序列中移除和给定值匹配的元素。谓词会决定一个元素是否被移除；它接受序列中的一个元素为参数，并返回一个布尔值。例如：
```
using Name = std::pair<string, string>； // First and second name
std::set<Name> blacklist {Name {"Al", "Bedo"}, Name {"Ann", "Ounce"}, Name {"Jo","King"}};
std::deque<Name> candidates {Name{"Stan", "Down"}, Name {"Al", "Bedo"}, Name {"Dan", "Druff"},Name {"Di", "Gress"}, Name {"Ann", "Ounce"}, Name {"Bea", "Gone"}}; candidates.erase(std::remove_if(std::begin(candidates), std::end(candidates),[&blacklist](const Name& name) { return blacklist.count(name); }), std::end(candidates)); std::for_each(std::begin(candidates), std::end(candidates), [] (const Name& name){std::cout << '"' << name.first << " " << name.second << "\" ";});
std::cout << std::endl;  // "Stan Down" "Dan Druff" "Di Gress" "Bea Gone"
```

这段代码用来模拟候选人申请成为倶乐部会员。那些众所周知的不安分人士的姓名被保存在 blacklist 中，它是一个集合。当前申请成为会员的候选人被保存在 candidates 容器中，它是一个 deque 容器。用 remove_if () 算法来保证不会有 blacklist 中的姓名通过甄选过程。这里的谓词是一个以引用的方式捕获 blacklist 容器的 lambda 表达式。当参数在容器中存在时，set 容器的成员函数 count () 会返回 1。谓词返回的值会被隐式转换为布尔值，因此对于每一个出现在 blacklist 中的候选人，谓词都会返回 true，然后会将它们从 candidates 中移除。注释中显示了通过甄选的候选人。

## remove_copy_if ()
remove_copy_if () 之于 remove_copy ()，就像 remove_if () 之于 remove。下面展示它是如何工作的：
```
std::set<Name> blacklist {Name {"Al", "Bedo"}, Name {"Ann", "Ounce"}, Name {"Jo", ,"King" } };
std::deque<Name> candidates {Name {"Stan", "Down"}, Name { "Al", "Bedo"},Name {"Dan", "Druff"}, Name {"Di", "Gress"}, Name {"Ann", "Ounce"},Name {"Bea", "Gone"}};
std::deque<Name> validated;
std::remove_copy_if(std::begin(candidates) , std::end(candidates), std::back inserter(validated), [&blacklist] (const Name& name) { return blacklist.count(name); });
```

这段代码实现了和前一段代码同样的功能，除了结果被保存在 validated 容器中和没有修改 candidates 容器之外。

