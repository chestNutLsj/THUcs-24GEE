## 题目描述
从键盘获取一串字符串，要求去除重复的字符，请使用 set 解决。

## 测试样例与预期输出
输入描述：
键盘输入任意字符串
输出描述：
输出去重后的内容（直接按 set 的默认顺序输出字符即可）

例如：
```
input: helloworld
output:dehlorw
```

## 代码与优化
set 是 STL 的集合容器，相同的元素在 set 中只保留一次，而且 set 还会使用红黑树按照 ASCⅡ码的大小排序自动排序。

建立一个set，遍历字符串，然后将每个字符都插入到set中，它会自动去重加排序，然后我们用迭代器遍历set，输出每个迭代器的元素即可。

![[set-insert-string.gif]]

### version 1: 字符串逐个插入
```cpp
#include <iostream>
#include <set>
using namespace std;

int main() {

	char str[100] = { 0 };
	cin.getline(str, sizeof(str));
    set<char> s;
    for(int i = 0; str[i] != '\0'; i++) //遍历字符串
        s.insert(str[i]); //将字符加入到set中
    for(auto iter = s.begin(); iter != s.end(); iter++) //遍历set输出字符
        cout << *iter;
	return 0;
}

```

时间复杂度：$O(nlogn)$，每次插入的代价是 $O(logn)$;
空间复杂度：$O(n)$;