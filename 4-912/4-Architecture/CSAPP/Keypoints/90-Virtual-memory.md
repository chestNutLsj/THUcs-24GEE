假装我们有用不完的物理内存

​[视频解说](https://www.bilibili.com/video/BV1RK4y1R7Kf?p=9)​

## 导读

**面试官：请解释一下什么是虚拟内存？** 有些面试官喜欢在面试的时候考察一下，或多或少能够反映出候选人的计算机基础素养，实际上内存管理（包含虚拟内存）的议题非常复杂，不是三言两语能讲清楚的，但至少你要做到心中有数，有宏观的整体把握，不至于一问三不知。

推荐你认真阅读一下Ulrich Drepper撰写的长达114页的经典论文：What Every Programmer Should Know About Memory，如果你实在没有耐心看完它，或者想了解其中的重点内容，那么也可以通过观看我自己录制的小视频来了解其中的重点内容：

​[每个程序员都应该知道的内存知识 (第3部分：虚拟内存)](https://www.bilibili.com/video/BV1Xy4y1b7SK?p=3)​

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MZ6eDXmxbk3PBQbymGD%2F-MZ6fcyQWQ-T6i2-L9N6%2Fimage.png?alt=media&token=3f1a1c52-6e4c-4164-a125-63ffdef497ad)

虚拟内存属于 **操作系统 + 硬件** 范畴，首先，并非所有系统都使用虚拟内存，简单的（或内存受限的）系统就有可能使用单一物理内存地址空间。

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MXq2q2OuFPvJF-pj2Xr%2F-MXq3rbvp2kJiuMlrirP%2F%E6%8D%95%E8%8E%B71_%E5%89%AF%E6%9C%AC.png?alt=media&token=39f390cd-bb5f-49b4-b58a-e6eb32a7c897)

**​**[**物理内存有什么问题？**](https://www.bilibili.com/video/BV1XB4y1A7vm?p=5) 1. 内存空间不够 2. 产生内存碎片 3. 没有内存保护

（虚拟内存闪亮登场，本质就是增加一个中间层，计算机科学的任何问题都可以通过增加间接层解决！）

**虚拟内存有什么优点？** 1**.** 可以使用磁盘交换空间 2. 虚拟地址到物理地址映射灵活 3. 进程地址空间隔离

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MXq4kTdu0dEP-EX0MnF%2F-MXq76Vvpc0JV_J76GyC%2F%E6%8D%95%E8%8E%B73_%E5%89%AF%E6%9C%AC.png?alt=media&token=7b917344-626c-4f29-a823-3bc5568b6d85)

**相关的一些基本概念：**

虚拟地址到物理地址的转换涉及哪些硬件/软件/过程？ （MMU & TLB & Page Fault & Lazy allocation）

Lazy allocation：当程序向操作系统申请动态内存时，系统会调用相应的函数分配内存，但是这种分配并不是实时的，内核会去修改页表 Page Table，但是只有当用户真的开始使用这块内存时，才会分配物理页面（通过Page Fault 来触发）。

​

**Linux Kernel Space** (kmalloc, vmalloc) **& User Space** (malloc)

动态内存管理是很重要的功能，内存一直都是很宝贵的资源，一个好的内存管理策略可以极大地提升系统性能，就用户空间的动态内存管理而言，主要的实现有：ptmalloc，tcmalloc，jemalloc ...

`GLIBC malloc` 内部通过 [`brk`](http://man7.org/linux/man-pages/man2/sbrk.2.html) 或 [`mmap`](http://man7.org/linux/man-pages/man2/mmap.2.html) 系统调用向内核申请堆内存

==> [glibc源码一瞥](https://elixir.bootlin.com/glibc/latest/source/malloc/malloc.c) （相关的补充参考资料： [Understanding glibc malloc](https://sploitfun.wordpress.com/2015/02/10/understanding-glibc-malloc/)）

目前glibc的实现是ptmalloc2，它使用 chunk 作为内存管理的基本单元（Allocated chunk，Free chunk，Top chunk，Last Remainder chunk），采用边界标记法，用户 free 掉的内存并不是都会马上归还给系统，ptmalloc 会统一管理空闲的 chunk，当用户进行下一次分配请求时，ptmalloc 会首先试图在空闲的 chunk 中挑选一块给用户，这样就避免了频繁的系统调用，降低了内存分配的开销，ptmalloc 将相似大小的 chunk 用链表链接起来，这样的一个链表被称为一个 bin，ptmalloc 按照大小维护了多个类型的bin（fast bin，small bin，large bin，unsorted bin，除了fast bin是单向链表之外，其他的都是双向链表）... 另外我们还可以通过mallopt()来改变相关的内存分配行为的参数。

***** 案例参考（遇到的问题 & 解决的方法）**：[阿里（华庭=庄明强）：Glibc内存管理ptmalloc源代码分析](https://paper.seebug.org/papers/Archive/refs/heap/glibc%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86ptmalloc%E6%BA%90%E4%BB%A3%E7%A0%81%E5%88%86%E6%9E%90.pdf)​

## 学习方式

​[CMU教授的视频教程 - Lecture17：虚拟内存概念](https://www.bilibili.com/video/BV1a54y1k7YE?p=21)​

​[CMU教授的视频教程 - Lecture18：虚拟内存系统](https://www.bilibili.com/video/BV1a54y1k7YE?p=22)​

​[CMU教授的视频教程 - Lecture19：动态内存分配（基础）](https://www.bilibili.com/video/BV1a54y1k7YE?p=23)​

​[CMU教授的视频教程 - Lecture20：动态内存分配（进阶）](https://www.bilibili.com/video/BV1a54y1k7YE?p=24)​

## 实验解读

自己动手写一个内存分配器（Memory Allocator）== 绝非易事！

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MbO_K4NNn_YesNhZIfQ%2F-MbO_MMeVutniCZDdD3R%2Fimage.png?alt=media&token=6276438f-1aad-4901-91ac-40ebc2622be0)

​[CMU助教的视频：Malloc Lab](https://www.bilibili.com/video/BV1a54y1k7YE?p=25) （其他的补充教程： [Malloc tutorial](https://danluu.com/malloc-tutorial/)）

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MbO_R4Kn-zU2_dBb2cz%2F-MbOmvhfRF_Sd7hgmW1S%2Fimage.png?alt=media&token=eed3f69e-2dd4-4eab-8a93-79a3a2eb9843)

​[CMU助教的视频：Debugging Malloc Lab](https://www.bilibili.com/video/BV1a54y1k7YE?p=26) （补充：[如何以聪明的方式提问（Eric Steven Raymond）](http://www.catb.org/~esr/faqs/smart-questions.html)）

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MbOn-gVdiXmn7-S9oOb%2F-MbP0Ax2G2BDVW6a24BT%2Fimage.png?alt=media&token=dd1415cd-9668-45ce-8e6f-8b617cca9a0d)

## 延伸阅读

- ​[Linux专题介绍：内存管理](https://www.bilibili.com/video/BV1XB4y1A7vm/)