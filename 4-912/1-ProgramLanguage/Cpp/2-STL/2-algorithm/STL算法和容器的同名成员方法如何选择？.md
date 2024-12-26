
通过前面的学习，我们已经掌握了一些 STL 算法的功能和用法。值得一提的是，STL 标准库提供有 70 多种算法函数，其中有些函数名称和 STL 容器模板类中提供的成员方法名相同。

例如，STL 标准库提供了 sort () 和 merge () 函数，而 list 容器模板类中也提供有同名的 sort () 和 merge () 成员方法。再比如，STL 标准库提供有 count ()、find ()、lower_bound ()、upper_bound () 以及 equal_range () 这些函数，而每个关联式容器（除哈希容器外）也提供有相同名称的成员方法。

那么，当某个 STL 容器提供有和算法同名的成员方法时，应该使用哪一个呢？大多数情况下，**我们应该使用 STL 容器提供的成员方法**，而不是同名的 STL 算法，原因包括：
1. 虽然同名，但它们的底层实现并不完全相同。相比同名的算法，容器的成员方法和自身结合地更加紧密。
2. 相比同名的算法，STL 容器提供的成员方法往往执行效率更高；

举个例子：
```
#include <iostream>    // std::cout
#include <algorithm>   // std::find
#include <set>         // std::set
#include <string>      // std::string
using namespace std;
//为 set 容器自定义排序规则，即按照字符串长度进行排序
class mycomp {
public:
    bool operator() (const string &i, const string &j) const {
        return i.length() < j.length();
    }
};

int main() {
//定义 set 容器，其排序规则为 mycomp
    std::set<string,mycomp> myset{"123","1234","123456"};
    //调用 set 容器成员方法
    set<string>::iterator iter = myset.find(string("abcd"));
    if (iter == myset.end()) {
        cout << "查找失败" << endl;
    }
    else {
        cout << *iter << endl;
    }

    //调用 find() 函数
    auto iter2 = find(myset.begin(), myset.end(), string("abcd"));
    if (iter2 == myset.end()) {
        cout << "查找失败" << endl;
    }
    else {
        cout << *iter << endl;
    }
    return 0;
}
```

程序执行结果为：
```
1234  
查找失败
```

可以看到，程序中分别调用了 find () 函数和 set 容器自带的 find () 成员方法，都用于查找 "abcd" 这个字符串，但查找结果却不相同。其中，find () 成员方法成功找到了和 "abcd" 长度相同的 "1234"，但 find () 函数却查找失败。

之所以会这样，是因为 find () 成员方法和 find () 函数底层的实现机制不同。前者会依照 mycomp () 规则查找和 `abcd` 匹配的元素，而 find () 函数底层仅会依据 `==` 运算符查找 myset 容器中和 `abcd` 相等的元素，所以会查找失败。

不仅如此，**无论是序列式容器还是关联式容器，成员方法的执行效率要高于同名的 STL 算法**。仍以 find () 函数和 set 容器中的 find () 成员方法为例。要知道，find () 函数是通过 “逐个比对” 来实现查找的，它以线性时间运行；而由于 set 容器底层存储结构采用的是红黑树，所以 find () 成员方法以对数时间运行，而非线性时间。

换句话说，对于含有一百万个元素的 set 容器，如果使用 find () 成员方法查找目标元素，其最差情况下的比对次数也不会超过 40 次（平均只需要比对 20 次就可以查找成功）；而使用同名的 find () 函数查找目标元素，最差情况下要比对一百万次（平均比对 50 万次才能查找成功）。

> 所谓 “最差情况”，指的是当前 set 容器中未存储有目标元素。

并且需要注意的一点是，虽然**有些容器提供的成员方法和某个 STL 算法同名，但该容器只能使用自带的成员方法，而不适用同名的 STL 算法**。比如，sort () 函数根本不能应用到 list 容器上，因为该类型容器仅支持双向迭代器，而 sort () 函数的参数类型要求为随机访问迭代器；merge () 函数和 list 容器的 merge () 成员方法之间也存在行为上的不同，即 merge () 函数是不允许修改源数据的，而 list:: merge () 成员方法就是对源数据做修改。

总之，当读者需要在 STL 算法与容器提供的同名成员方法之间做选择的时候，应优先考虑成员方法。几乎可以肯定地讲，成员方法的性能更优越，也更贴合当前要操作的容器。