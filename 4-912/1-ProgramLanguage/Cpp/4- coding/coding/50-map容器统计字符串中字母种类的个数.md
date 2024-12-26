## 题目描述
键盘输入一个字符串，统计字符串中各个字母字符的个数。例如：键盘输入"Hello World!"，上述字符串中各个字母字符的出现的次数，要求使用 map 实现，键的排序使用 map 默认排序即可。

## 测试样例与预期输出
**输入描述**：
键盘输入任意一个字符串

**输出描述**：
输出字母字符和字符对应的出现字数（注：相同字母的大小写算两个不同字符，字母字符和出现次数之间用:隔开，按 map 的默认顺序输出即可）

```
输入：
Hello World

输出：
H:1
W:1
d:1
e:1
l:3
o:2
r:1
```

## 代码与优化
### version 1: 逐字判断输入
```cpp
#include <iostream>
#include <map>
// write your code here......

using namespace std;

int main() {

    char str[100] = { 0 };
    cin.getline(str, sizeof(str));

    // write your code here......
    map<char, int> mymap;
    for (int i = 0; str[i] != '\0'; i++) {
        for (auto iter = mymap.begin(); iter != mymap.end(); iter++) {
            if(iter->first == ' ') continue;
            else if(iter->first < '9' && iter->first >'0') continue;
            else if (iter->first != str[i]) continue;
            else {
                (iter->second)++;
                i++;
                break;
            }
        }
        mymap.insert({str[i], 1});
    }

    for(auto iter=mymap.begin();iter!=mymap.end();iter++){
        cout<<iter->first<<":"<<iter->second<<endl;
    }
    return 0;
}
```
时间复杂度：$O(n^2)$;
缺点：没能正确利用 map 容器的特点，相当于创建了一个新式数组，非常低效。

### version 2: chatgpt 简化查找与判断
```cpp
#include <iostream>
#include <map>
#include <cctype> // For isalpha function

int main() {
    char str[100] = {0};
    std::cin.getline(str, sizeof(str));

    std::map<char, int> mymap;

    for (int i = 0; str[i] != '\0'; i++) {
        char currentChar = str[i];
        if (!std::isalpha(currentChar)) {
            continue; // Skip non-alphabet characters
        }

        // If the character is already in the map, increment the count
        mymap[currentChar]++;
    }

    // Output the character counts
    for (const auto& pair : mymap) {
        std::cout << pair.first << ":" << pair.second << std::endl;
    }

    return 0;
}

```

缺点：屎上雕花，没有利用 map 容器的特点，再如何也简化不了多少。

### version 3: 合理利用 map 容器的特点
```cpp
#include <iostream>
#include <map>
#include <cctype> // For std::isalpha()

int main() {
    std::string inputString;
    std::cout << "请输入一个字符串: ";
    std::getline(std::cin, inputString);

    // 使用 std::map 来统计字母字符的个数
    std::map<char, int> charCountMap;

    // 遍历输入字符串并统计字符出现次数
    for (char ch : inputString) {
        if (std::isalpha(ch)) {
            charCountMap[ch]++;
        }
    }

    // 打印结果
    for (const auto& entry : charCountMap) {
        std::cout << entry.first << ":" << entry.second << std::endl;
    }

    return 0;
}

```

#### 时间复杂度
- 遍历输入字符串的时间复杂度为 O (N)，其中 N 是输入字符串的长度。
- 在遍历过程中，执行 `std::isalpha(ch)` 检查当前字符是否为字母，这是一个常数时间操作，不影响总体时间复杂度。
- 在 `std::map` 中插入字符和更新计数的时间复杂度为 $O (log M)$，其中 M 是 `std::map` 中已有的元素数量（在这个问题中是字母字符的数量）。
- 打印结果的时间复杂度为 O (M)，其中 M 是不同的字母数量，也就是 `std::map` 中的键值对数量。
   
综上所述，整个代码的时间复杂度为 $O (N + log M + M)$。因为 $M$ 是常数（52 个字母），所以可以简化为 $O (N)$。

#### 空间复杂度
   - 输入字符串的存储需要 $O (N)$ 的空间。
   - 使用 `std::map` 来统计字符出现次数，最坏情况下需要存储所有不同的字母字符，因此空间复杂度为 $O (M)$。但由于 $M$ 是常数（52 个字母），所以可以简化为 $O (1)$。

综合考虑时间复杂度和空间复杂度，经过简化后，这段代码的时间复杂度为 $O (N)$，空间复杂度为 $O (1)$。