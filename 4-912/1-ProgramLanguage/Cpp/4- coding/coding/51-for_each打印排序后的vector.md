## 题目描述
键盘输入 5 个整数，使用 vector 进行存储，使用 STL 排序算法对元素进行排序（从大到小），再使用 STL 遍历算法输出元素。（元素和元素之间使用空格隔开）

## 测试样例与预期输出
```
input: 89 90 78 66 45

output:90 89 78 66 45
```

## 代码与优化
### version 1: 定义 print 方法
```cpp
#include <algorithm>
#include <iostream>
#include <vector>

using namespace std;

void print(int x) {
    printf("%d ", x);
}

int main() {

    int num;
    vector<int> v;
    for (int i = 0; i < 5; i++) {
        cin >> num;
        v.push_back(num);
    }

    // write your code here......
    sort(v.begin(), v.end(), greater<int>());

    for_each(v.begin(), v.end(), print);

    return 0;
}
```

还可以优化掉 print 函数的定义，直接使用 lambda 表达式内联：

```cpp
for_each(v.begin(), v.end(), [](int x){printf("%d ", x);});
```