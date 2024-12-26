### 本节导读

本节主要介绍了 QEMU 模拟的 RISC-V 计算机中的 virtio 设备的架构和重要组成部分，以及面向 virtio 设备的驱动程序主要功能；并对 virtio-blk 设备及其驱动程序，virtio-gpu 设备及其驱动程序进行了比较深入的分析。

这里选择 virtio 设备来进行介绍，主要考虑基于两点考虑，
1. 首先这些设备就是 QEMU 模拟的高性能物理外设，**操作系统可以面向这些设备编写出合理的驱动程序**（如 Linux 等操作系统中都有 virtio 设备的驱动程序，并被广泛应用于云计算虚拟化场景中。）；
2. 其次，各种类型的 virtio 设备，如块设备（virtio-blk）、网络设备（virtio-net）、键盘鼠标类设备（virtio-input）、显示设备（virtio-gpu）**具有对应外设类型的共性特征、专有特征和与具体处理器无关的设备抽象性**。通过对这些设备的分析和比较，能够比较快速地掌握各类设备的核心特点，并掌握编写裸机或操作系统驱动程序的关键技术。

## virtio 设备

### virtio 概述

Rusty Russell 在 2008 年左右设计了 virtio 协议，并开发了相应的虚拟化解决方案 lguest，形成了 VirtIO 规范（Virtual I/O Device Specification）。其主要目的是为了简化和统一虚拟机（Hypervisor）的设备模拟，并提高虚拟机环境下的 I/O 性能。virtio 协议是对 hypervisor 中的一组通用模拟设备的抽象，即 virtio 协议定义了虚拟设备的输入 / 输出接口。而基于 virtio 协议的 I/O 设备称为 virtio 设备。下图列出了两种在虚拟机中模拟外设的总体框架。

![[40-VirtIO-devices-driver-hypervisor-arch.png]]

在上图左侧的虚拟机模拟外设的传统方案中，如果 guest VM 要使用底层 host 主机的资源，==需要 Hypervisor 截获所有的 I/O 请求指令，然后模拟出这些 I/O 指令的行为==，这会带来较大的性能开销。

>[! note] **虚拟机（Virtual Machine，VM）**
>虚拟机是物理计算机的虚拟表示形式或仿真环境。 虚拟机通常被称为访客机（Guest Machine，简称 Guest）或访客虚拟机（Guest VM），而它们运行所在的物理计算机被称为主机（Host Machine，简称 Host）。
>
>**虚拟机监视器 Hypervisor**
>虚拟机监视器（Hypervisor 或 Virtual Machine Monitor，简称 VMM）是创造并且运行虚拟机的软件、固件、或者硬件。这样主机硬件上能同时运行一个至多个虚拟机，这些虚拟机能高效地分享主机硬件资源。

在上图右侧的虚拟机模拟外设的 virtio 方案中，模拟的外设实现了功能最小化，即==虚拟外设的数据面接口主要是与 guest VM 共享的内存、控制面接口主要基于内存映射的寄存器和中断机制==。这样 guest VM 通过访问虚拟外设来使用底层 host 主机的资源时，Hypervisor 只需对少数寄存器访问和中断机制进行处理，实现了高效的 I/O 虚拟化过程。

>[! note] **数据面（Data Plane）**
>设备与处理器之间的 I/O 数据传输相关的数据设定（地址、布局等）与传输方式（基于内存或寄存器等）
>
>**控制平面（Control Plane）**
>处理器发现设备、配置设备、管理设备等相关的操作，以及处理器和设备之间的相互通知机制。

另外，各种类型的 virtio 设备，如块设备（virtio-blk）、网络设备（virtio-net）、键盘鼠标类设备（virtio-input）、显示设备（virtio-gpu）具有共性特征和独有特征。==对于共性特征，virtio 设计了各种类型设备的统一抽象接口，而对于独有特征，virtio 尽量最小化各种类型设备的独有抽象接口==。这样，virtio 就形成了一套通用框架和标准接口（协议）来屏蔽各种 hypervisor 的差异性，实现了 guest VM 和不同 hypervisor 之间的交互过程。

![[40-VirtIO-devices-driver-hypervisor-arch-complete.png]]

上图意味着什么呢？它意味着在 guest VM 上看到的虚拟设备具有简洁通用的优势，这对运行在 guest VM 上的操作系统而言，可以设计出轻量高效的设备驱动程序（即上图的 Front-end drivers）。

从==本质上讲，virtio 是一个接口，允许运行在虚拟机上的操作系统和应用软件通过访问 virtio 设备使用其主机的设备==。这些 virtio 设备具备功能最小化的特征，==Guest VM 中的设备驱动程序（Front-end drivers 只需实现基本的发送和接收 I/O 数据即可，而位于 Hypervisor 中的 Back-end drivers 和设备模拟部分让主机处理其实际物理硬件设备上的大部分设置、维护和处理==。这种设计方案极大减轻了 virtio 驱动程序的复杂性。

virtio 设备是虚拟外设，存在于 QEMU 模拟的 RISC-V 64 virt 计算机中。而我们要在操作系统中实现 virtio 驱动程序，来管理和控制这些 virtio 虚拟设备。每一类 virtio 设备都有自己的 virtio 接口，virtio 接口包括了数据结构和相关 API 的定义。这些定义中，有共性内容，也有属于设备特定类型的非共性内容。

### virtio 架构

总体上看，virtio 架构可以分为上中下三层，
1. 上层包括运行在 QEMU 模拟器上的前端操作系统中各种驱动程序（Front-end drivers）；
2. 下层是在 QEMU 中模拟的各种虚拟设备 Device；
3. 中间层是传输（transport）层，就是驱动程序与虚拟设备之间的交互接口，包含两部分：
	- 上半部是 virtio 接口定义，即 I/O 数据传输机制的定义：virtio 虚拟队列（virtqueue）；
	- 下半部是 virtio 接口实现，即 I/O 数据传输机制的具体实现：virtio-ring，主要由环形缓冲区和相关操作组成，用于保存驱动程序和虚拟设备之间进行命令和数据交互的信息。

![[40-VirtIO-devices-driver-virtio-arch.png]]

==操作系统中 virtio 驱动程序的主要功能==包括：
* 接受来自用户进程或操作系统其它组件发出的 I/O 请求
* 将这些 I/O 请求通过 virqueue 发送到相应的 virtio 设备
* 通过中断或轮询等方式查找并处理相应设备完成的 I/O 请求


Qemu 或 ==Hypervisor 中 virtio 设备的主要功能==包括：
* 通过 virqueue 接受来自相应 virtio 驱动程序的 I/O 请求
* 通过设备仿真模拟或将 I/O 操作卸载到主机的物理硬件来处理 I/O 请求，使处理后的 I/O 数据可供 virtio 驱动程序使用
* 通过寄存器、内存映射或中断等方式通知 virtio 驱动程序处理已完成的 I/O 请求


运行在 Qemu 中的操作系统中的 virtio 驱动程序和 Qemu 模拟的 virtio 设备驱动的关系如下图所示：

![[40-VirtIO-devices-driver-virtio-driver-devices.png]]

### I/O 设备基本组成结构

virtio 设备代表了一类 I/O 通用设备，为了让设备驱动能够管理和使用设备。在程序员的眼里，I/O 设备基本组成结构包括如下几项：

* **呈现**模式：设备一般通过寄存器、内存或特定 I/O 指令等方式==让设备驱动能看到和访问到设备==；

* **特征描述**：让设备驱动能够==了解设备的静态特性==（可通过软件修改），从而==决定是否或如何使用该设备==；

* **状态表示**：让设备驱动能够==了解设备的当前动态状态==，从而确定如何进行设备管理或 I/O 数据传输；

* **交互机制**：交互包括事件==通知和数据传输==；
	* 对于事件通知，让设备驱动及时获知设备的状态变化的机制（可基于中断等机制），以及让设备及时获得设备驱动发出的 I/O 请求（可基于寄存器读写等机制）；
	* 对于数据传输，让设备驱动能处理设备给出的数据，以及让设备能处理设备驱动给出的数据，如（可基于 DMA 或 virtqueue 等机制）。


virtio 设备具体定义了设备驱动和设备之间的接口，包括设备呈现模式、设备状态域、特征位、通知、设备配置空间、虚拟队列等，覆盖了上述的基本接口描述。

### virtio 设备基本组成要素

virtio 设备的基本组成要素如下：

* 设备状态域（Device status field）

* 特征位（Feature bits）

* 通知（Notifications）

* 设备配置空间（Device Configuration space）

* 一个或多个虚拟队列（virtqueue）


其中的设备特征位和设备配置空间属于 virtio 设备的特征描述；设备状态域属于 virtio 设备初始化时的状态表示；通知和虚拟队列属于 virtio 设备的交互机制，也包含 virtio 设备运行时的状态表示。

### virtio 设备呈现模式

virtio 设备支持三种设备呈现模式：

* Virtio Over MMIO，虚拟设备直**接挂载到系统总线**上，我们实验中的虚拟计算机就是这种呈现模式；

* Virtio Over PCI BUS，遵循 PCI 规范，**挂载到 PCI 总线上**，作为 virtio-pci 设备呈现，在 QEMU 虚拟的 x86 计算机上采用的是这种模式；

* Virtio Over Channel I/O：主要用在虚拟 IBM s390 计算机上，virtio-ccw 使用这种基于 channel I/O 的机制。


在 Qemu 模拟的 RISC-V 计算机 – virt 上，采用的是 Virtio Over MMIO 的呈现模式。这样==在实现设备驱动时，我们只需要找到相应 virtio 设备的 I/O 寄存器等以内存形式呈现的地址空间==，就可以对 I/O 设备进行初始化和管理了。

### virtio 设备特征描述

virtio 设备特征描述包括设备特征位和设备配置空间。

1. **特征位**
	- 特征位用于表示 VirtIO 设备具有的各种特性和功能。
	- 其中 bit0 – 23 是特定设备可以使用的 feature bits， bit24 – 37 预给队列和 feature 协商机制，bit38 以上保留给未来其他用途。
	- 驱动程序与设备对设备特性进行协商，形成一致的共识，这样才能正确的管理设备。

2. **设备配置空间**
	- 设备配置空间通常用于==配置不常变动的设备参数==（属性），或者初始化阶段需要设置的设备参数。
	- 设备的特征位中包含表示配置空间是否存在的 bit 位，并可通过在特征位的末尾添加新的 bit 位来扩展配置空间。
	- 设备驱动程序在初始化 virtio 设备时，==需要根据 virtio 设备的特征位和配置空间来了解设备的特征，并对设备进行初始化==。

### virtio 设备状态表示

virtio 设备状态表示包括在==设备初始化过程中用到的设备状态域，以及在设备进行 I/O 传输过程中用到的 I/O 数据访问状态信息和 I/O 完成情况等==。

3. **设备状态域**：设备状态域包含对设备初始化过程中 virtio 设备的 6 种状态：
	* ACKNOWLEDGE（1）：驱动程序发现了这个设备，并且认为这是一个有效的 virtio 设备；
	* DRIVER (2) : 驱动程序知道该如何驱动这个设备；
	* FAILED (128) : 由于某种错误原因，驱动程序无法正常驱动这个设备；
	* FEATURES_OK (8) : 驱动程序认识设备的特征，并且与设备就设备特征协商达成一致；
	* DRIVER_OK (4) : 驱动程序加载完成，设备可以正常工作了；
	* DEVICE_NEEDS_RESET (64) ：设备触发了错误，需要重置才能继续工作。
	* 在设备驱动程序对 virtio 设备初始化的过程中，需要经历一系列的初始化阶段，这些阶段对应着设备状态域的不同状态。

4. **I/O 传输状态**
	- 设备驱动程序控制 virtio 设备==进行 I/O 传输过程中，会经历一系列过程和执行状态==，包括 I/O 请求状态、 I/O 处理状态、I/O 完成状态、 I/O 错误状态、 I/O 后续处理状态等。设备驱动程序在执行过程中，需要对上述状态进行不同的处理。
	- virtio 设备进行 I/O 传输过程中，
		- ==设备驱动会指出 *I/O 请求* 队列的当前位置状态信息==，这样设备能查到 *I/O 请求* 的信息，并根据 *I/O 请求* 进行 I/O 传输；
		- 而==设备会指出 *I/O 完成* 队列的当前位置状态信息==，这样设备驱动通过读取 *I/O 完成* 数据结构中的状态信息，就知道设备是否完成 I/O 请求的相应操作，并进行后续事务处理。
	- 比如，
		- virtio_blk 设备驱动发出一个读设备块的 I/O 请求，并在某确定位置给出这个 I/O 请求的地址，然后给设备发出’kick’通知 (读或写相关 I/O 寄存器映射的内存地址)，此时处于 I/O 请求状态；
		- 设备在得到通知后，此时处于 *I/O 处理* 状态，它解析这个 I/O 请求，完成这个 I/O 请求的处理，即把磁盘块内容读入到内存中，并给出读出的块数据的内存地址，再通过中断通知设备驱动，此时处于 *I/O 完成* 状态；
		- 如果磁盘块读取发生错误，此时处于 *I/O 错误* 状态；
		- 设备驱动通过中断处理例程，此时处于 *I/O 后续处理* 状态，设备驱动知道设备已经完成读磁盘块操作，会根据磁盘块数据所在内存地址，把数据传递给文件系统进行进一步处理；
		- 如果设备驱动发现磁盘块读错误，则会进行错误恢复相关的后续处理。

### virtio 设备交互机制

virtio 设备==交互机制包括基于 Notifications 的事件通知和基于 virtqueue 虚拟队列的数据传输==。事件通知是指设备和驱动程序必须通知对方，它们有数据需要对方处理。数据传输是指设备和驱动程序之间进行 I/O 数据（如磁盘块数据、网络包）传输。

5. **Notification 通知**
	- ==驱动程序和设备在交互过程中需要相互通知对方==：驱动程序组织好相关命令 / 信息要通知设备去处理 I/O 事务，设备处理完 I/O 事务后，要通知驱动程序进行后续事务，如回收内存，向用户进程反馈 I/O 事务的处理结果等。
	- 驱动程序通知设备可用 ` 门铃 doorbell` 机制，即==采用 PIO 或 MMIO 方式访问设备特定寄存器，**QEMU 进行拦截再通知**其模拟的设备==。设备通知驱动程序**一般用中断机制**，即在 QEMU 中进行中断注入，让 CPU 响应并执行中断处理例程，来完成对 I/O 执行结果的处理。

6. **virtqueue 虚拟队列**
	- 在 virtio 设备上进行批量数据传输的机制被称为虚拟队列（virtqueue），virtio 设备的虚拟队列（virtqueue）可以由各种数据结构（如数组、环形队列等）来具体实现。
	- 每个 virtio 设备==可以拥有零个或多个 virtqueue==，每个 virtqueue 占用多个物理页，可用于设备驱动程序给设备发 I/O 请求命令和相关数据（如磁盘块读写请求和读写缓冲区），也可用于设备给设备驱动程序发 I/O 数据（如接收的网络包）。

### virtqueue 虚拟队列

virtio 协议中一个关键部分是 virtqueue，在 virtio 规范中，virtqueue 是 virtio 设备上进行批量数据传输的机制和抽象表示。在设备驱动实现和 Qemu 中 virtio 设备的模拟实现中，virtqueue 是一种数据结构，用于设备和驱动程序中执行各种数据传输操作。

操作系统在 Qemu 上运行时，==virtqueue 是 virtio 驱动程序和 virtio 设备访问的同一块内存区域==。

当涉及到 virtqueue 的描述时，有很多不一致的地方。有将其与 vring（virtio-rings 或 VRings）等同表示，也有将二者分别单独描述为不同的对象。我们将在这里单独描述它们，因为 ==vring 是 virtqueues 的主要组成部分，是达成 virtio 设备和驱动程序之间数据传输的数据结构==， vring 本质是 virtio 设备和驱动程序之间的共享内存，但 virtqueue 不仅仅只有 vring。

virtqueue 由三部分组成（如下图所示）：
![[40-VirtIO-devices-driver-virtqueue.png]]
* 描述符表 Descriptor Table：描述符表是描述符为组成元素的数组，==每个描述符描述了一个内存 buffer 的 address/length==。而内存 buffer 中包含 I/O 请求的命令 / 数据（由 virtio 设备驱动填写），也可包含 I/O 完成的返回结果（由 virtio 设备填写）等。

* 可用环 Available Ring：一种 vring，==记录了 virtio 设备驱动程序发出的 I/O 请求索引==，即被 virtio 设备驱动程序更新的描述符索引的集合，需要 virtio 设备进行读取并完成相关 I/O 操作；

* 已用环 Used Ring：另一种 vring，==记录了 virtio 设备发出的 I/O 完成索引==，即被 virtio 设备更新的描述符索引的集合，需要 vrtio 设备驱动程序进行读取并对 I/O 操作结果进行进一步处理。

#### 描述符表 Descriptor Table

描述符表用来指向 virtio 设备 I/O 传输请求的缓冲区（buffer）信息，由 `Queue Size` 个 Descriptor（描述符）组成。
- 描述符中包括 buffer 的物理地址 – addr 字段，buffer 的长度 – len 字段，可以链接到 `next Descriptor` 的 next 指针（用于把多个描述符链接成描述符链）。
- buffer 所在物理地址空间需要设备驱动程序在初始化时分配好，并在后续由设备驱动程序在其中填写 IO 传输相关的命令 / 数据，或者是设备返回 I/O 操作的结果。
- 多个描述符（I/O 操作命令，I/O 操作数据块，I/O 操作的返回结果）形成的==描述符链可以表示一个完整的 I/O 操作请求==。

#### 可用环 Available Ring

可用环在结构上是一个环形队列，==其中的条目（item）仅由驱动程序写入，并由设备读出==。
- 可用环中的条目包含了一个描述符链的头部描述符的索引值。
- 可用环用头指针（idx）和尾指针（last_avail_idx）表示其可用条目范围。
- virtio 设备通过读取可用环中的条目可获取驱动程序发出的 I/O 操作请求对应的描述符链，然后 virtio 设备就可以进行进一步的 I/O 处理了。
- 描述符指向的缓冲区具有可读写属性，可读的缓冲区用于 Driver 发送数据，可写的缓冲区用于接收数据。

比如，对于 virtio-blk 设备驱动发出的一个读 I/O 操作请求包含了三部分内容，由三个 buffer 承载，需要用到三个描述符 ：
1) “读磁盘块”，
2) I/O 操作数据块 – “数据缓冲区”，
3) I/O 操作的返回结果 –“结果缓冲区”。
这三个描述符形成的一个 
- virtio-blk 从设备可通过读取第一个描述符指向的缓冲区了解到是 “读磁盘块” 操作，
- 这样就可把磁盘块数据通过 DMA 操作放到第二个描述符指向的 “数据缓冲区” 中，
- 然后把 “OK” 写入到第三个描述符指向的 “结果缓冲区” 中。然后把 “OK” 写入到第三个描述符指向的 “结果缓冲区” 中。

#### 已用环 Used Ring

已用环在结构上是一个环形队列，==其中的的条目仅由 virtio 设备写入，并由驱动程序读出==。已用环中的条目也一个是描述符链的头部描述符的索引值。已用环也有头指针（idx）和尾指针（last_avail_idx）表示其已用条目的范围。

比如，
- 对于 virtio-blk 设备驱动发出的一个读 I/O 操作请求（由三个描述符形成的请求链）后，virtio 设备==完成相应 I/O 处理==，即把磁盘块数据写入第二个描述符指向的 “数据缓冲区” 中，可用环中对应的 I/O 请求条目 “I/O 操作的返回结果” 的描述符索引值移入到已用环中，
- 把 “OK” 写入到第三个描述符指向的 “结果缓冲区” 中，再在已用环中添加一个已用条目，即 ==I/O 操作完成信息==；
- 然后 virtio 设备通过中断机制来==通知 virtio 驱动程序==，并让 virtio 驱动程序读取已用环中的描述符，获得 I/O 操作完成信息，即磁盘块内容。

#### virtqueue 细节步骤

上面主要说明了 virqueue 中的各个部分的作用。对如何基于 virtqueue 进行 I/O 操作的过程还缺乏一个比较完整的描述。我们把上述基于 virtqueue 进行 I/O 操作的过程小结一下，大致需要如下步骤：

1. 初始化过程：（驱动程序执行）
	- virtio 设备驱动在对设备进行初始化时，会==申请 virtqueue（包括描述符表、可用环、已用环）的内存空间==；
	- 并把 virtqueue 中的描述符、可用环、已用环三部分的==物理地址分别写入到 virtio 设备中对应的控制寄存器==（即设备绑定的特定内存地址）中。至此，设备驱动和设备就共享了整个 virtqueue 的内存空间。

2. I/O 请求过程：（驱动程序执行）
	- 设备驱动在发出 I/O 请求时，首先==把 I/O 请求的命令 / 数据等放到一个或多个 buffer 中==；
	- 然后在描述符表中分配新的描述符（或描述符链）来指向这些 buffer；
	- 再==把描述符（或描述符链的首描述符）的索引值写入到可用环==中，更新可用环的 idx 指针；
	- 驱动程序通过 kick 机制（即==写 virtio 设备中特定的通知控制寄存器）来通知设备有新请求==；

3. I/O 完成过程：（设备执行）
	- virtio 设备通过 kick 机制（知道有新的 I/O 请求，通过访问可用环的 idx 指针，）==解析出 I/O 请求==；
	- 根据 I/O 请求内容完成 I/O 请求，并把 I/O 操作的结果放到 I/O 请求中相应的 buffer 中；
	- 再==把描述符（或描述符链的首描述符）的索引值写入到已用环中==，更新已用环的 idx 指针；
	- 设备通过再==通过中断机制来通知设备驱动程序有 I/O 操作完成==；

4. I/O 后处理过程：（驱动程序执行）
	- 设备驱动程序读取已用环的 idx 信息，读取已用环中的描述符索引，获得 I/O 操作完成信息。

### 基于 MMIO 方式的 virtio 设备

基于 MMIO 方式的 virtio 设备**没有基于总线的设备探测机制**。 所以操作系统==采用 Device Tree 的方式来探测各种基于 MMIO 方式的 virtio 设备，从而操作系统能知道与设备相关的寄存器和所用的中断==。

基于 MMIO 方式的 virtio 设备提供了一组内存映射的控制寄存器，后跟一个设备特定的配置空间，在形式上是位于一个特定地址上的内存区域。==一旦操作系统找到了这个内存区域，就可以获得与这个设备相关的各种寄存器信息==。比如，我们在 virtio-drivers crate 中就定义了基于 MMIO 方式的 virtio 设备的寄存器区域：
```
//virtio-drivers/src/header.rs
pub struct VirtIOHeader {
   magic: ReadOnly<u32>,  //魔数 Magic value
   ...
   //设备初始化相关的特征/状态/配置空间对应的寄存器
   device_features: ReadOnly<u32>, //设备支持的功能
   device_features_sel: WriteOnly<u32>,//设备选择的功能
   driver_features: WriteOnly<u32>, //驱动程序理解的设备功能
   driver_features_sel: WriteOnly<u32>, //驱动程序选择的设备功能
   config_generation: ReadOnly<u32>, //配置空间
   status: Volatile<DeviceStatus>, //设备状态

   //virtqueue虚拟队列对应的寄存器
   queue_sel: WriteOnly<u32>, //虚拟队列索引号
   queue_num_max: ReadOnly<u32>,//虚拟队列最大容量值
   queue_num: WriteOnly<u32>, //虚拟队列当前容量值
   queue_notify: WriteOnly<u32>, //虚拟队列通知
   queue_desc_low: WriteOnly<u32>, //设备描述符表的低32位地址
   queue_desc_high: WriteOnly<u32>,//设备描述符表的高32位地址
   queue_avail_low: WriteOnly<u32>,//可用环的低32位地址
   queue_avail_high: WriteOnly<u32>,//可用环的高32位地址
   queue_used_low: WriteOnly<u32>,//已用环的低32位地址
   queue_used_high: WriteOnly<u32>,//已用环的高32位地址

   //中断相关的寄存器
   interrupt_status: ReadOnly<u32>, //中断状态
   interrupt_ack: WriteOnly<u32>, //中断确认
}
```

这里列出了部分关键寄存器和它的基本功能描述。在后续的设备初始化以及设备 I/O 操作中，会访问这里列出的寄存器。

在有了上述 virtio 设备的理解后，接下来，我们将进一步分析 virtio 驱动程序如何管理 virtio 设备来完成初始化和 I/O 操作。

## virtio 驱动程序

这部分内容是各种 virtio 驱动程序的共性部分，主要包括初始化设备，驱动程序与设备的交互步骤，以及驱动程序执行过程中的一些实现细节。

### 设备的初始化

操作系统通过某种方式（设备发现，基于设备树的查找等）找到 virtio 设备后，驱动程序进行设备初始化的常规步骤如下所示：

1. **重启设备状态**，设置设备状态域为 0

2. **设置设备状态域**为 `ACKNOWLEDGE` ，表明当前已经识别到了设备

3. 设置设备状态域为 `DRIVER` ，表明驱动程序知道如何驱动当前设备

4. **进行设备特定的安装和配置**，包括协商特征位，建立 virtqueue，访问设备配置空间等, 设置设备状态域为 `FEATURES_OK`

5. 设置设备状态域为 `DRIVER_OK` 或者 `FAILED` （如果中途出现错误）


注意，上述的步骤不是必须都要做到的，但最==终需要设置设备状态域为 `DRIVER_OK` ，这样驱动程序才能正常访问设备==。

在 virtio_driver 模块中，我们实现了通用的 virtio 驱动程序框架，各种 virtio 设备驱动程序的共同的初始化过程为：

1. ==确定协商特征位==，调用 VirtIOHeader 的 begin_init 方法进行 virtio 设备初始化的第 1-4 步骤；

2. 读取配置空间，==确定设备的配置情况==；

3. ==建立虚拟队列== 1~n 个 virtqueue；

4. ==调用 VirtIOHeader finish_init 方法==进行 virtio 设备初始化的第 5 步骤——报告初始化的结果 OK 还是FAILED。


比如，对于 virtio_blk 设备初始化的过程如下所示：
```
// virtio_drivers/src/blk.rs
//virtio_blk驱动初始化：调用header.begin_init方法
impl<H: Hal> VirtIOBlk<'_, H> {
   /// Create a new VirtIO-Blk driver.
   pub fn new(header: &'static mut VirtIOHeader) -> Result<Self> {
      header.begin_init(|features| {
            ...
            (features & supported_features).bits()
      });
      //读取virtio_blk设备的配置空间
      let config = unsafe { &mut *(header.config_space() ...) };
      //建立1个虚拟队列
      let queue = VirtQueue::new(header, 0, 16)?;
      //结束设备初始化
      header.finish_init();
      ...
   }
// virtio_drivers/src/header.rs
// virtio设备初始化的第1~4步骤
impl VirtIOHeader {
   pub fn begin_init(&mut self, negotiate_features: impl FnOnce(u64) -> u64) {
      self.status.write(DeviceStatus::ACKNOWLEDGE);
      self.status.write(DeviceStatus::DRIVER);
      let features = self.read_device_features();
      self.write_driver_features(negotiate_features(features));
      self.status.write(DeviceStatus::FEATURES_OK);
      self.guest_page_size.write(PAGE_SIZE as u32);
   }

   // virtio设备初始化的第5步骤
   pub fn finish_init(&mut self) {
      self.status.write(DeviceStatus::DRIVER_OK);
   }
```

### 驱动程序与设备之间的交互

驱动程序与外设可以共同访问约定的 virtqueue，virtqueue 将保存设备驱动的 I/O 请求信息和设备的 I/O 响应信息。virtqueue 由描述符表（Descriptor Table）、可用环（Available Ring）和已用环（Used Ring）组成。在上述的设备驱动初始化过程描述中已经看到了虚拟队列的创建过程。

当驱动程序向设备发送 I/O 请求（由命令 / 数据组成）时，它会在 buffer（设备驱动申请的内存空间）中填充命令 / 数据，各个 buffer 所在的起始地址和大小信息放在描述符表的描述符中，再把这些描述符链接在一起，形成描述符链。

而描述符链的起始描述符的索引信息会放入一个称为环形队列的数据结构中。该队列有两类，一类是包含由设备驱动发出的 I/O 请求所对应的描述符索引信息，即可用环。另一类由包含由设备发出的 I/O 响应所对应的描述符索引信息，即已用环。

一个用户进程发起的 **I/O 操作的处理过程大致可以分成如下四步**：

1. 用户进程==发出 I/O 请求，经过层层下传给到驱动程序==，驱动程序将 I/O 请求信息放入虚拟队列 virtqueue 的可用环中，并通过某种通知机制（如写某个设备寄存器）通知设备；

2. ==设备收到通知后，解析可用环和描述符表==，取出 I/O 请求并在内部进行实际 I/O 处理；

3. 设备==完成 I/O 处理或出错后，将结果作为 I/O 响应放入已用环中==，并以某种通知机制（如外部中断）通知 CPU；

4. ==驱动程序解析已用环，获得 I/O 响应的结果==，在进一步处理后，最终返回给用户进程。

![[40-VirtIO-devices-driver-vring.png]]

### 发出 I/O 请求的过程

虚拟队列的相关操作包括两个部分：向设备提供新的 I/O 请求信息（可用环–> 描述符–> 缓冲区），以及处理设备使用的 I/O 响应（已用环–> 描述符–> 缓冲区）。 比如，virtio-blk 块设备具有一个虚拟队列来支持 I/O 请求和 I/O 响应。在驱动程序进行 I/O 请求和 I/O 响应的具体操作过程中，需要注意如下一些细节。

驱动程序给设备发出 I/O 请求信息的具体步骤如下所示：

1. 将包含一个 I/O 请求内容的缓冲区的地址和长度信息==放入描述符表中的空闲描述符中，并根据需要把多个描述符进行链接==，形成一个描述符链（表示一个 I/O 操作请求）；

2. 驱动程序==将描述符链头的索引放入可用环的下一个环条目中==；

3. 如果可以进行批处理（batching），则可以重复执行步骤 1 和 2，这样通过（可用环–> 描述符–> 缓冲区）来添加多个 I/O 请求；

4. 根据添加到可用环中的描述符链头的数量，==更新可用环==；

5.  将 “有可用的缓冲区” 的==通知发送给设备==。

*注：在第 3 和第 4 步中，都需要指向适当的内存屏障操作（Memory Barrier），以确保设备能看到更新的描述符表和可用环。*

>[! note] 内存屏障 (Memory Barrier)
>大多数现代计算机为了提高性能而采取乱序执行，这使得内存屏障在某些情况下成为必须要执行的操作。
>
>内存屏障是一类同步屏障指令，它==使得 CPU 或编译器在对内存进行操作的时候, 严格按照一定的顺序来执行==, 也就是说在内存屏障之前的指令和内存屏障之后的指令不会由于系统优化等原因而导致乱序。
>
>内存屏障分为写屏障（Store Barrier）、读屏障（Load Barrier）和全屏障（Full Barrier），其作用是：
>- 防止指令之间的重排序
>- 保证数据的可见性


#### 将缓冲区信息放入描述符表的操作

==缓冲区用于表示一个 **I/O 请求的具体内容**==，由零个或多个设备可读 / 可写的物理地址连续的内存块组成（一般前面是可读的内存块，后续跟着可写的内存块）。我们把构成缓冲区的内存块称为缓冲区元素，把缓冲区映射到描述符表中以形成描述符链的具体步骤：

对于每个缓冲区元素 `b` 执行如下操作：

1. 获取下一个空闲描述符表条目 `d` ；

2. 将 `d.addr` 设置为 `b` 的的起始物理地址；
 
3. 将 `d.len` 设置为 `b` 的长度；
 
4. 如果 `b` 是设备可写的，则将 `d.flags` 设置为 `VIRTQ_DESC_F_WRITE` ，否则设置为 0；
 
5. 如果 `b` 之后还有一个缓冲元素 `c` ：
	- 将 `d.next` 设置为下一个空闲描述符元素的索引；
	- 将 `d.flags` 中的 `VIRTQ_DESC_F_NEXT` 位置 1；


#### 更新可用环的操作

描述符链头是上述步骤中的第一个条目 `d` ，即描述符表条目的索引，指向缓冲区的第一部分。一个驱动程序实现可以执行以下的伪代码的操作（假定在与小端字节序之间进行适当的转换）来==更新可用环==：

```
avail.ring[avail.idx % qsz] = head;  //qsz 表示可用环的大小
```

但是，==通常驱动程序可以在更新 idx 之前添加许多描述符链== （这时它们对于设备是可见的），因此通常要对驱动程序已添加的数目 `added` 进行计数：

```
avail.ring[(avail.idx + added++) % qsz] = head;
```

idx 总是递增，并在到达 `qsz` 后又回到 0：

```
avail.idx += added;
```

==一旦驱动程序更新了可用环的 `idx` 指针，这表示描述符及其它指向的缓冲区能够被设备看到==。这样设备就可以访问驱动程序创建的描述符链和它们指向的内存。==驱动程序必须在 idx 更新之前执行合适的内存屏障操作==，以确保设备看到最新描述符和 buffer 内容。

#### 通知设备的操作

在包含 virtio 设备的 Qemu virt 虚拟计算机中，驱动程序一般通过对代表通知” 门铃” 的特定寄存器进行写操作来发出通知。

``` hl:8
// virtio_drivers/src/header.rs
pub struct VirtIOHeader {
// Queue notifier 用户虚拟队列通知的寄存器
queue_notify: WriteOnly<u32>,
...
impl VirtIOHeader {
   // Notify device.
   pub fn notify(&mut self, queue: u32) {
      self.queue_notify.write(queue);
   }
```

### 接收设备 I/O 响应的操作

一旦设备完成了 I/O 请求，形成 I/O 响应，就会==更新描述符所指向的缓冲区，并向驱动程序发送已用缓冲区通知==（used buffer notification）。一般会采用中断这种更加高效的通知机制。设备驱动程序在收到中断后，就会对 I/O 响应信息进行后续处理。相关的伪代码如下所示：

```
// virtio_drivers/src/blk.rs
impl<H: Hal> VirtIOBlk<'_, H> {
   pub fn ack_interrupt(&mut self) -> bool {
      self.header.ack_interrupt()
   }

// virtio_drivers/src/header.rs
pub struct VirtIOHeader {
   // 中断状态寄存器 Interrupt status
   interrupt_status: ReadOnly<u32>,
   // 中断响应寄存器 Interrupt acknowledge
   interrupt_ack: WriteOnly<u32>,
impl VirtIOHeader {
   pub fn ack_interrupt(&mut self) -> bool {
      let interrupt = self.interrupt_status.read();
      if interrupt != 0 {
            self.interrupt_ack.write(interrupt);
            true
      }
      ...
```

这里给出了 virtio 设备驱动通过中断来接收设备 I/O 响应的共性操作过程。如果结合具体的操作系统，还需与操作系统的总体中断处理、同步互斥、进程 / 线程调度进行结合。