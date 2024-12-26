到目前为止，本书主要关注 RISC-V 对通用计算的支持：我们引入的所有指令都在用户模式（应用程序的代码在此模式下运行）下可用。本章介绍两种新的权限模式：
- 运行最可信的代码的机器模式（machine mode），
- 以及为 Linux，FreeBSD 和 Windows 等操作系统 提供支持的监管者模式（supervisor mode）。
这两种新模式都比用和模式有着更高的权限，这也是本章标题的来源。有更多权限的模式通常可以使用权限较低的模式的所用功能，并且它们还有一些低权限模式下不可用的额外功能，例如处理中断和执行 I/O 的功能。处理器通常大部分时间都运行在权限最低的模式下，处理中断和异常时会将控制权移交到更高权限的模式。

嵌入式系统运行时（runtime）和操作系统==用这些新模式的功能来响应外部事件==，如网络数据包的到达；支持多任务处理和任务间保护；抽象和虚拟化硬件功能等。鉴于这些主题的广度，为此而编撰的全面的程序员指南会是另外一本完整的书。但我们的这一章节旨在强调 RISC-V 这部分功能的亮点。对嵌入式系统运行时和操作系统不感兴趣的程序员可以跳过或略读本章。

![[A0-Privileged-architecture-instruction.png]]
图 10.1 是 RISC-V 特权指令的图形表示，图 10.2 列出了这些指令的操作码。显然，特权架构添加的指令非常少。作为替代，几个新的**控制状态寄存器**（CSR）显示了附加的功能。

本章将 RV32 和 RV64 特权架构一并介绍。一些概念仅在整数寄存器的大小上有所不同，因此为了描述简洁，我们引入术语 XLEN 来指代整数寄存器的宽度（以位为单位）。对于 RV32，XLEN 为 32；对 RV64，XLEN 则是 64。

## 简单嵌入式系统的机器模式

机器模式（缩写为 M 模式，M-mode）是 RISC-V 中 hart（hardware thread，硬件线程）可以执行的最高权限模式。在 M 模式下运行的 hart 对内存、I/O 和一些对于启动和配置系统来说必要的底层功能有着**完全的使用权**。因此它是唯一所有标准 RISC-V 处理器都必须实现的权限模式。实际上简单的 RISC-V 微控制器仅支持 M 模式。这类系统是本节的重点。

>[! note] hart
>hart 是硬件线程 (hardware thread)的缩略 形式。我们用该术语将 它们与大多数程序员熟悉的软件线程区分开来。软件线程在 harts 上进行分时复用。大多数处理器核都只有一个 hart。

### RISC-V 异常分类
**机器模式最重要的特性是拦截和处理异常**（不寻常的运行时事件）的能力。RISC-V 将异常分为两类。
- 一类是同步异常，这类异常在指令执行期间产生，如访问了无效的存储器地址或执行了具有无效操作码的指令时。
- 另一类是中断，它是与指令流异步的外部事件，比如鼠标的单击。

RISC-V 中实现精确异常：==保证异常之前的所有指令都完整地执行了，而后续的指令都没有开始执行==（或等同于没有执行）。图 10.3 列出了触发标准异常的原因。
![[A0-Privileged-architecture-exception.png]]

#### 五种同步异常
在 M 模式运行期间可能发生的同步异常有五种：
1. **访问错误异常**：当物理内存的地址不支持访问类型时发生（例如尝试写入 ROM）。
2. **断点异常**：在执行 ebreak 指令，或者地址或数据与调试触发器匹配时发生。
3. **环境调用异常**：在执行 ecall 指令时发生。
4. **非法指令异常**：在译码阶段发现无效操作码时发生。
5. **非对齐地址异常**：在有效地址不能被访问大小整除时发生，例如地址为 0x12 的 amoadd.w。

如果你还记得第二章声明允许非对齐的 load 和 store，你可能会问为什么图 10.3 中还会有非对齐的 load 和 store 地址异常。原因有两个，首先，第六章的原子内存操作需要自然对齐的地址；其次，一些实现者选择省略对于非对齐的常规 load 和 store 的硬件支持，因为它是一个难以实现的不常用特性。没有这种硬件的处理器依赖于异常处理程序，用一系列较小的对齐 load 和 store 来模拟软件中非对齐的 load 和 store。应用程序代码并没有变得更好：虽然速度很慢，非对齐访存操作仍按预期进行，而硬件仍然很简单。或者，在更高性能的处理器中可以硬件实现非对齐的 load 和 store。这种实现上的灵活性归功于 RISC-V 允许非对齐 load 和 store 与常规 load 和 store 共用操作码。这遵照了第一章将架构和具体 实现隔离开的准则。

#### 标准中断源
有**三种标准的中断源**：软件、时钟和外部来源。
- 软件中断通过向内存映射寄存器中存数来触发，并通常用于由一个 hart 中断另一个 hart（在其他架构中称为处理器间中断机制）。
- 当 hart 的时间比较器（一个名为 mtimecmp 的内存映射寄存器）大于实时计数器 mtime 时，会触发时钟中断。
- 外部中断由平台级中断控制器（大多数外部设备连接到这个中断控制器）引发。不同的硬件平台具有不同的内存映射并且需要中断控制器的不同特性，因此用于发出和消除这些中断的机制因平台而异。

所有 RISC-V 系统的共同问题是如何处理异常和屏蔽中断，这是下一节的主题。

## 机器模式下的异常处理

### 控制状态寄存器
八个控制状态寄存器（CSR）是机器模式下异常处理的必要部分：
- mtvec（Machine Trap Vector）它保存发生异常时处理器需要跳转到的地址。
- mepc（Machine Exception PC）它指向发生异常的指令。
- mcause（Machine Exception Cause）它指示发生异常的种类。
- mie（Machine Interrupt Enable）它指出处理器目前能处理和必须忽略的中断。
- mip（Machine Interrupt Pending）它列出目前正准备处理的中断。
- mtval（Machine Trap Value）它保存了 trap 的附加信息：地址异常中出错的地址、发生非法指令异常的指令本身，对于其他异常，它的值为 0。
- mscratch（Machine Scratch）它暂时存放一个字大小的数据。
- mstatus（Machine Status）它保存全局中断使能，以及许多其他的状态，如图 10.4 所示。
	- ![[A0-Privileged-architecture-mstatus.png]]

处理器在 M 模式下运行时，**只有在全局中断使能位 mstatus.MIE 置 1 时才会产生中断**。

此外，每个中断在控制状态寄存器 mie 中都有自己的使能位。这些位在 mie 中的位置对应于 [[A0-Privileged-architecture-exception.png|图 10.3]] 中的中断代码。例如，`mie[7]` 对应于 M 模式中的时钟中断。

控制状态寄存器 mip 具有相同的布局，并且它指示当前待处理的中断。==将所有三个控制状态寄存器合在一起考虑==，如果
- `mstatus.MIE = 1`，
- `mie[7] = 1`，
- 且 `mip[7] = 1`，则可以处理机器的时钟中断。

当一个 hart 发生异常时，硬件自动经历如下的状态转换：
1. 异常指令的 PC 被保存在 mepc 中，PC 被设置为 mtvec。（对于同步异常，mepc 指向导致异常的指令；对于中断，它指向中断处理后应该恢复执行的位置。）
2. 根据异常来源设置 mcause（如图 10.3 所示），并将 mtval 设置为出错的地址或者其它适用于特定异常的信息字。
3. 把控制状态寄存器 mstatus 中的 MIE 位置零以禁用中断，并把先前的 MIE 值保留到 MPIE 中。
4. 发生异常之前的权限模式保留在 mstatus 的 MPP 域中，再把权限模式更改为 M。图 10.5 显示了 MPP 域的编码（如果处理器仅实现 M 模式，则有效地跳过这个步骤）。
	- ![[A0-Privileged-architecture-mstatus_MPP.png]]

为避免覆盖整数寄存器中的内容，中断处理程序先在最开始用 mscratch 和整数寄存器（例如 a0）中的值交换。通常，软件会让 mscratch 包含指向附加临时内存空间的指针，处理程序用该指针来保存其主体中将会用到的整数寄存器。在主体执行之后，中断程序会恢复它保存到内存中的寄存器，然后再次使用 mscratch 和 a0 交换，将两个寄存器恢复到它们在发生异常之前的值。最后，处理程序用 mret 指令（M 模式特有的指令）返回。mret 将 PC 设置为 mepc，通过将 mstatus 的 MPIE 域复制到 MIE 来恢复之前的中断使能设置，并将权限模式设置为 mstatus 的 MPP 域中的值。这基本是前一段中描述的逆操作。

![[A0-Privileged-architecture-interrupt-handler.png]]
图 10.6 展示了遵循此模式的基本时钟中断处理程序的 RISC-V 汇编代码。它只对时间比较器执行了递增操作，然后继续执行之前的任务。更实际的时钟中断处理程序可能会调用调度程序，从而在任务之间切换。它是非抢占的，因此在处理程序的过程中会禁用中断。不考虑这些限制条件的话，它就是一个只有一页的 RISC-V 中断处理程序的完整示例！

有时需要在处理异常的过程中转到处理更高优先级的中断。而 mepc， mcause，mtval 和 mstatus 这些控制寄存器只有一个副本，处理第二个中断的时候如果软件不进行一些帮助的话，这些寄存器中的旧值会被破坏，导致数据丢失。==可抢占的中断处理程序可以在启用中断之前把这些寄存器保存到内存中的栈，然后在退出之前，禁用中断并从栈中恢复寄存器==。

除了上面介绍的 mret 指令之外，M 模式还提供了另外一条指令：wfi（Wait For Interrupt）。wfi 通知处理器目前没有任何有用的工作，所以它应该进入低功耗模式，直到任何使能有效的中断等待处理，即 mie&mip ≠ 0。RISC-V 处理器以多种方式实现该指令，包括到中断待处理之前都停止时钟。有的时候只把这条指令当作 nop 来执行。因此，wfi 通常在循环内使用。

>[!note] wfi 不论全局中断使能有效与否都有用
> 如果在全局中断使能有效（mstatus. MIE = 1）时执行 wfi，然后有一个使能有效的中断等待执行，则处理器跳转到异常处理程序。
> 
> 另一方面，如果在全局禁用中断时执行 wfi，接着 一个使能有效的中断等待执行，那么处理器继续执行 wfi 之后的代码。这些代码通常会检查控制状态寄存器 mip，以决定下一步该做什么。与跳转到异常处理程序相比，这个策略可以减少中断延迟，因为不需要保存和恢复整数寄存器。

## 嵌入式系统中的用户模式和进程隔离
虽然机器模式对于简单的嵌入式系统已经足够，但它仅适用于那些整个代码库都可信的情况，因为 M 模式可以自由地访问硬件平台。更常见的情况是，不能信任所有的应用程序代码，因为不能事先得知这一点，或者它太大，难以证明正确性。因此，RISC-V 提供了保护系统免受不可信的代码危害的机制，并且为不受信任的进程提供隔离保护。

### User-mode 实现隔离
**必须禁止不可信的代码执行特权指令（如 mret）和访问特权控制状态寄存器**（如 mstatus），因为这将允许程序控制系统。这样的限制很容易实现，只要加入一种额外的权限模式：用户模式（U 模式）。==这种模式拒绝使用这些功能，并在尝试执行 M 模式指令或访问 CSR 的时候产生非法指令异常==。其它时候，U 模式和 M 模式的表现十分相似。

通过将 mstatus.MPP 设置为 U（如图 10.5 所示，编码为 0），然后执行 mret 指令，软件可以从 M 模式进入 U 模式。如果在 U 模式下发生异常，则把控制移交给 M 模式。

### 地址访问限制实现
这些不可信的代码还必须被限制只能访问自己那部分内存。实现了 M 和 U 模式的处理器具有一个叫做物理内存保护（PMP，Physical Memory Protection）的功能，允许 M 模式指定 U 模式可以访问的内存地址。PMP 包括几个地址寄存器（通常为 8 到 16 个）和相应的配置寄存器。这些配置寄存器可以授予或拒绝读、写和执行权限。当处于 U 模式的处理器尝试取指或执行 load 或 store 操作时，将地址和所有的 PMP 地址寄存器比较。如果地 址大于等于 PMP 地址 i，但小于 PMP 地址 i+1，则 PMP i+1 的配置寄存器决定该访问是否可以继续，如果不能将会引发访问异常。

![[A0-Privileged-architecture-PMP-register.png]]
图 10.7 显示了 PMP 地址寄存器和配置寄存器的布局。两者都是 CSR，地址寄存器名为 pmpaddr0 到 pmpaddrN，其中 N+1 是实现的 PMP 个数。地址寄存器右移两位，因为 PMP 以四字节为单位。

![[A0-Privileged-architecture-pmpcfg.png]]
配置寄存器密集地填充在 CSR 中以加速上下文切换，如图 10.8 所示。PMP 的配置由 R、W 和 X 位组成，他们分别对于 load，store 和 fetch 操作，还有另一个域 A，当它为 0 时禁用此 PMP，当它为 1 时启用。PMP 配置还支持其他模式，还可以加锁，`[Waterman and Asanovic 2017]` 中描述了这些功能。

## 现代操作系统的监管者模式

### PMP 方案的不足
上一节中描述的 PMP 方案对嵌入式系统的实现很有吸引力，因为它以相对较低的成本提供了内存保护，但它的一些缺点限制了它在通用计算中的使用。
- 由于 ==PMP 仅支持固定数量的内存区域，因此无法对它进行扩展从而适应复杂的应用程序==。
- 而且由于这些区域必须在物理存储中连续，因此系统可能产生存储碎片化的问题。
- 另外，PMP 不能有效地支持辅存的分页。

### Supervisor mode 的中断和异常

更复杂的 RISC-V 处理器用和几乎所有通用架构相同的方式处理这些问题：使用基于页面的虚拟内存。这个功能构成了监管者模式（S 模式）的核心，这是一种可选的权限模式，旨在支持现代类 Unix 操作系统，如 Linux，FreeBSD 和 Windows。S 模式比 U 模式权限更高，但比 M 模式低。与 U 模式一样，S 模式下运行的软件不能使用 M 模式的 CSR 和指令，并且受到 PMP 的限制。本节介绍 S 模式的中断和异常，下一节将详细介绍 S 模式下的虚拟内存系统。

#### 中断委托机制
默认情况下，发生所有异常（不论在什么权限模式下）的时候，控制权都会被移交到 M 模式的异常处理程序。但是 Unix 系统中的大多数异常都应该进行 S 模式下的系统调用。M 模式的异常处理程序可以将异常重新导向 S 模式，但这些额外的操作会减慢大多数异常的处理速度。因此，RISC-V 提供了一种**异常/中断委托机制**。通过该机制可以选择性地将中断和同步异常交给 S 模式处理，而完全绕过 M 模式。

mideleg（Machine Interrupt Delegation，机器中断委托）CSR 控制将哪些中断委托给 S 模式。与 mip 和 mie 一样，mideleg 中的每个位对应于 [[A0-Privileged-architecture-exception.png|图 10.3]] 中相同的异常。例如， `mideleg[5]` 对应于 S 模式的时钟中断，如果把它置位，S 模式的时钟中断将会移交 S 模式的异常处理程序，而不是 M 模式的异常处理程序。

#### 委托给 S 模式的中断可被软件屏蔽

**委托给 S 模式的任何中断都可以被 S 模式的软件屏蔽**。sie 和 sip CSR 是 S 模式的控制状态寄存器，他们是 mie 和 mip 的子集。它们有着和 M 模式下相同的布局，但**在 sie 和 sip 中只有与由 mideleg 委托的中断对应的位才能读写**。那些没有被委派的中断对应的位始终为零。

#### 异常委托机制
M 模式还可以通过 medeleg CSR 将同步异常委托给 S 模式。该机制类似于刚才提到的中断委托，==但 medeleg 中的位对应的不再是中断，而是图 10.3 中的同步异常编码==。例如，置上 `medeleg[15]` 便会把 store page fault（store 过程中出现的缺页）委托给 S 模式。

<mark style="background: #FF5582A6;">请注意，无论委派设置是怎样的，发生异常时控制权都不会移交给权限更低的模式</mark>。在 M 模式下发生的异常总是在 M 模式下处理。在 S 模式下发生的异常，根据具体的委派设置，可能由 M 模式或 S 模式处理，但永远不会由 U 模式处理。

#### 异常处理 CSR 
S 模式有几个异常处理 CSR：sepc、stvec、scause、sscratch、stval 和 sstatus，它们执行与 10.2 中描述的 M 模式 CSR 相同的功能。图 10.9 显示了 sstatus 寄存器的布局。监管者异常返回指令 sret 与 mret 的行为相同，但它作用于 S 模式的异常处理 CSR，而不是 M 模式的 CSR。
![[A0-Privileged-architecture-sstatus.png]]
> [! note]
> S 模式不直接控制时钟中断和软件中断，而是使用 ecall 指令请求 M 模式设置定时器或代表它发送处理器间中断。该软件约定是监管者二进制接口(Supervisor Binary Interface)的一部分。

S 模式处理异常的行为已和 M 模式非常相似。如果 hart 接受了异常并且把它委派给了 S 模式，则硬件会原子地经历几个类似的状态转换，其中用到了 S 模式而不是 M 模式的 CSR：
1. 发生异常的指令的 PC 被存入 sepc，且 PC 被设置为 stvec。 
2. scause 按图 10.3 根据异常类型设置，stval 被设置成出错的地址或者其它特定异常的信息字。
3. 把 sstatus CSR 中的 SIE 置零，屏蔽中断，且 SIE 之前的值被保存在 SPIE 中。 
4. 发生异常时的权限模式被保存在 sstatus 的 SPP 域，然后设置当前模式为 S 模式。

## 基于分页的虚拟内存

S 模式提供了一种传统的虚拟内存系统，它将内存划分为固定大小的页来进行地址转换和对内存内容的保护。启用分页的时候，大多数地址（包括 load 和 store 的有效地址和 PC 中的地址）都是虚拟地址。要访问物理内存，它们必须被转换为真正的物理地址，这通过遍历一种称为页表的 radix-tree 实现。**页表中的叶节点指示虚地址是否已经被映射到了真正的物理页面**，如果是，则指示了哪些权限模式和通过哪种类型的访问可以操作这个页。访问未被映射的页或访问权限不足会导致页错误异常（page fault exception）。

![[A0-Privileged-architecture-pte-radixtree.png]]

### SV32

RISC-V 的分页方案以 SvX 的模式命名,其中 X 是以位为单位的虚拟地址的长度。RV32 的分页方案 Sv32 支持 4GiB 的虚址空间，这些空间被划分为 2^10 个 4 MiB 大小的巨页。每个巨页被进一步划分为 2^10个 4 KiB 大小的基页（分页的基本单位）。因此，Sv32 的页表是基数为 2^10的两级树结构。页表中每个项的大小是四个字节，因此页表本身的大小是 4 KiB。页表的大小和每个页的大小完全相同，这样的设计简化了操作系统的内存分配。

![[A0-Privileged-architecture-rv32-sv32.png]]
图 10.10 显示了 Sv32 页表项（page-table entry，PTE）的布局，从右到左分别包含如下所述的域：
- V 位决定了该页表项的其余部分是否有效（V = 1 时有效）。若 V = 0，则任何遍历到此页表项的虚址转换操作都会导致页错误。
- R、W 和 X 位分别表示此页是否可以读取、写入和执行。如果这三个位都是 0，那么这个页表项是指向下一级页表的指针，否则它是页表树的一个叶节点。
- U 位表示该页是否是用户页面。若 U = 0，则 U 模式不能访问此页面，但 S 模式可以。若 U = 1，则 U 模式下能访问这个页面，而 S 模式不能。
> 这里说的很不准确。请参考原版文档：
> If the SUM bit in the sstatus register is set, supervisor mode software may also access pages with U=1. However, supervisor code normally operates with the SUM bit clear, in which case, supervisor code will fault on accesses to user-mode pages.

- G 位表示这个映射是否对所有虚址空间有效，硬件可以用这个信息来提高地址转换的性能。这一位通常只用于属于操作系统的页面。
- A 位表示自从上次 A 位被清除以来，该页面是否被访问过。
- D 位表示自从上次清除 D 位以来页面是否被弄脏（例如被写入）。
- RSW 域留给操作系统使用，它会被硬件忽略。
- PPN 域包含物理页号，这是物理地址的一部分。若这个页表项是一个叶节点，那么 PPN 是转换后物理地址的一部分。否则 PPN 给出下一节页表的地址。（图 10.10 将 PPN 划分为两个子域，以简化地址转换算法的描述。）

> [! note] OS 的页面置换
> 操作系统依赖于A位和 D 位来决定将哪些页面交换到辅存。定期清除 A 位有助于 OS 判断哪些页面是最近最少使用的。置上 D 位表示换出该页面的成本更高，因为它必须写回辅存。

### SV39

RV64 支持多种分页方案，但我们只介绍最受欢迎的一种，Sv39。Sv39 使用和 Sv32 相同的 4 KiB 大的基页。页表项的大小变成 8 个字节，所以它们可以容纳更大的物理地址。为了保证页表大小和页面大小一致，树的基数相应地降到 2^9，树也变为三层。Sv39 的 512 GiB 地址空间划分为 2^9个 1 GiB 大小的吉页。每个吉页被进一步划分为 2^9 个巨页。在 Sv39 中这些巨页大小为 2 MiB，比 Sv32 中略小。每个巨页再进一步分为 2^9个 4 KiB 大小 的基页。

![[A0-Privileged-architecture-rv64-sv39.png]]
图 10.11 显示了 Sv39 页表项的布局。它和 Sv32 完全相同，只是 PPN 字段被扩展到了 44 位，以支持 56 位的物理地址，或者说 2^26 GiB 大小的物理地址空间。

> [! note] 未被使用的地址位
> 由于 Sv39 的虚拟地址比 RV64 整数寄存器要短，可能你想知道剩下的 35 位是什么。Sv39 要求地址位 63-39 是第 38 位的副本。因此有效的虚拟地址是 0000_0000_0000_0000hex0000_003f_ffff_ffffhex 和 ffff_ffc0_0000_0000hex-ffff_ffff_ffff_ffffhex。
> 
> 这两个区间之间间隔的大小是两个区间长度大小的 2^25 倍，看上去似乎浪费了 64 位寄存器可以表达范围的 99.999997%。为什么不充分地利用这额外的 25 位空间呢？答案是，随着程序的增长，它们可能会需要大于 512 GiB 的虚址空间。而架构师希望在不破坏向后兼容性的前提下增加地址空间。如果我们允许程序在高 25 位中存储额外的数据，那么以后就不可能把这些位回收从而存储更大的地址。像这样允许在未使用的地址位中存储数据的严重错误，在计算机的历史中已经重复出现了多次。

### SATP: 控制分页
一个叫 satp（Supervisor Address Translation and Protection，监管者地址转换和保护）的 S 模式控制状态寄存器控制了分页系统。

![[A0-Privileged-architecture-satp-csr.png]]
如图 10.12 所示，satp 有三个域。
- MODE 域可以开启分页并选择页表级数，图 10.13 展示了它的编码。
	- ![[A0-Privileged-architecture-satp-MODE.png]]
- ASID（Address Space Identifier， 地址空间标识符）域是可选的，它可以用来降低上下文切换的开销。
- 最后，PPN 字段保存了根页表的物理地址，它以 4 KiB 的页面大小为单位。

通常 M 模式的程序在第一次进入 S 模式之前会把零写入 satp 以禁用分页，然后 S 模式的程序在初始化页表以后会再次进行 satp 寄存器的写操作。

### 虚拟地址到物理地址的转换
当在 satp 寄存器中启用了分页时，S 模式和 U 模式中的虚拟地址会以从根部遍历页表的方式转换为物理地址。

![[A0-Privileged-architecture-virtual-addr-to-physical-addr.png]]
图 10.14 描述了这个过程：
1. `satp.PPN` 给出了一级页表的基址，`VA[31:22]` 给出了一级页号，因此处理器会读取位于地址(`satp.PPN × 4096 + VA[31: 22] × 4`)的页表项。 
2. 该 PTE 包含二级页表的基址，`VA[21:12]` 给出了二级页号，因此处理器读取位于地址(`PTE.PPN × 4096 + VA[21: 12] × 4`)的叶节点页表项。
3. 叶节点页表项的 PPN 字段和页内偏移（原始虚址的最低 12 个有效位）组成了最终结果：物理地址就是 (`LeafPTE.PPN × 4096 + VA[11: 0]`)

随后处理器会进行物理内存的访问。Sv39 的转换过程几乎和 Sv32 相同，区别在于其具有较大的 PTE 和更多级页表。本章末尾的图 10.19 给出了页表遍历算法的完整描述，详细说明了异常条件和超页面转换的特殊情况。

除了一点以外，我们几乎讲完了 RISC-V 分页系统的所有内容。如果所有取指，load 和 store 操作都导致多次页表访问，那么分页会大大地降低性能！所有现代的处理器都用地址转换缓存（通常称为 TLB，全称为 Translation Lookaside Buffer）来减少这种开销。为了降低这个缓存本身的开销，大多数处理器不会让它时刻与页表保持一致。这意味着如果操作系统修改了页表，那么这个缓存会变得陈旧而不可用。S 模式添加了另一条指令来解决这个问题。这条 `sfence.vma` 会通知处理器，软件可能已经修改了页表，于是处理器可以相应地刷新转换缓存。它需要两个可选的参数，这样可以缩小缓存刷新的范围。
- 一个位于 rs1，它指示了页表哪个虚址对应的转换被修改了；
- 另一个位于 rs2，它给出了被修改页表的进程的地址空间标识符（ASID）。

如果两者都是 x0，便会刷新整个转换缓存。

> [! note] `sfence.vma` 的作用就是刷新TLB
> The `SFENCE.VMA` is used to flush any local hardware caches related to address translation.
> 
> It is specified as a fence rather than a TLB flush to provide cleaner semantics with respect to which instructions are affected by the flush operation and to support a wider variety of dynamic caching structures and memory-management schemes. 
> 
> `SFENCE.VMA` is also used by higher privilege levels to synchronize page table writes and the address translation hardware.

>[!note] 多处理器中的地址转换缓存一致性 
> `sfence.vma` 仅影响执行当前指令的 hart 的地址转换硬件。当 hart 更改了另一个 hart 正在使用的页表时，前一个 hart 必须用处理器间中断来通知后一个 hart，他应该执行 `sfence.vma` 指令。这个过程通常被称为 TLB shootdown。

![[A0-Privileged-architecture-int-register.png]]

### 虚拟地址到物理地址的转换算法

A virtual address va is translated into a physical address pa as follows:
1. Let *a* be `satp.ppn × PAGESIZE`, and `let i = LEVELS − 1`. (For Sv32, PAGESIZE= $2^{12}$ and LEVELS=2.) The `satp` register must be active, i.e., the effective privilege mode must be S-mode or U-mode. 
2. Let *pte* be the value of the PTE at address `a+va.vpn[i]×PTESIZE`. (For Sv32, PTESIZE=4.) If accessing *pte* violates a PMA or PMP check, raise an access-fault exception corresponding to the original access type.
3. If `pte.v = 0`, or if `pte.r = 0` and `pte.w = 1`, or if any bits or encodings that are reserved for future standard use are set within *pte*, stop and raise a page-fault exception corresponding to the original access type.
4. Otherwise, the PTE is valid. If `pte.r = 1` or `pte.x = 1`, go to step 5. Otherwise, this PTE is a pointer to the next level of the page table. Let `i = i − 1`. If `i < 0`, stop and raise a page-fault exception corresponding to the original access type. Otherwise, let `a = pte.ppn × PAGESIZE` and go to step 2.
5. A leaf PTE has been found. Determine if the requested memory access is allowed by the `pte.r`, `pte.w`, `pte.x`, and `pte.u` bits, given the current privilege mode and the value of the SUM and MXR fields of the `mstatus` register. If not, stop and raise a page-fault exception corresponding to the original access type.
6. If `i > 0` and `pte.ppn[i − 1 : 0] != 0`, this is a misaligned superpage; stop and raise a page-fault exception corresponding to the original access type.
7. If `pte.a = 0`, or if the original memory access is a store and `pte.d = 0`, either raise a page-fault exception corresponding to the original access type, or:
	- If a store to *pte* would violate a PMA or PMP check, raise an access-fault exception corresponding to the original access type
	- Perform the following steps atomically: 
		- Compare *pte* to the value of the PTE at address `a + va.vpn[i] × PTESIZE`.
		- If the values match, set `pte.a` to 1 and, if the original memory access is a store, also set `pte.d` to 1. 
		- If the comparison fails, return to step 2
8. The translation is successful. The translated physical address is given as follows: 
	- `pa.pgoff = va.pgoff`.
	- If `i > 0`, then this is a superpage translation and `pa.ppn[i − 1 : 0] = va.vpn[i − 1 : 0]`.
	- `pa.ppn[LEVELS − 1 : i] = pte.ppn[LEVELS − 1 : i]`.

All implicit accesses to the address-translation data structures in this algorithm are performed using width PTESIZE.