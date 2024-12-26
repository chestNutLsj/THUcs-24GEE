​[视频解说](https://www.bilibili.com/video/BV1RK4y1R7Kf?p=7)​
## 导读

链接（以及装载）处于灰色地带，绝大部分人对它都是懵懂而已，但它们都是非常重要的系统软件。

## 学习方式

​[CMU教授的视频教程 - Lecture13：链接](https://www.bilibili.com/video/BV1a54y1k7YE?p=17)​

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MZ7Jrb8VFOJCabFE0SJ%2F-MZ7MouSC96vPjPUCsc0%2F%E6%8D%95%E8%8E%B71.PNG?alt=media&token=0c230456-0de7-45bf-8995-db12b405dfac)

静态链接

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MZ7Jrb8VFOJCabFE0SJ%2F-MZ7MvHgC_RAvjG4n-E3%2F%E6%8D%95%E8%8E%B72.PNG?alt=media&token=e28d2798-646d-411a-8709-c7b350c6c48b)

动态链接

这里推荐两本书：左边这本是比较经典的：**链接与装载 （英文版）**，初学者可能会觉得晦涩难懂。

右边这本书：**程序员的自我修养 -- 链接, 装载与库**，主要介绍系统软件的运行机制和原理，涉及在Windows和Linux两个系统平台上，一个应用程序在编译、链接和运行时刻所发生的各种事项，包括：代码指令是如何保存的，库文件如何与应用程序代码静态链接，应用程序如何被装载到内存中并开始运行，动态链接如何实现，C/C++运行库的工作原理，以及操作系统提供的系统服务是如何被调用的。每个技术专题都配备了大量图、表和代码实例，力求将复杂的机制以简洁的形式表达出来，我认为算是一部国产佳品。

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-M_Qb47yrcXSntm9rokh%2F-M_QcfTKJG610BM9gm4Q%2Fimage.png?alt=media&token=e0599514-9c37-4a55-bda2-ea897feeac94)

## 重点解读

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MaM7MDLJwZ53qJpXUHf%2F-MaM8N6rdF1ssYJWlYkj%2Fimage.png?alt=media&token=3231a252-0a2f-4f4d-8c56-679420833e7b)

Bin Yang - Senoir Android Framework Architect at Intel Corporation（from slideshare）

## 重点示例

**LD_PRELOAD**是Linux系统的一个环境变量，它可以影响程序的运行时的链接（Runtime linker），它允许你定义在程序运行前优先加载的动态链接库，一方面，我们可以以此功能来使用自己的或是更好的函数（比如，你可以使用Google开发的tcmalloc来提升效率），而另一方面，我们也可以向别人的程序注入程序，从而达到特定的目的。

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MaLyclZPXFKxyaUY2UL%2F-MaM0_MiG9w3YNQ6ACPp%2Fimage.png?alt=media&token=a4a1df7c-1d26-40e4-8cd0-1c014e65c42a)

**链接器：** GNU **ld** vs Google **gold** vs LLVM **lld** : [https://gcc.gnu.org/onlinedocs/gcc/Link-Options.html](https://gcc.gnu.org/onlinedocs/gcc/Link-Options.html) ​

![](https://1484576603-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MV9vJFv4kmvRLgEog6g%2F-MaMUZaJEbPYgE2GEWFq%2F-MaMVu00PKv7nyg4bB87%2Fimage.png?alt=media&token=58a9622e-a94b-4d1a-816e-d2b6d39b7b3b)

https://av.tib.eu/media/44657

## 延伸阅读

- 本人拙作（视频讲座）：[Linux环境下：程序的链接, 装载和库](https://www.bilibili.com/video/BV1hv411s7ew/) （比较详细，探讨了很多细节）