---
publish: ""
tags:
  - Network
  - 计算机网络原理
---
## 4.1 网络层概述

### 网络层服务
![[40-Network-layer-Data-plane-network-layer-instance.png]]
Figure 4.1 shows a simple network with two hosts, H1 and H2, and several routers on the path between H1 and H2. Let’s suppose that H1 is sending information to H2, and consider the role of the network layer in these hosts and in the intervening routers. 
- The network layer in H1 takes segments from the transport layer in H1, encapsulates each segment into a datagram, and then sends the datagrams to its nearby router, R1. 
- At the receiving host, H2, the network layer receives the datagrams from its nearby router R2, extracts the transport-layer segments, and delivers the segments up to the transport layer at H2. 
- The primary data-plane role of each router is to forward datagrams from its input links to its output links; 
- the primary role of the network control plane is to coordinate these local, per-router forwarding actions so that datagrams are ultimately transferred end-to-end, along paths of routers between source and destination hosts. 
- Note that the **routers** in Figure 4.1 are shown with a truncated protocol stack, that is, **with no upper layers above the network layer, because routers do not run application and transport-layer protocols** such as those we examined in Chapters 2 and 3.

网络层服务
- 在发送主机和接收主机对之间传送段(segment)
- 在发送端将段封装到数据报中
- 在接收端，将段上交给传输层实体
- 网络层协议存在于**每一个**主机和路由器（每一个都需要封装和解封装）
- 路由器检查每一个经过它的IP数据报的头部

网络层的关键功能
- 网络层功能：
    - **转发**（一个局部的概念，数据平面）：将分组从路由器的输入接口转发到合适的输出接口的路由器本地动作
    - **路由**（一个全局的功能，控制平面）：使用路由算法来决定分组从发送主机到目标接收主机的路径
        - 路由选择算法
        - 路由选择协议
- 旅行的类比：
    - 转发：通过单个路口的过程
    - 路由：从源到目的的路由路径规划过程

### 网络层：数据平面、控制平面

- 数据平面：分组从哪个端口输入，从哪个端口输出（微观，局部）
    - 本地进行的动作，每个路由器都要实现的功能
    - 转发发生的时间尺度很短（几纳秒），通常用硬件实现。
    - 转发功能：
        - 传统方式：基于目标 IP 地址得知哪个端口输入 + 转发表 forwarding table（路由表，router table）决定哪个端口输出
        - SDN方式：基于多个字段 + 与 **流表** 做匹配，通过匹配的表象进行相应的动作（如转发、阻止、泛洪、修改等）（不像传统方式只进行转发，更加灵活）

- 控制平面：决定分组在整个网络中的路径（宏观，全局）
    - 确定分组从源到目的地所采取的端到端路径的网络范围处理过程
    - 路由选择的时间尺度很长（几秒），通常用软件实现
    - 2个控制平面方法：
        - 传统的路由算法：在路由器中被实现，得到路由表
        - software-defined networking (SDN，软件定义网络)：在远程的服务器中实现，计算出流表通过南向接口交给分组交换设备，进而与分组的多个字段相匹配并根据匹配结果进行相应的动作

> [! example] forwarding table
> ![[40-Network-layer-Data-plane-forwarding-table.png]]
> 
> 路由器检查到达分组的首部的一个或多个字段值，
> - 进而使用这些值在转发表中进行索引，通过这种方式转发分组。
> - 这些值对应存储在转发表项中的值，指出了该分组将被转发的路由器的输出链路接口。


#### 控制平面：传统方法

转发表是网络层能够正常运行的关键，而如何配置转发表，是路由选择和转发间协调工作的核心问题。
- 暴力地，人类网络操作员可以配置物理上所有路由器的转发表内容，这意味着路由转发算法并不必需。
- 但实际上这一方法既不可为也不可行——人工配置极易出错、对于网络拓扑变化的响应极慢……
- 如果设计全局的路由选择算法，在每台路由器中根据路由选择算法进行转发与路由的协调、计算转发表的表项，是传统的改进方法。
	- 这种协调的通信是如何实现的呢？通过根据路由选择算法交换包含路由选择信息的路由选择报文。
	- 使用该方法，需要每台路由器都有一个与其它路由器的路由选择组件通信的路由选择组件。
	- 这在事实上，是控制平面和数据平面的紧耦合（集中于一台设备上实现），分布式地计算路由表，缺点是计算量大、难以修改路由设备的运行逻辑、模式僵化。


***SDN 方法应运而生！***

#### 控制平面：SDN 方法

![[40-Network-layer-Data-plane-SDN-method.png]]
- 一个不同的（通常是远程的）**控制器计算和分发转发表以供每台路由器使用**；
- 远程控制器与本地控制代理（CAs）交互，只用在控制器处改变流表就可以改变网络设备的行为逻辑，易修改、可编程；
- 注意到图 4.3 和 4.2 的区别：4.2 中路由选择算法和转发表是紧耦合的，而 4.3 中控制平面的路由选择功能与物理的路由器是分离的；

路由器与远程控制器是如何通信的呢？
- 通过交换包含转发表和其它路由信息的报文。
- 图 4.3 中的控制平面方法是 SDN (Software-Defined Networking)，顾名思义，计算转发表并与路由器交互的控制器是用软件实现的。

### 网络服务模型

- Q：从发送方主机到接收方主机传输数据报的“通道”，网络提供什么样的服务模型(service model)？服务模型有一系列的指标
    - 对于单个数据报的服务：
        - 确保交付
        - 延迟保证，如：少于40ms的延迟
    - 对于数据报流（一系列分组的序列）的服务：
        - 有序分组交付
        - 保证流的最小带宽
        - 安全性
        - 分组之间的延迟差 (jitter)

连接建立
- 在某些网络架构中是第三个重要的功能（继 **路由**、**转发** 之后）
    - ATM（有连接：建立连接&路径上所有主机进行维护）, 
    - frame relay, 
    - X.25
- 在分组传输之前，在两个主机之间，在通过一些路由器所构成的路径上建立一个网络层连接
    - 涉及到路由器
- 网络层和传输层连接服务区别:
    - 网络层：在2个主机之间，涉及到路径上的一些路由器，有连接
    - 传输层：在2个进程之间，很可能只体现在端系统上（TCP连接），面向连接

网络层服务模型：

| 网络架构 |        服务模型        |      保证带宽      | 不丢失 | 保序 | 延迟保证 |        拥塞反馈        |
|:--------:|:----------------------:|:------------------:|:------:|:----:|:--------:|:----------------------:|
| Internet | best effort “尽力而为” |        none        |   no   |  no  |    no    | no (inferred via loss) |
|   ATM    |     CBR(恒定速率)      |   constant rate    |  yes   | yes  |   yes    |     no congestion      |
|   ATM    |     VBR(变化速率)      |  guaranteed rate   |  yes   | yes  |   yes    |     no congestion      |
|   ATM    |    ABR(可用比特率)     | guaranteed minimum |   no   | yes  |    no    |          yes           |
|   ATM    |   UBR(不指名比特率)    |        none        |   no   | yes  |    no    |           no           |

The Internet’s network layer provides a single service, known as ***best-effort service***. With best-effort service, packets are neither guaranteed to be received in the order in which they were sent, nor is their eventual delivery even guaranteed. There is no guarantee on the end-to-end delay nor is there a minimal bandwidth guarantee. It might appear that best-effort service is a euphemism for no service at all—==a network that delivered no packets to the destination would satisfy the definition of best-effort delivery service==! 
> hah😆

Other network architectures have defined and implemented service models that go beyond the Internet’s best-effort service. For example, the ATM network architecture `[Black 1995]` provides for guaranteed in-order delay, bounded delay, and guaranteed minimal bandwidth. There have also been proposed service model extensions to the Internet architecture; for example, the Intserv architecture `[RFC 1633]` aims to provide end-end delay guarantees and congestion-free communication. 

Interestingly, in spite of these well-developed alternatives, the Internet’s basic best-effort service model combined with adequate bandwidth provisioning and bandwidth-adaptive application-level protocols such as the DASH protocol we encountered in Section 2.6.2 have arguably proven to be more than “good enough” to enable an amazing range of applications, including streaming video services such as Netflix and video-over-IP, real-time conferencing applications such as Skype and Facetime.
> 能用就是好！😠

## 4.2 路由器内究

### 路由器结构概况
![[40-Network-layer-Data-plane-router-structure.png]]

- 通用路由器体系架构：输入端口 + 输出端口 + 交换结构 + 路由选择处理器
    - 路由：运行路由选择算法/协议 (RIP, OSPF, BGP) - 生成路由表
    - 转发：从输入到输出链路交换数据报-根据路由表进行分组的转发

- 输入端口：（从左到右三个功能块，分别完成以下三个任务）
	- 终结物理链路的物理层功能
	- 与位于入链路远端的数据链路层交互，来执行数据链路层功能
	- 在输入端口执行查找功能（通过查询转发表决定路由器的输出端口）
		- 普通分组通过路由器的交换结构转发到输出端口
		- 控制分组（携带路由选择协议信息的分组）转发到路由选择处理器
- 交换结构：
	- 将路由器的输入端口连接到输出端口
	- 是一个网络路由器中的网络，完全包含在路由器之中
- 输出端口：
	- 存储从交换结构接收的分组，并通过执行必要的链路层和物理层功能在输出链路上传输这些分组
	- 一条链路是双向的时，输出端口和输入端口成对地出现在同一线路卡上
- 路由选择处理器：
	- 执行控制平面功能。
		- 在传统路由器中，执行路由选择协议，维护路由选择表与关联链路状态信息，并为该路由器计算转发表
		- SDN 路由器中，路由选择处理器负责与远程控制器通信，目的是接收由远程控制器计算的转发表项，并在该路由器的输入端口安装这些表项
	- 还执行网络管理功能。

>[! warning] 路由器端口是物理端口！
>与应用层和传输层所说的端口不同，路由器端口指的是物理上可以插拔网线、控制线的实体接口，而应用层等端口是软件实现的、socket 端口。不要混淆。

>[! note] 输入端口、输出端口、交换结构用硬件实现！
>To appreciate why a hardware implementation is needed, consider that with a 100 Gbps input link and a 64-byte IP datagram, ==the input port has only 5.12 ns to process the datagram before another datagram may arrive==.
>
>If N ports are combined on a line card (as is often done in practice), ***the datagram-processing pipeline must operate N times faster—far too fast for software implementation***. Forwarding hardware can be implemented either using a router vendor’s own hardware designs, or constructed using purchased merchant-silicon chips (for example, as sold by companies such as Intel and Broadcom).

>[!note] 控制平面的功能用软件实现！
>While the data plane operates at the nanosecond time scale, a router’s **control functions**—executing the routing protocols, responding to attached links that go up or down, communicating with the remote controller (in the SDN case) and performing management functions—operate at the millisecond or second timescale.
>
>These ***control plane functions are thus usually implemented in software and execute on the routing processor*** (typically a traditional CPU).

> [! note] 区分基于目的地的转发和通用转发
> 
> ***Destination-based forwarding***. Suppose the car stops at an entry station and ==indicates its final destination== (not at the local roundabout, but the ultimate destination of its journey). An attendant at the entry station looks up the final destination, determines the roundabout exit that leads to that final destination, and tells the driver which roundabout exit to take. 
> 
> ***Generalized forwarding***. The attendant could also determine the car’s exit ramp on the basis of many other factors besides the destination. For example, the selected exit ramp might depend on the car’s origin, for example the state that issued the car’s license plate. Cars from a certain set of states might be directed to use one exit ramp (that leads to the destination via a slow road), while cars from other states might be directed to use a different exit ramp (that leads to the destination via superhighway). The same decision might be made based on the model, make and year of the car. Or a car not deemed roadworthy might be blocked and not be allowed to pass through the roundabout. In the case of generalized forwarding, any number of factors may contribute to the attendant’s choice of the exit ramp for a given car.

### 输入端口处理和基于目的地的转发

![[40-Network-layer-Data-plane-input-port-processing.png]]

- 正是在输入端口，路由器使用转发表来查找输出端口，使得到达的分组能够经过交换结构转发到该输出端口。
- 转发表由路由选择处理器经过独立的总线（如 PCI）复制到线路卡（4.4 图中的虚线流向）。
- 使用在**每个输入端口**的 copy，转发决策能在每个输入端口本地做出，无须基于每个分组调用集中式的路由选择处理器

![[40-Network-layer-Data-plane-input-function.png]]

>[! note] 转发表基于目的地的转发规则
>显然，暴力地将每个目标地址的 IP 作为一个转发表的表项，是过于庞大、不可行的。
>
>通常使用基于前缀的转发规则：
> ![[40-Network-layer-Data-plane-ip-based-forward-rules.png]]
>如果分组的目标地址的 IP 的前缀与某个转发表表项相同，则向对应的输出端口转发。如果不满足任何表项，则向默认端口转发。
>For example, suppose the packet’s destination address is 11001000 00010111 00010110 10100001; because the 21-bit prefix of this address matches the first entry in the table, the router forwards the packet to link interface 0. If a prefix doesn’t match any of the first three entries, then the router forwards the packet to the default interface 3.
>
>当有多个前缀匹配时，遵循**最长前缀匹配原则**longest prefix matching rule)：即在转发表中寻找最长的匹配项，并向与最长前缀匹配相关联的链路接口转发分组。
>映射到现实就很好理解这一原则：我（分组）从太原（源地址）开车前往深圳（目的地址），每个高速路口（路由器）时都要决定下一个岔路往哪里开。比如到郑州时，有三个岔路，分别通向武汉、上海、西安，那么我的目标前缀（向南前进）决定了我要开向武汉岔口。每到一个岔口，我的路线就更进一步地细分——如何准确地到达深圳，而不走向其它岔路。所谓前缀匹配，就是看目标方向是否一致，前缀匹配得越长，与目标所在方向越精确。

由于查找要求的时间非常苛刻，因此不仅要使用硬件查找，而且对于大型转发表要使用超出简单线性搜索的技术：
- surveys of fast lookup algorithms can be found in `[Gupta 2001, Ruiz-Sanchez 2001]`. 
- Special attention must also be paid to memory access times, resulting in designs with embedded on-chip DRAM and faster SRAM memories. In practice, ***Ternary Content Addressable Memories (TCAMs)*** are also often used for lookup `[Yu 2004]`. 
- With a TCAM, a 32-bit IP address is presented to the memory, which returns the content of the forwarding table entry for that address in essentially constant time. 
	- The Cisco Catalyst 6500 and 7600 Series routers and switches can hold upwards of a million TCAM forwarding table entries `[Cisco TCAM 2014]`.

### 交换

![[40-Network-layer-Data-plane-switch-three-techniques.png]]
- 3 种典型的交换机构：memory, bus, interconnection
- 通过内存交换（**memory**） —— 第一代路由器：
	- 在 CPU (路由选择处理器) 直接控制下的交换，
	- 与传统的计算机类似，输入端口和输出端口就像传统的 I/O 设备一样，一个分组到达一个输入端口时，该端口先通过中断的方式向路由选择器发出信号，于是该分组从输入端口被复制到处理器的内存中，路由选择处理器从分组的首部提取出目的地址，在转发表中找到适当的输出端口，并将该分组复制到输出端口的缓存中。
	- 缺点：
		- 内存带宽为 B 时，总的转发吞吐量不会超过 B/2
		- 转发速率被内存的带宽限制（数据报通过 BUS 两遍（进+出），系统总线本身就会成为瓶颈，速率低）
		- 共享的系统总线一次仅能执行一个内存读/写——即一次只能转发一个分组
	- 一种改进方式是：目的地址的查找和将分组存储到适当的内存存储位置由输入线路卡来处理——类似共享内存的多处理器

- 通过总线交换（**bus**） —— 第二代路由器
	- 数据报通过共享总线，从输入端口转发到输出端口，不需要路由选择处理器的干预
	- 输入端口为分组预先计划一个交换机内部标签，指示本地输出端口，使分组在总线上传送到输出端口，所有输出端口都能查看分组，但只有指定的与标签匹配的端口才能接收、完成转发
	- 缺点：
		- 虽然相比于 memory 方式，只经过 bus 一次，交换速率大大提升，但是还是有问题 —— **总线竞争**：即使有多个分组等待交换，但一次只能有一个分组使用总线进行传送——交换速度受限于总线带宽
	- 1 Gbps bus, Cisco 1900； 32 Gbps bus, Cisco 5600；对于接入或企业级路由器，速度足够（但不适合区域或骨干网络）

- 通过互联网络（**interconnection**等）的交换
	- 通过 2N 条总线组成互联网络，连接 N 个输入端口和 N 个输出端口，每条垂直的总线在交叉点与每条水平的总线交叉，交叉点通过交换结构控制器（交换结构自身的一部分）能够在任何时间开启和闭合
	- When a packet arrives from port A and needs to be forwarded to port Y, the switch controller closes the crosspoint at the intersection of busses A and Y, and port A then sends the packet onto its bus, which is picked up (only) by bus Y. Note that a packet from port B can be forwarded to port X at the same time, since the A-to-Y and B-to-X packets use different input and output busses.
	- 同时并发转发多个分组，克服总线带宽限制，并且是非阻塞式的——a packet being forwarded to an output port will not be blocked from reaching that output port as long as no other packet is currently being forwarded to that output port. However, if two packets from two different input ports are destined to that same output port, then one will have to wait at the input, since only one packet can be sent over any given bus at a time.


>[! note] 多级交换策略
>The Cisco CRS employs a three-stage non-blocking switching strategy. A router’s switching capacity can also be scaled by running multiple switching fabrics in parallel.
>
>In this approach, input ports and output ports are connected to N switching fabrics that operate in parallel. An input port breaks a packet into K smaller chunks, and sends (“sprays”) the chunks through K of these N switching fabrics to the selected output port, which reassembles the K chunks back into the original packet.
>> 在路由器内部的、通过交换结构的复用与解复用。

- 交换速率：分组可以按照该速率从输入传输到输出
    - 运行速度经常是输入/输出链路速率的若干倍
    - N个输入端口：交换机构的交换速度是输入线路速度的N倍比较理想，才不会成为瓶颈


### 输出端口处理

![[40-Network-layer-Data-plane-output-port-processing.png]]
- 取出已存放在输出端口缓冲中的分组并将其发送到输出链路上
- 包括选择和取出排队的分组进行传输，执行所需的链路层和物理层传输功能

### 排队

输入端口和输出端口都需要一个队列——输入/输出端口缓存：输入链路交给输入端口的速度、或输出端口交付给输出链路的速度与分组交换结构的速度可能不匹配。

而队列的位置和长度取决于流量负载、交换结构的相对速率和输出速率——由于排队队列的增长会耗尽缓冲区，进而导致丢包——由于输入缓存溢出造成的丢失！

- 假设输入线路速度和输出线路速度相同，均为 $R_{line}$，并且有 N 个输入端口和 N 个输出端口；
- 定义交换接口的传送速率为 $R_{switch}$，若比 $R_{line}$ 快 N 倍，此时输入端口的排队情况可以忽略不计，因为即使在最坏情况（N 个输入端口同时有分组到达），也会在下一轮分组到达前通过交换结构的处理

#### 输入端口排队

- 当交换结构转发给输出端口的速率小于输入端口的汇聚速率时，在输入端口需要排队
- 或者当交换结构正在被其它输入端口的分组使用时，新的分组会被阻塞，这会占据缓存的区域
- 如此，造成排队延迟，甚至输入端口缓冲区的溢出！
- 不要忘记在 HTTP/1.1 中提及的 Head-of-the-Line (HOL) blocking：排在队头的数据报阻止了队列中其他数据报向前移动

![[40-Network-layer-Data-plane-HOL-input-port.png]]
- 分组策略是 FCFS，发向同一输出端口的分组会阻塞等待，不同输出端口的分组则并行传输；
- Figure 4.8 shows an example in which two packets (darkly shaded) at the front of their input queues are destined for the same upper-right output port. Suppose that the switch fabric chooses to transfer the packet from the front of the upper-left queue. In this case, the darkly shaded packet in the lower-left queue must wait.
- But not only must this darkly shaded packet wait, so too must the lightly shaded packet that is queued behind that packet in the lower-left queue, even though there is no contention for the middle-right output port (the destination for the lightly shaded packet). This phenomenon is known as ***head-of-the-line*** (HOL) blocking in an input-queued switch—a queued packet in an input queue must wait for transfer through the fabric (even though its output port is free) because it is blocked by another packet at the head of the line. 
- `[Karol 1987]` shows that ==due to HOL blocking, the input queue will grow to unbounded length== (informally, this is equivalent to saying that significant packet loss will occur) ==under certain assumptions as soon as the packet arrival rate on the input links reaches only 58 percent of their capacity==. A number of solutions to HOL blocking are discussed in `[McKeown 1997]`.

#### 输出端口排队
- 当数据报从交换结构的到达输出端口的速度比输出端口向外传输数据的速率快时，就需要输出端口缓存
- 具体地，$R_{switch}$ 最大不应超过 $R_{\text{out line}}$ ，否则在转发出输出端口前，新的分组就会到达，从而拥挤在缓冲区，渐渐地耗尽可用缓冲区。
- 没有足够的缓存接收分组时，就要做出决定：丢弃到达的分组，或者删除一个或多个已排队的分组来腾出空间
	- 某些情况下，在缓存即将填满之前丢弃一个分组是有利的，这样的丢包最终会向发送方提供一个拥塞信号，这样的策略称为主动队列管理 Active Queue Management, AQM
	- One of the most widely studied and implemented AQM algorithms is the ***Random Early Detection*** (RED) algorithm `[Christiansen 2001]`. More recent AQM policies include PIE (the Proportional Integral controller Enhanced `[RFC 8033]`), and CoDel `[Nichols 2012]`.

![[40-Network-layer-Data-plane-output-queuing.png]]
- At time *t*, a packet has arrived at each of the incoming input ports, each destined for the uppermost outgoing port.
- Assuming identical line speeds and a switch operating at three times the line speed, one time unit later (that is, in the time needed to receive or send a packet), all three original packets have been transferred to the outgoing port and are queued awaiting transmission. 
- In the next time unit, one of these three packets will have been transmitted over the outgoing link.
- In our example, two new packets have arrived at the incoming side of the switch; one of these packets is destined for this uppermost output port. A consequence of such queuing is that a ***packet scheduler*** at the output port must choose one packet, among those queued, for transmission.

#### 缓冲多大才是“足够”？

Our study above has shown how a packet queue forms when bursts of packets arrive at a router’s input or (more likely) output port, and the packet arrival rate temporarily exceeds the rate at which packets can be forwarded. ***The longer the amount of time that this mismatch persists, the longer the queue will grow, until eventually a port’s buffers become full and packets are dropped***. 
> 之前的讨论说明了突发的分组到达时输出队列是如何形成的。当分组到达速率大于输出端口分发速率时，这个时间持续越久，队列越长，直至耗尽缓冲区、产生丢包。

One natural question is how much buffering should be provisioned at a port. It turns out the answer to this question is much more complicated than one might imagine and can teach us quite a bit about the subtle interaction among congestion-aware senders at the network’s edge and the network core!

For many years, ***the rule of thumb*** `[RFC 3439]` for buffer sizing was that ==the amount of buffering (B) should be equal to an average round-trip time (RTT) times the link capacity (C)==. Thus, a 10-Gbps link with an RTT of 250 msec would need an amount of buffering equal to $B = RTT\times  C = 2.5 Gbits$ of buffers. This result was based on an analysis of the queueing dynamics of a relatively small number of TCP flows `[Villamizar 1994]`. 
> RFC 3439 提出的拇指理论给出了缓冲区大小与平均 RTT、链路容量的关系：$B=RTT\times C$。但是这一规律基于相对少量的 TCP 流的动态排队分析。

More recent theoretical and experimental efforts `[Appenzeller 2004]`, however, suggest that when a large number of independent TCP flows (N) pass through a link, the amount of buffering needed is $B = RTT \times C / \sqrt{N}$. In core networks, where a large number of TCP flows typically pass through large backbone router links, the value of N can be large, with the decrease in needed buffer size becoming quite significant. `[Appenzeller 2004; Wischik 2005; Beheshti 2008]` provide very readable discussions of the buffer-sizing problem from a theoretical, implementation, and operational standpoint. 
> 然而基于大量 TCP 流的链路研究表明，其所需的缓冲区大小仅是 $B=RTT \times C / \sqrt{N}$ ，即随着 TCP 流的数量增多，缓冲区所需大小实际上在下降。

It’s temping to think that more buffering must be better—larger buffers would allow a router to absorb larger fluctuations in the packet arrival rate, thereby decreasing the router’s packet loss rate. ==But larger buffers also mean potentially longer queueing delays==. For gamers and for interactive teleconferencing users, tens of milliseconds count. Increasing the amount of per-hop buffer by a factor of 10 to decrease packet loss could increase the end-end delay by a factor of 10! Increased RTTs also make TCP senders less responsive and slower to respond to incipient congestion and/or packet loss. These delay-based considerations show that buffering is a double-edged sword—==buffering can be used to absorb short-term statistical fluctuations in traffic but can also lead to increased delay and the attendant concerns==. Buffering is a bit like salt—just the right amount of salt makes food better, but too much makes it inedible!
> 缓冲区并非越大越好。大缓冲区意味着更长的排队时延、更长的 RTT，这对延迟敏感的用户来说不可接受，会显著降低他们的响应性体验。
> 缓冲区是一把双刃剑——大缓冲区意味着有效应对突发流量，但同样会导致过长的延迟。

![[40-Network-layer-Data-plane-bufferbloat.png]]
In the discussion above, we’ve implicitly assumed that many independent senders are competing for bandwidth and buffers at a congested link. While this is probably an excellent assumption for routers within the network core, at the network edge this may not hold.
> 独立的发送方会竞争带宽和缓冲区，但这一假设对于网络核心中的路由器来说可能是成立的，但对网络边缘的设备来说并非如此。

Figure 4.10(a) shows a home router sending TCP segments to a remote game server. Following `[Nichols 2012]`, suppose that it takes 20ms to transmit a packet (containing a gamer’s TCP segment), that there are negligible queueing delays elsewhere on the path to the game server, and that the RTT is 200 ms. As shown in Figure 4.10(b), suppose that at time t = 0, a burst of 25 packets arrives to the queue. One of these queued packets is then transmitted once every 20 ms, so that at t = 200 msec, the first ACK arrives, just as the 21st packet is being transmitted. 
> 考虑一种情形，发送分组的延迟是 20ms，暂时忽视排队延迟，RTT 是 200ms，那么突发的 25 个分组需要转发时，每 20ms 会转发一个分组，第一个分组的 ACK 在 200ms 时返回，这时正要发送第 21 个分组。

This ACK arrival causes the TCP sender to send another packet, which is queued at the outgoing link of the home router. At t = 220, the next ACK arrives, and another TCP segment is released by the gamer and is queued, as the 22nd packet is being transmitted, and so on. You should convince yourself that in this scenario, ACK clocking results in a new packet arriving at the queue every time a queued packet is sent, resulting in queue size at the home router’s outgoing link that is always five packets!
> ACK 到达会导致 TCP 发送方继续发送下一个分组，这个分组一直等待在家庭路由器的输出链路上。第 220ms 时第二个 ACK 分组到达，发送下一个... 以此类推，这时，应当注意到 ACK 时钟导致每发送一个输出队列中的分组，才会有新的分组到达队列，从而家庭路由器的输出链路上的队列始终为 5 个分组大小！

That is, the end-end-pipe is full (delivering packets to the destination at the path bottleneck rate of one packet every 20 ms), but the amount of queueing delay is constant and persistent. As a result, the gamer is unhappy with the delay, and the parent (who even knows wireshark!) is confused because he or she doesn’t understand why delays are persistent and excessively long, even when there is no other traffic on the home network. 
> 也即，尽管端到端链路确实盈满（确实每 20ms 发送一个分组），但排队延迟却是固定的。（吞吐量能够实现，但延迟却居高不下）

This scenario above of long delay due to persistent buffering is known as ***bufferbloat*** and illustrates that not only is throughput important, but also minimal delay is important as well `[Kleinrock 2018]`, and that the interaction among senders at the network edge and queues within the network can indeed be complex and subtle. 
> 缓冲膨胀指的是持续地缓冲导致较长的延迟。这表明吞吐量和最小延迟都非常重要。

The DOCSIS 3.1 standard for cable networks that we will study in Chapter 6, recently added a specific AQM mechanism `[RFC 8033, RFC 8034]` to combat bufferbloat, while preserving bulk throughput performance.

### 分组调度
- 由**调度规则**选择排队的数据报进行传输（先来的不一定先传）
    - 调度：选择下一个要通过链路传输的分组
    - 调度策略：
        - **==FIFO scheduling==**：按照分组到来的次序发送（先到先服务）
            - ![[40-Network-layer-Data-plane-fifo.png]]
            - ![[40-Network-layer-Data-plane-fifo-timeline.png]]
            - 丢弃策略：如果分组到达一个满的队列，哪个分组将会被抛弃？
                - tail drop：丢弃刚到达的分组（抛尾部）
                - priority：根据优先权丢失/移除分组
                - random：随机地丢弃/移除
        - **==优先权调度==**：发送最高优先权的分组
            - 多类，不同类别有不同的优先权
                - 类别可能依赖于标记或者其他的头部字段，e.g. IP source/dest, port numbers, ds, etc.
                - 先传高优先级的队列中的分组，除非没有
                - 高（低）优先权中的分组传输次序：FIFO
            - ![[40-Network-layer-Data-plane-priority.png]]
            - ![[40-Network-layer-Data-plane-priority-timeline.png]]
            - Packets 1, 3, and 4 belong to the high-priority class, and packets 2 and 5 belong to the low-priority class.
            - Packet 4 (a high-priority packet) arrives during the transmission of packet 2 (a low-priority packet). Under a ***non-preemptive priority queuing discipline***, the transmission of a packet is not interrupted once it has begun. In this case, packet 4 queues for transmission and begins being transmitted after the transmission of packet 2 is completed.
        - **==Round Robin (RR) scheduling==**:
            - 循环扫描不同类型的队列，发送完一类的一个分组，再发送下一个类的一个分组，循环所有类
            - In the simplest form of round robin scheduling, a class 1 packet is transmitted, followed by a class 2 packet, followed by a class 1 packet, followed by a class 2 packet, and so on. 
            - A so-called ***work-conserving queuing discipline*** will never allow the link to remain idle whenever there are packets (of any class) queued for transmission. A work-conserving round robin discipline that looks for a packet of a given class but finds none will immediately check the next class in the round robin sequence.
            - ![[40-Network-layer-Data-plane-round-robin-timeline.png]]
            - In this example, packets 1, 2, and 4 belong to class 1, and packets 3 and 5 belong to the second class.
        - **==Weighted Fair Queuing==** (WFQ): 
            - ![[40-Network-layer-Data-plane-wfq.png]]
            - Arriving packets are classified and queued in the appropriate per-class waiting area. As in round robin scheduling, a WFQ scheduler will serve classes in a circular manner—first serving class 1, then serving class 2, then serving class 3, and then (assuming there are three classes) repeating the service pattern.
            - WFQ differs from round robin in that *each class may receive a differential amount of service in any interval of time*. Specifically, each class, i, is assigned a weight, $w_i$. Under WFQ, during any interval of time during which there are class i packets to send, class i will then be guaranteed to receive a fraction of service equal to $\frac{w_{i}}{\sum\limits w_{j}}$, where the sum in the denominator is taken over all classes that also have packets queued for transmission.
            - 在一段时间内，每个队列得到的服务时间是： $(W_i/\sum{W_i}) * t$ ，和权重成正比
            - 每个类在每一个循环中获得不同权重的服务量


## 4.3 IP：Internet Protocol

IP协议主要实现数据平面的转发功能

### 数据报格式

![[40-Network-layer-Data-plane-IP-datagram-format.png]]
- ***Version number***. These 4 bits ==specify the IP protocol version of the datagram==. By looking at the version number, the router can determine how to interpret the remainder of the IP datagram. Different versions of IP use different datagram formats. The datagram format for IPv4 is shown in Figure 4.17. The datagram format for the new version of IP (IPv6) is discussed in Section 4.3.4. 
- ***Header length***. Because an IPv4 datagram can contain a variable number of options (which are included in the IPv4 datagram header), these 4 bits are needed to ==determine where in the IP datagram the payload== (for example, the transport-layer segment being encapsulated in this datagram) ==actually begins==. Most IP datagrams do not contain options, so the typical IP datagram has a 20-byte header.
- ***Type of service***. The type of service (TOS) bits were included in the IPv4 header to ==allow different types of IP datagrams to be distinguished from each other==. For example, it might be useful to distinguish real-time datagrams (such as those used by an IP telephony application) from non-real-time traffic (e.g., FTP). The specific level of service to be provided is a policy issue determined and configured by the network administrator for that router. We also learned in Section 3.7.2 that ==two of the TOS bits are used for Explicit Congestion Notification==. ^84039d
- ***Datagram length***. This is the ==total length of the IP datagram (header plus data)==, measured in bytes. Since this field is 16 bits long, the theoretical maximum size of the IP datagram is 65,535 bytes. However, datagrams are rarely larger than 1,500 bytes, which allows an IP datagram to fit in the payload field of a maximally sized Ethernet frame.（数据报实际很少超过 1500 字节，这与以太网帧的长度有关）
- ***Identifier, flags, fragmentation offset***. These three fields have to do with so-called ==IP fragmentation==, when a large IP datagram is broken into several smaller IP datagrams which are then forwarded independently to the destination, where they are reassembled before their payload data is passed up to the transport layer at the destination host. Interestingly, the new version of IP, ==IPv6, does not allow for fragmentation==. We’ll not cover fragmentation here; but readers can find a detailed discussion online, among the “retired” material from earlier versions of this book.
> - flags 中有 DF 和 MF 两种，前者是 Don't fragment ，告知 router 不要对该数据报进行分片；后者是 More fragment，告知 router 这是否是分片数据报的最后一个，若不是，则为 1（即后续还有分片之意）
> 
> - fragmentation offset 描述片偏移，占据 13 位，即提供 2^13=8192 字节的偏移描述；

- ***Time-to-live***. The time-to-live (TTL) field is included to ==ensure that datagrams do not circulate forever== (due to, for example, a long-lived routing loop) in the network. This field is decremented by one each time the datagram is processed by a router. If the TTL field reaches 0, a router must drop that datagram.
- ***Protocol***. This field is ==typically used only when an IP datagram reaches its final destination==. The value of this field indicates the specific transport-layer protocol to which the data portion of this IP datagram should be passed. For example, a value of 6 indicates that the data portion is passed to TCP, while a value of 17 indicates that the data is passed to UDP. For a list of all possible values, see `[IANA Protocol Numbers 2016]`. Note that ==the protocol number in the IP datagram has a role that is analogous to the role of the port number field in the transport-layer segment==. The protocol number is the glue that binds the network and transport layers together, whereas the port number is the glue that binds the transport and application layers together. We’ll see in Chapter 6 that the link-layer frame also has a special field that binds the link layer to the network layer.
- ***Header checksum***. The header checksum ==aids a router in detecting bit errors in a received IP datagram==. The header checksum is computed by treating each 2 bytes in the header as a number and summing these numbers using 1s complement arithmetic. As discussed in Section 3.3, the 1s complement (反码) of this sum, known as the Internet checksum, is stored in the checksum field. A router computes the header checksum for each received IP datagram and detects an error condition if the checksum carried in the datagram header does not equal the computed checksum. ==Routers typically discard datagrams for which an error has been detected==. Note that ==the checksum **must be recomputed and stored again at each router**, since the TTL field, and possibly the options field as well, will change==. An interesting discussion of fast algorithms for computing the Internet checksum is `[RFC 1071]`. 
	- A question often asked at this point is, why does TCP/IP perform error checking at both the transport and network layers? There are several reasons for this repetition.
	- First, note that only the IP header is checksummed at the IP layer, while the TCP/ UDP checksum is computed over the entire TCP/UDP segment.
	- Second, TCP/ UDP and IP do not necessarily both have to belong to the same protocol stack. TCP can, in principle, run over a different network-layer protocol (for example, ATM) and IP can carry data that will not be passed to TCP/UDP.
- ***Source and destination IP addresses***. When a source creates a datagram, it inserts its IP address into the source IP address field and inserts the address of the ultimate destination into the destination IP address field. Often the source host determines the destination address via a DNS lookup, as discussed in Chapter 2. We’ll discuss IP addressing in detail in Section 4.3.2. 
- ***Options***. The options fields allow an IP header to be extended. Header options were meant to be used rarely—hence the decision to save overhead by not including the information in options fields in every datagram header. However, the mere existence of options does complicate matters—since datagram headers can be of variable length, one cannot determine a priori where the data field will start. Also, since some datagrams may require options processing and others may not, ==the amount of time needed to process an IP datagram at a router can vary greatly==. These considerations become particularly important for IP processing in high-performance routers and hosts. For these reasons and others, ==IP options were not included in the IPv6 header==, as discussed in Section 4.3.4.
- ***Data (payload)***. Finally, we come to the last and most important field—the *raison d’etre*（存在的理由，法语） for the datagram in the first place! In most circumstances, the data field of the IP datagram contains the transport-layer segment (TCP or UDP) to be delivered to the destination. However, the ==data field can carry other types of data, such as ICMP messages== (discussed in Section 5.6).

Note that an IP datagram has a total of 20 bytes of header (assuming no options). If the datagram carries a TCP segment, then each datagram carries a total of 40 bytes of header (20 bytes of IP header plus 20 bytes of TCP header) along with the application-layer message.
> IP 数据报的头部有 20 字节，TCP 的头部有 20 字节，再加上应用层的数据（应用层当然也有各自协议的头部+数据体）。

### IPv4分片和重组
> IPv6 禁止分片

- 网络链路有 MTU（Maximum Transmission Unit） —— 链路层帧所携带的最大数据长度
    - 不同的链路类型有不同的 MTU 
- 大的IP数据报在网络上被分片(“fragmented”)
    - 一个数据报被分割成若干个小的数据报
        - 相同的ID，知道属于同一个数据报
        - 不同的偏移量(offset)：小数据报的第一个字节在字节流中的位置除以8
        - 最后一个分片标记为0（fragmentation flag 标识位），其他分片的 fragmentation flag 标识位标记为1
    - “重组”只在最终的目标主机进行（不占用路由器的资源）
    - IP头部的信息被用于标识，排序相关分片
    - 若某一片丢失，整个全部丢弃

> When a destination host receives a series of datagrams from the same source, it needs to determine whether any of these datagrams are fragments of some original, larger datagram. If some datagrams are fragments, it must further determine when it has received the last fragment and how the fragments it has received should be pieced back together to form the original datagram.
> 
> To allow the destination host to perform these reassembly tasks, the designers of IPv4 put ***identification, flag, and fragmentation offset fields*** in the IP datagram header. When a datagram is created, the sending host stamps the datagram with an identification number as well as source and destination addresses.
> 当生成一个数据报时，发送方设置源和目的地址时，同时贴上标识号
> 
> Typically, the sending host ==increments the identification number for each datagram it sends==. When a router needs to fragment a datagram, each resulting datagram (that is, fragment) is stamped with the source address, destination address, and identification number of the original datagram. When the destination receives a series of datagrams from the same sending host, it can examine the identification numbers of the datagrams to determine which of the datagrams are actually fragments of the same larger datagram. 
> 
> Because IP is an unreliable service, one or more of the fragments may never arrive at the destination. For this reason, in order for the destination host to be absolutely sure it has received the last fragment of the original datagram, ==the last fragment has a flag bit set to 0, whereas all the other fragments have this flag bit set to 1==. Also, in order for the destination host to determine whether a fragment is missing (and also to be able to reassemble the fragments in their proper order), the ==***offset field*** is used to specify where the fragment fits within the original IP datagram==.

> [! example] 例：
> ![[40-Network-layer-Data-plane-fragmetation.png]]
> - $4000$ 字节数据报
>     - $20$ 字节头部
>     - $3980$ 字节数据
> - $MTU = 1500 bytes$
> - 第一片： $20$ 字节头部 + $1480$ 字节数据
>     - 偏移量： $0$
> - 第二片： $20$ 字节头部 + $1480$ 字节数据（ $1480$ 字节应用数据）
>     - 偏移量： $1480/8=185$
> - 第三片： $20$ 字节头部 + $1020$ 字节数据（ $1020$ 字节应用数据）
>     - 偏移量： $2960/8=370$

### IPv4 编址

#### 接口
- 接口：主机/路由器和物理链路的连接处
    - 一台主机通常只有一条链路连接到网络；当主机中的 IP 想要发送一个数据报时，它就在该链路上发送；
    - 路由器通常拥有多个接口（路由器连接若干个物理网络，在多个物理网络之间进行分组转发）
    - 主机可能有一个或多个接口
    - IP 要求每台主机和路由器的接口都有自己的 IP 地址——即，IP 地址和每一个接口关联，该接口对应一个“网卡”

每个 IPv4 地址长度为 32 位:
- 理论上有 2^32 个可能的 IPv4 地址（实际上并没有）
- 这些地址通常按照点分十进制记法书写——地址中每个字节用它的十进制形式书写，各字节以句点隔开：
- IPv4 binary:  11000001 00100000 11011000 00001001
- IPv4 decimal: 193.32.216.9

Q：这些接口是如何连接的？     
A：有线以太网网口链接到以太网络交换机连接       
目前：无需担心一个接口是如何接到另外一个接口（中间没有路由器，一跳可达）

#### 子网(Subnets)
- IP地址：
    - 子网部分（高位bits）
    - 主机部分（低位 bits）
- 什么是子网(subnet)？
    - 一个子网内的节点（主机或者路由器）它们的**IP地址的高位部分相同**，这些节点构成的网络的一部分叫做子网
    - **无需路由器介入**，子网内各主机可以在物理上相互直接到达，在IP层面一跳可达（但是在数据链路层可能需要借助交换机）

![[40-Network-layer-Data-plane-subnets.png]]

子网判断方法：
- 要判断一个子网，将每一个接口从主机或者路由器上分开，构成了一个个网络的孤岛
- 每一个孤岛（网络）都是一个都可以被称之为 subnet。

![[40-Network-layer-Data-plane-subnet-mask.png]]
- 相同高位的一组 IP（包括主机和路由器端口）组成一个子网，它们的划分通过子网掩码来得到：
- 上图中子网掩码的记号为 `/24`，表明高 24 位相同的 IP 组成一个子网，第 8 位用以区分子网内的主机——一个子网支持 2^8=256 台主机；

![[40-Network-layer-Data-plane-router-port-subset.png]]
- Three subnets, 223.1.1.0/24, 223.1.2.0/24, and 223.1.3.0/24, are similar to the subnets we encountered in Figure 4.18. 
- But note that there are three additional subnets in this example as well: 
	- one subnet, 223.1.9.0/24, for the interfaces that connect routers R1 and R2; 
	- another subnet, 223.1.8.0/24, for the interfaces that connect routers R2 and R3; 
	- and a third subnet, 223.1.7.0/24, for the interfaces that connect routers R3 and R1.
- For a general interconnected system of routers and hosts, we can use the following recipe to define the subnets in the system: ***To determine the subnets, detach each interface from its host or router, creating islands of isolated networks, with interfaces terminating the end points of the isolated networks. Each of these isolated networks is called a subnet***.
- If we apply this procedure to the interconnected system in Figure 4.20, we get six islands or subnets.

互联网中的路由通过网络号进行一个个子网的计算，以网络为单位进行传输，而非具体到单个主机。

#### 因特网地址分配策略：CIDR
IP 编址：CIDR (Classless InterDomain Routing, 无类域间路由)
- 子网部分可以在任意的位置，按需划分
- 地址格式：a.b.c.d/x，其中 x 是地址中子网号的长度（网络号的长度），前 x 位由此称为该地址的前缀，通常一个组织被分配一块连续的地址——具有相同前缀的一段地址
- 仅考虑 x 位即可完成转发，极大地减少了路由器中转发表的长度
- 剩余 32-x 位可认为是用于组织内部的设备分配，只有在组织内部的路由器中转发分组才会考虑这一字段，进一步地，根据这一字段组织可以在内部继续细分子网：
	- a.b.c.d/21 表示组织的子网
	- a.b.c.d/24 表示组织内部继续细分的子网

#### 分类编址

CIDR 采用前，IP 地址的网络部分限定长度为 8、16、24 字节——分类编址 classful addressing 方案：
- Subnets with 8-, 16-, and 24-bit subnet addresses were known as class A, B, and C networks, respectively.
- Class C (/24) subnet could accommodate only up to `2^8-2=254` hosts (two of the 28 5 256 addresses are reserved for special use)—too small for many organizations. 
- However, a class B (/16) subnet, which supports up to 65,634 hosts, was too large. 
- Under classful addressing, an organization with, say, 2,000 hosts was typically allocated a class B (/16) subnet address. This ==led to a rapid depletion of the class B address space and poor utilization of the assigned address space==. 
	- For example, the organization that used a class B address for its 2,000 hosts was allocated enough of the address space for up to 65,534 interfaces—leaving more than 63,000 addresses that could not be used by other organizations.

#### 层次编址
This example of an ISP that connects eight organizations to the Internet nicely illustrates how carefully allocated CIDRized addresses facilitate routing. Suppose, as shown in Figure 4.21, that the ISP (which we’ll call Fly-By-Night-ISP) advertises to the outside world that it should be sent any datagrams whose first 20 address bits match 200.23.16.0/20. The rest of the world need not know that within the address block 200.23.16.0/20 there are in fact eight other organizations, each with its own subnets. This ability to use a single prefix to advertise multiple networks is often referred to as address aggregation (also route aggregation or route summarization). 
![[40-Network-layer-Data-plane-hierarchical-addressing.png]]

Address aggregation works extremely well when addresses are allocated in blocks to ISPs and then from ISPs to client organizations. But what happens when addresses are not allocated in such a hierarchical manner? What would happen, for example, if Fly-By-Night-ISP acquires ISPs-R-Us and then has Organization 1 connect to the Internet through its subsidiary ISPs-R-Us? As shown in Figure 4.21, the subsidiary ISPs-R-Us owns the address block 199.31.0.0/16, but Organization 1’s IP addresses are unfortunately outside of this address block. What should be done here? Certainly, Organization 1 could renumber all of its routers and hosts to have addresses within the ISPs-R-Us address block. But this is a costly solution, and Organization 1 might well be reassigned to another subsidiary in the future. The solution typically adopted is for Organization 1 to keep its IP addresses in 200.23.18.0/23. In this case, as shown in Figure 4.22, Fly-By-Night-ISP continues to advertise the address block 200.23.16.0/20 and ISPs-R-Us continues to advertise 199.31.0.0/16. However, ISPs-R-Us now also advertises the block of addresses for Organization 1, 200.23.18.0/23. When other routers in the larger Internet see the address blocks 200.23.16.0/20 (from Fly-By-Night-ISP) and 200.23.18.0/23 (from ISPsR-Us) and want to route to an address in the block 200.23.18.0/23, they will use longest prefix matching (see Section 4.2.1), and route toward ISPs-R-Us, as it advertises the longest (i.e., most-specific) address prefix that matches the destination address.

![[40-Network-layer-Data-plane-ISPs-R-Us.png]]
#### IP 地址分类
![[40-Network-layer-Data-plane-ip-address-classify.png]]
- Class A：126 networks ( $2^7-2$ , 0.0.0.0 and 1.1.1.1 not available), 16 million hosts ( $2^{24}-2$ , 0.0.0.0 and 1.1.1.1 not available)
- Class B：16382 networks ( $2^{14}-2$ , 0.0.0.0 and 1.1.1.1 not available), 64 K hosts ( $2^{16}-2$ , 0.0.0.0 and 1.1.1.1 not available)
- Class C：2 million networks, 254 host ( $2^8-2$ , 0.0.0.0 and 1.1.1.1 not available)
- Class D：multicast
- Class E：reserved for future

*注：A、B、C类称为 单播地址 （发送给单个），D类称为 主播地址 （发送给特定的组的所有人）*

![[40-Network-layer-Data-plane-special-ip.png]]
特殊 IP 地址的约定
- 一些约定：
    - 子网部分：全为0---本网络
    - 主机部分：全为0---本主机
    - 主机部分：全为1---广播地址，这个网络的所有主机
- 特殊IP地址


内网（专用）IP地址
- 专用地址：地址空间的一部份供专用地址使用
- 永远不会被当做公用地址来分配，不会与公用地址重复
    - 只在局部网络中有意义，用来区分不同的设备
- 路由器不对目标地址是专用地址的分组进行转发
- 专用地址范围
    - Class A 10.0.0.0-10.255.255.255  MASK 255.0.0.0
    - Class B 172.16.0.0-172.31.255.255  MASK 255.255.0.0
    - Class C 192.168.0.0-192.168.255.255 MASK 255.255.255.0

#### 子网掩码与转发表
子网掩码(subnet mask)
- 32bits , 0 or 1 in each bit
    - 1：bit位置表示子网部分
    - 0：bit位置表示主机部分（主机号在查询路由表时没有意义，路由信息的计算以网络为单位）
- 原始的A、B、C类网络的子网掩码分别是
    - A：255.0.0.0：11111111 00000000 0000000 00000000
    - B：255.255.0.0：11111111 11111111 0000000 00000000
    - C：255.255.255.0：11111111 11111111 11111111 00000000
- CIDR下的子网掩码例子：
    - 11111111 11111111 11111100 00000000
- 另外的一种表示子网掩码的表达方式
    - /# ： 例如 /22 表示前面22个bit为子网部分

转发表和转发算法

| Destination Subnet Num |      Mask       | Next hop | Interface |
|:----------------------:|:---------------:|:--------:|:---------:|
|      202.38.73.0       | 255.255.255.192 |   IPx    |   Lan1    |
|      202.38.64.0       | 255.255.255.192 |   IPy    |   Lan2    |
|          ...           |       ...       |   ...    |    ...    |
|        Default         |        -        |   IPz    |   Lan0    |
- 获得IP数据报的目标地址
- 对于转发表中的每一个表项
- 如 (IP Des addr) & (mask)== destination，则按照表项对应的接口转发该数据报
- 如果都没有找到，则使用默认表项，通过默认端口（通常是一个网络的出口路由器所对应的IP）转发数据报

#### 主机如何分配、得到一个 IP 地址?

为了获取一块 IP 地址用于一个组织的子网内，网络管理员首先会与 ISP 联系，ISP 从自己能够支配的地址块中再划分给网络管理员。

> [! example] 例：IP 地址划分举例     
> 
>ISP地址:200.23.16.0/20 <u>11001000 00010111 0001</u>0000 00000000 
>组织 0: 200.23.16.0/23 <u>11001000 00010111 0001000</u>0 00000000 
>组织 1: 200.23.18.0/23 <u>11001000 00010111 0001001</u>0 00000000 
>组织 2: 200.23.20.0/23 <u>11001000 00010111 0001010</u>0 00000000
> …… 
>组织 7: 200.23.30.0/23 <u>11001000 00010111 0001111</u>0 00000000
>
>地址 21、22、23 一共 3 位，故可划分处 2^3=8 个子网

>[! note] 全球权威机构：管理 IP 总空间
>ICANN (Internet Corporation for Assigned Names and Numbers)
>- 分配地址
>- 管理 DNS 根服务器
>- 分配域名，解决冲突

#### 如何获取一个主机的地址？

- 网络管理员将组织得到的 IP 配置在主机的一个文件中
    - Windows: control-panel -> network -> configuration -> tcp/ip -> properties
    - UNIX: /etc/rc.config

- DHCP (Dynamic Host Configuration Protocol)：
	- 从服务器中动态获得一个 IP 地址
		- 以及子网掩码 Mask、
		- local name server（DNS 服务器的域名和 IP 地址）、
		- default getaway（第一跳路由器的 IP 地址（默认网关））
	- “plug-and-play”，自动配置，接上即用
		- 另外，根据网络管理员的配置不同，分配得到的 IP 也不同，既可以是固定的 IP，也可以是临时的 IP
		- 临时的 IP 会变化，只用在用户上网时分配，其余时间该 IP 可以被其他上网用户使用，提高效率
	- 目标：允许主机在加入网络的时候，动态地从服务器那里获得 IP 地址及相关配置信息：
		- 可以更新对主机在用 IP 地址的租用期——租期快到了
		- 重新启动时，允许重新使用以前用过的 IP 地址
		- 支持移动用户加入到该网络（短期在网）
	- DHCP 工作概况：
		- DHCP 是客户-服务器协议；最简单场合下，每个子网都有一个 DHCP 服务器，若子网没有 DHCP 服务器，则需要一个 DHCP 中继代理（通常是路由器），代理知道用于该网络的 DHCP 服务器地址。如下图：
		- ![[40-Network-layer-Data-plane-DHCP-client-server.png]]
		- 主机上线时广播“DHCP discover” 报文（可选）（目标 IP：255.255.255.255，进行广播）
		- DHCP 服务器用 “DHCP offer”提供报文响应（可选）
		- 主机请求 IP 地址：发送 “DHCP request” 报文（这第二次握手是因为可能有多个 DHCP 服务器，要确认用哪一个）
		- DHCP 服务器发送地址：“DHCP ack” 报文


> [! example] DHCP 实例：
> 
> ![[40-Network-layer-Data-plane-DHCP-interaction.png]]
>- ***DHCP server discovery***. The first task of a newly arriving host is to find a DHCP server with which to interact. This is done using a ==DHCP discover message==, which a client sends within a **UDP packet** to port 67. The UDP packet is encapsulated in an IP datagram. But to whom should this datagram be sent? The host doesn’t even know the IP address of the network to which it is attaching, much less the address of a DHCP server for this network. ==Given this, the DHCP client creates an IP datagram containing its DHCP discover message along with the broadcast destination IP address of 255.255.255.255 and a “this host” source IP address of 0.0.0.0==. The DHCP client passes the IP datagram to the link layer, which then broadcasts this frame to all nodes attached to the subnet (we will cover the details of link-layer broadcasting in Section 6.4).
>
>- ***DHCP server offer***(s). A DHCP server receiving a DHCP discover message responds to the client with a ==DHCP offer message== that is broadcast to all nodes on the subnet, ==again using the IP broadcast address of 255.255.255.255==. (You might want to think about why this server reply must also be broadcast. 因为它也不知道 DHCP 客户端的 IP 地址). Since several DHCP servers can be present on the subnet, the client may find itself in the enviable position of being able to choose from among several offers. (DHCP 客户端可能会收到多份服务端发来的 DHCP server offer，选取其中最近、最好的服务器即可) 
>	- Each server offer message contains the ==transaction ID== of the received discover message, the ==proposed IP address== for the client, the ==network mask==, and an ==IP address lease time==—the amount of time for which the IP address will be valid. It is common for the server to set the lease time to several hours or days `[Droms 2002]`. 
>
>- ***DHCP request***. The newly arriving client will choose from among one or more server offers and ==respond to its selected offer with a DHCP request message, echoing back the configuration parameters==.
>
>- ***DHCP ACK***. The server responds to the DHCP request message with a DHCP ACK message, confirming the requested parameters.
>
>Once the client receives the DHCP ACK, the interaction is complete and the client can use the DHCP-allocated IP address for the lease duration. Since a client may want to use its address beyond the lease’s expiration, DHCP also provides a mechanism that allows a client to renew its lease on an IP address.（在 IP 租借期内可以使用，并且通过 DHCP 也可以方便地续约租借IP）

>[! warning] DHCP 的严重缺陷——移动性差
>From a mobility aspect, DHCP does have one very significant shortcoming. Since a new IP address is obtained from DHCP each time a node connects to a new subnet, a ==TCP connection to a remote application cannot be maintained as a mobile node moves between subnets==. 
>
>In Chapter 7, we will learn how mobile cellular networks allow a host to retain its IP address and ongoing TCP connections as it moves between base stations in a provider’s cellular network. Additional details about DHCP can be found in [Droms 2002] and [dhc 2020]. An open source reference implementation of DHCP is available from the Internet Systems Consortium [ISC 2020].

### NAT(Network Address Translation)

由前文所述，IPv4 地址其实远远不够用，划分子网时经常利用率极低。因此出现了网络地址转换技术：分组离开子网时共用一个子网的 IP 地址，到达子网的路由器分界时再转换为内网地址。
![[40-Network-layer-Data-plane-NAT.png]]

- 动机：本地网络只有一个有效IP地址：
    - 不需要从ISP分配一块地址，可用一个IP地址用于所有的（局域网）设备 —— 省钱
    - 可以在局域网改变设备的地址情况下而无须通知外界
    - 可以改变ISP（地址变化）而不需要改变内部的设备地址
    - 局域网内部的设备没有明确的地址，对外是不可见的 —— 安全

![[40-Network-layer-Data-plane-NAT-instance.png]]
- The NAT-enabled router, residing in the home, has an interface that is part of the home network on the right of Figure 4.25. Addressing within the home network is exactly as we have seen above—all four interfaces in the home network have the same subnet address of 10.0.0.0/24. The address space 10.0.0.0/8 is one of three portions of the IP address space that is reserved in `[RFC 1918]` for a *private network* or a *realm with private addresses*, such as the home network in Figure 4.25. ***A realm with private addresses refers to a network whose addresses only have meaning to devices within that network***. 
> NAT-enabled 的路由器端口向子网内有一个子网 IP (10.0.0.1/24)，这个子网 IP 专用于子网内的主机之间分组发送。而对外，该端口又有一个公网 IP (128.119.40.186)

- To see why this is important, consider the fact that there are hundreds of thousands of home networks, many using the same address space, 10.0.0.0/24. Devices within a given home network can send packets to each other using 10.0.0.0/24 addressing. However, packets forwarded beyond the home network into the larger global Internet clearly cannot use these addresses (as either a source or a destination address) because there are hundreds of thousands of networks using this block of addresses. ***That is, the 10.0.0.0/24 addresses can only have meaning within the given home network***. But if private addresses only have meaning within a given network, how is addressing handled when packets are sent to or received from the global Internet, where addresses are necessarily unique? The answer lies in understanding NAT.

采用 NAT 技术，如果客户端需要连接在 NAT 后面的服务器，会出现**NAT 穿透问题**：出去没问题，可以找得到服务器，但是若外面想进来和内网的主机通信确做不到，无法找到通信主机。如客户端需要连接地址为 10.0.0.1 的服务器，但是服务器地址 10.0.0.1 是 LAN 本地地址（客户端不能够使用其作为目标地址），整网只有一个外部可见地址：138.76.29.7 。有以下解决方案：
- 方案 1：静态配置 NAT：转发进来的对服务器特定端口连接请求
    - e.g. (123.76.29.7, port 2500) 总是转发到 10.0.0.1 port 25000
- 方案 2：Universal Plug and Play (UPnP) Internet Gateway Device (IGD) 协议。允许 NATted 主机可以：
    - 获知网络的公共 IP 地址 (138.76.29.7)
    - 列举存在的端口映射
    - 增/删端口映射（在租用时间内）
    - i.e. 自动化静态 NAT 端口映射配置
- 方案 3：中继
    - NAT 后面的服务器建立和中继的连接
    - 外部的客户端链接到中继
    - 中继在 2 个连接之间桥接

总之：
- 实现：NAT路由器必须：
    - 数据包外出：替换源地址和端口号为NAT IP地址和新的端口号，目标IP和端口不变
        - 远端的C/S将会用NAP IP地址，新端口号作为目标地址
    - 记住每个转换替换对（在NAT转换表中）
        - 源IP，端口 vs  NAP IP ，新端口
    - 数据包进入：替换目标IP地址和端口号，采用存储在NAT表中的mapping表项，用（源IP，端口）

*实际上就是用外网的某个IP代替内网里面的网络号*     
*出去的时候替换 原来IP 和 端口号*       
*进来的时候替换 目标IP 和 端口号*       

> [! example] NAT 进行 IP、端口号替换的例子
> ![[40-Network-layer-Data-plane-NAT-instance.png]]
> Consider the example in Figure 4.25. Suppose a user sitting in a home network behind host 10.0.0.1 requests a Web page on some Web server (port 80) with IP address 128.119.40.186. 
> 
> 1. The host 10.0.0.1 assigns the (arbitrary) source port number 3345 and sends the datagram into the LAN. The NAT router receives the datagram, generates a new source port number 5001 for the datagram, replaces the source IP address with its WAN-side IP address 138.76.29.7, and replaces the original source port number 3345 with the new source port number 5001. When generating a new source port number, the NAT router can select any source port number that is not currently in the NAT translation table. (*Note that because a port number field is 16 bits long, the NAT protocol can support over 2^16=65,536 simultaneous connections with a single WAN-side IP address for the router!*) NAT in the router also adds an entry to its NAT translation table.
> 
>2. The Web server, blissfully unaware that the arriving datagram containing the HTTP request has been manipulated by the NAT router, ==responds with a datagram whose destination address is the IP address of the NAT router, and whose destination port number is 5001==. When this datagram arrives at the NAT router, the router indexes the NAT translation table using the destination IP address and destination port number to obtain the appropriate IP address (10.0.0.1) and destination port number (3345) for the browser in the home network. The router then rewrites the datagram’s destination address and destination port number, and forwards the datagram into the home network.

>[! note] NAT 的弊端，或者说，争议
>- 路由器只应该对第 3 层做信息处理，而这里对端口号（4 层）作了处理
>- 违反了 end-to-end 原则
>	- 端到端原则：复杂性放到网络边缘
>	- 无需借助中转和变换，就可以直接传送到目标主机
>	- NAT 可能要被一些应用设计者考虑, eg, P2P applications
>- 外网的机器无法主动连接到内网的机器上
>
>==IPv6 势在必行！NAT 不过是饮鸩止渴==。

>[! note] NAT 情况下家庭主机和路由器如何获取其 IP 地址？
>Often, the answer is the same—***DHCP***!
>
>The router gets its address from the ISP’s DHCP server, and the router runs a DHCP server to provide addresses to computers within the NAT-DHCP-router-controlled home network’s address space.

### IPv6

#### 动机
- 初始动机：32-bit地址空间将会被很快用完
- 另外的动机：
    - 头部格式改变帮助加速处理和转发
        - TTL-1
        - 头部checksum
        - 分片
    - 头部格式改变帮助QoS 

#### IPv6数据报格式
![[40-Network-layer-Data-plane-IPv6-datagram-format.png]]
IPv6 新增的格式变化：
- ***Expanded addressing capabilities***. IPv6 increases the size of the ==IP address from 32 to 128 bits==. This ensures that the world won’t run out of IP addresses.  In addition to unicast and multicast addresses, IPv6 has introduced a new type of address, called an ==anycast address==, that allows a datagram to be delivered to any one of a group of hosts. (This feature could be used, for example, to send an HTTP GET to the nearest of a number of mirror sites that contain a given document.) 
> IPv6 最直接的改动就是 IP 地址从 32 位增长到 128 位。其次除了单播和多播外，还引入了任播地址 anycast address，它允许将数据报交付给一组主机中的任何一个，比如用在向镜像网站发送 GET 请求时。

- ***A streamlined 40-byte header***. The resulting ==40-byte fixed-length header== allows for faster processing of the IP datagram by a router. A new encoding of options allows for more flexible options processing.
> IPv6 首部长度为定长 40 字节，其中源地址和目的地址各占 16 字节。删减了很多 IPv4 中不必要的标记，请看下文。

- ***Flow labeling***. IPv6 has an elusive definition of a flow. RFC 2460 states that this allows “==labeling of packets belonging to particular flows for which the sender requests special handling, such as a non-default quality of service or real-time service==.” For example, audio and video transmission might likely be treated as a flow. On the other hand, the more traditional applications, such as file transfer and e-mail, might not be treated as flows. It is possible that the traffic carried by a high-priority user (for example, someone paying for better service for their traffic) might also be treated as a flow. What is clear, however, is that the designers of IPv6 foresaw the eventual need to be able to differentiate among the flows, even if the exact meaning of a flow had yet to be determined.
> 所谓流，就是指用于特殊服务的流量，如需要实时服务的流量、或者高优先级用户的流量，而传统的服务一般不是流。这个定义还很模糊。

IPv6 分组各字段的含义：
- ***Version***. This 4-bit field identifies the IP version number. Not surprisingly, IPv6 carries a value of 6 in this field. Note that putting a 4 in this field does not create a valid IPv4 datagram. (If it did, life would be a lot simpler—see the discussion below regarding the transition from IPv4 to IPv6.)
> IPv6 头部的 Version 字段值为 6，表明要用 IPv6 的方式解释分组，仅仅修改 Version 6 为 4 不能得到合法的 IPv4 分组。

- ***Traffic class***. The 8-bit traffic class field, like the TOS ([[#^84039d|type of service]]) field in IPv4, can be used to give priority to certain datagrams within a flow, or it can be used to give priority to datagrams from certain applications (for example, voice-over-IP) over datagrams from other applications (for example, SMTP e-mail).
> 流量类型用于赋予不同的数据报以特定优先级，完成不同的服务。

- ***Flow label***. As discussed above, this 20-bit field is used to identify a flow of datagrams. 
> 流标签有 20 位长。

- ***Payload length***. This 16-bit value is treated as an unsigned integer giving the number of bytes in the IPv6 datagram following the fixed-length, 40-byte datagram header.
> 给出有效载荷数据的长度，以无符号整数解释，理论最长为 2^16=65536 个字节

- ***Next header***. ==This field identifies the protocol to which the contents (data field) of this datagram will be delivered (for example, to TCP or UDP)==. The field uses the same values as the protocol field in the IPv4 header.
>给出该数据报要交付给传输层的哪一个协议。

- ***Hop limit***. The contents of this field are decremented by one by each router that forwards the datagram. If the hop limit count reaches zero, a router must discard that datagram.
> 每一跳就减一，值为 0 时丢弃数据报，防止在链路中过长时间存在。

- ***Source and destination addresses***. The various formats of the IPv6 128-bit address are described in RFC 4291. 
- ***Data***. This is the payload portion of the IPv6 datagram. When the datagram reaches its destination, the payload will be removed from the IP datagram and passed on to the protocol specified in the next header field.

相比较 IPv4，IPv6 中少了如下几个字段：
- ***Fragmentation/reassembly***. IPv6 does not allow for fragmentation and reassembly at intermediate routers; these operations can be performed only by the source and destination. ==If an IPv6 datagram received by a router is too large to be forwarded over the outgoing link, the router simply drops the datagram and sends a “Packet Too Big” ICMP error message (see Section 5.6) back to the sender==. The sender can then resend the data, using a smaller IP datagram size. Fragmentation and reassembly is a time-consuming operation; removing this functionality from the routers and placing it squarely in the end systems considerably speeds up IP forwarding within the network. 
> IPv6 不允许分组和重整，如果 IPv6 的数据报被接收路由器认为过长，就会将其丢弃并回送一个分组过长的 ICMP 错误信息给发送方。
> 
> 分组和重整是一件耗时很长的操作，IPv6 将这项任务交给端系统去做，而不是浪费路由器的关键时间。

- ***Header checksum***. Because the transport-layer (for example, TCP and UDP) and link-layer (for example, Ethernet) protocols in the Internet layers perform checksumming, the designers of IP probably felt that this functionality was sufficiently redundant in the network layer that it could be removed. Once again, fast processing of IP packets was a central concern. Recall from our discussion of IPv4 in Section 4.3.1 that since the IPv4 header contains a TTL field (similar to the hop limit field in IPv6), the IPv4 header checksum needed to be recomputed at every router. As with fragmentation and reassembly, this too was a costly operation in IPv4.
> 头部校验和也被舍弃，这是考虑到链路层和传输层都提供了足够有效的冗余措施用以检错。并且由于 IPv4 中 TTL 在每一跳的自减，都会导致重新计算校验和并写回，这极大地增加了路由器的处理时间。

- ***Options***. An options field is no longer a part of the standard IP header. However, it has not gone away. ==Instead, the options field is one of the possible next headers pointed to from within the IPv6 header==. That is, just as TCP or UDP protocol headers can be the next header within an IP packet, so too can an options field. The removal of the options field results in a fixed-length, 40-byte IP header.
> 选项字段并非消失，而是作为 next header 字段的指示目标存在，就像 TCP 或 UDP 可以作为 next header 的指示目标一样。

#### 从 IPv4到 IPv6的迁移

- IPv6 是后向兼容的，能够处理 IPv4 的数据报，但 IPv4 不能处理 IPv6 的数据报
- 不是所有的路由器都能够同时升级的，设置一个标记日 “flag days”，在那一天全部宕机升级，这不现实
- 在 IPv4和 IPv6路由器混合时，网络如何运转？——建隧道！
	- 隧道：在 IPv4 路由器之间传输的 IPv4 数据报中携带 IPv6 数据报
	- ![[40-Network-layer-Data-plane-tunneling.png]]
	- Suppose two IPv6 nodes (in this example, B and E in Figure 4.27) want to interoperate using IPv6 datagrams but are connected to each other by intervening IPv4 routers. We refer to ==the intervening set of IPv4 routers between two IPv6 routers as a **tunnel**==, as illustrated in Figure 4.27. 
	- With tunneling, the IPv6 node on the sending side of the tunnel (in this example, B) ==takes the entire IPv6 datagram and puts it in the data (payload) field of an IPv4 datagram==. This IPv4 datagram is then addressed to the IPv6 node on the receiving side of the tunnel (in this example, E) and sent to the first node in the tunnel (in this example, C). 
	- The intervening IPv4 routers in the tunnel route this IPv4 datagram among themselves, just as they would any other datagram, ==blissfully unaware that the IPv4 datagram itself contains a complete IPv6 datagram==. 
	- The IPv6 node on the receiving side of the tunnel eventually receives the IPv4 datagram (it is the destination of the IPv4 datagram!), determines that the IPv4 datagram contains an IPv6 datagram (by observing that the protocol number field in the IPv4 datagram is 41 `[RFC 4213]`, indicating that the IPv4 payload is a IPv6 datagram), extracts the IPv6 datagram, and then routes the IPv6 datagram exactly as it would if it had received the IPv6 datagram from a directly connected IPv6 neighbor.

[IPv6中国发展史 —— 温竣岩](https://www.bilibili.com/video/BV1i14y157YV/?spm_id_from=333.999.0.0&vd_source=77e5fb53d88adf1084faadbdb466558d)

## 4.4 通用转发和SDN

之前介绍的路由大部分都是传统方式，下面来看**通用转发**和**SDN的方式**。

### 传统路由的弊端

传统方式：1. 每台设备上既实现控制功能，又实现数据平面 2. 控制功能分布式实现 3. 路由表-粘连
- 传统方式的缺陷：
    - 垂直集成（每台路由器或其他网络设备，包括：1.专用的硬件、私有的操作系统；2.互联网标准协议(IP, RIP, IS-IS, OSPF, BGP)的私有实现；从上到下都由一个厂商提供（代价大、被设备上“绑架”“）） --> 昂贵、不便于创新的生态
    - 分布式、固化设备功能 --> 网络设备种类繁多
        - 交换机；防火墙；NAT；IDS；负载均衡设备
        - 未来：不断增加的需求和相应的网络设备
        - 需要不同的设备去实现不同的网络功能
            - 每台设备集成了控制平面和数据平面的功能
            - 控制平面分布式地实现了各种控制平面功能
            - **升级和部署网络设备非常困难**
        - 无法改变路由等工作逻辑，设备基本上只能（分布式升级困难）按照固定方式工作，**控制逻辑固化**，无法实现流量工程等高级特性
        - 配置错误影响全网运行；升级和维护会涉及到全网设备：管理困难
        - 要增加新的网络功能，需要设计、实现以及部署新的特定设备，设备种类繁

考虑到以上缺点，在2005年前后，开始重新思考网络控制平面的处理方式：SDN
- 集中：远程的控制器集中实现控制逻辑，通过南向接口将流表发送给每个设备中的控制代理
- 远程：数据平面和控制平面的分离
### 通用转发

回顾基于目的地的转发：[[#输入端口处理和基于目的地的转发]]
- 查找目的 IP 地址（匹配）
- 然后将分组发送到有特定输出端口的交换结构（动作）

通用匹配：
- 对协议栈的多个首部字段进行“**匹配**”，这些首部字段与不同层次的不同协议相关联
- **动作**：
	- 将分组转发到一个或多个输出端口（类似于基于目的地的转发）、
	- 跨越多个通向服务的离开接口进行负载均衡分组
	- 重写首部值（类似 NAT）
	- 有意识地阻挡/丢弃某个分组（类似防火墙）
	- 为进一步处理和动作而向某个特定的服务器发送一个分组（类似 DPI，Deep Packet Inspection）

![[40-Network-layer-Data-plane-generalized-forwarding.png]]
- 通用转发中，匹配+动作表将基于目的地转发的转发表泛化。
- 更广泛地，基于网络层或链路层的源和目的地址做出转发决定，因此通用转发中的转发设备不是简单的 router 或 switcher，而是称作 packet switches —— 分组交换机。
- 图 4-28 中每台分组交换机都有一章匹配+动作表，该表由远程控制器计算、安装和更新。注意与 4-2、4-3 两张图对比。

### Openflow

> OpenFlow — a highly visible standard that has pioneered the notion of the match-plus-action forwarding abstraction and controllers, as well as the SDN revolution more generally.

匹配+动作的转发表在 Openflow 中称为流表 flow table，其中表项包含有：
- ***A set of header field values to which an incoming packet will be matched***. As in the case of destination-based forwarding, hardware-based matching is most rapidly performed in TCAM memory, with more than a million destination address entries being possible [Bosshart 2013]. A packet that matches no flow table entry can be dropped or sent to the remote controller for more processing. In practice, a flow table may be implemented by multiple flow tables for performance or cost reasons [Bosshart 2013], but we’ll focus here on the abstraction of a single flow table.
- ***A set of counters that are updated as packets are matched to flow table entries***. These counters might include the number of packets that have been matched by that table entry, and the time since the table entry was last updated.
- ***A set of actions to be taken when a packet matches a flow table entry***. These actions might be to forward the packet to a given output port, to drop the packet, makes copies of the packet and sent them to multiple output ports, and/or to rewrite selected header fields.

### Openflow 的匹配
![[40-Network-layer-Data-plane-flow-table.png]]
- 上图是流表的分组匹配字段，显示了 11 个分组首部字段和入端口 ID，这个 ID 用来匹配规则、行以动作
- 众所周知，链路层中的帧、网络层中的数据报、传输层中的分组是逐层嵌套的，因此 Openflow 允许对来自三个层次的协议首部所选择的字段进行匹配：
	- 图 4.29 所示的源 MAC 地址和目的 MAC 地址是与帧的发送接口和接收接口相关的链路层地址；
	- 通过基于以太网地址而非 IP 地址进行转发，我们可以看到，启用 OpenFlow 的设备既可以作为路由器（第 3 层设备）转发数据报，也可以作为交换机（第 2 层设备）转发帧。
	- 以太网类型字段与上层协议（如 IP）相对应，帧的有效载荷将根据该协议进行解复用，而 VLAN 字段则与我们将在第 6 章中学习的所谓虚拟局域网相关。
- The ingress port refers to the input port at the packet switch on which a packet is received. The packet’s IP source address, IP destination address, IP protocol field, and IP type of service fields were discussed earlier in Section 4.3.1. The transport-layer source and destination port number fields can also be matched. 
- Flow table entries may also have wildcards. For example, an IP address of 128.119.*.* in a flow table will match the corresponding address field of any datagram that has 128.119 as the first 16 bits of its address. Each flow table entry also has an associated priority. If a packet matches multiple flow table entries, the selected match and corresponding action will be that of the highest priority entry with which the packet matches.

### Openflow 的动作

Each flow table entry has a list of zero or more actions that ==determine the processing that is to be applied to a packet that matches a flow table entry==. If there are multiple actions, they are performed in the order specified in the list.

Among the most important possible actions are:
- ***Forwarding***. An incoming packet may be forwarded to a particular physical output port, broadcast over all ports (except the port on which it arrived) or multicast over a selected set of ports. The packet may be encapsulated and sent to the remote controller for this device. That controller then may (or may not) take some action on that packet, including installing new flow table entries, and may return the packet to the device for forwarding under the updated set of flow table rules.
- ***Dropping***. A flow table entry with no action indicates that a matched packet should be dropped.
- ***Modify-field***. The values in 10 packet-header fields (all layer 2, 3, and 4 fields shown in Figure 4.29 except the IP Protocol field) may be re-written before the packet is forwarded to the chosen output port.

### Openflow 的实例

![[40-Network-layer-Data-plane-openflow-instance.png]]

#### A First Example: Simple Forwarding

As a very simple example, suppose that the desired forwarding behavior is that packets from h5 or h6 destined to h3 or h4 are to be forwarded from s3 to s1, and then from s1 to s2 (thus completely avoiding the use of the link between s3 and s2). The flow table entry in s1 would be: 
```
# s1 Flow Table (Example 1) 
Match                     Action
Ingress Port = 1 ; IP Src = 10.3.*.* ; IP Dst = 10.2.*.* Forward(4) … … 
```

Of course, we’ll also need a flow table entry in s3 so that datagrams sent from h5 or h6 are forwarded to s1 over outgoing interface 3: 
```
# s3 Flow Table (Example 1)
Match Action
IP Src = 10.3.*.* ; IP Dst = 10.2.*.* Forward(3)
… … 
```

Lastly, we’ll also need a flow table entry in s2 to complete this first example, so that datagrams arriving from s1 are forwarded to their destination, either host h3 or h4: 
```
# s2 Flow Table (Example 1) 
Match Action
Ingress port = 2 ; IP Dst = 10.2.0.3 Forward(3)
Ingress port = 2 ; IP Dst = 10.2.0.4 Forward(4)
… … 
```

#### A Second Example: Load Balancing

As a second example, let’s consider a load-balancing scenario, where datagrams from h3 destined to 10.1.*.* are to be forwarded over the direct link between s2 and s1, while datagrams from h4 destined to 10.1.*.* are to be forwarded over the link between s2 and s3 (and then from s3 to s1). Note that this behavior couldn’t be achieved with IP’s destination-based forwarding. In this case, the flow table in s2 would be:
```
# s2 Flow Table (Example 2)
Match Action
Ingress port = 3; IP Dst = 10.1.*.* Forward(2)
Ingress port = 4; IP Dst = 10.1.*.* Forward(1)
… …
```

Flow table entries are also needed at s1 to forward the datagrams received from s2 to either h1 or h2; and flow table entries are needed at s3 to forward datagrams received on interface 4 from s2 over interface 3 toward s1. See if you can figure out these flow table entries at s1 and s3.

#### A Third Example: Firewalling
As a third example, let’s consider a firewall scenario in which s2 wants only to receive (on any of its interfaces) traffic sent from hosts attached to s3.
```
# s2 Flow Table (Example 3)
Match Action
IP Src = 10.3.*.* IP Dst = 10.2.0.3 Forward(3)
IP Src = 10.3.*.* IP Dst = 10.2.0.4 Forward(4)
… …
```

If there were no other entries in s2’s flow table, then only traffic from 10.3.*.* would be forwarded to the hosts attached to s2. 

Although we’ve only considered a few basic scenarios here, the versatility and advantages of generalized forwarding are hopefully apparent. In homework problems, we’ll explore how flow tables can be used to create many different logical behaviors, including virtual networks—two or more logically separate networks (each with their own independent and distinct forwarding behavior)—that use the same physical set of packet switches and links. In Section 5.5, we’ll return to flow tables when we study the SDN controllers that compute and distribute the flow tables, and the protocol used for communicating between a packet switch and its controller.

The match-plus-action flow tables that we’ve seen in this section are actually a limited form of programmability, specifying how a router should forward and manipulate (e.g., change a header field) a datagram, based on the match between the datagram’s header values and the matching conditions. One could imagine an even richer form of programmability—a programming language with higher-level constructs such as variables, general purpose arithmetic and Boolean operations, variables, functions, and conditional statements, as well as constructs specifically designed for datagram processing at line rate. P4 (Programming Protocol-independent Packet Processors) [P4 2020] is such a language, and has gained considerable interest and traction since its introduction five years ago [Bosshart 2014]

### SDN 总结
SDN的主要思路
- 网络设备数据平面和控制平面分离
- 数据平面-分组交换机
    - 将路由器、交换机和目前大多数网络设备的功能进一步抽象成：按照流表（由控制平面设置的控制逻辑）进行PDU（帧、分组）的动作（包括转发、丢弃、拷贝、泛洪、阻塞）
    - 统一化设备功能：SDN交换机（分组交换机），执行控制逻辑
- 控制平面-控制器+网络应用
    - 分离、集中
    - 计算和下发控制逻辑：流表

SDN控制平面和数据平面分离的优势
- **水平集成**控制平面的开放实现（而非私有实现），创造出好的产业生态，促进发展
    - 分组交换机、控制器和各种控制逻辑网络应用app可由不同厂商生产，专业化，引入竞争形成良好生态
- **集中式**实现控制逻辑，网络管理容易：
    - 集中式控制器了解网络状况，编程简单，传统方式困难
    - 避免路由器的误配置
- 基于流表的匹配+行动的工作方式允许“**可编程的**”分组交换机
    - 实现流量工程等高级特性
    - 在此框架下实现各种新型（未来）的网络设备

<img src="http://knight777.oss-cn-beijing.aliyuncs.com/img/image-20211001155703049.png" style="zoom:60%"/>

流量工程：传统路由比较困难

<img src="http://knight777.oss-cn-beijing.aliyuncs.com/img/image-20211001160320932.png" style="zoom:64%"/>

Q1：网管如果需要u到z的流量走uvwz，x到z的流量走xwyz，怎么办？       
A1：需要定义链路的代价，流量路由算法以此运算（IP路由面向目标，无法操作）（或者需要新的路由算法）       
*链路权重只是控制旋钮，错！*

<img src="http://knight777.oss-cn-beijing.aliyuncs.com/img/image-20211001160555463.png" style="zoom:60%"/>

Q2：如果网管需要将u到z的流量分成2路：uvwz和uxyz（负载均衡），怎么办？（IP路由面向目标）      
A2：无法完成（在原有体系下只有使用新的路由选择算法，而在全网部署新的路由算法是个大的事情）      

<img src="http://knight777.oss-cn-beijing.aliyuncs.com/img/image-20211001160622576.png" style="zoom:56%"/>

Q3：如果需要w对蓝色的和红色的流量采用不同的路由，怎么办？       
A3：无法操作（基于目标的转发，采用LS，DV路由）

SDN特点

<img src="http://knight777.oss-cn-beijing.aliyuncs.com/img/image-20211001160657605.png" />

SDN架构
- 数据平面交换机
    - 快速，简单，商业化交换设备采用硬件实现通用转发功能
    - 流表被控制器计算和安装
    - 基于南向API（例如OpenFlow），SDN控制器访问基于流的交换机
        - 定义了哪些可以被控制哪些不能
    - 也定义了和控制器的协议（e.g., OpenFlow）
- SDN控制器（网络OS）： 
    - 维护网络状态信息
    - 通过上面的北向API和网络控制应用交互
    - 通过下面的南向API和网络交换机交互
    - 逻辑上集中，但是在实现上通常由于性能、可扩展性、容错性以及鲁棒性在物理上采用分布式方法
- 网络控制应用：
    - 控制的大脑：采用下层提供的服务（SDN控制器提供的API），实现网络功能
        - 路由器 交换机
        - 接入控制 防火墙
        - 负载均衡
        - 其他功能
    - 非绑定：可以被第三方提供，与控制器厂商以通常上不同，与分组交换机厂商也可以不同

通用转发和SDN：每个路由器包含一个流表（被逻辑上集中的控制器计算和分发）

<img src="http://knight777.oss-cn-beijing.aliyuncs.com/img/image-20211001161538845.png" style="zoom:80%"/>

OpenFlow数据平面抽象
- 流：由分组（帧）头部字段所定义
- 通用转发：简单的分组处理规则
    - 模式(pattern)：将分组头部字段和流表进行匹配（路由器中的流表定义了路由器的匹配+行动规则（流表由控制器计算并下发））
    - 行动(action)：对于匹配上的分组，可以是丢弃、转发、修改、将匹配的分组发送给控制器
    - 优先权Priority：几个模式匹配了，优先采用哪个，消除歧义
    - 计数器Counters：#bytes 以及 #packets

OpenFlow: 流表的表项结构

<img src="http://knight777.oss-cn-beijing.aliyuncs.com/img/image-20211001161724509.png" />

OpenFlow抽象
- match + action：统一化各种网络设备提供的功能 *目前几乎所有的网络设备都可以在这个 匹配+行动 模式框架进行描述，具体化为各种网络设备包括未来的网络设备*
- 路由器
    - match：最长前缀匹配
    - action：通过一条链路转发
- 交换机
    - match：目标MAC地址
    - action：转发或者泛洪
- 防火墙
    - match：IP地址和TCP/UDP端口号
    - action：允许或者禁止
- NAT
    - match：IP地址和端口号
    - action：重写地址和端口号
