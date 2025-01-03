到此为止，本教程已经讲解了 C++ STL 标准库中所有容器的特性、功能以及用法，但考虑到一些读者可能在纠结“什么场景中选用哪个容器”这个问题，本节将带领大家系统回顾一下所学的这些容器，并给出一个解决此问题的思路。

值得一提的是，虽然 STL 标准库还有迭代器、算法、函数对象等，但容器仍是大多数 C++ 程序员关注的焦点。首先，和普通数组相比，容器支持动态扩容和收缩，还可以自行管理存储的元素（例如排序），同时还提供有诸多成员方法，大大提高了开发效率等等。其次，每个容器的底层实现，都采用的是精心挑选的数据结构，这意味着在使用这些容器时，不用担心它们的执行效率。

总的来说，C++ STL 标准库（以 C++ 11 为准）提供了以下几种容器供我们选择：

1. 序列式容器：array、vector、deque、list 和 forward_list；
2. 关联式容器：map、multimap、set 和 multiset；
3. 无序关联式容器：unordered_map、unordered_multimap、unordered_set 和 unordered_multiset；
4. 容器适配器：stack、queue 和 priority_queue。

> 注意，容器适配器本质上也属于容器，关于以上各个容器适配器，后续章节会做详细讲解。

上面是依据容器类型进行分类的。实际上，每个容器所具有的特性都和其底层选用的存储结构息息相关。根据容器底层采用的是连续的存储空间，还是分散的存储空间（以链表或者树作为存储结构），还可以将上面容器分为如下两类：

1. 采用连续的存储空间：array、vector、deque；
2. 采用分散的存储空间：list、forward_list 以及所有的关联式容器和哈希容器。

> 注意，这里将 deque 容器归为使用连续存储空间的这一类，是存在争议的。因为 deque 容器底层采用一段一段的连续空间存储元素，但是各段存储空间之间并不一定是紧挨着的。关于 deque 容器的底层存储结构（可阅读《[C++ STL deque底层实现原理](http://www.cdsy.xyz/computer/programme/stl/20210307/cd161510779711978.html)》一节详细了解），读者理解即可，这里不必深究。

既然 C++ STL 标准库提供了这么多种容器，在实际场景中我们应该如何选择呢？

要想选择出适用于该特定场景的最佳容器，需要综合考虑多种实际因素，例如：

- 是否需要在容器的指定位置插入新元素？如果需要，则只能选择序列式容器，而关联式容器和哈希容器是不行的；
- 是否对容器中各元素的存储位置有要求？如果没有，则可以考虑使用哈希容器，反之就要避免使用哈希容器；
- 是否需要使用指定类型的迭代器？举个例子，如果必须是随机访问迭代器，则只能选择 array、vector、deque；如果必须是双向迭代器，则可以考虑 list 序列式容器以及所有的关联式容器；如果必须是前向迭代器，则可以考虑 forward_list 序列式容器以及所有的哈希容器；
- 当发生新元素的插入或删除操作时，是否要避免移动容器中的其它元素？如果是，则要避开 array、vector、deque，选择其它容器；
- 容器中查找元素的效率是否为关键的考虑因素？如果是，则应优先考虑哈希容器。

当然，以上问题并没有涵盖所有的情形，只是起到一个抛砖引玉的作用。在实际场景中，我们需要考虑更多的因素（例如对比各个容器解决当前问题所需的时间复杂度），经过层层筛选，最终找到适合该场景的那个容器。