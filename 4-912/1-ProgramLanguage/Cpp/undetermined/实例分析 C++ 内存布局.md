---
url: http://notes.maxwi.com/2016/06/11/cpp-memory-layout/
title: 实例分析 C++ 内存布局 | blueyi's notes
date: 2023-07-25 17:18:51
time: 1690276731604
tag: 
summary: 
---

本文所使用的环境：

* Ubuntu-14.04 x64 kernel 4.2.0-36-generic
* GCC version 4.8.4

需要用到的工具（都是系统自带的，以下是 man 的基本信息，详细信息可以直接查看 man 手册）：

* size - list section sizes and total size. 可以列出 section 大小和总的大小，能够用于粗略估计，但不适合深度研究 section 大小
* readelf - Displays information about ELF files. 显示 ELF 文件的信息，readelf 非常强大，能够显示出 ELF 非常多的信息
* objdump - display information from object files. 显示 object 文件的信息，也就是目标文件，功能与 readelf 类似，但没有 readelf 强大，有些 readelf 可以显示的信息，它无法显示

如果对编译过程不甚了解，可以参看这里[实例验证 C/C++ 源代码如何变成程序的过程](http://notes.maxwi.com/2016/06/05/source-to-program/)

关于内存布局，首先需要了解 ELF（Executable and Linkable Format）文件，因为 ELF 文件格式即是可执行文件通用格式，几乎所有 UNIX 系统的可执行文件都是采用 ELF 格式，ELF 将被加载器（loader）载入到内存中被操作系统执行，所以 ELF 中指定了可执行程序的内存布局，ELF 格式有三种不同的类型：

*   可重定位的目标文件（relocatable 或 object file），也就是编译之后用于链接的文件
*   可执行文件（Executable），链接之后可以直接运行的文件
*   共享库（Shared Object，或 Shared Library）  
    具体一个 ELF 文件是哪种类型由 ELF 文件中的 ELF Header、Section Header Table 和 Program Header Table 指定。详情可以参考清华大学的一个课件 [ELF 文件](http://learn.tsinghua.edu.cn/kejian/data/77130/138627/html-chunk/ch18s05.html)和 Wikipedia 上的 [Executable and Linkable Format](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)  
    或者参见 ·[elf-man](http://man7.org/linux/man-pages/man5/elf.5.html)，当然自己`man elf`也是一样的。  
    下面两个图 ELF 文件的布局：

![](1690276731638.png)

[ELF layout](http://notes.maxwi.com/2016/06/11/cpp-memory-layout/elf-layout.png "ELF layout")

![](1690276733102.png)

从上图中可以清楚的看到与可链接的 (linkable) 对应的是 sections，与可执行的（executable）对应的是 segments。ELF 格式文件提供了两个视角，如后面的图中所示，左边是从链接器的视角来看 ELF 文件，右边是从加载器的视角来看 ELF 文件（左右是相对的，在前面图中则是相反中），链接器把 ELF 文件看成是 Section 的集合，sections 中包含了链接和重定位的所有重要信息，可重定位的 ELF 中任意一个字节都最多对应一个 section，也可以有一些不属于任何一个 section 的孤立字节。而加载器（loader）则把 ELF 文件看成是 Segment 的集合，segments 中包含了可执行文件或共享库需要被加载到内存中的必要信息，每个 segment 中可以由一个或多个 section 组成，每个 segment 都有一个长度和一组与之关联的权限（如 read、write、execute），一个进程只有在权限允许且在 segment 中的偏移长度在 segment 指定的长度之内，才能正常引用 segment，否则将会出现 segmentation fault 的异常，关于 segments 的解释可以参看 Wikipedia 的 [Memory segmentation](https://en.wikipedia.org/wiki/Memory_segmentation)。  
简单分析一下 ELF 文件各部分的信息，引用自上面提到的清华的一个课件。

开头的 ELF Header 描述了体系结构和操作系统等基本信息，并指出 Section Header Table 和 Program Header Table 在文件中的什么位置，Program Header Table 在链接过程中用不到，所以是可有可无的，Section Header Table 中保存了所有 Section 的描述信息，通过 Section Header Table 可以找到每个 Section 在文件中的位置。右边是从加载器的视角来看 ELF 文件，开头是 ELF Header，Program Header Table 中保存了所有 Segment 的描述信息，Section Header Table 在加载过程中用不到，所以是可有可无的。从上图可以看出，一个 Segment 由一个或多个 Section 组成，这些 Section 加载到内存时具有相同的访问权限。有些 Section 只对链接器有意义，在运行时用不到，也不需要加载到内存，那么就不属于任何 Segment。注意 Section Header Table 和 Program Header Table 并不是一定要位于文件的开头和结尾，其位置由 ELF Header 指出，上图这么画只是为了清晰

目标文件需要链接器做进一步处理，所以一定有 Section Header Table；可执行文件需要加载运行，所以一定有 Program Header Table；而共享库既要加载运行，又要在加载时做动态链接，所以既有 Section Header Table 又有 Program Header Table。

这样就区别出了 ELF 文件的三种类型。  
链接器是如何将可重定位的目标程序中的 sections 映射到可执行目标程序中的 segments，详情可以参见 [Airs 上的 Linkers](http://www.airs.com/blog/archives/45)，总结来说：  
链接器从输入的可重定位的目标程序中读取 section，然后根据访问权限将所有可载入的 section 对应地写入到可执行文件中的 segments 中。也就是说 segments 可以直接与内存相映射（终于扯出了内存布局），而 setcion 则是根据访问权限与 segment 映射，通常情况下所有的只读 setcions 被映射到一个 segment 中，而所有的可写 sections 被映射到其他 segment。

下图是使用 readelf 工具读出的[实例验证 C/C++ 源代码如何变成程序的过程](http://notes.maxwi.com/2016/06/05/source-to-program/)中`main.cpp`生成的可重定位的目标程序和可执行的目标程序的一部分输出，readelf 命令是`readelf -a main.o`

![](1690276734443.png)

[REL_EXEC](http://notes.maxwi.com/2016/06/11/cpp-memory-layout/REL_EXEC.jpg "REL_EXEC")

使用 vimdiff 进行的比对，从图的 ELF Header 中可以清楚地看到前面一个 ELF 类型是可重定位的文件（REL），而后面一个是可执行的文件（EXEC）。通过 ELF 的魔数（Magic）可以看出这个 ELF 是 64 位，字节序为小端格式（后面将进行验证），更多关于 ELF 魔数可以参见

[Wikipedia](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format)

，或下图：

![](1690276735150.png)

通过上图中的 Section Headers 部分信息可以看出链接之前的可重定位的目标文件中的 section 地址段全是 0，因为这些加载地址要在链接时才会添加，所以可执行的文件中该地址都存在（当然这个地址是虚拟地址，需要根据程序运行时的实际地址进行偏移）。

通过命令`readelf --segments <input>`命令可以查看 Section 到 Segment 的映射，由于可重定位的目标文件中并不存在 segments，上面也说了可执行的目标文件有 program header 进行标识，所以该命令作用于`main.o`（可重定位的目标文件）时会提示”There are no program headers in this file.”。对于`main`（可执行的目标程序）输出如下，最下面标出了 Section 到 Segment 的映射。

```
Elf file type is EXEC (Executable file)
Entry point 0x400970
There are 9 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  PHDR           0x0000000000000040 0x0000000000400040 0x0000000000400040
                 0x00000000000001f8 0x00000000000001f8  R E    8
  INTERP         0x0000000000000238 0x0000000000400238 0x0000000000400238
                 0x000000000000001c 0x000000000000001c  R      1
      [Requesting program interpreter: /lib64/ld-linux-x86-64.so.2]
  LOAD           0x0000000000000000 0x0000000000400000 0x0000000000400000
                 0x0000000000001dc9 0x0000000000001dc9  R E    200000
  LOAD           0x0000000000001de8 0x0000000000601de8 0x0000000000601de8
                 0x00000000000002b0 0x00000000000003d0  RW     200000
  DYNAMIC        0x0000000000001e08 0x0000000000601e08 0x0000000000601e08
                 0x00000000000001f0 0x00000000000001f0  RW     8
  NOTE           0x0000000000000254 0x0000000000400254 0x0000000000400254
                 0x0000000000000044 0x0000000000000044  R      4
  GNU_EH_FRAME   0x000000000000147c 0x000000000040147c 0x000000000040147c
                 0x00000000000001bc 0x00000000000001bc  R      4
  GNU_STACK      0x0000000000000000 0x0000000000000000 0x0000000000000000
                 0x0000000000000000 0x0000000000000000  RW     10
  GNU_RELRO      0x0000000000001de8 0x0000000000601de8 0x0000000000601de8
                 0x0000000000000218 0x0000000000000218  R      1

 Section to Segment mapping:
  Segment Sections...
   01     .interp 
   02     .interp .note.ABI-tag .note.gnu.build-id .gnu.hash .dynsym .dynstr .gnu.version .gnu.version_r .rela.dyn .rela.plt .init .plt .text .fini .rodata .eh_frame_hdr .eh_frame .gcc_except_table 
   03     .init_array .fini_array .jcr .dynamic .got .got.plt .data .bss 
   04     .dynamic 
   05     .note.ABI-tag .note.gnu.build-id 
   06     .eh_frame_hdr 
   08     .init_array .fini_array .jcr .dynamic .got
```

当然可以使用`readelf --sections <input>`命令读取所有的 sections。

上面已经说了 ELF 中的 Segments 对应的就是内存的布局，所以这里要先来分析 Segments。  
Segments 主要包含以下几个部分

## [](#text-segment ".text segment").text segment

它是固定大小的只读 segment  
text segment 也称代码段（segment），它包含了编译器和汇编器提供的可执行指令。

## [](#data-segment ".data segment").data segment

它是固定大小的可读写 segment  
data segment 也称为**已初始化**（initialized）的数据段（segment），它包含已初始化的：

*   全局变量，包括全局静态变量
*   局部静态变量（即 static 声明的变量）

该 segment 的大小由源代码中相应变量所占用的大小决定，这些变量的值可以在运行时改变。

## [](#rdata-rodata-segment ".rdata/.rodata segment").rdata/.rodata segment

它是一段只读的 segment  
该 segment 存储静态的无名数据以及 const 修饰的常量，例如字符串常量，const 修饰的变量。

## [](#bss-segment ".bss segment").bss segment

它是可读写的 segment，与. data segment 相邻  
BSS segment 也称为未初始化的数据段，包含全局或静态未初始化的变量，该部分数据中的值会在程序启动时直接初始为 0，BSS 是 Block Started by Symbol 的简写。

## [](#堆（head）和栈（stack） "堆（head）和栈（stack）")堆（head）和栈（stack）

堆和栈是每个程序都有的内存区域，详情可以参见 [Data segment](https://en.wikipedia.org/wiki/Data_segment)

### [](#堆 "堆")堆

维基百科上的解释，堆起始于. bss 和. data segments 之后，从该处朝着地址空间变大的方向增长。堆通过 malloc、realloc、free 来管理，具体这三个函数的实现，在 linux 下有可能是使用 brk/sbrk 来实现，当然也有可能是通过 mmap 从虚拟内存的非连续内存中获取后给进程的虚拟地址空间的（brk/sbrk/mmap 都是 linux 下管理内存的函数）。堆被同一个进程的所有线程、共享库以及动态加载的模块共享。  
关于 heap 和 free store 的区别，SOF 上也有强烈的争论，Herb Sutter 在 [Memory Management - Part I](http://www.gotw.ca/gotw/009.htm) 中详细说明了内存管理时的几个内存区域的区别，他认为 heap 和 free store 是不同的，一个是 C 语言下的，一个是 C++ 下的。而 Bjarne Stroustrup 的说法我觉得应该更合适，在这里 [C++ : Free-store versus Heap](http://zamanbakshifirst.blogspot.com/search/label/heap)，为了避免可能无法打开该链接，将全文引在下面：

```
C++ : Free-store versus Heap

What's the difference between the heap and the free-store? The C++ Programming Language keeps on referring them interchangeably. There was as huge cry over this issue in C/C++ programmer's community in Orkut. I had to shoot a mail to Dr. Bjarne Stroustrup. Here's our conversation:

My Mail:

Dear Mr Stroustrup,

Sorry to disturb you again. You have mentioned several times in the TC++PL that 'new' allocates memory from the 'free store (or heap)'. There has been a huge cry on the C++ community at Orkut (that I am moderating) as to whether free-store is the same as heap. The argument given against is that Mr Herb Sutter has mentioned that the free-store is different from the heap:

http://www.gotw.ca/gotw/009.htm

and that global 'new' has nothing to do with the heap.

So, if so, why has TC++PL used 'free store (or heap)' instead of mentioning the use of 'heap' separately.

Waiting anxiously for the response.

Regards,
Zaman Bakshi

His Reply:

Note that Herb says: "Note about Heap vs. Free Store: We distinguish between "heap" and "free store" because the draft deliberately leaves unspecified the question of whether these two areas are related. For example, when memory is deallocated via operator delete, 18.4.1.1 states:"

In other word, the "free store" vs "heap" distinction is Herb's attempt to distinguish malloc() allocation from new allocation.

>
> So, if so, why has TC++PL used 'free store (or heap)' instead of
> mentioning the use of 'heap' separately.

Because even though it is undefined from where new and malloc() get their memory, they typically get them from exactly the same place. It is common for new and malloc() to allocate and free storage from the same part of the computer's memory. In that case, "free store" and "heap" are synonyms. I consistently use "free store" and "heap" is not a defined term in the C++ standard (outside the heap standard library algorithms, which are unrelated to new and malloc()). In relation to new, "heap" is simply a word someone uses (typically as a synonym to "free store") - usually because they come from a different language background.

My Reply:

Thank you Mr. Stroustrup, I had inferred the same thing (about using free store as general -- or better, synonym -- term) and had explained the community. But, I had been requested to reconfirm.

With warm regards,
Zaman Bakshi
```

### [](#栈 "栈")栈

首先栈是先进后出的数据结构，其位于内存的高地址空间中。  
自动变量存储在栈上，包括函数中定义的变量（也就是 {} 中的变量，但不包括 static，上面说了 static 存储在 data 段）。函数调用时的参数也会被压入发起调用的进程栈中。

下图可以形象地表示内存布局  

![](1690276735663.png)

以下这段程序的内存布局图来自 [pdf](http://www.cs.uleth.ca/~holzmann/C/system/memorylayout.pdf)

```
void func(int x, int y) {  
    int a;  
    int b[3];  
      
    ... 
}  

void main() {  
    ...
    func(72,73);  
    ...  
}
```

上述代码内存布局分析  

![](1690276737311.png)

## [](#关于堆和栈的增长方向 "关于堆和栈的增长方向")关于堆和栈的增长方向

从上面的图以及无数的解释中我们可以明显知到堆是沿低地址向高地址增长（低地址指较小的址，高地址指较大的地址，通常所说的向上增长就是指向高地址增长），而栈是沿高地址向低地址增加，栈和堆共用一块内存空间，当然它们有可能会出现重叠的情况。  
内存中将这两部分称为堆和栈是因为它们的表现类似于数据结构中的堆（heap）和栈（stack）。很多童鞋，例如我想尝试验证堆和栈的增长方式，简单地在函数内定义几个自动变量，再使用 malloc 或者 new 创建几个变量，然后来输出地址比较，这种方法是行不通的，因为存在栈帧的问题。所谓栈帧就是每个函数在每次的调用过程中都会对应一个栈帧，栈帧中保存了该函数在调用过程中所需要的所有信息，包括返回地址、局部变量等。所以两个不同的函数调用时的局部变量将保存在不同的栈帧中，所以函数调用者栈帧中的信息必然比被调用者先入栈，如果被调用函数的局部变量地址比调用者的大，则栈肯定是沿低地址向高地址增长，即向上增长，反之则是向下增长。  
使用下面这段代码分别在 windows 及 linux 下来验证栈的增长方向、栈帧内栈的增长方向以及 malloc 与 new 分配的空间是否是在同一块内存中：

```
#include <iostream>

void func2(int *first) {
    int second = 0;
    std::cout << "&fir: " << first << std::endl
              << "&sec: " << &second << std::endl;
}

void func1(void) {
    int first = 0;
    std::cout << "******stack direction******" << std::endl;
    func2(&first);
}

void func(void) {
    
    int t1 = 0;
    int t2 = 0;
    int t3 = 0;
    int t4 = 0;
    std::cout << std::endl << "*******stack*****" << std::endl;
    std::cout << &t1 << std::endl;
    std::cout << &t2 << std::endl;
    std::cout << &t3 << std::endl;
    std::cout << &t4 << std::endl;
    
    int *n5 = new int;
    int *n6 = new int;
    int *n7 = new int;
    int *n8 = new int;
    std::cout << "*******heap: new allocate*****" << std::endl;
    std::cout << n5 << std::endl;
    std::cout << n6 << std::endl;
    std::cout << n7 << std::endl;
    std::cout << n8 << std::endl;
    
    int *m5 = (int *)malloc(sizeof(int));
    int *m6 = (int *)malloc(sizeof(int));
    int *m7 = (int *)malloc(sizeof(int));
    int *m8 = (int *)malloc(sizeof(int));
    std::cout << "*******heap: malloc allocate*****" << std::endl;
    std::cout << m5 << std::endl;
    std::cout << m6 << std::endl;
    std::cout << m7 << std::endl;
    std::cout << m8 << std::endl;
    
    delete n5;
    delete n6;
    delete n7;
    delete n8;
    free(m5);
    free(m6);
    free(m7);
    free(m8);
}
int main() {
    func1();
    func();
    getchar();
    return 0;
}
```

这是在 Windows 7 x64 系统下使用 GCC 5.1.0 x64 编译运行的结果：

```
C:\Windows\system32\cmd.exe /c (mem.exe)
******stack direction******
&fir: 0x22fe1c
&sec: 0x22fddc

*******stack*****
0x22fddc
0x22fdd8
0x22fdd4
0x22fdd0
*******heap: new allocate*****
0x8f7b00
0x8f7b20
0x788ab0
0x788ac0
*******heap: malloc allocate*****
0x788ad0
0x788ae0
0x788af0
0x788b00
```

这是在 Ubuntu-14.10 x64 系统下使用 GCC 4.8.4 x64 编译运行的结果：

```
******stack direction******
&fir: 0x7ffdf88d276c
&sec: 0x7ffdf88d274c

*******stack*****
0x7ffdf88d2720
0x7ffdf88d2724
0x7ffdf88d2728
0x7ffdf88d272c
*******heap: new allocate*****
0x753010
0x753030
0x753050
0x753070
*******heap: malloc allocate*****
0x753090
0x7530b0
0x7530d0
0x7530f0
```

从以上结果可以清楚的看到不管是在 windows 下还是在 linux 下，first 的地址都比 second 的地址大，也就是说先入栈的地址比后入栈的地址大，所以栈是向下增长。  
而在一个栈帧内，即 stack 部分的输出结果可以看出 windows 下地址是减小的，而 linux 下地址却是增大，不管是由于编译器的问题还是系统的分配问题，或者都有可能（至少此处的例子编译器一样，虽然版本不同），所以在一个栈帧内并无法判断栈的增长方向。  
而根据堆的输出结果可以看出不管是 windows 下还是 linux 下，地址都是变大的，也就是向上增长，从 windows 下的输出可以看出堆分配的内存有可能不连续，而不管是在 windows 下还是 linux 下都没有显示出 new 和 malloc 分配的内存不是在同一个内存区域上，而且据说 new 就是使用 malloc 进行实现的。  
之所以要设计堆和栈两种不同的内存管理方式，根据函数的调用来说显然是栈的存在的非常有必要的，考虑到逻辑与数据的分享堆和栈也同样的非常有必要的。  
总结堆和栈的区别：

1.  分配方式不同，栈用于存储定义的自动变量、函数的返回地址、函数的参数等，由系统自动分配，而堆需要使用 malloc、realloc 或 new 手动分配
2.  回收方式不同，栈在函数调用结束或程序运行完成时由系统自动释放，而堆需要使用 delete 或 free 来手动释放，当然在程序运行结束后也有可能 OS 会自动回收
3.  增长方向不同，栈由高地址向低地址增长，而堆则是由低地址向高地址增长
4.  大小限制不同，栈可申请的空间通常在编译时由操作系统确定，程序运行之前就已经确定大小，且通常较小，而堆则可在运行时扩展，大小受限于系统剩余的虚拟内存
5.  申请效率不同，栈由系统自动分配通常效率较高，堆由于在分配和释放时需要由系统维护一个空闲内存地址的链表，容易产生碎片化的内存，在申请时，系统首先需要从空闲内存地址链表中查询空闲内存地址，找到后返回给程序并从该链表中删除，所以速度较慢。
6.  堆内存只能通过指针使用，而栈可以通过变量名
7.  栈的使用不会引起内存碎片化，但在堆上进行大量的分配和释放有可能造成内存碎片化
8.  失败后的结果不同，当分配的栈太大或者死循环，或者递归的太深时会引起栈溢出（stack overflow）。而在申请堆时，请求的内存太大会引起内存分配失败。
9.  网上有人说存取效率不同，栈的存取更快，堆较慢。这个应该根据不同的系统实现可能并不一定。

关于堆和栈的区别也可以看 SOF 上的高票 [SOF](http://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap)

从以下代码开始：

```
int main() {
    return 0;
}
```

查看它的 section 大小：

```
blueyi@vm:~/cpp/b$ g++ --save-temps -o main main.cpp
blueyi@vm:~/cpp/b$ size main.o 
   text	   data	    bss	    dec	    hex	filename
     67	      0	      0	     67	     43	main.o
```

因为没有定义任何变量，所以. data 和. bss 都是 0，现在将程序修改为如下所示：

```
int global_uninitialized;

float global_ini = 3.1415;
int main() {
    
    static int static_uninitialized;
    
    static char ch_ini = 'a';
    return 0;
}
```

查看 section 大小：

```
blueyi@vm:~/cpp/b$ g++ --save-temps -o main main.cpp
blueyi@vm:~/cpp/b$ size main.o 
   text	   data	    bss	    dec	    hex	filename
     67	      5	      8	     80	     50	main.o
```

现在. data section 大小变成了 5，刚好是一个已初始化的全局 float 占用的 4 个字节加上局部已初始化的静态 char 所占用的字节，而. bss section 的大小刚好是两个 int 占用的字节 8，它们分别来自全局未初始化的 global_uninitialized 和局部静态未初始化的 static_uninitialized。

对可执行程序执行反汇编并查看. data 中的内容如下：

```
blueyi@vm:~/cpp/b$ objdump -CS -s -j .data main

main:     file format elf64-x86-64

Contents of section .data:
 601028 00000000 00000000 00000000 00000000  ................
 601038 560e4940 61                          V.I@a 

Disassembly of section .data:

0000000000601028 <__data_start>:
	...

0000000000601030 <__dso_handle>:
	...

0000000000601038 <global_ini>:
  601038:	56 0e 49 40                                         V.I@

000000000060103c <main::ch_ini>:
  60103c:	61                                                  a
```

可以清楚地看到. data 中有`<global_ini>`和`<main::ch_ini>`，它们正是我们的两个变量 global_ini 和 ch_ini。

反汇编并查看. bss 中的内容如下：

```
blueyi@vm:~/cpp/b$ objdump -CS -s -j .bss main

main:     file format elf64-x86-64

Disassembly of section .bss:

0000000000601040 <completed.6973>:
  601040:	00 00 00 00                                         ....

0000000000601044 <global_uninitialized>:
  601044:	00 00 00 00                                         ....

0000000000601048 <main::static_uninitialized>:
	...
```

同样可以看到与我们源代码中对应的`<global_uninitialized>`和`<main::static_uninitialized>`

通过反汇编查看. rodata 中的数据如下：

```
blueyi@vm:~/cpp/b$ objdump -CS -s -j .rodata main

main:     file format elf64-x86-64

Contents of section .rodata:
 400580 01000200                             ....            

Disassembly of section .rodata:

0000000000400580 <_IO_stdin_used>:
  400580:	01 00 02 00                                         ....
```

与之前. data 的输出结果类比可以看出. rodata 中并没有数据，因为我们的程序中没有定义需要存储在. rodata section 中的数据。  
将程序修改为如下：

```
int global_uninitialized;

float global_ini = 3.1415;

const int MAX = 100;
const int MIN = 10;

int main() {
    
    static int static_uninitialized;
    
    static char ch_ini = 'a';

    
    const float pi = 3.14;
    const char *str = "maxwi.com";
    return 0;
}
```

再来查看 sections 大小，以及反汇编之后. rodata 的内容：

```
blueyi@vm:~/cpp/b$ g++ --save-temps -o main main.cpp
blueyi@vm:~/cpp/b$ size main.o
   text	   data	    bss	    dec	    hex	filename
    108	      5	      8	    121	     79	main.o
blueyi@vm:~/cpp/b$ objdump -CS -s -j .rodata main

main:     file format elf64-x86-64

Contents of section .rodata:
 400590 01000200 6d617877 692e636f 6d000000  ....maxwi.com...
 4005a0 64000000 0a000000 c3f54840           d.........H@    

Disassembly of section .rodata:

0000000000400590 <_IO_stdin_used>:
  400590:	01 00 02 00 6d 61 78 77 69 2e 63 6f 6d 00 00 00     ....maxwi.com...

00000000004005a0 <MAX>:
  4005a0:	64 00 00 00                                         d...

00000000004005a4 <MIN>:
  4005a4:	0a 00 00 00 c3 f5 48 40                             ......H@
```

从输出中可以看出. data 和. bss 的大小没有增加，而从. rodata section 的输出出我们看到了程序中定义的全局 const 变量对应的内容`<MAX>`和`<MIN>`，但没有 main 函数中的 pi 和 * str，仔细查看输出内容发现`<_IO_stdin_used>`的后面正是`maxwi.com`的十六进制 ASCII 码，紧接在`<MIN>`后面的是 3.14 的内容。

C++ 变量的内存布局中主要有. data、.bss、.rodata、heap 和 stack 五个部分。.text 属于 ELF，用于存放源代码指令。

大端（Big-Endian）和小端（Little-Endian）是指字节在内存中的存储顺序：

*   小端（Little-Endian）就是低位字节排放在内存的低地址端，高位字节排放在内存的高地址端。
*   大端（Big-Endian）就是高位字节排放在内存的低地址端，低位字节排放在内存的高地址端。  
    下图应该很能说明问题：  
    
    ![](1690276737888.png)
    
      
    与之相关的两个关键词：
*   MSB:MoST Significant Bit ——- 最高有效位
*   LSB:Least Significant Bit ——- 最低有效位

下面用代码来验证本机是大端还是小端，都知道可以通过定义一个值为 1 的 int 型变量，然后将其强制转换为 char，由于 int 占 4 个字节，而 char 只占 1 个字节，所以对于 char 来说就存在低位有效还是高位有效。默认情况下对于一个元素来说它在内存中存储是由低地址到高地址，例如一个 int 占 4 个字节，第二个字节在内存中的地址会比第一个字节大，依此类推，下面的程序中会验证。这样如果是最低位有效，即 LSB，也就是与之对应的小端，反之就是 MSB，与之对应的就是大端。参见 [Endianness-Wikipedia](https://en.wikipedia.org/wiki/Endianness) 和 [Embedded Systems programming: Little Endian/Big Endian & TCP Sockets - 2016](http://www.bogotobogo.com/Embedded/Little_endian_big_endian_htons_htonl.php)  
说再多也没用，”Talk is cheap. Show me the code.“

```
#include <iostream>
#include <cstdio>

void int_address() {
    int a = 12345;  
    char *p = (char*)(&a);
    for (std::size_t i = 0; i < sizeof(a); ++i) {
        printf("%p\t0x%.2x\n", p + i, *(p + i));
    }
}

bool is_big_end_pointer() {
    int p = 1;
    if (*((char*)&p) == 1)
        return false;
    else
        return true;
}

bool is_big_end_union() {
    union {
        int i;
        char ch;
    } endn;
    endn.i = 1;
    if (endn.ch == 1)
        return false;
    else
        return true;
}

int main(void) {
    std::cout << "address increase direction" << std::endl;
    int_address();

    std::cout << std::endl;
    std::cout << "**pointer**" << std::endl;
    if (is_big_end_pointer())
        std::cout << "big endian" << std::endl;
    else
        std::cout << "little endian" << std::endl;

    std::cout << std::endl;
    std::cout << "**union**" << std::endl;
    if (is_big_end_union())
        std::cout << "big endian" << std::endl;
    else
        std::cout << "little endian" << std::endl;

    return 0;
}
```

Windows 下的输出结果如下：

```
C:\Windows\system32\cmd.exe /c (big_and_littel_endian.exe)
address increase direction
000000000022fe0c        0x39
000000000022fe0d        0x30
000000000022fe0e        0x00
000000000022fe0f        0x00

**pointer**
little endian

**union**
little endian
```

Linux 下的输出结果如下：

```
address increase direction
0x7ffd9b54628c      0x39
0x7ffd9b54628d      0x30
0x7ffd9b54628e      0x00
0x7ffd9b54628f      0x00

**pointer**
little endian

**union**
little endian
```

显然，我机器的内存存储模式是小端格式，其他根据 int_address() 函数即能看出来单个元素在内存中的存储方向，同样可以清楚的看到低位存储在低地址，高位存储在高地址，即小端模式。  
**关于字节序的一些知识**

*   采用小端模式的处理器包括：PDP-11、VAX、Intel 系列微处理器和一些网络通信设备； 采用大端模式的处理器包括：IBM3700 系列、PDP-10、Mortolora 微处理器系列和绝大多数的 RISC 处理器 所以通常我们自己的机器多数都是小端字节序。
*   网络字节序是确定的，网络字节序定义为大端模式，所以两台主机之间通过 TCP/IP 协议进行通信的时候，在向对方发送报文前，都需要调用相应的函数把自己的主机序（Little-Endian）模式的报文转换成网络序（Big-Endian）模式；同样，在接收到对方的报文信息后，都需要将报文（网络序）转换成主机序（Little-Endian）。

最后提一下 register 定义的寄存器变量，显然上述没有提到寄存器这个存储区，因为寄存器是 CPU 的存储单元，寄存器可以直接访问而不需要通过总线，所以速度较快。早期的编程环境，特别是较老硬件下的 C 语言编程通常会考虑将使用比较频繁的变量定义为 register，以加快访问速度，但现在的 C++ 编译器通常会忽略掉 register，而是采用自己的优化策略。register 在 C 和 C++ 中的用法也有点区别：  
在 c++ 中：

1.  register 关键字无法在全局中定义变量，否则会被提示为不正确的存储类。
2.  register 关键字在局部作用域中声明时，可以用 & 操作符取地址，一旦使用了取地址操作符，被定义的变量会强制存放在内存中。  
    在 c 中:
3.  register 关键字可以在全局中定义变量，当对其变量使用 & 操作符时，只是警告 “有坏的存储类”。
4.  register 关键字可以在局部作用域中声明，但这样就无法对其使用 & 操作符。否则编译不通过。

注意. rdata 区，这是一段只读的数据区，该区域存储的常量数据通常会在编译阶段用于替换程序中相应的常量类型的变量（也就是所谓的常量折叠）  
常量折叠发生在编译阶段，而不是预编译阶段，预编译阶段只是对宏定义这类进行替换（如 #define、#include 等定义），可以参考： [实例验证 C/C++ 源代码变成程序的过程](http://notes.maxwi.com/2016/06/05/source-to-program/)  
而编译阶段的常量折叠是一种编译优化技术，会使用常量值来替换常量表达式，如：

```
const int a = 5;
int i = 2 * 3 * 6;
int ca = a;
```

编译完成之后实际上相当于：

```
int i = 36;
int ca = 5;
```

所以当我们将某个变量声明为 const 之后，虽然该变量及其值会被存储在. rdata（只读）区，编译阶段就已经确定了该变量的值，且无法修改该变量的值（实际上该变量的值在编译阶段会直接被替换为相应的值）。虽然程序运行时可以通过指针强行修改 const 变量所指向的内存区域的值，但当通过这个 const 变量来使用其值时，由于常量折叠，该值在编译完成之后就已经被替换成了实际的值（注意是编译阶段，而不是预编译阶段）

如：

```
#include <iostream>

int main(void) {
    const int a = 4;
    int *p = (int *)&a;   
    std::cout << p << " : " << *p << std::endl << std::endl;
    *p = 5;
    std::cout << &a << " : " << a << std::endl;  
    std::cout << p << " : " << *p << std::endl << std::endl;
    int *p2 = (int *)&a;  
    *p2 = 6;
    std::cout << &a << " : " << a << std::endl;  
    std::cout << p << " : " << *p << std::endl;
    std::cout << p2 << " : " << *p2 << std::endl;
    return 0;
}
```

输出结果为：

```
0x7fffce11a0ec : 4

0x7fffce11a0ec : 4
0x7fffce11a0ec : 5

0x7fffce11a0ec : 4
0x7fffce11a0ec : 6
0x7fffce11a0ec : 6
```

由于输出结果可知虽然 a 和 p、p2 的地址一样，但 a 的值并不会发生变化，因为实际上 a 的值在编译阶段就已经被替换为 4，且被固定在. rodata 区，编译器虽然为 const 定义的变量分配了地址但并不会分析存储空间。  
当对 const 定义的变量强行取地址时，此时编译器才会为这些常量分配存储空间，并会从. rodata 区域中取出它的值，并重新在内存中创建一个它的拷贝，所以第一次对 p 赋值后，它的值是 5，当通过 p2 对这块内存赋值为 6 时，p 和 p2 所指向的同一块内存值为 6，但输出中显示的 a 依然是. rodata 中存储的 4（编译阶段就已经发生了替换）。  
看如下程序：  
rodata.cpp

```
const int a5 = 33;

int main() {
    int *p = (int *)&a5;   
    *p = 6;
    int dd = a5;
    return 0;
}
```

使用`g++ -S rodata.cpp`编译生成汇编源文件 rodata.s，并使用 c++filt 对变量名 unmangling：

```
.file   "rodata.cpp"
        .text
        .globl  main
        .type   main, @function
main:
.LFB0:
        .cfi_startproc
        pushq   %rbp
        .cfi_def_cfa_offset 16
        .cfi_offset 6, -16
        movq    %rsp, %rbp
        .cfi_def_cfa_register 6
        movq    a5, -8(%rbp)  //从.rodata中取出a5的值，并拷贝到分配的内存中
        movq    -8(%rbp), %rax
        movl    $6, (%rax)
        movl    $33, -12(%rbp)  //变量dd的值直接使用33而不是a5
        movl    $0, %eax
        popq    %rbp
        .cfi_def_cfa 7, 8
        ret
        .cfi_endproc
.LFE0:
        .size   main, .-main
        .section        .rodata
        .align 4
        .type   a5, @object
        .size   a5, 4
a5:
        .long   33
        .ident  "GCC: (Ubuntu 4.8.4-2ubuntu1~14.04.3) 4.8.4"
        .section        .note.GNU-stack,"",@progbits
```

volatile 关键字用于修饰变量与 const 用法类似，但功能恰好相反。volatile 关键字修饰变量表示该变量可以被某些编译器未知的因素更改，比如：操作系统、硬件或者其它线程等。遇到这个关键字声明的变量，编译器对访问该变量的代码就不再进行优化，从而可以提供对特殊地址的稳定访问。声明时语法：int volatile vInt; 当要求使用 volatile 声明的变量的值的时候，系统总是重新从它所在的内存读取数据，即使它前面的指令刚刚从该处读取过数据。而且读取的数据立刻被保存。  
也就是说 volatile 修饰的变量，不管在什么时候通过变量名使用该变量，都要直接从内存中读取，而不允许编译器对其修饰的变量做任何编译优化，例如常量折叠。volatile 可以与 const 一起使用，const 修饰的变量，只是说不能修改该变量的值，毕竟该变量会在编译时优化替换为原值，就像上面的例子一样，依然可以通过指针来修改 const 变量所指向的内存中的内容。  
同样是上面的两段代码，如果将前面那段代码的`const int a = 4;`前面再加上 volatile，改成`volatile const int a = 4`，则输出结果为

```
0x7fffda9fdf5c : 4

1 : 5
0x7fffda9fdf5c : 5

1 : 6
0x7fffda9fdf5c : 6
0x7fffda9fdf5c : 6
```

注意这个地方在 g++ 编译时会有警告`The address of 'a' will always evaluate as 'true' [-Waddress]`，然后 a 的地址都会输出成了 1，这是因为 operator<<重载的运算符没有 volatile void * 类型的形参，默认情况下 iostream 会将指针类型隐式转换为 void * 以用于显示，但对于 volatile 类型的指针却不转换，所以上面对于 a 地址的输出，可以加个到 (void *)&a 的强制转换，当然输出地址与 p 是一样的。

如果将后面那段代码中的`const int a5 = 33;`改成`volatile const int a5 = 33;`，则编译后的汇编中可以发现，程序不再采用常量折叠的方式替换 dd 后面 a5 的值为 33，而是从 a5 中获取  
volatile 用于多线程环境较多，当两个线程都要用到某一个变量且该变量的值会被改变时，应该用 volatile 声明，该关键字的作用是防止优化编译器把变量从内存装入 CPU 寄存器中。如果变量被装入寄存器，那么两个线程有可能一个使用内存中的变量，一个使用寄存器中的变量，这会造成程序的错误执行。  
主要可用于以下三个方面：

*   中断服务程序中修改的供其它程序检测的变量需要加 volatile；
*   多任务环境下各任务间共享的标志应该加 volatile；
*   存储器映射的硬件寄存器通常也要加 volatile 说明，因为每次对它的读写都可能由不同意义。

volatile 在修饰指针时，也存在类似 const 修饰指针时的常量指针和指针常量两种情况。所以 volatile 可用于修饰指针，或修饰指针所指的对象