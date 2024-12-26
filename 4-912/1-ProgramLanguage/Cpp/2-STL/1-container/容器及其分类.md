
## 容器container
------------------------------

容器：包括序列式容器和关联式容器；即各种数据结构，如 vector，list，deque，set，map 等用来存储数据；从实现的角度来看，STL 容器是一种 class template。

任何特定的数据结构都是为了实现某种特定的算法。

![[container.png]]

| 容器种类                                                                                                                                                        | 功能                                                                                                                                                      |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------|
| 序列容器                                                                                                                                                        | 主要包括 vector 向量容器、list 列表容器以及 deque 双端队列容器。之所以被称为序列容器，是因为元素在容器中的位置同元素的值无关，即容器不是排序的。将元素插入容器时，指定在什么位置，元素就会位于什么位置。                                          |
| 排序容器                                                                                                                                                        | 包括 set 集合容器、multiset多重集合容器、map映射容器以及 multimap 多重映射容器。排序容器中的元素默认是由小到大排序好的，即便是插入元素，元素也会插入到适当位置。所以关联容器在查找时具有非常好的性能。                                        |
| 哈希容器                                                                                                                                                        | C++ 11 新加入 4 种关联式容器，分别是 unordered_set 哈希集合、unordered_multiset 哈希多重集合、unordered_map 哈希映射以及 unordered_multimap 哈希多重映射。和排序容器不同，哈希容器中的元素是未排序的，元素的位置由哈希函数确定。 |
>[! warning] HASH!
>由于哈希容器直到 C++ 11 才被正式纳入 C++ 标准程序库，而在此之前，“民间”流传着 hash_set、hash_multiset、hash_map、hash_multimap 版本，不过该版本只能在某些支持 C++ 11 的编译器下使用（如 VS），有些编译器（如 gcc/g++）是不支持的。

### 序列式容器 (sequence container

* array (C++ 提供，build-in)
* vector
* heap (内含一个 vector)
* priority-queue (内含一个 heap)
* list
* slist (非标准)
* deque
* stack (内含一个 deque) (adapter 配接器)
* queue (内含一个 deque) (adapter 配接器)

怎么理解序列式容器，其中的元素都可序 (orderable), 但未必有序 (sorted)

ordered 是容器集合被排序，可以使用指定的顺序去遍历集合。 sorted 是一个容器集合根据某些规则确定排序的。

![[sequence-container.png]]

### 关联式容器 (associative container)

*   RB-tree (非公开)
*   set (内含一个 RB-tree)
*   map (内含一个 RB-tree)
*   multiset (内含一个 RB-tree)
*   multimap (内含一个 RB-tree)
*   hashtable (非标准)
*   hash_set (内含一个 hashtable) (非标准)
*   hash_map (内含一个 hashtable) (非标准)
*   hash_multiset (内含一个 hashtable) (非标准)
*   hash_multimap (内含一个 hashtable) (非标准)

![[assosiative-container.png]]

熟悉关联式容器，需要有 [RB-tree](https://github.com/steveLauwh/Data-Structures-And-Algorithms/tree/master/Tree/RB-tree) (红黑树原理) 和 [hash table](https://github.com/steveLauwh/Data-Structures-And-Algorithms/tree/master/Hash%20Table) (哈希表原理) 基础。

### 哈希容器 (hash container)