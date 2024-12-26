## 题目描述
有一对兔子，从出生后第 3 个月起每个月都生一对兔子，小兔子长到第三个月后每个月又生一对兔子，假如兔子都不死，问第 n 个月的兔子对数为多少？

## 测试样例与预期输出
```
input: 1
output: 1

input: 2
output: 1

input: 3
output: 2
```

## 代码与优化

| 月份   | 兔子数（对）                           |
|------|----------------------------------|
| 第 1 个月 | 1 对兔子                             |
| 第 2 个月 | 1 对兔子                             |
| 第 3 个月 | 2 对兔子（第一对生出第二对）                   |
| 第 4 个月 | 3 对兔子（第一对生出第三对）                   |
| 第 5 个月 | 5 对兔子（第一对生出第四对、第二对生出第五对）          |
| 第 6 个月 | 8 对兔子（第一对生出第六对、第二对生出第七对、第三对生出第八对） |

我们发现除了最前面两个月，其余月份的兔子数都是有它前面的两个月相加而来，因此我们有如下结论： $f(n)=f(n-1)+f(n-2)$，即斐波那契数列。

### version 1: 递归法
![[fibonacci.png]]

```cpp
#include <iostream>
using namespace std;

int getSum(int n);

int main() {

	int n;
	cin >> n;
    
	cout << getSum(n) << endl; //输出结果

	return 0;
}

int getSum(int n) {
    if(n == 1 || n == 2) //n=1或2跳出递归
        return 1;
    return getSum(n - 1) + getSum(n - 2); //返回前两个月相加
}

```

时间复杂度：$O(2^n)$;
空间复杂度: $O(n)$;

### version 2: 迭代法
```cpp
int getSum(int n) {
    if(n <= 2) //2及以下直接返回
        return 1;
    int a = 1; //表示当前要计算的月份前一个月
    int b = 1; //表示当前要计算月份的前两个月
    int res = 0; 
    for(int i = 3; i <= n; i++){ //遍历3-n
        res = a + b; //直接相加
        b = a; //更新前两个
        a = res; //
    }
    return res;
}

```

时间复杂度：$O(n)$;
空间复杂度：$O(1)$;