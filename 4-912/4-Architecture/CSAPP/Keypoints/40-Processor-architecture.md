---
url: https://fengmuzi2003.gitbook.io/csapp3e/di-4-zhang-chu-li-qi-ti-xi-jie-gou
title: 第 04 章：处理器体系结构 - CSAPP 重点解读
date: 2023-09-14 23:16:47
---
第 04 章开篇第一句话就是：**现代微处理器可以称得上是人类创造出的最复杂的系统之一**。

![](1694704607805.png)

要讨论现代处理器，如此单薄的一章肯定是远远不够的，要再开设一门课程才行，实际上本章的内容隶属于计算机组成原理（计算机体系结构）的范畴，这里仅介绍一些大家必须知道的东西，或者是值得借鉴和思考的东西。

**RISC** （精简指令集） vs **CISC**（复杂指令集），如今很多 RISC 风格的处理器为了加入很多新的特性，正在变得越来越像 CISC，而反过来 CISC 风格的处理器 (比如 x86) 在具体实现的时候又转换成了类似 RISC 的微指令，因此两者的界限越来越模糊了，本质上都是“取其精华，去其糟粕”，或者说是某种程度的折中。

![](1694704610189.png)

From：MIT 6.823 Computer System Architecture

另外，这不仅仅是技术上孰优孰劣的问题，还有市场因素，**Technical elegance ≠ market success**，例如今天我们很不幸地看到 MIPS 已经开始日落西山，回顾历史，MIPS（特别优雅）在 90 年代曾经一度辉煌过，很多半导体公司都采用 MIPS 的设计来制造芯片，其生产的芯片也被 Sony 和 Nintendo 的游戏机，以及被超级计算机所使用，但是它错失了智能手机时代，由于 MIPS 的产品从设计之初就以 Intel 的 x86 为对标产品，主打高性能，反观 ARM，从诞生开始就瞄准嵌入式低功耗领域，在智能手机市场大爆发的年代，ARM 一下子走上舞台中央，而 MIPS 由于聚焦在中高端并没有功耗的优势，限制了其在智能手机上大展拳脚，另一方面 MIPS 的迟缓也导致它失去了最关键的十年， 如今 MIPS 已是几经转手，命途多舛，反观 Intel（特别不优雅），它的指令集架构并不优雅和完美，但是它的兼容性做的很好，在个人电脑和服务器领域还是占据了重大的份额，我们不得不感慨科技产业的快速变迁，正所谓 “适者生存”，类似的，Linux 并不完美，但是它同样很务实，也取得了巨大的成功。

![](1694704611204.png)

近年来，基于领域的设计也开始再次兴起，传统的商业产品（比如 x86-64）已经达不到某些公司对性能的极致需求，或者对需求的响应过于缓慢，因此很多大公司都在自行设计能够满足自身特定性能需求的软硬件，比如 Google 的 TPU，Apple 的 M1 芯片等，Patterson 和 Hennessy 称之为计算机体系结构的新黄金时代。

![](1694704613786.png)

图片：Apple 于 2020 年发布的 M1 SoC 芯片

CMU 的教授没有给出对应的视频，我也不推荐大家阅读原书第 4 章，原因：Y86-64 不是一个商业可用的指令集架构，意味着你学了之后基本上没有实用价值，另外，Y86-64 的设计并不优雅，不是特别值得学习。

**推荐教材**：计算机组成与设计: 硬件 / 软件接口：MIPS 版本 或者 RISC-V 版本，以下简称 P&H Book。

**下载链接**：如果你没有以上推荐教材的纸质书籍的话，可以跳到附录部分直接下载电子版教材。

![](1694704614876.png)

备注：Patterson 和 Hennessy 由于 RISC 方面的贡献而获得 2017 年图灵奖（相当于计算机界的诺贝尔奖）

![](1694704616811.png)

**补充说明 1**：MIPS 的芯片比较难买到，如果感兴趣的话你可以通过模拟器来学习。

**补充说明 2**：推荐学习 RISC-V 指令集，因为它是开放的指令集架构，发展迅猛，未来可期，如果你已经有了良好的基础（比如学习过 MIPS），那么你学习 RISC-V 指令集仅仅需要如下两页纸（a.k.a. Green Card）：

RISC-V Reference Data Card (Green Card).pdf

4MB

RISC-V Green Card (from P&H Book)

![](1694704619863.png)

**单周期处理器设计**：数据通路 + 实现控制逻辑 (本质 = 组合逻辑电路 + 时序逻辑电路)

**引申**：控制与数据分离的设计思想也广泛应用在其他领域，比如网络通信领域中引入了 SDN

备注：数字电路不是本课程的重点，如果不熟悉的朋友请参考 P&H Book. 附录. The Basics of Logic Design

![](1694704622147.png)

From P&H Book：组合逻辑单元，时序逻辑单元

示例：以下是简单的 RISC-V 单周期处理器的设计（MIPS 也类似），来自 P&H Book

![](1694704623212.png)

From P&H Book：single-cycle datapath

流水线技术：现代处理器借鉴了汽车生产的流水线技术，使得指令能够并行执行（ILP）

示例：以下是简单的 RISC-V 流水线处理器的设计（MIPS 也类似），来自 P&H Book

![](1694704624344.png)

From P&H Book：pipelined version of the datapath

提示：流水线并不会缩短单条指令的执行时间（甚至会增加时间）， 但是提高了指令的吞吐率。流水线深度也不是越深越好，一方面由于流水线寄存器本身会带来额外的开销，另外一方面流水线越深，由分支预测失败带来的性能损失会更大，下图是各种处理器的流水线深度和功耗对比，因此实际需要考虑折中。

![](1694704625659.png)

流水线冒险：英文称为 Harzard，指的是阻止下一条指令在下一个时钟周期开始执行的情况

**结构冒险**：所需的硬件资源正在被之前的指令工作，解决方案：等待 Stall，或者增加硬件资源，比如增加寄存器的端口数量，或者指令和数据 Cache 分开存储，请参考下图的场景 a 和场景 b 对照理解。

![](1694704626590.png)

场景 a：增加寄存器端口，场景 b：指令 / 数据分开存储

**数据冒险**：需要等待之前的指令完成写操作，解决方案：等待 Stall，或者提前转发 Forwarding，但是无法解决所有的数据依赖问题（例如需要访存的情况），请参考下图的场景 a 和场景 b 对照理解。

![](1694704627599.png)

编译器也可以进行指令调度，在一定程度上避免等待 Stall，请参考下图的场景进行理解：

![](1694704628529.png)

**控制冒险**： 需要根据之前指令的结果决定下一步的行为，解决方案：等待 Stall，分支预测（硬件实现的动态预测，比如 branch target buffer，branch history table 等机制），编译器实现的分支合并等。

提升性能：现代处理器为了进一步提升 ILP，采用了一些较为 “激进” 的设计，譬如乱序执行等

Out-of-Order Execution： 乱序执行（动态调度）：大致上包含的组件有 Register renaming，Reservation stations 等。注意：虽然表面上看起来是乱序执行，但是最终还是要确保按序递交结果（从程序员的角度看顺序是一致的）

![](1694704629483.png)

**历史**：世界上第一台乱序执行和执行超标量的计算机是 CDC 6600，超级计算机之父 Semour Cray 带领的团队在这台堪称是天神下凡一般的机器上实现了体系结构设计的两项重大突破，值得永载史册，出于对 CDC 6600 的敬意，乱序执行中用来描述指令发射的术语（issue）沿用了 CDC 6600 设计文档中的原始称呼并一直保留至今。

![](1694704632347.png)

**现代处理器实例 - 1**：Intel Core-i7 使用的 Nehalem 架构，如果看不太清楚的话可以通过下方链接下载文档

Intel-Nehalem-microarchitecture.pdf

804KB

Intel Nehalem microarchitecture

![](1694704633456.png)

source from：Mohammad Radpour & Amirali Sharifian

**现代处理器实例 - 2**：ARM Cortex-A57 的架构图，如果看不太清楚的话可以通过下方链接下载图片

arm-Cortex-A57 Block DiagramInstruction.pdf

44KB

ARM Cortex-A57 Block Diagram

![](1694704634383.png)

source from： Hiroshige Goto

![](1694704637329.png)

From：https://compas.cs.stonybrook.edu/~nhonarmand/courses/fa15/cse610/

![](1694704639296.png)

![](1694704641233.png)

*   毕尔肯大学 CS224：[计算机组成原理](https://www.bilibili.com/video/BV1Q5411E7e4/) - 这个教授讲的不错，推荐它的另一个原因是视频与教材同步
    

*   加州大学伯克利分校 CS61C：[计算机架构的伟大思想](https://www.bilibili.com/video/BV1g5411K7Z7/) - 授课教授 Krste，目前担任 RISC-V 基金会主席
    

*   卡内基梅隆大学 18-447：[计算机体系结构](https://www.bilibili.com/video/BV1PT4y1M7gM/) - 授课教授 Onur Mutlu，目前在苏黎世联邦理工学院任教
    

*   伊利诺伊大学：[异构并行编程](https://www.bilibili.com/video/BV1z541137iG/) - 胡文美教授 - 重点讲解 CUDA 编程，目前是 Nvidia 高级杰出研究科学家