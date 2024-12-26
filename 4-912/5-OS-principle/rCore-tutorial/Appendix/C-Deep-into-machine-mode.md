## RustSBI

RISC-V 指令集的 SBI 标准规定了类 Unix 操作系统之下的运行环境规范。这个规范拥有多种实现，RustSBI 是它的一种实现。

RISC-V架构中，存在着定义于操作系统之下的运行环境。这个运行环境不仅将引导启动RISC-V下的操作系统， 还将常驻后台，为操作系统提供一系列二进制接口，以便其获取和操作硬件信息。 RISC-V给出了此类环境和二进制接口的规范，称为“操作系统二进制接口”，即“SBI”。

SBI的实现是在M模式下运行的特定于平台的固件，它将管理S、U等特权上的程序或通用的操作系统。

RustSBI项目发起于鹏城实验室的“rCore代码之夏-2020”活动，它是完全由Rust语言开发的SBI实现。 现在它能够在支持的RISC-V设备上运行rCore教程和其它操作系统内核。

RustSBI项目的目标是，制作一个从固件启动的最小Rust语言SBI实现，为可能的复杂实现提供参考和支持。 RustSBI也可以作为一个库使用，帮助更多的SBI开发者适配自己的平台，以支持更多处理器核和片上系统。

当前项目实现源码：[https://github.com/rustsbi/rustsbi](https://github.com/rustsbi/rustsbi)

## BIOS 与 SBI 的关系
SBI 是 RISC-V Supervisor Binary Interface 规范的缩写，OpenSBI 是 RISC-V 官方用 C 语言开发的 SBI 参考实现；RustSBI 是用 Rust 语言实现的 SBI。

BIOS 是 Basic Input/Output System，作用是引导计算机系统的启动以及硬件测试，并向OS提供硬件抽象层。

机器上电之后，会从ROM中读取引导代码，引导整个计算机软硬件系统的启动。而整个启动过程是分为多个阶段的，现行通用的多阶段引导模型为：

ROM -> LOADER -> RUNTIME -> BOOTLOADER -> OS

Loader 要干的事情，就是内存初始化，以及加载 Runtime 和 BootLoader 程序。而Loader自己也是一段程序，常见的Loader就包括 BIOS 和 UEFI，后者是前者的继任者。

Runtime 固件程序是为了提供运行时服务（runtime services），它是对硬件最基础的抽象，对OS提供服务，当我们要在同一套硬件系统中运行不同的操作系统，或者做硬件级别的虚拟化时，就离不开Runtime服务的支持。SBI就是RISC-V架构的Runtime规范。

BootLoader 要干的事情包括文件系统引导、网卡引导、操作系统启动配置项设置、操作系统加载等等。常见的 BootLoader 包括GRUB，U-Boot，LinuxBoot等。

而 BIOS/UEFI 的大多数实现，都是 Loader、Runtime、BootLoader 三合一的，所以不能粗暴的认为 SBI 跟 BIOS/UEFI 有直接的可比性。

如果把BIOS当做一个泛化的术语使用，而不是指某个具体实现的话，那么可以认为 SBI 是 BIOS 的组成部分之一。

也可参考这份文稿《[An Introduction to RISC-V Boot Flow](https://riscv.org/wp-content/uploads/2019/12/Summit_bootflow.pdf)》的P5, P7, P9-11。

题外话：  
计算机最重要的思想之一就是分层抽象，在任意两层之间，还可以按照设计者的意愿再次添加抽象层。而软件架构的设计和实现，是为了解决现实世界的具体问题，会面临资源、财力、物力、人力、时间等多种因素的掣肘，就会诞生一些“不那么规矩”、“不那么单纯”的架构或组件/软件，它们往往会跨层次，跨模块，大模块拆小，小模块合并，甚至打破一些“金科玉律”等等。

所以，相比于弄懂一个名词，更多的精力应该放在理解事物的本质上，只要把解决问题的流程和方法弄明白了，解决问题的过程中所用到的子流程、工具、方法，你爱怎么叫怎么叫，甚至自己发明名词也可以（只是与外人沟通可能会不太顺畅）。