
有些场景中，我们需要将 2 个有序序列合并为 1 个有序序列，这时就可以借助 merge () 或者 inplace_merge () 函数实现。

值得一提的是，merge () 和 inplace_merge () 函数都定义在 `<algorithm>` 头文件中，因此在使用它们之前，程序中必须提前引入该头文件：

```
#include <algorithm> 
```

## merge () 函数
--------------

merge () 函数用于将 2 个有序序列合并为 1 个有序序列，**前提是这 2 个有序序列的排序规则相同**（要么都是升序，要么都是降序）。并且最终借助该函数获得的新有序序列，其排序规则也和这 2 个有序序列相同。

举个例子，假设有 2 个序列，分别为 `5,10,15,20,25` 和 `7,14,21,28,35,42`，显然它们不仅有序，而且都是升序序列。因此借助 merge () 函数，我们就可以轻松获得如下这个有序序列：
```
5 7 10 15 17 20 25 27 37 47 57
```

可以看到，该序列不仅包含以上 2 个序列中所有的元素，并且其本身也是一个升序序列。

值得一提的是，C++ STL 标准库的开发人员考虑到用户可能需要自定义排序规则，因此为 merge () 函数设计了以下 2 种语法格式：
```
//以默认的升序排序作为排序规则
OutputIterator merge (InputIterator1 first1, 
					  InputIterator1 last1,
                      InputIterator2 first2,
                      InputIterator2 last2,
                      OutputIterator result);

//以自定义的 comp 规则作为排序规则
OutputIterator merge (InputIterator1 first1, 
					  InputIterator1 last1,
                      InputIterator2 first2, 
	                  InputIterator2 last2,
                      OutputIterator result,
                      Compare comp);
```

可以看到，first 1、last 1、first 2 以及 last 2 都为输入迭代器，\[first 1, last 1\) 和 \[first 2, last 2\) 各用来指定一个有序序列；result 为输出迭代器，用于为最终生成的新有序序列指定存储位置；comp 用于自定义排序规则。同时，该函数会返回一个输出迭代器，其指向的是新有序序列中最后一个元素之后的位置。

> 注意，当采用第一种语法格式时，\[first 1, last 1\) 和 \[first 2, last 2\) 指定区域内的元素必须支持 < 小于运算符；同样当采用第二种语法格式时，\[first 1, last 1\) 和 \[first 2, last 2\) 指定区域内的元素必须支持 comp 排序规则内的比较运算符。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::merge
#include <vector>       // std::vector
using namespace std;
int main() {
    //first 和 second 数组中各存有 1 个有序序列
    int first[] = { 5,10,15,20,25 };
    int second[] = { 7,17,27,37,47,57 };
    //用于存储新的有序序列
    vector<int> myvector(11);
    //将 [first,first+5) 和 [second,second+6) 合并为 1 个有序序列，并存储到 myvector 容器中。
    merge(first, first + 5, second, second + 6, myvector.begin());
    //输出 myvector 容器中存储的元素
    for (vector<int>::iterator it = myvector.begin(); it != myvector.end(); ++it) {
        cout << *it << ' ';
    }   
    return 0;
}
```

程序执行结果为：
```
5 7 10 15 17 20 25 27 37 47 57
```

可以看到，first 数组和 second 数组中各存有 1 个升序序列，通过借助 merge () 函数，我们成功地将它们合并成了一个有序序列，并存储到 myvector 容器中。

> 注意，merge () 函数底层是通过拷贝的方式实现合并操作的。换句话说，上面程序在采用 merge () 函数实现合并操作的同时，并不会对 first 和 second 数组有任何影响。有关该函数的具体实现过程，可查看 [C++ STL merge() 官网](http://www.cplusplus.com/reference/algorithm/merge/)。

实际上，对于 2 个有序序列是各自存储（像 first 和 second 这样）还是存储到一起，merge () 函数并不关心，只需要给它传入恰当的迭代器（或指针），该函数就可以正常工作。因此，我们还可以将上面程序改写为：
```
//该数组中存储有 2 个有序序列
int first[] = { 5,10,15,20,25,7,17,27,37,47,57 };
//用于存储新的有序序列
vector<int> myvector(11);
//将 [first,first+5) 和 [first+5,first+11) 合并为 1 个有序序列，并存储到 myvector 容器中。
merge(first, first + 5,  first + 5, first +11 , myvector.begin());
```

可以看到，2 个有序序列全部存储到了 first 数组中，但只要给 merge () 函数传入正确的指针，仍可以将它们合并为 1 个有序序列。

## inplace_merge () 函数
----------------------

事实上，当 2 个有序序列存储在同一个数组或容器中时，如果想将它们合并为 1 个有序序列，除了使用 merge () 函数，更推荐使用 inplace_merge () 函数。

和 merge () 函数相比，inplace_merge () 函数的语法格式要简单很多：
```
//默认采用升序的排序规则
void inplace_merge (BidirectionalIterator first, 
					BidirectionalIterator middle,
                    BidirectionalIterator last);

//采用自定义的 comp 排序规则
void inplace_merge (BidirectionalIterator first, 
					BidirectionalIterator middle,
                    BidirectionalIterator last, 
	                Compare comp);
```

其中，first、middle 和 last 都为双向迭代器，\[first, middle\) 和 \[middle, last\) 各表示一个有序序列。

和 merge () 函数一样，inplace_merge () 函数也要求 \[first, middle\) 和 \[middle, last\) 指定的这 2 个序列必须遵循相同的排序规则，且当采用第一种语法格式时，这 2 个序列中的元素必须支持 < 小于运算符；同样，当采用第二种语法格式时，这 2 个序列中的元素必须支持 comp 排序规则内部的比较运算符。

不同之处在于，merge () 函数会将最终合并的有序序列存储在其它数组或容器中，而 inplace_merge () 函数则将最终合并的有序序列存储在 \[first, last\) 区域中。

举个例子：
```
#include <iostream>     
#include <algorithm>    
using namespace std;
int main() {
    
    int first[] = { 5,10,15,20,25,7,17,27,37,47,57 };
    
    inplace_merge(first, first + 5,first +11);
    for (int i = 0; i < 11; i++) {
        cout << first[i] << " ";
    }
    return 0;
}

```

程序执行结果为：
```
5 7 10 15 17 20 25 27 37 47 57
```
可以看到，first 数组中包含 2 个升序序列，借助 inplace_merge () 函数，实现了将这 2 个序列合并为 1 个升序序列，且新序列仍存储在 first 数组中。