---
url: https://github.com/sebastiankliem/eos2017_memory-management-demo
---
## LinkedList
### .h
```
#ifndef LINKEDLIST_H  
#define LINKEDLIST_H  
  
#define DEFAULT_BLOCK_SIZE 64  
  
struct list_block {  
    void *address;  
    list_block *next;  
};  
  
class LinkedList {  
private:  
    list_block *_head;  
    list_block *_tail;  
    int _blockSizeKb;  
    unsigned int _length;  
  
    void _init(int blockSizeKb);  
  
public:  
    LinkedList();  
  
    explicit LinkedList(int blockSize);  
  
    bool isEmpty();  
  
    int getBlockSizeKb();  
  
    int getLength();  
  
    void print();  
  
    list_block *getBlockAt(unsigned int position);  
  
    void addBlockStart(void *address);  
  
    void addBlockEnd(void *address);  
  
    void addBlockAt(unsigned int position, void *address);  
  
    void removeBlockStart();  
  
    void removeBlockEnd();  
  
    void removeBlockAt(unsigned int position);  
};  
  
#endif //LINKEDLIST_H
```
- 头文件开始处使用了条件编译指令，防止多次包含同一个头文件。
- 定义了一个结构体 `list_block`，其中包含一个指向数据块地址的指针 `void *address` 和一个指向下一个节点的指针 `list_block *next`。
- 声明了一个类 `LinkedList`，它包括了私有成员变量和公共成员函数。
- 私有成员变量包括 `_head` 和 `_tail`，分别表示链表的头部和尾部，`_blockSizeKb` 表示数据块的大小（以KB为单位），`_length` 表示链表的长度。
- 构造函数 `LinkedList()` 和 `LinkedList(int blockSize)` 分别用于创建链表对象，可以传入数据块大小。
- 公共成员函数包括用于查询链表状态和操作链表的函数，如 `isEmpty()`, `getBlockSizeKb()`, `getLength()`, `print()`, `getBlockAt()`, `addBlockStart()`, `addBlockEnd()`, `addBlockAt()`, `removeBlockStart()`, `removeBlockEnd()`, `removeBlockAt()`。

### .cpp
```cpp
#include "LinkedList.h"  
#include <iostream>  
#include <cstdlib>  
  
LinkedList::LinkedList() {  
    LinkedList::_init(DEFAULT_BLOCK_SIZE);  
}  
  
LinkedList::LinkedList(int blockSizeKb) {  
    LinkedList::_init(blockSizeKb);  
}  
  
void LinkedList::_init(int blockSizeKb) {  
    _blockSizeKb = blockSizeKb;  
    _head = NULL;  
    _tail = NULL;  
    _length = 0;  
}  
  
bool LinkedList::isEmpty() {  
    return _head == NULL && _tail == NULL;  
}  
  
int LinkedList::getBlockSizeKb() {  
    return _blockSizeKb;  
}  
  
int LinkedList::getLength() {  
    return _length;  
}  
  
void LinkedList::print() {  
    list_block *iterator = new list_block;  
    iterator = _head;  
  
    while (iterator != NULL) {  
        std::cout << iterator->address << " ";  
        iterator = iterator->next;  
    }  
    std::cout << std::endl;  
}  
  
list_block *LinkedList::getBlockAt(unsigned int position) {  
    list_block *current = new list_block;  
    current = _head;  
    if (position > 0) {  
        for (int i = 0; i < position; i++) {  
            current = current->next;  
        }  
    }  
    return current;  
}  
  
void LinkedList::addBlockStart(void *address) {  
    list_block *newBlock = new list_block;  
    newBlock->address = address;  
    newBlock->next = NULL;  
    if (_length == 0) {  
        _head = newBlock;  
        _tail = newBlock;  
    } else {  
        newBlock->next = _head;  
        _head = newBlock;  
    }  
    _length++;  
}  
  
void LinkedList::addBlockEnd(void *address) {  
    list_block *newBlock = new list_block;  
    newBlock->address = address;  
    newBlock->next = NULL;  
    if (_length == 0) {  
        _head = newBlock;  
        _tail = newBlock;  
    } else {  
        list_block *secondLastBlock = getBlockAt((_length == 1) ? 0 : _length - 2);  
        secondLastBlock->next = newBlock;  
        _tail = newBlock;  
    }  
    _length++;  
}  
  
void LinkedList::addBlockAt(unsigned int position, void *address) {  
    if (_length == 0 || position == 0) {  
        addBlockStart(address);  
    } else {  
        list_block *newBlock = new list_block;  
        newBlock->address = address;  
        list_block *beforeBlock = new list_block;  
        beforeBlock = getBlockAt(position - 1);  
        newBlock->next = beforeBlock->next;  
        beforeBlock->next = newBlock;  
        _length++;  
    }  
}  
  
void LinkedList::removeBlockStart() {  
    removeBlockAt(0);  
}  
  
void LinkedList::removeBlockEnd() {  
    removeBlockAt(_length - 1);  
}  
  
void LinkedList::removeBlockAt(unsigned int position) {  
    if (_length != 0 && position < _length) {  
        list_block *removeBlock = new list_block;  
        removeBlock = getBlockAt(position);  
        if (_length == 1) {  
            _head = NULL;  
            _tail = NULL;  
        }  
            // else if (position == 0) {  
            //     removeBlockStart();            // }        else {  
            if (position == 0) {  
                _head = removeBlock->next;  
            } else {  
                list_block *blockBefore = new list_block;  
                blockBefore = getBlockAt(position - 1);  
                blockBefore->next = removeBlock->next;  
                if (position == _length - 1) {  
                    _tail = blockBefore;  
                }  
            }  
        }  
        delete removeBlock;  
        _length--;  
    }  
}
```

## BuddyAllocator
### .h
```h
#include "LinkedList.h"  
  
#ifndef BUDDYALLOCATOR_H  
#define BUDDYALLOCATOR_H  
  
#define MEMORY_DEFAULT_SIZE_KB 1024  
#define BLOCK_MIN_SIZE_KB 64  
  
struct buddy_block {  
    char *startAddress;  
    unsigned int sizeKb;  
};  
  
class BuddyAllocator {  
private:  
    unsigned long _size;  
    char *_memory;  
    LinkedList *_blocks;  
  
    void _init(int sizeKb);  
  
    int _getListsSize();  
  
    int _getListNo(int sizeKb);  
  
    void _sortList(int listNo);  
  
    int *_findBuddies(int listNo);  
  
    void _merge(int listNo);  
  
    void _mergeAll();  
  
public:  
    BuddyAllocator();  
  
    BuddyAllocator(int sizeKb);  
  
    unsigned long getSize();  
  
    unsigned long getSizeKb();  
  
    void dumpLists();  
  
    char *getMemoryPointer();  
  
    buddy_block *allocate(int sizeKb);  
  
    void deallocate(buddy_block *);  
};  
  
#endif //BUDDYALLOCATOR_H
```
- 头文件开始处包含了 "LinkedList.h" 头文件，表明它依赖于 LinkedList 类。
- 定义了一些常量，包括默认内存大小 `MEMORY_DEFAULT_SIZE_KB` 和最小块大小 `BLOCK_MIN_SIZE_KB`。
- 定义了一个结构体 `buddy_block`，用于表示内存块的起始地址和大小。
- 声明了一个类 `BuddyAllocator`，它包括了私有成员变量和公共成员函数。
- 私有成员变量包括 `_size`，表示总内存大小；`_memory`，表示分配的内存块；和 `_blocks`，一个包含多个 LinkedList 的数组，用于管理不同大小的内存块。
- 构造函数 `BuddyAllocator()` 和 `BuddyAllocator(int sizeKb)` 用于创建 BuddyAllocator 对象，可以指定内存大小。
- 公共成员函数包括查询内存状态和进行内存分配和释放的函数，如 `getSize()`, `getSizeKb()`, `dumpLists()`, `getMemoryPointer()`, `allocate()`, 和 `deallocate()`。
### .cpp

```cpp
#include "BuddyAllocator.h"  
#include <iostream>  
#include <cmath>  
#include <cstdlib>  
  
BuddyAllocator::BuddyAllocator() {  
    _init(MEMORY_DEFAULT_SIZE_KB);  
}  
  
BuddyAllocator::BuddyAllocator(int sizeKb) {  
    _init(sizeKb);  
}  
  
void BuddyAllocator::_init(int sizeKb) {  
    _size = sizeKb * 1000 * sizeof(char);  
    _memory = (char *) malloc(_size);  
    if (_memory == NULL) {  
        std::cerr << "ERROR: could not allocate memory\n";  
        exit(EXIT_SUCCESS);  
    }  
  
    _blocks = new LinkedList[_getListsSize()];  
    for (int i = 0; i < _getListsSize(); i++) {  
        _blocks[i] = LinkedList(pow(2, i + log2(BLOCK_MIN_SIZE_KB)));  
    }  
    _blocks[_getListNo(getSizeKb())].addBlockEnd(_memory);  
}  
  
int BuddyAllocator::_getListsSize() {  
    return _getListNo(getSizeKb()) + 1;  
}  
  
int BuddyAllocator::_getListNo(int sizeKb) {  
    return ceil(log2(sizeKb) - log2(BLOCK_MIN_SIZE_KB));  
}  
  
unsigned long BuddyAllocator::getSize() {  
    return _size;  
}  
  
unsigned long BuddyAllocator::getSizeKb() {  
    return _size / 1000;  
}  
  
void BuddyAllocator::_sortList(int listNo) {  
    LinkedList unsorted = _blocks[listNo];  
    LinkedList sorted = LinkedList(unsorted.getBlockSizeKb());  
    sorted.addBlockStart(unsorted.getBlockAt(0)->address);  
    for (int i = 1; i < unsorted.getLength(); i++) {  
        void *insertAddress = unsorted.getBlockAt(i)->address;  
        for (int j = 0; j < sorted.getLength(); j++) {  
            if (insertAddress <= (sorted.getBlockAt(j)->address)) {  
                sorted.addBlockStart(insertAddress);  
                break;  
            } else if (j == sorted.getLength() - 1 && insertAddress > (sorted.getBlockAt(j)->address)) {  
                sorted.addBlockEnd(insertAddress);  
                break;  
            }  
        }  
    }  
  
    _blocks[listNo] = sorted;  
}  
  
int *BuddyAllocator::_findBuddies(int listNo) {  
    LinkedList list = _blocks[listNo];  
    if (list.getLength() > 1) {  
        int *buddies = (int *) malloc(sizeof(int) * 2);  
        int blockSize = list.getBlockSizeKb() * 1000;  
        for (int i = 0; i <= list.getLength() - 2; i++) {  
            if (list.getBlockAt(i)->address == (char *) list.getBlockAt(i + 1)->address - blockSize) {  
                buddies[0] = i;  
                buddies[1] = i + 1;  
                return buddies;  
            }  
        }  
    }  
    return NULL;  
}  
  
void BuddyAllocator::_merge(int listNo) {  
    int *buddies = _findBuddies(listNo);  
    while (buddies != NULL) {  
        void *startAddress = _blocks[listNo].getBlockAt(buddies[0])->address;  
        _blocks[listNo].removeBlockAt(buddies[1]);  
        _blocks[listNo].removeBlockAt(buddies[0]);  
        _blocks[listNo + 1].addBlockStart(startAddress);  
        _sortList(listNo + 1);  
  
        buddies = _findBuddies(listNo);  
    }  
}  
  
void BuddyAllocator::_mergeAll() {  
    for (int i = 0; i < _getListsSize(); i++) {  
        _merge(i);  
    }  
}  
  
void BuddyAllocator::dumpLists() {  
    for (int i = 0; i < _getListsSize(); i++) {  
        std::cout << _blocks[i].getBlockSizeKb() << ": ";  
        _blocks[i].print();  
    }  
}  
  
char *BuddyAllocator::getMemoryPointer() {  
    return _memory;  
}  
  
buddy_block *BuddyAllocator::allocate(int sizeKb) {  
    int listNo = _getListNo(sizeKb);  
    bool found = false;  
    list_block *allocateBlock = new list_block;  
    while (!found) {  
        if (_blocks[listNo].getLength() > 0) {  
            allocateBlock = _blocks[listNo].getBlockAt(0);  
            _blocks[listNo].removeBlockAt(0);  
            found = true;  
        } else if (listNo < _getListsSize()) {  
            listNo++;  
            if (_blocks[listNo].getLength() > 0) {  
                list_block *removeBlock = new list_block;  
                removeBlock = _blocks[listNo].getBlockAt(0);  
                _blocks[listNo - 1].addBlockStart(  
                        ((char *) removeBlock->address + (_blocks[listNo - 1].getBlockSizeKb() * 1000)));  
                _blocks[listNo - 1].addBlockStart(removeBlock->address);  
                _blocks[listNo].removeBlockAt(0);  
                listNo = _getListNo(sizeKb);  
            }  
        } else {  
            break;  
        }  
    }  
    if (found) {  
        buddy_block *foundBlock = new buddy_block;  
        foundBlock->startAddress = (char *) allocateBlock->address;  
        foundBlock->sizeKb = _blocks[listNo].getBlockSizeKb();  
        return foundBlock;  
    } else {  
        return NULL;  
    }  
}  
  
void BuddyAllocator::deallocate(buddy_block *freeBlock) {  
    int listNo = _getListNo(freeBlock->sizeKb);  
    _blocks[listNo].addBlockStart(freeBlock->startAddress);  
    _sortList(listNo);  
    _mergeAll();  
}
```

1. `BuddyAllocator:: BuddyAllocator ()` 和 `BuddyAllocator:: BuddyAllocator (int sizeKb)`
   - 这两个构造函数用于初始化 BuddyAllocator 对象。
   - `_init(sizeKb)` 被调用，它设置了总内存大小和分配内存块的 LinkedList 数组。
   - 如果内存分配失败，会输出错误消息并终止程序。

2. `void BuddyAllocator:: _init (int sizeKb)`
   - 这个函数初始化 BuddyAllocator 的内部状态。
   - 计算总内存大小 `_size`，并分配内存块 `_memory`。
   - 创建一个包含不同大小内存块的 LinkedList 数组 `_blocks`，并将第一个块添加到适当的链表中。

3. `int BuddyAllocator:: _getListsSize ()`
   - 这个函数用于计算 LinkedList 数组 `_blocks` 的大小。
   - 根据总内存大小 `_size` 计算，包括了不同大小的内存块链表。

4. `int BuddyAllocator:: _getListNo (int sizeKb)`
   - 这个函数根据内存块大小（以 KB 为单位）计算应该使用的 LinkedList 的索引。
   - 使用对数运算将内存块大小映射到合适的链表。

5. `unsigned long BuddyAllocator:: getSize ()` 和 `unsigned long BuddyAllocator:: getSizeKb ()`
   - 这两个函数分别返回总内存大小（以字节为单位）和（以 KB 为单位）。

6. `void BuddyAllocator:: _sortList (int listNo)`
   - 这个函数用于对特定大小的内存块链表进行排序。
   - 创建一个新的 LinkedList `sorted`，然后逐个将节点按顺序插入到 `sorted` 中。
   - 最后，将 `sorted` 赋值给原链表 `_blocks[listNo]`，以实现排序。

7. `int *BuddyAllocator:: _findBuddies (int listNo)`
   - 这个函数查找特定大小的内存块链表中是否存在"伙伴"块，即大小相同且相邻的块。
   - 如果找到伙伴块，返回一个包含它们在链表中的索引的整数数组。

8. `void BuddyAllocator:: _merge (int listNo)`
   - 这个函数用于合并特定大小的内存块链表中的伙伴块。
   - 使用 `_findBuddies(listNo)` 查找伙伴块，然后将它们合并为一个更大的块。
   - 调用 `_sortList(listNo + 1)` 来确保新块按顺序插入。

9. `void BuddyAllocator:: _mergeAll ()`
   - 这个函数用于在所有不同大小的内存块链表上执行合并操作，以最大程度减少碎片。
   - 遍历所有链表，连续地合并伙伴块。

10. `void BuddyAllocator:: dumpLists ()`
    - 这个函数用于打印每个不同大小的内存块链表的状态。
    - 遍历所有链表并打印每个链表的块大小和块地址。

11. `char *BuddyAllocator:: getMemoryPointer ()`
    - 这个函数返回分配的内存块的起始地址。

12. `buddy_block *BuddyAllocator:: allocate (int sizeKb)`
    - 这个函数用于分配特定大小的内存块。
    - 根据请求的块大小，找到合适的链表 `listNo`。
    - 循环查找合适的块，直到找到或遍历完所有链表。
    - 如果找到块，将其从链表中移除，并返回其起始地址和大小。

13. `void BuddyAllocator:: deallocate (buddy_block *)`
    - 这个函数用于释放内存块。
    - 根据释放的块大小找到对应的链表 `listNo`。
    - 将释放的块添加到链表中，然后调用 `_mergeAll()` 函数来合并伙伴块以减少碎片。

这些函数共同实现了 Buddy 分配器的内存管理功能，包括内存的分配和释放，以及碎片的最小化处理。
## BuddySystem
### .cpp
```cpp
#include <iostream>  
#include "BuddyAllocator.h"  
  
using namespace std;  
  
int main(int argc, char* argv[]) {  
    if (argc != 2) {  
        cout << "Usage: buddy <MemorySizeKiloBytes>\n";  
        return EXIT_SUCCESS;  
    }  
    // read memory size in kB  
    int buddySize = atoi(argv[1]);  
    BuddyAllocator ba = BuddyAllocator(buddySize);  
  
    cout << "BUDDY SYSTEM ALLOCATION\n";  
    cout << "Memory size: " << ba.getSizeKb() << " kB" << endl;  
    cout << "Min block size: " << BLOCK_MIN_SIZE_KB << " kB" << endl;  
    cout << "Memory begins at: " << (void *)(ba.getMemoryPointer()) << endl;  
    cout << "Memory status: " << endl;  
    ba.dumpLists();  
    cout << endl;  
  
    cout << "request: " << 120 << endl;  
    buddy_block *myblock = ba.allocate(120);  
    cout << "[1] start addres: " << (void *)myblock->startAddress << endl;  
    ba.dumpLists();  
    cout << endl;  
  
    cout << "request: " << 128 << endl;  
    buddy_block *myblock2 = ba.allocate(128);  
    cout << "[2] start addres: " << (void *)myblock2->startAddress << endl;  
    ba.dumpLists();  
    cout << endl;  
  
    cout << "request: " << 40 << endl;  
    buddy_block *myblock3 = ba.allocate(40);  
    cout << "[3] start addres: " << (void *)myblock3->startAddress << endl;  
    ba.dumpLists();  
    cout << endl;  
  
    cout << "request: " << 356 << endl;  
    buddy_block *myblock4 = ba.allocate(356);  
    cout << "[4] start addres: " << (void *)myblock4->startAddress << endl;  
    ba.dumpLists();  
    cout << endl;  
  
    cout << "request: " << 359 << endl;  
    buddy_block *myblock5 = ba.allocate(359);  
    cout << "[5] start addres: " << (void *)myblock5->startAddress << endl;  
    ba.dumpLists();  
    cout << endl;  
  
    cout << "free [5]: " << myblock5->sizeKb << "KB" << endl;  
    ba.deallocate(myblock5);  
    ba.dumpLists();  
    cout << endl;  
  
    cout << "free [4]: " << myblock4->sizeKb << "KB" << endl;  
    ba.deallocate(myblock4);  
    ba.dumpLists();  
    cout << endl;  
  
    cout << "free [3]: " << myblock3->sizeKb << "KB" << endl;  
    ba.deallocate(myblock3);  
    ba.dumpLists();  
    cout << endl;  
    cout << endl;  
  
    cout << "free [2]: " << myblock2->sizeKb << "KB" << endl;  
    ba.deallocate(myblock2);  
    ba.dumpLists();  
    cout << endl;  
  
    cout << "free [1]: " << myblock->sizeKb << "KB" << endl;  
    ba.deallocate(myblock);  
    ba.dumpLists();  
    cout << endl;  
  
    return EXIT_SUCCESS;  
}
```

1. **头文件和命名空间**:
   - 包括了 `<iostream>` 和 `"BuddyAllocator. h"` 头文件，后者用于引入 BuddyAllocator 的定义。
   - 使用 `using namespace std;` 将 `std` 命名空间引入，以简化对标准库的使用。

2. **main 函数**:
   - 检查命令行参数数量，如果不是 2 个，输出用法并终止程序。

3. **读取内存大小**:
   - 使用 `atoi(argv[1])` 从命令行参数中获取内存大小，以 KB 为单位，存储在 `buddySize` 变量中。

4. **创建 BuddyAllocator 对象**:
   - 创建一个 `BuddyAllocator` 对象 `ba`，并传递内存大小 `buddySize` 作为参数。

5. **输出内存信息**:
   - 打印内存大小、最小块大小、内存起始地址以及当前内存状态。

6. **内存分配和输出**:
   - 依次进行多次内存分配操作，每次分配后输出分配的块的起始地址和当前内存状态。

7. **内存释放和输出**:
   - 依次进行多次内存释放操作，每次释放后输出释放的块的大小和当前内存状态。

## Make and Run
### cmake file
```cmake
# 设置项目名称
project(Buddy_System)

# 设置要求的CMake最低版本
cmake_minimum_required(VERSION 3.0)

# 添加可执行文件
add_executable(BuddySystem 
	    BuddySystem.cpp  
        BuddyAllocator.cpp   
        LinkedList.cpp
        )

# 包含头文件目录
target_include_directories(Buddy_System PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

# 指定C++标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 添加自定义运行目标
add_custom_target(run
    COMMAND BuddySystem 2048  # 这里传递了参数，你可以根据需要修改
    DEPENDS BuddySystem
    WORKING_DIRECTORY ${CMAKE_PROJECT_DIR}
)


```

### Make
```shell
# 生成构建文件
mkdir cmake-build-debug
cd cmake-build-debug
cmake ..

# 编译、构建、运行
cmake --build . --target run

```

### Run with params
```shell
# file_name    wanted_memory_pool_size_in_KB
./buddy_system 2048
```
The example allocates and deallocates memory in the following order:  
+ 120 KB  
+ 128 KB  
+ 40 KB  
+ 356 KB  
+ 359 KB  
- 359 KB  
- 356 KB  
- 40 KB  
- 128 KB  
- 120 KB

After each allocation or deallocation the state of all free-lists is printed. An entry in a free-list ist the start address of a free block in that list.

```
BUDDY SYSTEM ALLOCATION
Memory size: 2048 kB
Min block size: 64 kB
Memory begins at: 0x109f16000
Memory status:
64:
128:
256:
512:
1024:
2048: 0x109f16000

request: 120
[1] start addres: 0x109f16000
64:
128: 0x109f35400
256: 0x109f54800
512: 0x109f93000
1024: 0x10a010000
2048:

request: 128
[2] start addres: 0x109f35400
64:
128:
256: 0x109f54800
512: 0x109f93000
1024: 0x10a010000
2048:

request: 40
[3] start addres: 0x109f54800
64: 0x109f64200
128: 0x109f73c00
256:
512: 0x109f93000
1024: 0x10a010000
2048:

request: 356
[4] start addres: 0x109f93000
64: 0x109f64200
128: 0x109f73c00
256:
512:
1024: 0x10a010000
2048:

request: 359
[5] start addres: 0x10a010000
64: 0x109f64200
128: 0x109f73c00
256:
512: 0x10a08d000
1024:
2048:

free [5]: 512KB
64: 0x109f64200
128: 0x109f73c00
256:
512:
1024: 0x10a010000
2048:

free [4]: 512KB
64: 0x109f64200
128: 0x109f73c00
256:
512: 0x109f93000
1024: 0x10a010000
2048:

free [3]: 64KB
64:
128:
256: 0x109f54800
512: 0x109f93000
1024: 0x10a010000
2048:


free [2]: 128KB
64:
128: 0x109f35400
256: 0x109f54800
512: 0x109f93000
1024: 0x10a010000
2048:

free [1]: 128KB
64:
128:
256:
512:
1024:
2048: 0x109f16000
```