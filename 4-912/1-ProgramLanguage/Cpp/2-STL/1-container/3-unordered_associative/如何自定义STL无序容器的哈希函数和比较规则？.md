
前面在讲解 unordered_map、unordered_multimap、unordered_set 以及 unordered_multiset 这 4 种无序关联式容器（哈希容器）时，遗留过一个共性问题，即如何给无序容器自定义一个哈希函数和比较规则？

注意，虽然每种无序容器都指定了默认的 hash\<key\> 哈希函数和 equal_to\<key\> 比较规则，但它们仅适用于存储基本类型（比如 int、double、float、string 等）数据的无序容器。换句话说，如果无序容器存储的数据类型为自定义的结构体或类，则 STL 标准库提供的 hash\<key\> 和 equal_to\<key\> 将不再适用。

## 无序容器自定义哈希函数
---------------

我们知道，无序容器以键值对的方式存储数据（unordered_set 和 unordered_multiset 容器可以看做存储的是键和值相等的键值对），且底层采用哈希表结构存储各个键值对。在此存储结构中，哈希函数的功能是根据各个键值对中键的值，计算出一个哈希值（本质就是一个整数），哈希表可以根据该值判断出该键值对具体的存储位置。

简单地理解哈希函数，它可以接收一个元素，并通过内部对该元素做再加工，最终会得出一个整形值并反馈回来。需要注意的是，哈希函数只是一个称谓，其本体并不是普通的函数形式，而是一个函数对象类。因此，如果我们想自定义个哈希函数，就需要自定义一个函数对象类。

举个例子，假设有如下一个 Person 类：
```
class Person {
public:
    Person (string name, int age) : name (name), age (age) {};
    string getName () const;
    int getAge () const;
private:
    string name;
    int age;
};
string Person:: getName () const {
    return this->name;
}
int Person:: getAge () const {
    return this->age;
}
```
在此基础上，假设我们想创建一个可存储 Person 类对象的 unordered_set 容器，考虑到 Person 为自定义的类型，因此默认的 hash\<key\> 哈希函数不再适用，这时就需要以函数对象类的方式自定义一个哈希函数。比如：

```
class hash_fun {
public:
    int operator ()(const Person &A) const {
        return A.getAge ();
    }
};
```

> 注意，重载 ( ) 运算符时，其参数必须为 const 类型，且该方法也必须用 const 修饰。

可以看到，我们利用 hash_fun 函数对象类的 ( ) 运算符重载方法，自定义了适用于 Person 类对象的哈希函数。该哈希函数每接收一个 Person 类对象，都会返回该对象的 age 成员变量的值。

> 事实上，默认的 hash\<key\> 哈希函数，其底层也是以函数对象类的形式实现的。

由此，在创建存储 Person 类对象的 unordered_set 容器时，可以将 hash_fun 作为参数传递给该容器模板类中的 Pred 参数：

```
std::unordered_set<Person, hash_fun> myset；
```

但是，此时创建的 myset 容器还无法使用，因为该容器使用的是默认的 std:: equal_to\<key\> 比较规则，但此规则并不适用于该容器。

## 无序容器自定义比较规则
---------------

和哈希函数一样，无论创建哪种无序容器，都需要为其指定一种可比较容器中各个元素是否相等的规则。

值得一提的是，默认情况下无序容器使用的 std:: equal_to\<key\> 比较规则，其本质也是一个函数对象类，底层实现如下：

```
template<class T>
class equal_to
{
public:   
    bool operator ()(const T& _Left, const T& _Right) const{
        return (_Left == _Right);
    }   
};
```

可以看到，该规则在底层实现过程中，直接用 == 运算符比较容器中任意 2 个元素是否相等，这意味着，如果容器中存储的元素类型，支持直接用 == 运算符比较是否相等，则该容器可以使用默认的 std:: equal_to\<key\> 比较规则；反之，就不可以使用。

显然，对于我们上面创建的 myset 容器，其内部存储的是 Person 类对象，不支持直接使用 == 运算符做比较。这种情况下，有以下 2 种方式可以解决此问题：

1.  在 Person 类中重载 == 运算符，这会使得 std:: equal_to\<key\> 比较规则中使用的 == 运算符变得合法，myset 容器就可以继续使用 std:: equal_to\<key\> 比较规则；
2.  以函数对象类的方式，自定义一个适用于 myset 容器的比较规则。

### 1) 重载 == 运算符

如果选用第一种解决方式，仍以 Person 类为例，在此类的外部添加如下语句：

```
bool operator==(const Person &A, const Person &B) {
    return (A.getAge () == B.getAge ());
}
```

> 注意，这里在重载 == 运算符时，2 个参数必须用 const 修饰。

可以看到，通过此方式重载的运算符，当 std:: equal_to\<key\> 函数对象类中直接比较 2 个 Person 类对象时，实际上是在比较这 2 个对象的 age 成员变量是否相等。换句话说，此时的 std:: equal_to\<key\> 规则的含义为：只要 2 个 Person 对象的 age 成员变量相等，就认为这 2 个 Person 对象是相等的。

重载 == 运算符之后，就能以如下方式创建 myset 容器：

```
std::unordered_set<Person, hash_fun> myset{ {"zhangsan", 40},{"zhangsan", 40},{"lisi", 40},{"lisi", 30} };
```

注意，虽然这里给 myset 容器初始化了 4 个 Person 对象，但由于比较规则以各个类对象的 age 值为准，myset 容器会认为前 3 个 Person 对象是相等的，因此最终 myset 容器只会存储 {"zhangsan", 40} 和 {"lisi", 30}。

### 2) 以函数对象类的方式自定义比较规则

除此之外，还可以完全舍弃 std:: equal_to\<key\>，以函数对象类的方式自定义一个比较规则。比如：

```
class mycmp {
public:
    bool operator ()(const Person &A, const Person &B) const {
        return (A.getName () == B.getName ()) && (A.getAge () == B.getAge ());
    }
};
```

在 mycmp 规则的基础上，我们可以像如下这样创建 myset 容器：

```
std::unordered_set<Person, hash_fun, mycmp> myset{ {"zhangsan", 40},{"zhangsan", 40},{"lisi", 40},{"lisi", 30} };
```

由此创建的 myset 容器，虽然初始化了 4 个 Person 对象，但 myset 容器根据 mycmp 比较规则，可以识别出前 2 个是相等的，因此最终该容器内部存有  {"zhangsan", 40}、{"lisi", 40} 和 {"lisi", 30} 这 3 个 Person 对象。

## 总结

总的来说，当无序容器中存储的是基本类型（int、double、float、string）数据时，自定义哈希函数和比较规则，都只能以函数对象类的方式实现。

而当无序容器中存储的是用结构体或类自定义类型的数据时，自定义哈希函数的方式仍只有一种，即使用函数对象类的形式；而自定义比较规则的方式有两种，要么也以函数对象类的方式，要么仍使用默认的 std:: equal_to\<key\> 规则，但前提是必须重载 == 运算符。

如下是本节的完整代码，读者可直接拷贝下来，加深对本节知识的理解：

```
#include <iostream>
#include <string>
#include <unordered_set>

using namespace std;

class Person {
public:
    Person(string name, int age) : name(name), age(age) {};

    string getName() const;

    int getAge() const;

private:
    string name;
    int age;
};

string Person::getName() const {
    return this->name;
}

int Person::getAge() const {
    return this->age;
}

//自定义哈希函数
class hash_fun {
public:
    int operator()(const Person &A) const {
        return A.getAge();
    }
};

//重载 == 运算符，myset 可以继续使用默认的 equal_to<key> 规则
bool operator==(const Person &A, const Person &B) {

    return (A.getAge() == B.getAge());
}

//完全自定义比较规则，弃用 equal_to<key>
class mycmp {
public:
    bool operator()(const Person &A, const Person &B) const {
        return (A.getName() == B.getName()) && (A.getAge() == B.getAge());
    }
};

int main() {
    //使用自定义的 hash_fun 哈希函数，比较规则仍选择默认的 equal_to<key>,前提是必须重载 == 运算符
    std::unordered_set<Person, hash_fun> myset1
            {{"zhangsan", 40},
             {"zhangsan", 40},
             {"lisi",     40},
             {"lisi",     30}};
    //使用自定义的 hash_fun 哈希函数，以及自定义的 mycmp 比较规则
    std::unordered_set<Person, hash_fun, mycmp> myset2
            {{"zhangsan", 40},
             {"zhangsan", 40},
             {"lisi",     40},
             {"lisi",     30}};

    cout << "myset 1: " << endl;
    for (const auto & iter : myset1) {
        cout << iter.getName() << " " << iter.getAge() << endl;
    }

    cout << "myset 2: " << endl;
    for (const auto & iter : myset2) {
        cout << iter.getName() << " " << iter.getAge() << endl;
    }
    return 0;
}
```

程序执行结果为：
```
myset 1:  
zhangsan 40  
lisi 30  
myset 2:  
lisi 40  
zhangsan 40  
lisi 30
```