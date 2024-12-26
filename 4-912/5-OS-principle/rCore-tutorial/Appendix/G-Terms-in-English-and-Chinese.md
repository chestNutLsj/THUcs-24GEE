## 第一章：RV64 裸机应用[]( https://rcore-os.cn/rCore-Tutorial-Book-v3/terminology.html#rv64 "永久链接至标题")

|中文|英文|出现章节|
|---|---|---|
|执行环境|Execution Environment|[应用程序运行环境与平台支持](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/1app-ee-platform.html#term-execution-environment)|
|系统调用|System Call|[应用程序运行环境与平台支持](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/1app-ee-platform.html#term-system-call)|
|指令集体系结构|ISA, Instruction Set Architecture|[应用程序运行环境与平台支持](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/1app-ee-platform.html#term-isa)|
|抽象|Abstraction|[应用程序运行环境与平台支持](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/1app-ee-platform.html#term-abstraction)|
|平台|Platform|[应用程序运行环境与平台支持](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/1app-ee-platform.html#term-platform)|
|目标三元组|Target Triplet|[应用程序运行环境与平台支持](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/1app-ee-platform.html#term-target-triplet)|
|裸机平台|Bare-Metal|[应用程序运行环境与平台支持](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/1app-ee-platform.html#term-bare-metal)|
|交叉编译|Cross Compile|[移除标准库依赖](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/2remove-std.html#term-cross-compile)|
|物理地址|Physical Address|[内核第一条指令（原理篇）](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#term-physical-address)|
|物理内存|Physical Memory|[内核第一条指令（原理篇）](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#term-physical-memory)|
|引导加载程序|Bootloader|[内核第一条指令（原理篇）](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#term-bootloader)|
|控制流|Control Flow|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-control-flow)|
|函数调用|Function Call|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-function-call)|
|源寄存器|Source Register|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-source-register)|
|立即数|Immediate|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-immediate)|
|目标寄存器|Destination Register|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-destination-register)|
|伪指令|Pseudo Instruction|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-pseudo-instruction)|
|上下文|Context|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter0/3os-hw-abstract.html#term-context)|
|活动记录|Activation Record|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-activation-record)|
|保存/恢复|Save/Restore|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-save-restore)|
|被调用者保存|Callee-Saved|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-callee-saved)|
|调用者保存|Caller-Saved|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-caller-saved)|
|开场白|Prologue|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-prologue)|
|收场白|Epilogue|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-epilogue)|
|调用规范|Calling Convention|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-calling-convention)|
|栈/栈指针/栈帧|Stack/Stack Pointer/Stackframe|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-stack)|
|后入先出|LIFO, Last In First Out|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-lifo)|
|段|Section|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#term-section)|
|内存布局|Memory Layout|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#term-memory-layout)|
|堆|Heap|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#term-heap)|
|编译器|Compiler|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#term-compiler)|
|汇编器|Assembler|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#term-assembler)|
|链接器|Linker|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#term-linker)|
|目标文件|Object File|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/3first-instruction-in-kernel1.html#term-object-file)|
|链接脚本|Linker Script|[为内核支持函数调用](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/4first-instruction-in-kernel2.html#term-linker-script)|
|可执行和链接格式|ELF, Executable and Linkable Format|[手动加载、运行应用程序](https://rcore-os.cn/rCore-Tutorial-Book-v3/appendix-b/index.html#term-elf)|
|元数据|Metadata|[手动加载、运行应用程序](https://rcore-os.cn/rCore-Tutorial-Book-v3/appendix-b/index.html#term-metadata)|
|魔数|Magic|[手动加载、运行应用程序](https://rcore-os.cn/rCore-Tutorial-Book-v3/appendix-b/index.html#term-magic)|
|裸指针|Raw Pointer|[手动加载、运行应用程序](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-raw-pointer)|
|解引用|Dereference|[手动加载、运行应用程序](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter1/5support-func-call.html#term-dereference)|

## 第二章：批处理系统[](https://rcore-os.cn/rCore-Tutorial-Book-v3/terminology.html#id2 "永久链接至标题")

|中文|英文|出现章节|
|---|---|---|
|批处理系统|Batch System|[引言](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/0intro.html#term-batch-system)|
|特权级|Privilege|[引言](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/0intro.html#term-privilege)|
|监督模式执行环境|SEE, Supervisor Execution Environment|[RISC-V 特权级架构](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/1rv-privilege.html#term-see)|
|异常控制流|ECF, Exception Control Flow|[RISC-V 特权级架构](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter0/3os-hw-abstract.html#term-ecf)|
|陷入|Trap|[RISC-V 特权级架构](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/1rv-privilege.html#term-trap)|
|异常|Exception|[RISC-V 特权级架构](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/1rv-privilege.html#term-exception)|
|执行环境调用|Environment Call|[RISC-V 特权级架构](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/1rv-privilege.html#term-environment-call)|
|监督模式二进制接口|SBI, Supervisor Binary Interface|[RISC-V 特权级架构](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/1rv-privilege.html#term-sbi)|
|应用程序二进制接口|ABI, Application Binary Interface|[RISC-V 特权级架构](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/1rv-privilege.html#term-abi)|
|控制状态寄存器|CSR, Control and Status Register|[RISC-V 特权级架构](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/1rv-privilege.html#term-csr)|
|胖指针|Fat Pointer|[实现应用程序](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/2application.html#term-fat-pointer)|
|内部可变性|Interior Mutability|[实现应用程序](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/3batch-system.html#term-interior-mutability)|
|指令缓存|i-cache, Instruction Cache|[实现批处理系统](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/3batch-system.html#term-icache)|
|数据缓存|d-cache, Data Cache|[实现批处理系统](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter2/3batch-system.html#term-dcache)|
|原子指令|Atomic Instruction|[处理 Trap](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter8/2lock.html#term-atomic-instruction)|

## 第三章：多道程序与分时多任务[](https://rcore-os.cn/rCore-Tutorial-Book-v3/terminology.html#id3 "永久链接至标题")

|中文|英文|出现章节|
|---|---|---|
|多道程序|Multiprogramming|[引言](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/0intro.html#term-multiprogramming)|
|分时多任务系统|Time-Sharing Multitasking|[引言](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/0intro.html#term-time-sharing-multitasking)|
|任务上下文|Task Context|[任务切换](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/2task-switching.html#term-task-context)|
|输入/输出|I/O, Input/Output|[多道程序与协作式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/3multiprogramming.html#term-input-output)|
|任务控制块|Task Control Block|[多道程序与协作式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/3multiprogramming.html#term-task-control-block)|
|吞吐量|Throughput|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-throughput)|
|后台应用|Background Application|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-background-application)|
|交互式应用|Interactive Application|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-interactive-application)|
|协作式调度|Cooperative Scheduling|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-cooperative-scheduling)|
|时间片|Time Slice|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-time-slice)|
|公平性|Fairness|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter8/2lock.html#term-fairness)|
|时间片轮转算法|RR, Round-Robin|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-round-robin)|
|中断|Interrupt|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-interrupt)|
|同步|Synchronous|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-sync)|
|异步|Asynchronous|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-async)|
|并行|Parallel|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-parallel)|
|软件中断|Software Interrupt|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-software-interrupt)|
|时钟中断|Timer Interrupt|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-timer-interrupt)|
|外部中断|External Interrupt|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-external-interrupt)|
|嵌套中断|Nested Interrupt|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-nested-interrupt)|
|轮询|Busy Loop|[分时多任务系统与抢占式调度](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter3/4time-sharing-system.html#term-busy-loop)|

## 第四章：地址空间[](https://rcore-os.cn/rCore-Tutorial-Book-v3/terminology.html#id4 "永久链接至标题")

|中文|英文|出现章节|
|---|---|---|
|幻象|Illusion|[引言](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/0intro.html#term-illusion)|
|时分复用|TDM, Time-Division Multiplexing|[引言](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/0intro.html#term-time-division-multiplexing)|
|地址空间|Address Space|[地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#term-address-space)|
|虚拟地址|Virtual Address|[地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#term-virtual-address)|
|内存管理单元|MMU, Memory Management Unit|[地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#term-mmu)|
|地址转换|Address Translation|[地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#term-address-translation)|
|插槽|Slot|[地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#term-slot)|
|位图|Bitmap|[地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#term-bitmap)|
|内碎片|Internal Fragment|[地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#term-internal-fragment)|
|外碎片|External Fragment|[地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#term-external-fragment)|
|页面|Page|[地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#term-page)|
|虚拟页号|VPN, Virtual Page Number|[地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#term-virtual-page-number)|
|物理页号|PPN, Physical Page Number|[地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#term-physical-page-number)|
|页表|Page Table|[地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/2address-space.html#term-page-table)|
|静态分配|Static Allocation|[Rust 中的动态内存分配](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/1rust-dynamic-allocation.html#term-static-allocation)|
|动态分配|Dynamic Allocation|[Rust 中的动态内存分配](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/1rust-dynamic-allocation.html#term-dynamic-allocation)|
|智能指针|Smart Pointer|[Rust 中的动态内存分配](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/1rust-dynamic-allocation.html#term-smart-pointer)|
|集合|Collection|[Rust 中的动态内存分配](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/1rust-dynamic-allocation.html#term-collection)|
|容器|Container|[Rust 中的动态内存分配](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/1rust-dynamic-allocation.html#term-container)|
|借用检查|Borrow Check|[Rust 中的动态内存分配](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/1rust-dynamic-allocation.html#term-borrow-check)|
|引用计数|Reference Counting|[Rust 中的动态内存分配](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/1rust-dynamic-allocation.html#term-reference-counting)|
|垃圾回收|GC, Garbage Collection|[Rust 中的动态内存分配](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/1rust-dynamic-allocation.html#term-garbage-collection)|
|资源获取即初始化|RAII, Resource Acquisition Is Initialization|[Rust 中的动态内存分配](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/1rust-dynamic-allocation.html#term-raii)|
|页内偏移|Page Offset|[实现 SV39 多级页表机制（上）](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/3sv39-implementation-1.html#term-page-offset)|
|类型转换|Type Conversion|[实现 SV39 多级页表机制（上）](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/3sv39-implementation-1.html#term-type-conversion)|
|字典树|Trie|[实现 SV39 多级页表机制（上）](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/3sv39-implementation-1.html#term-trie)|
|多级页表|Multi-Level Page Table|[实现 SV39 多级页表机制（上）](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/3sv39-implementation-1.html#term-multi-level-page-table)|
|页索引|Page Index|[实现 SV39 多级页表机制（上）](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/3sv39-implementation-1.html#term-page-index)|
|大页|Huge Page|[实现 SV39 多级页表机制（上）](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/3sv39-implementation-1.html#term-huge-page)|
|恒等映射|Identical Mapping|[实现 SV39 多级页表机制（下）](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/4sv39-implementation-2.html#term-identical-mapping)|
|页表自映射|Recursive Mapping|[实现 SV39 多级页表机制（下）](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/4sv39-implementation-2.html#term-recursive-mapping)|
|跳板|Trampoline|[内核与应用的地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/6multitasking-based-on-as.html#term-trampoline)|
|隔离|Isolation|[内核与应用的地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/5kernel-app-spaces.html#term-isolation)|
|保护页面|Guard Page|[内核与应用的地址空间](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/5kernel-app-spaces.html#term-guard-page)|
|快表|Translation Lookaside Buffer|[基于地址空间的分时多任务](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/6multitasking-based-on-as.html#term-tlb)|
|熔断|Meltdown|[基于地址空间的分时多任务](https://rcore-os.cn/rCore-Tutorial-Book-v3/chapter4/6multitasking-based-on-as.html#term-meltdown)|