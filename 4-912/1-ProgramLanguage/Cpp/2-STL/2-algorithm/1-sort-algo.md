
C++ STL 标准库提供有很多实用的排序函数，如下表所示。通过调用它们，我们可以很轻松地实现对普通数组或者容器中指定范围内的元素进行排序。

| 函数名                                                     | 用法                                                                                                                                                                                                |
| ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| sort (first, last)                                         | 对容器或普通数组中 \[first, last) 范围内的元素进行排序，默认进行升序排序。                                                                                                                          |
| stable_sort (first, last)                                  | 和 sort () 函数功能相似，不同之处在于，对于 \[first, last) 范围内值相同的元素，该函数不会改变它们的相对位置。                                                                                       |
| partial_sort (first, middle, last)                         | 从 \[first, last) 范围内，筛选出 middle-first 个最小的元素并排序存放在 \[first，middle) 区间中。                                                                                                    |
| partial_sort_copy (first, last, result_first, result_last) | 从 \[first, last) 范围内筛选出 result_last-result_first 个元素排序并存储到 \[result_first, result_last) 指定的范围中。                                                                              |
| is_sorted (first, last)                                    | 检测 \[first, last) 范围内是否已经排好序，默认检测是否按升序排序。                                                                                                                                  |
| is_sorted_until (first, last)                              | 和 is_sorted () 函数功能类似，唯一的区别在于，如果 \[first, last) 范围的元素没有排好序，则该函数会返回一个指向首个不遵循排序规则的元素的迭代器。                                                    |
| void nth_element (first, nth, last)                        | 找到 \[first, last) 范围内按照排序规则（默认按照升序排序）应该位于第 nth 个位置处的元素，并将其放置到此位置。同时使该位置左侧的所有元素都比其存放的元素小，该位置右侧的所有元素都比其存放的元素大。 |

对于表中罗列的这些函数，本教程会一一进行讲解，这里先介绍 sort () 函数。

## sort () 排序函数
---------------

C++ STL 标准库中的 sort ()  函数，本质就是一个模板函数。正如表中描述的，该函数**专门用来对容器或普通数组中指定范围内的元素进行排序**，排序规则默认以元素值的大小做升序排序，除此之外我们也可以选择标准库提供的其它排序规则（比如 `std::greater<T>` 降序排序规则），甚至还可以自定义排序规则。

> sort () 函数是基于快速排序实现的。

需要注意的是，sort () 函数受到底层实现方式的限制，它仅适用于普通数组和部分类型的容器。换句话说，只有普通数组和具备以下条件的容器，才能使用 sort () 函数：
1. 容器支持的迭代器类型必须为**随机访问迭代器**。这意味着，sort () 只对 **array**、**vector**、**deque** 这 3 个容器提供支持。
2. 如果对容器中指定区域的元素做默认升序排序，则元素类型必须支持 `<` 小于运算符；同样，如果选用标准库提供的其它排序规则，元素类型也必须支持该规则底层实现所用的比较运算符；
3. sort () 函数在实现排序时，需要交换容器中元素的存储位置。这种情况下，如果容器中存储的是自定义的类对象，则该类的内部必须提供移动构造函数和移动赋值运算符。

另外还需要注意的一点是，对于指定区域内值相等的元素，sort () 函数**无法保证**它们的相对位置不发生改变——即 sort()函数是不稳定的。例如，有如下一组数据：

```
2a 1 2b 3 2c
```
可以看到，该组数据中包含多个值为 2 的元素，此时如果使用 sort () 函数进行排序，则值为 2 的这 3 个元素的相对位置可能会发生改变，比如排序结果为：
```
1 2b 2a 2c 3
```

> 实际场景中，如果需要保证值相等元素的相对位置不发生改变，可以选用 stable_sort () 排序函数。有关该函数的具体用法，后续章节会做详细讲解。

值得一提的是，sort () 函数位于 `<algorithm>` 头文件中，因此在使用该函数前，程序中应包含如下语句：
```
#include <algorithm>
```

sort () 函数有 2 种用法，其语法格式分别为：
```
//对 [first, last) 区域内的元素做默认的升序排序
void sort (RandomAccessIterator first, RandomAccessIterator last);
//按照指定的 comp 排序规则，对 [first, last) 区域内的元素进行排序
void sort (RandomAccessIterator first, RandomAccessIterator last, Compare comp);
```

其中，first 和 last 都为随机访问迭代器，它们的组合 \[first, last\) 用来指定要排序的目标区域；另外在第 2 种格式中，comp 可以是 C++ STL 标准库提供的排序规则（比如 std:: greater\<T\>），也可以是自定义的排序规则。

> 关于如何自定义一个排序规则，除了《[[如何自定义STL关联式容器的排序规则？]]》一节介绍的 2 种方式外，还可以直接定义一个具有 2 个参数并返回 bool 类型值的函数作为排序规则。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::sort
#include <vector>       // std::vector
//以普通函数的方式实现自定义排序规则
bool mycomp (int i, int j) {
    return (i < j);
}
//以函数对象的方式实现自定义排序规则
class mycomp 2 {
public:
    bool operator () (int i, int j) {
        return (i < j);
    }
};

int main () {
    std::vector<int> myvector{ 32, 71, 12, 45, 26, 80, 53, 33 };
    //调用第一种语法格式，对 32、71、12、45 进行排序
    std:: sort (myvector.begin (), myvector.begin () + 4); //(12 32 45 71) 26 80 53 33
    //调用第二种语法格式，利用 STL 标准库提供的其它比较规则（比如 greater<T>）进行排序
    std:: sort (myvector.begin (), myvector.begin () + 4, std::greater<int>()); //(71 45 32 12) 26 80 53 33
   
    //调用第二种语法格式，通过自定义比较规则进行排序
    std:: sort (myvector.begin (), myvector.end (), mycomp 2 ());//12 26 32 33 45 53 71 80
    //输出 myvector 容器中的元素
    for (std::vector<int>:: iterator it = myvector.begin (); it != myvector.end (); ++it) {
        std:: cout << *it << ' ';
    }
    return 0;
}
```

程序执行结果为：
```
12 26 32 33 45 53 71 80
```

可以看到，程序中分别以函数和函数对象的方式自定义了具有相同功能的 mycomp 和 mycomp 2 升序排序规则。需要注意的是，和为关联式容器设定排序规则不同，给 sort () 函数指定排序规则时，需要为其传入一个函数名（例如 mycomp ）或者函数对象（例如 std:: greater\<int\>() 或者 mycomp2 ()）。

那么，sort () 函数的效率怎么样呢？该函数实现排序的平均时间复杂度为 $O(nlogn)$（其中 n 为指定区域 \[first, last)\ 中 last 和 first 的距离）。

## stable_sort ()
当指定范围内包含多个相等的元素时，sort () 排序函数无法保证不改变它们的相对位置。那么，如果既要完成排序又要保证相等元素的相对位置，可以使用 stable_sort () 函数。

> 有些场景是需要保证相等元素的相对位置的。例如对于一个保存某种事务（比如银行账户）的容器，在处理这些事务之前，为了能够有序更新这些账户，需要按照账号对它们进行排序。而这时就很有可能出现相等的账号（即同一账号在某段时间做多次的存取钱操作），它们的相对顺序意味着添加到容器的时间顺序，此顺序不能修改，否则很可能出现账户透支的情况。

值得一提的是，stable_sort () 函数完全可以看作是 sort () 函数在功能方面的升级版。换句话说，stable_sort () 和 sort () 具有相同的使用场景，就连语法格式也是相同的（后续会讲），只不过前者在功能上除了可以实现排序，还可以保证不改变相等元素的相对位置。

> stable_sort () 函数是基于归并排序实现的。

和 sort () 函数一样，实现 stable_sort () 的函数模板也位于 `<algorithm>` 头文件中，因此在使用该函数前，程序也应包含如下语句：

```
#include <algorithm>
```

并且，table_sort () 函数的用法也有 2 种，其语法格式和 sort () 函数完全相同（仅函数名不同）：

```
//对 [first, last) 区域内的元素做默认的升序排序
void stable_sort ( RandomAccessIterator first, RandomAccessIterator last );
//按照指定的 comp 排序规则，对 [first, last) 区域内的元素进行排序
void stable_sort ( RandomAccessIterator first, RandomAccessIterator last, Compare comp );
```

其中，first 和 last 都为随机访问迭代器，它们的组合 \[first, last\) 用来指定要排序的目标区域；另外在第 2 种格式中，comp 可以是 C++ STL 标准库提供的排序规则（比如 std:: greater\<T\>），也可以是自定义的排序规则。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::stable_sort
#include <vector>       // std::vector
//以普通函数的方式实现自定义排序规则
bool mycomp (int i, int j) {
    return (i < j);
}
//以函数对象的方式实现自定义排序规则
class mycomp 2 {
public:
    bool operator () (int i, int j) {
        return (i < j);
    }
};

int main () {
    std::vector<int> myvector{ 32, 71, 12, 45, 26, 80, 53, 33 };
    //调用第一种语法格式，对 32、71、12、45 进行排序
    std:: stable_sort (myvector.begin (), myvector.begin () + 4); //(12 32 45 71) 26 80 53 33
    //调用第二种语法格式，利用 STL 标准库提供的其它比较规则（比如 greater<T>）进行排序
    std:: stable_sort (myvector.begin (), myvector.begin () + 4, std::greater<int>()); //(71 45 32 12) 26 80 53 33

    //调用第二种语法格式，通过自定义比较规则进行排序, 这里也可以换成 mycomp 2 ()
    std:: stable_sort (myvector.begin (), myvector.end (), mycomp);//12 26 32 33 45 53 71 80
    //输出 myvector 容器中的元素
    for (std::vector<int>:: iterator it = myvector.begin (); it != myvector.end (); ++it) {
        std:: cout << *it << ' ';
    }
    return 0;
}
```

程序执行结果为：
```
12 26 32 33 45 53 71 80
```
那么，stable_sort () 函数的效率怎么样呢？当可用**空间足够**的情况下，该函数的时间复杂度可达到 $O (nlogn)$；反之，时间复杂度为 $O (nlog(n)^2)$，其中 n 为指定区域 \[first, last\) 中 last 和 first 的距离。

## partial_sort ()

假设这样一种情境，有一个存有 100 万个元素的容器，但我们只想从中提取出值最小的 10 个元素，该如何实现呢？

通过前面的学习，读者可能会想到使用 sort () 或者 stable_sort () 排序函数，即通过对容器中存储的 100 万个元素进行排序，就可以成功筛选出最小的 10 个元素。但仅仅为了提取 10 个元素，却要先对 100 万个元素进行排序，可想而知这种实现方式的效率是非常低的。

对于解决类似的问题，C++ STL 标准库提供了更高效的解决方案，即使用 partial_sort () 或者 partial_sort_copy () 函数，本节就对这 2 个排序函数的功能和用法做详细的讲解。

首先需要说明的是，partial_sort () 和 partial_sort_copy () 函数都位于 \<algorithm\> 头文件中，因此在使用这 2 个函数之前，程序中应引入此头文件：

```
#include <algorithm> 
```

### partial_sort () 
-----------------------

partial_sort () 函数可以从指定区域中提取出部分数据，并对它们进行排序。

但 “部分排序” 仅仅是对 partial_sort () 函数功能的一个概括，如果想彻底搞清楚它的功能，需要结合该函数的语法格式。partial_sort () 函数有 2 种用法，其语法格式分别为：

```
//按照默认的升序排序规则，对 [first, last) 范围的数据进行筛选并排序
void partial_sort (RandomAccessIterator first,
 RandomAccessIterator middle,
 RandomAccessIterator last);
 
//按照 comp 排序规则，对 [first, last) 范围的数据进行筛选并排序
void partial_sort (RandomAccessIterator first,
 RandomAccessIterator middle,
 RandomAccessIterator last,
 Compare comp);
```

其中，first、middle 和 last 都是随机访问迭代器，comp 参数用于自定义排序规则。

partial_sort () 函数会以交换元素存储位置的方式实现部分排序的。具体来说，partial_sort () 会将 \[first, last\) 范围内最小（或最大）的 $middle-first$ 个元素移动到 \[first, middle\) 区域中，并对这部分元素做升序（或降序）排序。

需要注意的是，partial_sort () 函数受到底层实现方式的限制，它**仅适用于普通数组和部分类型的容器**。换句话说，只有普通数组和具备以下条件的容器，才能使用 partial_sort () 函数：
* 容器支持的迭代器类型必须为*随机访问迭代器*。这意味着，partial_sort () 函数只适用于 array、vector、deque 这 3 个容器。
* 当选用默认的升序排序规则时，容器中存储的元素类型必须*支持 < 小于运算符*；同样，如果选用标准库提供的其它排序规则，元素类型也必须支持该规则底层实现所用的比较运算符；
* partial_sort () 函数在实现过程中，需要交换某些元素的存储位置。因此，如果容器中存储的是自定义的类对象，则该类的内部*必须提供移动构造函数和移动赋值运算符*。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::partial_sort
#include <vector>       // std::vector
using namespace std;
//以普通函数的方式自定义排序规则
bool mycomp 1 (int i, int j) {
    return (i > j);
}
//以函数对象的方式自定义排序规则
class mycomp 2 {
public:
    bool operator () (int i, int j) {
        return (i > j);
    }
};

int main () {
    std::vector<int> myvector{ 3,2,5,4,1,6,9,7};

    //以默认的升序排序作为排序规则，将 myvector 中最小的 4 个元素移动到开头位置并排好序
    std:: partial_sort (myvector.begin (), myvector.begin () + 4, myvector.end ());
    cout << "第一次排序:\n";
    for (std::vector<int>:: iterator it = myvector.begin (); it != myvector.end (); ++it)
        std:: cout << *it << ' ';
    cout << "\n 第二次排序:\n";

    // 以指定的 mycomp 2 作为排序规则，将 myvector 中最大的 4 个元素移动到开头位置并排好序
    std:: partial_sort (myvector.begin (), myvector.begin () + 4, myvector.end (), mycomp 2 ());
    for (std::vector<int>:: iterator it = myvector.begin (); it != myvector.end (); ++it)
        std:: cout << *it << ' ';
    return 0;
}
```

程序执行结果为：
```
第一次排序:  
1 2 3 4 5 6 9 7  
第二次排序:  
9 7 6 5 1 2 3 4
```

值得一提的是，partial_sort () 函数实现排序的平均时间复杂度为 $O(nlogm)$，其中 n 指的是 \[first, last\) 范围的长度，m 指的是 \[first, middle\) 范围的长度。

### partial_sort_copy () 排序函数
----------------------------

partial_sort_copy () 函数的功能和 partial_sort () 类似，唯一的区别在于，前者不会对原有数据做任何变动，而是**先将选定的部分元素拷贝到另外指定的数组或容器中，然后再对这部分元素进行排序**。

partial_sort_copy () 函数也有 2 种语法格式，分别为：
```
//默认以升序规则进行部分排序
RandomAccessIterator partial_sort_copy (
    InputIterator first,
    InputIterator last,
	RandomAccessIterator result_first,
    RandomAccessIterator result_last);

//以 comp 规则进行部分排序
RandomAccessIterator partial_sort_copy (
    InputIterator first,
	InputIterator last,
	RandomAccessIterator result_first,
    RandomAccessIterator result_last,
    Compare comp);
```

其中，first 和 last 为输入迭代器；result_first 和 result_last 为随机访问迭代器；comp 用于自定义排序规则。

partial_sort_copy () 函数会将 \[first, last\) 范围内最小（或最大）的 $result\_last-result\_first$ 个元素复制到 \[result_first, result_last\) 区域中，并对该区域的元素做升序（或降序）排序。

值得一提的是，\[first, last\] 中的这 2 个迭代器类型仅限定为输入迭代器，这意味着相比 partial_sort () 函数，partial_sort_copy () 函数**放宽了对存储原有数据的容器类型的限制**。换句话说，partial_sort_copy () 函数**还支持对 list 容器或者 forward_list 容器中存储的元素进行 “部分排序”**，而 partial_sort () 函数不行。

但是，介于 result_first 和 result_last 仍为随机访问迭代器，因此 \[result_first, result_last\) 指定的区域仍仅限于普通数组和部分类型的容器，这和 partial_sort () 函数对容器的要求是一样的。

举个例子：
```
#include <iostream>     
#include <algorithm>    
#include <list>       
using namespace std;
bool mycomp 1 (int i, int j) {
    return (i > j);
}
class mycomp 2 {
public:
    bool operator () (int i, int j) {
        return (i > j);
    }
};
int main () {
    int myints[5] = { 0 };
    std::list<int> mylist{ 3,2,5,4,1,6,9,7 };
    
    std:: partial_sort_copy (mylist.begin (), mylist.end (), myints, myints + 5);
    cout << "第一次排序：\n";
    for (int i = 0; i < 5; i++) {
        cout << myints[i] << " ";
    }
    
    std:: partial_sort_copy (mylist.begin (), mylist.end (), myints, myints + 5, mycomp 2 ());
    cout << "\n 第二次排序：\n";
    for (int i = 0; i < 5; i++) {
        cout << myints[i] << " ";
    }
    return 0;
}

```

程序执行结果为：
```
第一次排序：  
1 2 3 4 5  
第二次排序：  
9 7 6 5 4
```

可以看到，程序中调用了 2 次 partial_sort_copy () 函数，其作用分别是：
* 第 20 行：采用默认的升序排序规则，在 mylist 容器中筛选出最小的 5 个元素，然后将它们复制到 myints\[5\] 数组中，并对这部分元素进行升序排序；
* 第 27 行：采用自定义的 mycomp 2 降序排序规则，从 mylist 容器筛选出最大的 5 个元素，同样将它们复制到 myints\[5\] 数组中，并对这部分元素进行降序排序；

值得一提的是，partial_sort_copy () 函数实现排序的平均时间复杂度为 $mlog(min (n,m))$，其中 n 指的是 \[first, last\) 范围的长度，m 指的是 \[result_first, result_last\) 范围的长度。

## nth_element ()

在系统讲解 nth_element () 函数之前，我们先形成一个共识，即在有序序列中，我们可以称第 n 个元素为整个序列中 “第 n 大” 的元素。比如，下面是一个升序序列：
```
2 4 6 8 10
```

在这个序列中，我们可以称元素 6 为整个序列中 “第 3 小” 的元素，并位于第 3 的位置处；同样，元素 8 为整个序列中 “第 4 小” 的元素，并位于第 4 的位置处。

简单的理解 nth_element () 函数的功能，当采用默认的升序排序规则（std:: less\<T\>）时，该函数可以从某个序列中找到第 n 小的元素 K，并将 K 移动到序列中第 n 的位置处。不仅如此，整个序列经过 nth_element () 函数处理后，所有位于 K 之前的元素都比 K 小，所有位于 K 之后的元素都比 K 大。

当然，我们也可以将 nth_element () 函数的排序规则自定义为降序排序，此时该函数会找到第 n 大的元素 K 并将其移动到第 n 的位置处，同时所有位于 K 之前的元素都比 K 大，所有位于 K 之后的元素都比 K 小。

以下面这个序列为例：
```
3 4 1 2 5
```

假设按照升序排序，并通过 nth_element () 函数查找此序列中第 3 小的元素，则最终得到的序列可能为：
```
2 1 3 4 5
```

显然，nth_element () 函数找到了第 3 小的元素 3 并将其位于第 3 的位置，同时**元素 3 之前的所有元素都比该元素小**，元素 3 之后的所有元素都比该元素大。

> 策略使用了 `k 选取 `，并且同时结合了快速排序中临界点的思想，可能是为了平摊过高的常系数时间复杂度。（对于 k 选取，时间复杂度可能达到 $O(20n)$ ）

nth_element () 本质也是一个函数模板，定义在`<algorithm>`头文件中。因此，如果程序中想使用该函数，就需要提前引入这个头文件：

```
#include <algorithm>
```

nth_element () 函数有以下 2 种语法格式：

```
//排序规则采用默认的升序排序
void nth_element (
	RandomAccessIterator first,
    RandomAccessIterator nth,
    RandomAccessIterator last);
    
//排序规则为自定义的 comp 排序规则
void nth_element (
	RandomAccessIterator first,
    RandomAccessIterator nth,
    RandomAccessIterator last,
    Compare comp);
```

其中，各个参数的含义如下：
* first 和 last：都是随机访问迭代器，\[first, last\) 用于指定该函数的作用范围（即要处理哪些数据）；
* nth：也是随机访问迭代器，其功能是令函数查找 “第 nth 大” 的元素，并将其移动到 nth 指向的位置；
* comp：用于自定义排序规则。

注意，鉴于 nth_element () 函数中各个参数的类型，其**只能对普通数组或者部分容器进行排序**。换句话说，只有普通数组和符合以下全部条件的容器，才能使用使用 nth_element () 函数：
1. 容器支持的迭代器类型**必须为随机访问迭代器**。这意味着，nth_element () 函数只适用于 array、vector、deque 这 3 个容器。
2. 当选用默认的升序排序规则时，容器中存储的元素类型**必须支持 < 小于运算符**；同样，如果选用标准库提供的其它排序规则，元素类型也必须支持该规则底层实现所用的比较运算符；
3. nth_element () 函数在实现过程中，需要交换某些元素的存储位置。因此，如果容器中存储的是自定义的类对象，则该类的内部**必须提供移动构造函数和移动赋值运算符**。

举个例子：
```
#include <iostream>
#include <algorithm>    // std::nth_element
#include <vector>       // std::vector
using namespace std;
//以普通函数的方式自定义排序规则
bool mycomp 1 (int i, int j) {
    return (i > j);
}

//以函数对象的方式自定义排序规则
class mycomp 2 {
public:
    bool operator () (int i, int j) {
        return (i > j);
    }
};
int main () {
    std::vector<int> myvector{3,1,2,5,4};
    //默认的升序排序作为排序规则
    std:: nth_element (myvector.begin (), myvector.begin ()+2, myvector.end ());
    cout << "第一次 nth_element 排序：\n";
    for (std::vector<int>:: iterator it = myvector.begin (); it != myvector.end (); ++it) {
        std:: cout << *it << ' ';
    }
    //自定义的 mycomp 2 () 或者 mycomp 1 降序排序作为排序规则
    std:: nth_element (myvector.begin (), myvector.begin () + 3, myvector.end (), mycomp 1);
    cout << "\n 第二次 nth_element 排序：\n";
    for (std::vector<int>:: iterator it = myvector.begin (); it != myvector.end (); ++it) {
        std:: cout << *it << ' ';
    }
    return 0;
}
```

程序执行结果可能为（不唯一）：
```
第一次 nth_element 排序：  
1 2 3 4 5  
第二次 nth_element 排序：  
5 4 3 2 1
```

上面程序中，共调用了 2 次 nth_elelment () 函数：
* 第 20 行：nth_element () 函数采用的是默认的升序排序，nth 参数设置为 myvector.begin ()+2，即指向的是 myvector 容器中第 3 个元素所在的位置。因此，nth_element () 函数会查找 “第 3 小” 的元素 3，并将其移动到 nth 指向的位置，同时使 nth 之前的所有元素都比 3 小，使 nth 之后的所有元素都比 3 大。
* 第 26 行：nth_element () 函数采用的是默认的降序排序，nth 参数设置为 myvector. begin ()+3，即指向的是 myvector 容器中第 4 个元素所在的位置。因此，nth_element () 函数会查找 “第 4 大” 的元素 2，并将其移动到 nth 指向的位置，同时使 nth 之前的所有元素都比 2 大，使 nth 之后的所有元素都比 2 小。

## is_sorted ()

我们知道，排序操作是比较耗费时间的，尤其当数据量很大的时候。因此在设计程序时，我们应该有意识的去避免执行一些不必要的排序操作。

那么，何谓不必要的排序操作呢？举个例子，有这样一组数据：
```
1 2 3 4 5
```
这本就是一组有序的数据，如果我们恰巧需要这样的升序序列，就没有必要再执行排序操作。

因此，当程序中涉及排序操作时，我们应该为其包裹一层判断语句，像如下这样：

```
// ...
if(not sorted sequence){
	// sort it!
}

// ...
```

> 注意这里的 “不是有序序列”，即只要该序列不符合我们指定的排序规则，就不是有序序列。

那么，怎样才能判断一个序列是否为有序序列呢？很简单，使用 is_sorted () 函数即可，此函数专门用于判断某个序列是否为有序序列。

### is_sorted ()
------------------

和之前学习的其它排序函数（比如 sorted () 函数）一样，is_sorted () 函数本质上就是一个函数模板，定义在 `<algorithm>` 头文件中。因为，在使用该函数之前，程序中必须先引入此头文件：
```
#include <algorithm> 
```

is_sorted () 函数有 2 种语法格式，分别是：
```
//判断 [first, last) 区域内的数据是否符合 std::less<T> 排序规则，即是否为升序序列
bool is_sorted (ForwardIterator first, ForwardIterator last);

//判断 [first, last) 区域内的数据是否符合 comp 排序规则  
bool is_sorted (ForwardIterator first, ForwardIterator last, Compare comp);
```

其中，first 和 last 都为正向迭代器（这意味着该函数适用于大部分容器），\[first, last\) 用于指定要检测的序列；comp 用于指定自定义的排序规则。

> 注意，如果使用默认的升序排序规则，则 \[first, last\) 指定区域内的元素必须支持使用 < 小于运算符做比较；同样，如果指定排序规则为 comp，也要保证 \[first, last\) 区域内的元素支持该规则内部使用的比较运算符。

另外，该函数会返回一个 bool 类型值，即如果 \[first, last\) 范围内的序列符合我们指定的排序规则，则返回 true；反之，函数返回 false。值得一提得是，如果 \[first, last\) 指定范围内只有 1 个元素，则该函数始终返回 true。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::is_sorted
#include <vector>       // std::array
#include <list>         // std::list
using namespace std;
//以普通函数的方式自定义排序规则
bool mycomp1(int i, int j) {
    return (i > j);
}
//以函数对象的方式自定义排序规则
class mycomp2 {
public:
    bool operator() (int i, int j) {
        return (i > j);
    }
};

int main() {
    vector<int> myvector{ 3,1,2,4 };
    list<int> mylist{ 1,2,3,4 };
    //调用第 2 种语法格式的 is_sorted() 函数，该判断语句会得到执行
    if (!is_sorted(myvector.begin(), myvector.end(),mycomp2())) {
        cout << "开始对 myvector 容器排序" << endl;
        //对 myvector 容器做降序排序
        sort(myvector.begin(), myvector.end(),mycomp2());
        //输出 myvector 容器中的元素
        for (auto it = myvector.begin(); it != myvector.end(); ++it) {
            cout << *it << " ";
        }
    }
   
    //调用第一种语法格式的 is_sorted() 函数，该判断语句得不到执行
    if (!is_sorted(mylist.begin(), mylist.end())) {
        cout << "开始对 mylist 排序" << endl;
        //......
    }
    return 0;
}
```

程序执行结果为：
```
开始对 myvector 容器排序  
4 3 2 1
```

结合输出结果可以看到，虽然 myvector 容器中的数据为降序序列，但我们需要的是升序序列。因此第 22 行代码中 is_sorted () 函数的返回值为 false，而 !false 即 true，所以此 if 判断语句会得到执行。

同样在 33 行代码中，mylist 容器中存储的数据为升序序列，和 is_sorted () 函数的要求相符，因此该函数的返回值为 true，而 !true 即 false，所以此 if 判断语句将无法得到执行。

> C++ 标准库官方网站给出了 is_sorted () 函数底层实现的等效代码，感兴趣的读者可自行前往查看。[std::is\_sorted - cppreference.com](https://en.cppreference.com/w/cpp/algorithm/is_sorted)

### is_sorted_until ()
------------------------

和 is_sorted () 函数相比，is_sorted_until () 函数不仅能检测出某个序列是否有序，还会返回一个正向迭代器，该迭代器指向的是当前序列中第一个破坏有序状态的元素。

is_sorted_until () 函数的定义也位于 `<algorithm>` 头文件中。因此，在使用该函数之前，程序中必须先引入此头文件：

```
#include <algorithm> 
```

is_sorted_until () 函数有以下 2 种语法格式：

```
//排序规则为默认的升序排序
ForwardIterator is_sorted_until (
	ForwardIterator first,
	ForwardIterator last);
	
//排序规则是自定义的 comp 规则
ForwardIterator is_sorted_until (
	ForwardIterator first,
    ForwardIterator last,
	Compare comp);
```

其中，first 和 last 都为正向迭代器（这意味着该函数适用于大部分容器），\[first, last\) 用于指定要检测的序列；comp 用于指定自定义的排序规则。

> 注意，如果使用默认的升序排序规则，则 \[first, last\) 指定区域内的元素必须支持使用 < 小于运算符做比较；同样，如果指定排序规则为 comp，也要保证 \[first, last\) 区域内的元素支持该规则内部使用的比较运算符。

可以看到，该函数会返回一个正向迭代器。对于第一种语法格式来说，该函数返回的是指向序列中第一个破坏升序规则的元素；对于第二种语法格式来说，该函数返回的是指向序列中第一个破坏 comp 排序规则的元素。注意，如果 \[first, last\) 指定的序列完全满足默认排序规则或者 comp 排序规则的要求，则该函数将返回一个和 last 迭代器指向相同的正向迭代器。

举个例子：
```
#include <iostream>     // std::cout
#include <algorithm>    // std::is_sorted_until
#include <vector>        // std::array
#include <list>            // std::list
using namespace std;
//以普通函数的方式自定义排序规则
bool mycomp1(int i, int j) {
    return (i > j);
}
//以函数对象的方式自定义排序规则
class mycomp2 {
public:
    bool operator() (int i, int j) {
        return (i > j);
    }
};

int main() {
    vector<int> myvector{ 3,1,2,4 };
    list<int> mylist{ 1,2,3,4 };
    //如果返回值为 myvector.end()，则表明 myvector 容器中的序列符合 mycomp2() 规则
    if (is_sorted_until(myvector.begin(), myvector.end(),mycomp2()) != myvector.end()) {
        cout << "开始对 myvector 容器排序" << endl;
        //对 myvector 容器做降序排序
        sort(myvector.begin(), myvector.end(),mycomp2());
        //输出 myvector 容器中的元素
        for (auto it = myvector.begin(); it != myvector.end(); ++it) {
            cout << *it << " ";
        }
    }
   
    //该判断语句得不到执行
    if (is_sorted_until(mylist.begin(), mylist.end()) != mylist.end()) {
        cout << "开始对 mylist 排序" << endl;
        //......
    }
    return 0;
}
```

程序执行结果为：
```
开始对 myvector 容器排序  
4 3 2 1
```

## 如何判断使用哪个排序算法？

> 当操作对象为 list 或者 forward_list 序列式容器时，其容器模板类中都提供有 sort () 排序方法，借助此方法即可实现对容器内部元素进行排序。其次，对关联式容器（包括哈希容器）进行排序是没有实际意义的，因为这类容器会根据既定的比较函数（和哈希函数）维护内部元素的存储位置。

那么，当需要对普通数组或者 array、vector 或者 deque 容器中的元素进行排序时，怎样选择最合适（效率最高）的排序函数呢？这里为大家总结了以下几点：
1. 如果需要对所有元素进行排序，则选择 sort () 或者 stable_sort () 函数；
2. 如果需要保持排序后各元素的相对位置不发生改变，就只能选择 stable_sort () 函数，而另外 3 个排序函数都无法保证这一点；
3. 如果需要对最大（或最小）的 n 个元素进行排序，则优先选择 partial_sort () 函数；
4. 如果只需要找到最大或最小的 n 个元素，但不要求对这 n 个元素进行排序，则优先选择 nth_element () 函数。

除此之外，很多读者都关心这些排序函数的性能。总的来说，函数功能越复杂，做的工作越多，它的性能就越低（主要体现在时间复杂度上）。对于以上 4 种排序函数，综合考虑它们的时间和空间效率，其性能之间的比较如下所示：

```
nth_element ()> partial_sort () > sort () > stable_sort ()       <-- 从左到右，性能由高到低
```

建议大家，在实际选择排序函数时，应更多从所需要完成的功能这一角度去考虑，而不是一味地追求函数的性能。换句话说，如果你选择的算法更有利于实现所需要的功能，不仅会使整个代码的逻辑更加清晰，还会达到事半功倍的效果。
