---
publish: ""
tags:
  - Network
  - 计算机网络原理
---
网络层解决了一个网络如何到达另外一个网络的路由问题。在**一个网络内部**如何**由一个节点（主机或者路由器）到达另外一个相邻节点**则需要用到链路层的**点到点**传输功能

## 内容简介

目标：
- 理解数据链路层服务的原理：
    - 检错和纠错
    - 共享广播信道：多点接入（多路访问）
    - 链路层寻址
    - LAN:以太网、WLAN、VLANs
    - 可靠数据传输，流控制：解决！
- 实例和各种链路层技术的实现

网络节点的连接方式：一个子网中的若干节点是如何连接到一起的？
- 点到点连接（广域），只有封装/解封装的功能
- 多点连接：（局域），“一发全收”，有寻址和MAC问题，链路层的功能更加复杂，有两种方式实现：
    - 通过共享型介质，如同轴电缆
    - 通过网络交换机

数据链路层和局域网
- WAN：网络形式采用点到点链路
    - 带宽大、距离远（延迟大）
    - 带宽延迟积大
    - 如果采用多点连接方式
        - 竞争方式：一旦冲突代价大
        - 令牌等协调方式：在其中协调节点的发送代价大
- 点到点链路的链路层服务实现非常简单，封装和解封装
- LAN一般采用多点连接方式
    - 连接节点非常方便
    - 接到共享型介质上（或网络交换机），就可以连接所有其他节点
- 多点连接方式网络的链路层功能实现相当复杂
    - 多点接入：协调各节点对共享性介质的访问和使用
    - 竞争方式：冲突之后的协调；
    - 令牌方式：令牌产生，占有和释放等

## 6.1 链路层概述

- 一些术语：
    - 运行链路层协议的所有设备都称作：**节点 nodes**（包括主机、路由器、交换机、WiFi接入点、网桥等）
    - 沿着通信路径，连接相邻节点的通信信道：**链路 links**
        - 有线链路
        - 无线链路
        - 局域网，共享性链路
        - 为了将一个数据报从源主机传输到目的主机，数据报必须通过沿端到端路径上的各段链路传输；
    - 链路层协议的数据单元称为：**帧 frame**，封装链路层数据报；

数据链路层负责从一个节点通过链路将（**帧**中的）数据报发送到相邻的物理节点。     

>[! note] 链路层传递数据的步骤
> Consider sending a datagram from one of the wireless hosts to one of the servers.
> ![[60-Link-layer-and-LAN-wireless-link-layer.png]]
> This datagram will actually pass through ***six links***:
> - a WiFi link between sending host and WiFi access point,
> - an Ethernet link between the access point and a link-layer switch;
> - a link between the link-layer switch and the router,
> - a link between the two routers;
> - an Ethernet link between the router and a link-layer switch; 
> - and finally an Ethernet link between the switch and the server.
> 
> Over a given link, a transmitting node encapsulates the datagram in a link-layer frame and transmits the frame into the link.

链路层：上下文
- 数据报（分组）在不同的链路上以不同的链路协议传送：
    - 第一跳链路：以太网
    - 中间链路：帧中继链路
    - 最后一跳802.11：无线网络协议
- 不同的链路协议提供不同的服务

> 传输类比
> - 从Princeton到Lausanne
>     - 轿车：Princeton to JFK
>     - 飞机：JFK to Geneva
>     - 火车：Geneva to Lausanne
> - 旅行者 = 数据报 datagram
> - 交通段 = 通信链路 communication link
> - 交通工具 = 链路层协议：数据链路层和局域网protocol
> - 票务代理 = 路由算法 routing algorithm

### 链路层提供的服务
> 一般化的链路层服务，不是所有的链路层都提供这些服务，一个特定的链路层只是提供其中一部分的服务（子集）

- 成帧：
    - 将网络数据报封装在帧中的数据字段，再加上帧头、帧尾部（具体的结构由链路层协议规定）
    - 在帧头部使用“MAC”（物理）地址来标示源和目的
        - 不同于IP地址

- 链路接入：
    - MAC (Medium Access Control, 媒体访问控制)协议规定了帧在链路上传输的规则；
    - 在链路两端各自仅有一个发送方或接收方的点对点链路，MAC 允许只要链路空闲，就可以发送帧；
    - 若是多个节点共享单个广播链路，MAC 则会协调多个节点的帧传输。

- 可靠交付：
    - 在（一个网络内）相邻两个节点完成可靠数据传递——保证无差错地经链路层移动每个网络层数据报；
    - 与传输层的可靠交付类似，链路层可靠交付的实现也是通过确认+重传
    - 在低出错率的链路上（光纤和双绞线电缆）很少使用——有线链路层协议通常不提供可靠交付服务
		- 出错率低，没有必要在每一个帧中做差错控制的工作，协议复杂
			- 发送端对每一帧进行差错控制编码，根据反馈做相应的动作
			- 接收端进行差错控制解码，反馈给发送端（ACK，NAK）
			- 在本层放弃可靠控制的工作，在网络层或者是传输层做可靠控制的工作，或者根本就不做可靠控制的工作
    - 在高差错链路上需要进行可靠的数据传送
        - 高差错链路：无线链路
        - Q：为什么要在采用无线链路的网络上，链路层做可靠数据传输工作，还要在传输层做端到端的可靠性工作？
        - 原因：出错率高，如果在链路层不做差错控制工作，漏出去的错误比较高；到了上层如果需要可靠控制的数据传输代价会很大

- 流量控制：
    - 使得相邻的发送和接收方节点的速度匹配

- 差错检测与纠正
    - 差错由信号衰减和噪声引起，导致接收方的硬件在判断电平时认错
    - 传输层和网络层也有有限的差错检测功能，但链路层的差错检测通常由硬件实现，也更复杂；
    - 接收方检测出的错误： 
        - 通知发送端进行重传或丢
    - 错误不太严重时，接收端检查和根据网络编码纠正bit错误，不通过重传来纠正错误

- 半双工和全双工：
    - 半双工：链路可以双向传输，但一次只有一个方向

### 链路层在何处实现？

![[60-Link-layer-and-LAN-Network-Adapter.png]]

- 链路层主体功能在“网络适配器”上实现
    - 也叫 network interface card, NIC
    - 或者在一个芯片组上，核心部分是链路层控制器
    - 以太网卡，802.11网卡；以太网芯片组
    - 实现链路层和相应的物理层功能
    - 在每一个主机上，网卡实现链路层和物理层的功能
        - 也在每个路由器上，插多个网卡，实现链路层和相应物理层的功能
        - 交换机的每个端口上
        - 接到主机的系统总线上
    - 硬件、软件和固件的综合体
        - 其中软件组件实现了高层链路层的功能，如组装链路层寻址信息、激活控制器硬件等；接收端链路层软件响应控制器中断、处理差错和传递数据给网络层；
        - 链路层是硬件和软件分界的位置。

适配器（网卡）通信
- 发送方：
    - 在帧中封装数据报
    - 加上差错控制编码，实现RDT和流量控制功能等
    - 交给物理层打出
- 接收方
    - 从物理层接收bit
    - 检查有无出错，执行rdt和流量控制功能等
    - 解封装数据报，将帧交给上层

## 6.2 差错检测和纠正

![[60-Link-layer-and-LAN-EDC.png]]
- EDC = 差错检测和纠正位（冗余位）
- D = 数据（由差错检测保护，可以包含头部字段）
- 错误检测不是100%可靠的！
    - 协议会漏检一些错误，但是很少
    - 更长的EDC字段可以得到更好的检测和纠正效果

### 奇偶校验

- 加一个校验位，使得整个出现的1的个数是奇数还是偶数，是奇数->奇校验，是偶数->偶校验
- Receiver operation is also simple with a single parity bit. The receiver need only count the number of 1s in the received d+1 bits. *If an odd number of 1-valued bits are found with an even parity scheme, the receiver knows that at least one bit error has occurred*. More precisely, it knows that some odd number of bit errors have occurred.
> 以奇校验为例，接收方只需检查收到的数据是否仍是奇数个 1。
> 这种校验方式能够检验奇数个位的错误。
> 然而，如果是偶数个位的错误，则 1的数量仍是奇数个，因此无法检验。

- If the probability of bit errors is small and errors can be assumed to occur independently from one bit to the next, the probability of multiple bit errors in a packet would be extremely small. In this case, a single parity bit might suffice. However, measurements have shown that, rather than occurring independently, errors are often clustered together in “bursts.” Under burst error conditions, the probability of undetected errors in a frame protected by single-bit parity can approach 50 percent `[Spragins 1991]`.
> 如果位发生错误之间是独立的且概率较低，那么单个位奇偶校验足够有用。
> 但是研究显示，错误会突发地集中在某个区域内出错，这种情况下单个位奇偶校验的成功率仅仅是 50%。

- 二维奇偶校验：
	- ![[60-Link-layer-and-LAN-two-dimensional-even-parity.png]]
	- Figure 6.5 shows a two-dimensional generalization of the single-bit parity scheme. Here, the *d* bits in *D* are divided into *i* rows and *j* columns. A parity value is computed for each row and for each column. The resulting *i + j + 1* parity bits comprise the link-layer frame’s error-detection bits.
	- Suppose now that a single bit error occurs in the original d bits of information. With this ***two-dimensional parity*** scheme, the parity of both the column and the row containing the flipped bit will be in error. The receiver can thus not only detect the fact that a single bit error has occurred, but can use the column and row indices of the column and row with parity errors to actually identify the bit that was corrupted and correct that error!（二维奇偶校验策略位每一行和每一列都设置奇偶校验位，因此一个位出错会在行和列同时显示，也因此具有定位出错位的位置的能力）
	- Figure 6.5 shows an example in which the 1-valued bit in position (2,2) is corrupted and switched to a 0—an error that is both detectable and correctable at the receiver. Although our discussion has focused on the original d bits of information, a single error in the parity bits themselves is also detectable and correctable.
	- Two-dimensional parity can also detect (but not correct!) any combination of two errors in a packet.（二维奇偶校验也能够检测到一个分组中任意两位的错误，但是无法纠正）

### 校验和

- The ***Internet checksum*** is that bytes of data are treated as 16-bit integers and summed. The 1s complement of this sum then forms the Internet checksum that is carried in the segment header.
- As discussed in Section 3.3, the receiver checks the checksum by taking the 1s complement of the sum of the received data (including the checksum) and checking whether the result is all 0 bits. If any of the bits are 1, an error is indicated. `RFC 1071` discusses the Internet checksum algorithm and its implementation in detail.
> 校验和是将字节流数据视作 16bit 整数相加，然后取对 1 的补码填入分组首部的校验和字段。
> 当接收方收到分组时，通过校验和字段与数据字段相加进行检验——结果是 0 表明没有错，否则意味着差错发生。

- In the TCP and UDP protocols, the Internet checksum is computed over all fields (header and data fields included).
- In IP, the checksum is computed over the IP header (since the UDP or TCP segment has its own checksum).
- In other protocols, for example, XTP `[Strayer 1992]`, one checksum is computed over the header and another checksum is computed over the entire packet.
> TCP 和 UDP 的校验和会计算所有域（即包括首部和数据域），IP 只计算首部。

- 通常校验和用于 TCP 和 UDP 这样的传输层协议中，CRC 用于链路层协议中。这是因为传输层通过软件实现，因此简单的差错检测可以在不过分影响性能的情况下使用；而链路层的差错检测通过硬件实现，速度更快，因此可以使用 CRC 这样繁琐的差错检测方案。

### 循环冗余校验

1. 模2运算（加法不进位，减法不借位，位和位之间没有关系）：同0异1，异或运算
	- 模 2 加减都相当于 XOR 异或运算
	- 模 2 除与算术除类似，从被除数最高位开始，如果最高位（包括运算过程中的情况）是 1，则商的该位是 1，其余位做模 2 减；如果最高位是 0，则商的该位是 0.
	- 模 2 乘，乘 $2^r$ 相当于左移 r 位。

2. 位串的两种表示：位串 or 多项式 的表示方式
$$1011 \iff 1 * x^3 + 0 * x^2 + 1 * x^1 + 1 * x^0 = x^3 + x + 1$$

3. 生成多项式：r+1 位（发送方和接收方要预先约定好），记为 *G*

4. 在发送方发送的 D 位数据比特后附上 r 位的冗余位 R，使得序列正好被生成多项式整除，则没有出错
	- ![[60-Link-layer-and-LAN-CRC.png]]
	- 将数据比特 D，看成是 d 位二进制的数据
	- 生成多项式 G：双方协商 r+1 位模式串
	- 选择 r 位 CRC 附加位 R，得到 D+R 模式串（d+r 位模式串），其正好能被 G 整除 (mod 2) 
	- 接收方知道 G，将收到的 d+r 位模式串除以 G，如果非 0 余数：检查出错误！
	- 用公式表示上述过程，即
		- 发送方进行的操作是 $\text{D} \times 2^{r} \text{ XOR R} = n\text{G}$，
		- 接收方则对该式两边都用 R 异或—— $\text{D}\times 2^{r}=n\text{G XOR R}$，其含义就是如果用 G 来除 $\text{D}\times 2^r$，余数值刚好是 R
		- 即 $\text{R}=\text{remainder} \frac{D\times 2^{r}}{G}$ 

>[! example] CRC 实例
> ![[60-Link-layer-and-LAN-CRC-calc.png]]
> Figure 6.7 illustrates this calculation for the case of D = 101110, d = 6, G = 1001, and r = 3. The 9 bits transmitted in this case are 101110 011.
> 
> You should check these calculations for yourself and also check that indeed $D \times 2^{r} = 101011 \times G \text{ XOR } R$.

5. International standards have been defined for 8-, 12-, 16-, and 32-bit generators, G. The CRC-32 32-bit standard, which has been adopted in a number of link-level IEEE protocols, uses a generator of $$G_{CRC-32} = 100000100110000010001110110110111$$

7. CRC 性能分析
	- 突发错误和突发长度
	- CRC检错性能描述
	- 能够检查出所有的1bit 错误
	- 能够检查出所有的双 bits 的错误
	- 能够检查出所有长度小于等于 r 位的错误
	- 出现长度为 r+1 的突发错误，检查不出的概率是 $\frac{1}{2^{r-1}}$
	- 出现长度大于 r+1 的突发错误，检查不出的概率是 $\frac{1}{2^r}$

## 6.3 多点访问链路和协议

> 点到点不存在多点访问的问题（两个访问端都确定好了），多点连接则需要考虑。

### 链路类型

- 两种类型的链路（一个子网内部链路连接形式）：
    - 点对点链路
        - 链路两端的接收方和发送方各自只有 1 个设备
        - 拨号访问的 PPP（Point-to-Point Protocol，点对点协议）
        - 高级数据链路控制 HDLC（High-level Data Link Control）
    - 广播，也称为多点连接的网络
        - 多个发送或接收节点连接到相同、单一、共享的信道
        - 广播的含义，即任何一个节点传输一个帧时，信道广播该帧，链路上其他节点都收到一个副本
        - 举例：
            - 传统以太网（同轴电缆连接所有节点，所有节点通过比较 MAC 地址确认帧发送的目标地址）
            - HFC 上行链路
            - 802.11无线局域网

### 多路访问控制协议

多路访问协议
- 作用：规范、处理共享的广播信道上的传输行为
- 冲突/碰撞 (collision)：
	- 多个节点在同一个时刻发送，则接收方会收到 2 个或多个信号叠加
	- 碰撞发生时，所有接收节点都无法获得有效的传输帧，即碰撞帧将所有帧都污染了；
	- 碰撞发生就会极大地浪费广播信道的带宽，因此决定节点如何使用共享信道以致碰撞频率降低，可以很大程度地提高广播信道的带宽利用率。
- **共享控制**：传输控制信息，使得各节点协调信道的使用
    - 没有带外的信道，只有这一个信道，即要占用信道本身的带宽；

理想的多路访问协议
- 给定：*R* bps 的广播信道
- 必要条件：
    1. 当一个节点要发送时，可以 R 速率发送 —— 满速
    2. 当 M 个节点要发送，每个可以以 R/M 的平均速率发送 —— 公平、均分
    3. 分布式的：不会因为中心节点的崩溃而导致系统崩溃
    4. 简单又好用

MAC（多路访问控制，既可以是 multiple，也可以是 medium）协议：
- 分类：3大类：
	- 信道划分 (partition)
		- 把信道划分成小片（时间、频率、编码）
		- 分配片给每个节点专用
	- 随机访问 (random)：“想用就用”
		- 信道不划分，允许冲突/碰撞
		- 检测冲突，冲突后恢复
	- 依次轮流：分为 完全分布式的（令牌方式） 和 主节点协调式的（主节点轮流询问）
		- 节点依次轮流
		- 但是有很多数据传输的节点可以获得较长的信道使用权（发送数据较长者）

#### 信道划分协议

- TDMA(time division multiple access)分时复用
    - 将时间划分为时间帧 time frame，进一步划分每个时间帧为 N 个时隙 slot；
    - 每个站点使用每周期中固定的时隙（长度=帧传输时间）传输数据帧
    - 通常时隙的长度要能容纳一个数据帧；
    - 优势：每个节点得到的传输速率都是 R/N bps，绝对公平
    - 缺点：
        - 如果站点无帧传输，时隙空闲->浪费 （改进方法，统计时分复用）
        - 即使只有一个节点在使用信道，也只能获得 R/N bps 的传输速率
        - 节点必须等待传输序列中的轮次才能发送数据帧

![[60-Link-layer-and-LAN-TDM-FDM.png]]

- FDMA(frequency division multiple access)频分复用
    - 信道的有效频率范围被分成一个个小的频段
    - 每个站点被分配一个固定的频段
    - 分配给站点的频段如果没有被使用，则空闲
    - 优缺点与 TDMA 类似；

- CDMA(code division multiple access)码分复用
    - 有站点在整个频段上同时进行传输，采用编码原理加以区分
    - 如果编码理想地不会冲突，那么不同节点同时传输的问题即刻解决——各自相应的接收方仍能正确接收来自发送方的数据帧
    - 假定：信号同步很好，线性叠加
    - 应用：蜂窝电话（无线信道）

- 比方
    - TDM：不同的人在不同的时刻讲话
    - FDM：不同的组在不同的小房间里通信
    - CDMA：不同的人使用不同的语言讲话

#### 随机接入协议

- 当节点有帧要发送时
    - 总是以信道带宽的全部 R bps 发送，遇到碰撞就反复地重发帧，直到该帧不碰撞地正确传输（当然不是立即重发，而是涉及到碰撞的节点，就各自独立地随机等待一段时间后重发）
    - 没有节点间的预先协调
- 两个或更多节点同时传输，会发生->冲突“collision”
- 随机存取协议规定: 
    - 如何检测冲突
    - 如何从冲突中恢复（如：通过重传）
    - 随机MAC协议：
        - 时隙ALOHA
        - 纯ALOHA（非时隙）
        - CSMA, CSMA/CD, CSMA/CA

1. **时隙 ALOHA** 协议
	- 假设：
		- 所有帧是等长的 L 比特，能够持续一个时隙
		- 时间被划分成相等的时隙（长度为 L/R 秒），每个时隙可发送一帧
		- 节点只在时隙开始时发送帧
		- 共享信道的所有节点在时钟上是同步的——每个节点都知道时隙何时开始
		- 如果两个或多个节点在一个时隙中碰撞，所有的站点在该时隙结束之前都能检测到碰撞事件
	- 运行：
		- 当节点有一个新的帧要发送，在下一个时隙内传输整个帧
		- 传输时没有检测到冲突（可从信道内的信息能量的幅度判断），成功，并且节点能够在下一时隙发送新帧
		- 如果发生碰撞，则节点在时隙结束前检测到这次碰撞，只有在每一个随后的时隙以概率 p 重传帧直到成功（可能仍然发生冲突，但是时间越长，冲突概率越低）
			- 这里概率 p，指的是几何分布——每次判断，有 p 的概率重发后不冲突，直到n次后重发成功的总概率
	- 优点
		- 节点可以以信道带宽全速连续传输，尤其是在只有一个节点占用信道时
		- 高度分散：每个节点检测碰撞、独立地决定什么时候重传
		- 简单
	- 缺点
		- 时隙ALOHA需要在节点中对时隙同步
		- 存在冲突，浪费时隙
		- 即使有帧要发送，仍然有可能存在空闲的时隙
		- 节点检测冲突的时间 小于 帧传输的时间
		- 必须传完
	- 效率 (Efficiency)
		- 定义：当有大量活跃节点且每个节点都由大量的帧需要发送，长期运行中成功时隙所占的份额。（成功时隙指的是恰好只有一个节点传输的时隙）
		- ![[60-Link-layer-and-LAN-slot-ALOHA.png]]
		- 假设 N 个节点，每个节点都有很多帧要发送，在每个时隙中的传输概率是 $p$ 
		- 一个节点成功传输概率是 $p(1-p)^{N-1}$ （几何分布）
		- 任何一个节点的成功概率是 $N \times p(1-p)^{N-1}$
		- $N$ 个节点的最大效率：求出使 $f(p) = N \times p(1-p)^{N-1}$ 最大的 $p^{ ' }$
		- 代入 $p^{ ' }$ 得到最大 $f(p^{ ' }) = N \times p^{ ' }(1-p^{ ' })^{N-1}$ 
		- $N$ 为无穷大时的极限为 $1/e=0.37$ ，即最好情况：信道利用率 $37\%$ 


2. **纯 ALOHA**（非时隙）：数据帧一形成立即发送
	- 无时隙 ALOHA：简单、无须节点间在时间上同步（时隙 ALOHA 需要在时隙上同步，每个时隙开始时进行传输）
	- 当有帧从网络层传递下来，就立刻传输；如果一个传输的帧与其它帧的传输发生了碰撞，这个节点在完整传输完碰撞帧后，立即以概率 p 尝试重传，否则等待一个帧传输时间；
	- 冲突的概率增加:
		- 帧在 $t_0$ 发送，和其它在 $[t_0-1, t_0+1]$ 区间内开始发送的帧冲突
		- 和当前帧冲突的区间（其他帧在此区间开始传输）增大了一倍
		- ![[60-Link-layer-and-LAN-ALOHA.png]]
	- 纯ALOHA的效率$$
\begin{split}
P(\text{指定节点成功}) &= P(\text{节点传输}) \times \\ 
&\quad P(\text{其它节点在 $[t_0-1, t_0]$ 不传}) \times \\ 
&\quad P(\text{其它节点在 $[t_0, t_0+1\text{不传}]$ }) \\ 
&= p(1-p)^{N-1}(1-p)^{N-1} \\ 
&= p(1-p)^{2(N-1)}
\end{split}
$$
		- 选择最佳的 $p$ 且 $N$ 趋向无穷大时，效率为 $\frac{1}{2e} = 17.5\%$ ，效率比时隙 ALOHA 更差！

>[! note] ALOHA 的特点与改进目标——CSMA
> ALOHA 协议的特点就是：在发送自己的帧之前不查看链路的情况，这种情况既自私又无礼——把公共链路当成自己独占的；
> 
> 因此改进方向就是在发送前先查看链路的状态，如果空闲再发送，这就引出了两种思想：
> - ***Listen before speaking***. If someone else is speaking, wait until they are finished. In the networking world, this is called carrier sensing—a node listens to the channel before transmitting. If a frame from another node is currently being transmitted into the channel, a node then waits until it detects no transmissions for a short amount of time and then begins transmission. —— 载波侦听
> - ***If someone else begins talking at the same time, stop talking***. In the networking world, this is called ***collision detection***—a transmitting node listens to the channel while it is transmitting. If it detects that another node is transmitting an interfering frame, it stops transmitting and waits a random amount of time before repeating the sense-and-transmit-when-idle cycle. —— 碰撞检测

3. ***CSMA***(载波侦听多路访问) —— “说之前先听”
	- CSMA：在传输前先侦听信道：
		- 如果侦听到信道空闲，传送整个帧
		- 如果侦听到信道忙，推迟传送
	- CSMA 冲突
		- 冲突仍然可能发生——由传播延迟造成：两个节点可能侦听不到正在进行的传输
	- 冲突：
		- 整个冲突帧的传输时间都被浪费了，是无效的传输
	- 本质：
		- ==传播延迟（距离）决定了冲突的概率==
		- 本质上是节点依据本地的信道使用情况来判断全部信道的使用情况——距离越远，延时越大，发生冲突的可能性就越大
	- 举例：
		- ![[60-Link-layer-and-LAN-CSMA.png]]
		- Figure 6.12 shows a space-time diagram of four nodes (A, B, C, D) attached to a linear broadcast bus. The horizontal axis shows the position of each node in space; the vertical axis represents time.
		- At time t0, node B senses the channel is idle, as no other nodes are currently transmitting. Node B thus begins transmitting, with its bits propagating in both directions along the broadcast medium. The downward propagation of B’s bits in Figure 6.12 with increasing time indicates that a nonzero amount of time is needed for B’s bits actually to propagate (albeit at near the speed of light) along the broadcast medium.（t0时刻，B 意识到链路空闲，于是开始传输帧，上图中蓝色区域向下、向两边蔓延，即数据在链路上向两侧传播，并且传播需要时间）
		- At time t1 (t1 > t0), node D has a frame to send. Although node B is currently transmitting at time t1, the bits being transmitted by B have yet to reach D, and thus D senses the channel idle at t1. In accordance with the CSMA protocol, D thus begins transmitting its frame. A short time later, B’s transmission begins to interfere with D’s transmission at D.（t1时刻 D 也有帧需要发送，此时也检测到链路空闲——实际上 B 发送的帧还未传播到 D 处——但是根据 CSMA 协议，D 仍然头铁地传输帧，于是碰撞发生了）
		- From Figure 6.12, it is evident that the end-to-end ***channel propagation delay*** of a broadcast channel—the time it takes for a signal to propagate from one of the nodes to another—will play a crucial role in determining its performance. The longer this propagation delay, the larger the chance that a carrier-sensing node is not yet able to sense a transmission that has already begun at another node in the network.（信道传播时延在此处至关重要，传播时延越长，载波侦听策略所不能帧听到远处已发送帧的节点的可能就越大）
	- 改进：CSMA --> CSMA/CD（目前有线介质的局域网如“以太网”采用的方式）

4. ***CSMA/CD***(冲突检测) —— “边说边听” 
	- CSMA 的缺点：只是在发送前进行碰撞检测，不能确定远端是否有帧已经开始发送；并且即使发送过程中出现了碰撞，还是头铁地将帧传输完；
	- 改进：加入碰撞检测
		- 在传输帧的同时，检测是否发生了碰撞，一旦检测到碰撞，就各自停止传输——减少对信道的浪费
		- ![[60-Link-layer-and-LAN-CSMA-CD.png]]
		- ![[60-Link-layer-and-LAN-CSMA-CD-flow.png]]
	- 更具体的步骤：
		1. The adapter obtains a datagram from the network layer, prepares a link-layer frame, and puts the frame adapter buffer.（从网络层获取的数据报在链路层打包成帧，然后放入帧适配器的缓存区中）
		2. If the adapter senses that the channel is idle (that is, there is no signal energy entering the adapter from the channel), it starts to transmit the frame. If, on the other hand, the adapter senses that the channel is busy, it waits until it senses no signal energy and then starts to transmit the frame.（如果适配器检测到链路空闲，就开始传输帧；否则就等待，直到链路不忙）
		3. While transmitting, the adapter monitors for the presence of signal energy coming from other adapters using the broadcast channel.（传输帧过程中，适配器监视信道，查看是否有来自其它适配器的信号）
		4. If the adapter transmits the entire frame without detecting signal energy from other adapters, the adapter is finished with the frame. If, on the other hand, the adapter detects signal energy from other adapters while transmitting, it aborts the transmission, and transmits a ***jam*** signal.（如果适配器传输完整的帧且没有检测到其它适配器的能量，则该次传输帧完成；否则丢弃当前的传输，并传输一个 *jam* 信号，表示当前信道已被占用）
			- *jam* 信号的作用：强化冲突，确保整条链路上的所有接收者都能检测到碰撞的发生。
		5. After aborting, the adapter waits a random amount of time and then returns to step 2.（停止当前传输后，适配器各自等待随机时间后再回到步骤 2，开始尝试新的传输）
	- 碰撞后等待的时间间隔如何设置？**二进制指数后退**！
		- 在第 n 次失败（发送时有碰撞）后，适配器随机选择一个 $\{ 0，1，2，3，……，2^{n-1}\}$ 中的 *K* 值，等待 `K*512` 比特时间（即 发送 512 比特进入以太网所需时间量的 *K* 倍），n 能够取得的最大值不超过 10；
		- 可选 K 的集合随着碰撞次数指数增长，而选取 K 的方式是等概率地随机选取，因此随着 n 的增大，能够成功发送的概率越来越大；（不成功就是有两个适配器选取的 K 值相同）
		- 另外有新的帧刚刚到达，而其它适配器正在退避过程中——新的帧会抢先发送，因为它直接运行了 CSMA/CD 策略；
		- 指数退避的特点：
			- 高负载（重传节点多）：重传窗口时间大，减少冲突，但等待时间长
			- 低负载（重传节点少）：使得各站点等待时间少，但冲突概率大
		- CSMA/CD 的效率：
			- 有大量活跃节点和每个节点都由大量帧要发送的理想情况下，帧在信道中无碰撞地传输时间在长期运行时间中的所占份额：
			- 令 $t_{prop}$ 为 LAN 上 2 个节点的最大传播延迟，$t_{trans}$ 为传输最大以太网帧的时间，则效率计算式为：
			- $$ efficiency = \frac{1}{1 + 5 \times t_{prop} / t_{trans}} $$
			- 当 $t_{prop}$ 变成 $0$ ——信道传播速度足够快，或当 $t_{trans}$ 变成无穷大——帧无穷大，将使效率增加并接近 1；

5. 无线局域网 CSMA/CA
	- 无线信道倾向于选择 CSMA/CA：因为无线信道中传输者在传输数据报的过程中，会关闭接收器，因而不适用 CSMA/CD；
	- 由于[隐藏节点问题](https://en.wikipedia.org/wiki/Hidden_node_problem)，CSMA/CA 并不是绝对可靠的；
		- ![[60-Link-layer-and-LAN-hidden-node-problem.png]]
		- In one scenario, Station A can communicate with Station B. Station C can also communicate with Access Point Station B. However, Stations A and C cannot communicate with each other as they are out of range of each other, and thus start to transmit simultaneously preventing B from receiving messages intended for it.
	- 碰撞避免的策略：
		- ![[60-Link-layer-and-LAN-CSMA-CA-flow.png]]
		- 发送方：
			- 如果站点侦测到信道空闲持续 DIFS 长，则传输整个帧
			- 如果侦测到信道忙碌，那么选择一个随机回退值，并在信道空闲时递减该值；如果信道忙碌，回退值不会变化；回退值递减到 0 时（只发生在信道闲时）发送整个帧。
			- 如果没有收到 ACK，增加回退值，重复这段的整个过程；
			- ![[60-Link-layer-and-LAN-CSMACA-DIFS.png]]
		- 802.11 接收方
			- 如果帧正确，则在 SIFS 后发送 ACK
		- *无线链路特性，需要每帧都请求、确认（有线网由于边发边确认，不需要接收 ACK 信息就可以知道是否发送成功），这里的失败原因还是在担心隐藏节点问题。*
	- 细节问题：
		- 在随机回退值 count down 时，侦听到了信道空闲为什么不发送，而要等到0时在发送？以下例子可以很好地说明这一点
		- 考虑 2 个站点有数据帧需要发送，第 3 个节点正在发送的情况：
		- 有线局域网使用CD：让 2 者听完第三个节点发完，立即发送
			- 冲突：放弃当前的发送，避免了信道的浪费于无用冲突帧的发送
			- 代价不昂贵
		- 无线局域网使用 CA：
			- 无法 CD，一旦发送就必须发完，如冲突信道浪费严重，代价高昂
			- 思想：尽量事先避免冲突，而不是在发生冲突时放弃然后重发
			- 听到发送的站点，分别选择随机值，回退到 0 发送
				- 不同的随机值，一个站点会胜利
				- 失败站点会冻结计数器，当胜利节点发完再发
	- 进一步地冲突避免：RTS-CTS 交换 —— 对长帧的可选项
		- ![[60-Link-layer-and-LAN-CSMA-CA-DIFS.png]]
		- 思想：允许发送方“预约”信道，而不是随机访问该信道：避免长数据帧的冲突（可选项）
		- 发送方首先使用CSMA向接收方发送一个小的RTS分组
			- RTS 可能会冲突（但是由于比较短，浪费信道较少）
		- 接收方广播 CTS，作为 RTS 的响应
			- CTS 能够被所有涉及到的节点听到
			- 正确的发送方能够发送数据帧，而其它节点抑制发送
		- 采用小的预约分组，可以完全避免数据帧的冲突

#### 轮流协议

- 回顾：
	- MAC 协议的两个理想目标：(1) when only one node is active, the active node has a throughput of R bps, and (2) when M nodes are active, then each active node has a throughput of nearly R/M bps.
	- 信道划分协议：固定分配、平分
		- 共享信道在高负载时是有效和公平的
		- 在低负载时效率低下：只能等到自己的时隙开始发送或者利用1/N 的信道频率发送，当只有一个节点有帧传时，也只能够得到1/N 个带宽分配
	- 随机访问MAC协议
		- 在低负载时效率高：单个节点可以完全利用信道全部带宽
		- 高负载时：冲突开销较大，效率极低，时间很多浪费在冲突中

- 轮流协议：有两种策略：轮询 polling 和令牌 token-passing
	- 轮询：
		- 从链路上所有节点中选出一个主节点，主节点以循环的方式轮询每个节点——In particular, the master node first sends a message to node 1, saying that it (node 1) can transmit up to some maximum number of frames. After node 1 transmits some frames, the master node tells node 2 it (node 2) can transmit up to the maximum number of frames. (The master node can determine when a node has finished sending its frames by observing the lack of a signal on the channel.) The procedure continues in this manner, with the master node polling each of the nodes in a cyclic manner.
		- 从节点没有自己的链路支配权；
		- 缺点：
			- 轮询开销：轮询本身消耗信道带宽
			- 轮询时延：通知一个节点可以开始传输的时间
			- 等待时间：每个节点需等到主节点轮询后开始传输，即使只有一个节点，也需要等到轮询一周后才能够发送
			- 单点故障：主节点失效时造成整个系统无法工作
	- 令牌传递：（没有主节点）
		- 控制令牌 (token)循环从一个节点到下一个节点传递 （“击鼓传花”）。如果令牌到达的节点需要发送信息，就将令牌位置位为 0，将令牌帧变为数据帧，发送的数据绕行一周后再由自己收下
			- 为什么不是由目标节点收下？“一发多收”，目标节点有多个，如果由目标节点收下会导致后面的目标节点收不到，所以需要轮转一圈，确认经过了所有节点后由自己收下
		- 令牌报文：特殊的帧
		- 缺点：
			- 令牌开销：本身消耗带宽
			- 延迟：只有等到抓住令牌，才可传输
			- 单点故障 (token)：
				- 一个节点的故障会导致整个信道故障——单向传输的弊端；因此改进目标就是双向传输，可是又涉及复杂的令牌设计问题
				- 令牌丢失系统级故障，整个系统无法传输
				- 复杂机制重新生成令牌

### DOCSIS：综合三种访问方式的例子

Recall from Section 1.2.1 that a cable access network typically connects several thousand residential cable modems to a cable modem termination system (CMTS) at the cable network headend. The Data-Over-Cable Service Interface Specifications (DOCSIS) specifies the cable data network architecture and its protocols.
> 电缆接入网通常将几千个住宅电缆调制解调器与**电缆调制解调器端系统**在电缆网头端连接，而 DOCSIS 接口规定了电缆数据网络体系结构和相关协议。

DOCSIS uses FDM to divide the downstream (CMTS to modem) and upstream (modem to CMTS) network segments into multiple frequency channels. Each downstream channel is between 24 MHz and 192 MHz wide, with a maximum throughput of approximately 1.6 Gbps per channel; each upstream channel has channel widths ranging from 6.4 MHz to 96 MHz, with a maximum upstream throughput of approximately 1 Gbps. Each upstream and downstream channel is a broadcast channel. Frames transmitted on the downstream channel by the CMTS are received by all cable modems receiving that channel; since there is just a single CMTS transmitting into the downstream channel, however, there is no multiple access problem. The upstream direction, however, is more interesting and technically challenging, since multiple cable modems share the same upstream channel (frequency) to the CMTS, and thus collisions can potentially occur.
> - DOCSIS 采用 FDM 进行信道的划分——将上行、下行网络段划分为多个不同频率的信道
> - 下行信道：
> 	- 信道频率在 24MHz 到192MHz，最大吞吐量 1.6Gbps per channel
> 	- 下行、上行信道都是广播信道，CMTS 在下行信道中传输的帧被信道上所有接收方调制解调器都能收到；
> 	- 单一的 CMTS 在下行信道传输可以避免多路访问问题；但反之上行信道的碰撞问题比较复杂；
> 	- 在下行 MAP 帧中：CMTS 告诉各节点微时隙分配方案，分配给各站点的上行微时隙
> 	- 另外：头端传输下行数据（给各个用户）
> - 上行信道：
> 	- 信道频率在 6.4MHz 到 96MHz，最大吞吐量约 1Gbps.
> 	- 采用 TDM 的方式将上行信道分成若干微时隙：MAP 指定
> 	- 站点采用分配给它的微时隙上行数据传输：分配
> 	- 在特殊的上行微时隙中，各站点请求上行微时隙：竞争
> 	- 各站点对于该时隙的使用是随机访问的
> 	- 一旦碰撞（请求不成功，结果是：在下行的 MAP 中没有为它分配，则二进制退避）选择时隙上传输

![[60-Link-layer-and-LAN-DOCSIS.png]]
As illustrated in Figure 6.14, each upstream channel is divided into intervals of time (TDM-like), each containing a sequence of mini-slots during which cable modems can transmit to the CMTS. The CMTS explicitly grants permission to individual cable modems to transmit during specific mini-slots. The CMTS accomplishes this by sending a control message known as a MAP message on a downstream channel to specify which cable modem (with data to send) can transmit during which mini-slot for the interval of time specified in the control message. Since mini-slots are explicitly allocated to cable modems, the CMTS can ensure there are no colliding transmissions during a mini-slot.
> 上行线路划分为时间间隔，每个时间间隔有一个微时隙序列，电缆 modem 在微时隙中向 CMTS 传输帧。
> CMTS 显式地允许每个电缆 modem 在特定的微时隙中传输——在下行信道中通过发送 MAP 报文，控制指定的电缆 modem 获得微时隙中传输指定时间间隔的权限。——轮询的思路，从而确保没有碰撞。

如何确定电缆 modem 有数据要发送？This is accomplished by having cable modems send mini-slot-request frames to the CMTS during a special set of interval mini-slots that are dedicated for this purpose, as shown in Figure 6.14. These mini-slot-request frames are transmitted in a random access manner and so may collide with each other. A cable modem can neither sense whether the upstream channel is busy nor detect collisions. Instead, the cable modem infers that its mini-slot-request frame experienced a collision if it does not receive a response to the requested allocation in the next downstream control message. When a collision is inferred, a cable modem uses binary exponentia backoff to defer the retransmission of its mini-slot-request frame to a future time slot. When there is little traffic on the upstream channel, a cable modem may actually transmit data frames during slots nominally assigned for mini-slot-request frames (and thus avoid having to wait for a mini-slot assignment).
> - 电缆 modem 在特定的微时隙间隔内，发送微时隙间隔请求——获取发送权。
> - 这些微时隙请求帧是随机接入的方式进行传输，因此可能互相碰撞。然而电缆 modem 既不能侦听上行信道是否繁忙，也不能检测碰撞。
> - 事实上，电缆 modem 如果没有在下一个下行控制报文中收到请求分配的响应，于是推断出微时隙请求帧发生了碰撞——当检测到碰撞，就使用二进制指数回退将微时隙请求帧延后重新发送。

## 6.4 交换局域网

链路层设备如链路层交换机等，不识别网络层地址，因此 RIP、OSPF 这类路由选择算法无法用于确定链路层帧的路径——因而需要链路层地址转发链路层帧。

### 链路层寻址和ARP

- MAC地址
    - IP地址：
        - 网络层地址
        - 前 n-1 跳：用于使数据报到达目的 IP 子网
        - 最后一跳：到达子网中的目标节点 —— IP地址的主机号部分
    - LAN（MAC/物理/以太网）地址：
        - 用于使帧从一个网卡传递到与其物理连接的另一个网卡（在同一个物理网络中）（注意到，并不是每个主机或路由器都由链路层地址，实际上链路层地址与适配器一一对应）
        - 48bit MAC地址固化在适配器的ROM，有时也可以通过软件设定
            - 理论上全球任何2个网卡的 MAC 地址都不相同
        - 链路层交换机的任务是在主机与路由器之间承载链路层帧，即主机或路由器不必明确地将帧寻址到具体的交换机：
        - ![[60-Link-layer-and-LAN-switcher.png]]

>[! note] MAC 如何确保不冲突？
> The answer is that the ==IEEE manages the MAC address space==.
> In particular, when a company wants to manufacture adapters, it purchases a chunk of the address space consisting of $2^{24}$ addresses for a nominal fee. IEEE allocates the chunk of $2^{24}$ addresses by fixing the first 24 bits of a MAC address and letting the company create unique combinations of the last 24 bits for each adapter.

- 网络地址和 MAC 地址分离
	- IP 地址和 MAC 地址的结构不同
		1) IP 地址是分层的
			- 一个子网所有站点网络号一致，路由聚集，减少路由表的长度
			- 需要一个网络中的站点地址网络号一致，如果捆绑需要定制网卡非常麻烦
			- 希望网络层地址是动态配置的，随着主机移动（局域网的网段）变化而变化
			- IP 地址完成网络到网络的交付
		2) MAC 地址是扁平的的
			- 网卡在生产时不知道被用于哪个网络，因此给网卡一个唯一的标示，用于区分一个网络内部不同的网卡即可
			- 可以完成一个物理网络内部的节点到节点的数据交付网络地址和 MAC 地址分离
	- 分离好处：
		1) 网卡坏了，IP 不变，可以捆绑到另外一个 MAC 的网卡上
		2) 物理网络还可以除 IP 之外支持其他网络层协议，链路协议为任意上层网络协议，如 IPX 等
	- 捆绑的问题
		1) 如果仅仅使用 IP 地址，不用 MAC 地址，那么它仅支持 IP 协议
		2) 每次上电都要重新写入网卡 IP 地址；
		3) 另外一个选择就是不使用任何地址；不用MAC地址，则每到来一个帧都要上传到IP层次，由它判断是不是需要接受，干扰一次

- 帧的发送与广播
	- When an adapter wants to send a frame to some destination adapter, ==the sending adapter inserts the destination adapter’s MAC address into the frame and then sends the frame into the LAN==. Thus, when an adapter receives a frame, it will check to see whether the destination MAC address in the frame matches its own MAC address. If there is a match, the adapter extracts the enclosed datagram and passes the datagram up the protocol stack. If there isn’t a match, the adapter discards the frame, without passing the network-layer datagram up. Thus, the destination only will be interrupted when the frame is received.（要发送给指定目的适配器时，会将目的适配器的 MAC 地址插入帧中再发送到 LAN。当适配器收到帧时，检查其中的 MAC 地址是否匹配，若匹配则提取帧中的数据并上交给网络层；若不匹配，则丢弃。）
	- However, sometimes a sending adapter does want all the other adapters on the LAN to receive and process the frame it is about to send. In this case, the sending adapter inserts a special MAC broadcast address into the destination address field of the frame. For LANs that use 6-byte addresses (such as Ethernet and 802.11), the broadcast address is a string of 48 consecutive 1s (that is, FF-FF-FF-FF-FF-FF in hexadecimal notation).（若将插入的 MAC 地址设置为 FF-FF-FF-FF-FF-FF，则意味着在LAN中广播该帧）

- 地址解析协议：Address Resolution Protocol
	- 提供 IP 地址和 MAC 地址之间的转换：
	- 在同一个 LAN 的例子：
		- ![[60-Link-layer-and-LAN-ARP-instra-LAN.png]]
		- 假设 C 向 A 发送 IP 数据报，则源主机需要向其适配器提供 IP 数据报和目的主机的 MAC 地址，然后由适配器构造成一个包含目的地 MAC 地址的链路层帧，再发送到 LAN 中；
		- ARP 的作用是，将相同 LAN 上的任何 IP 作为输入，返回相应的 MAC 地址——类似 DNS，但 DNS 支持的主机范围在全世界，而 ARP 只支持当前 LAN；
		- ARP 解析地址的工具是 ARP 表，在 LAN 上的每个 IP 节点都有一个 ARP 表
			- ARP 表：包括一些 LAN 节点 IP/MAC 地址的映射 —— `<IP address; MAC address; TTL>`
			- ![[60-Link-layer-and-LAN-ARP-table.png]]
			- TTL 时间是指地址映射失效的时间，典型是 20min。20min 内直接使用缓存 —— 高效；20min 后删除 —— 保持最新；
		- 如果 ARP 表中没有目的主机的表项：
			- 发送方构造一个 ARP 分组，其中包括发送方和接收方的 IP 地址和 MAC 地址，广播之，从而每个适配器都受到该分组，并将其传递给 ARP 模块，
			- ARP 模块检查后确定是否与其中的目的地址匹配，若匹配则在自己的 ARP 表中添加源主机的映射，并且单播回送一个正确的 ARP 响应分组，从而让源主机也正确地建立 ARP 映射；
		- 节点在自己的 ARP 表中，缓存 IP-to-MAC 地址映射关系，直到信息超时
			- 软状态：靠定期刷新维持的系统状态
			- 定期刷新周期之间维护的状态信息可能和原有系统不一致
		- ARP 是即插即用的，即节点自己创建和管理 ARP 的表项，无需网络管理员的干预。
	- 跨越路由器发送到不同 LAN 的例子：
		- ![[60-Link-layer-and-LAN-ARP-inter-LAN.png]]
		- 每台主机仅有一个 IP 地址和一个适配器（即一个 MAC 地址），而路由器对每个端口都有一个 IP 地址，因此路由器中对每个端口都有一个 ARP 模块和适配器；
		- 子网由掩码划分，其中子网 1 的前缀是 111.111.111，而子网 2 的前缀是 222.222.222。因此考查从子网 1 的一台主机向子网 2 的一台主机发送数据报：
			- In order for a datagram to go from 111.111.111.111 to a host on Subnet 2, the datagram must first be sent to the router interface 111.111.111.110, which is the IP address of the first-hop router on the path to the final destination. Thus, the appropriate MAC address for the frame is the address of the adapter for router interface 111.111.111.110, namely, E6-E9-00-17- BB-4B. How does the sending host acquire the MAC address for 111.111.111.110? By using ARP, of course!（IP 为111.111.111.111的主机向子网 2发送一个数据报，则应当首先发送到 IP 为111.111.111.110的路由器端口，因此正确的 MAC 地址也应当是该路由器端口的适配器的 MAC 地址。而这个 MAC 地址属于子网 1内部，因此通过 ARP 协议获取）
			- Once the sending adapter has this MAC address, it creates a frame (containing the datagram addressed to 222.222.222.222) and sends the frame into Subnet 1. The router adapter on Subnet 1 sees that the link-layer frame is addressed to it, and therefore passes the frame to the network layer of the router. Hooray—the IP datagram has successfully been moved from source host to the router!
			- But we are not finished. We still have to move the datagram from the router to the destination. The router now has to determine the correct interface on which the datagram is to be forwarded. As discussed in Chapter 4, this is done by consulting a forwarding table in the router. The forwarding table tells the router that the datagram is to be forwarded via router interface 222.222.222.220. This interface then passes the datagram to its adapter, which encapsulates the datagram in a new frame and sends the frame into Subnet 2. （在路由器内部如何判断正确的转发接口？路由表！因此该数据报（此时已经被脱去链路层附加的头部）被查表后转发到 IP 为222.222.222.220的接口，然后重新封装新的链路层帧头（因为目的MAC地址发生了改变））
			- This time, the destination MAC address of the frame is indeed the MAC address of the ultimate destination. And how does the router obtain this destination MAC address? From ARP, of course!

### Ethernet

特点：
- 目前最主流的有线 LAN 技术：98%占有率
- 廉价：30元 RMB 100Mbps！——广泛应用、流行，均摊下来成本很低。
- 最早广泛应用的 LAN 技术——先发优势
- 比令牌环和 ATM 网络简单、廉价
- 带宽不断提升：10M, 100M, 1G, 10G——其它 LAN 技术的优势也可以被以太网吸收

#### 物理拓扑
- 总线：初代拓扑结构，在上个世纪 80-90年代很流行
    - 广播局域网
    - 所有节点在一个碰撞域内，一次只允许一个节点发送
    - 可靠性差，如果介质破损（总线长，破损概率大），截面形成信号的反射，发送节点误认为是冲突，总是冲突 —— 需要中继器吸收电磁能量
- 星型：90年代后期
    - 连接选择：集线器 Hub 
    - Hub 是物理层设备，作用于各比特而不是帧——简单地将比特流在输出端口重新生成，抵消传输过程中的能量损耗
    - 基于集线器的星型拓扑以太网也是广播局域网，一个接口收到比特流，其他接口都要输出；
    - 仍有碰撞：如果 Hub 从两个不同接口同时收到帧，则在内部出现碰撞，需要源发送方重新发送该帧；
- 星型：21 世纪
    - 中心部位：集线器 Hub 改进成交换机 Switch
    - 交换机每个节点以及相连的交换机端口使用（独立的）以太网协议（不会和其他节点的发送产生碰撞）
    - 交换机是接收-存储-转发，而不是直接转发；另外交换机只运行在第二层（链路层）
    - *（不要与网络层交换机混淆，那里只是一个概念，“用于转发分组的机器”，实际上对应的物理实体是路由器或三层交换机）*

![[60-Link-layer-and-LAN-Ethernet-Topo.png]]

![[60-Link-layer-and-LAN-Ethernet-logical-topo.png]]

#### 以太网帧

**发送方适配器在以太网帧中封装 IP 数据报，或其他网络层协议数据单元**

考虑以下情景：发送方适配器的 MAC 地址为 AA-AA-AA-AA-AA-AA，接收方适配器的 MAC 地址为 BB-BB-BB-BB-BB-BB；发送方适配器封装一个以太网帧，其负载是一个 IP 数据报，然后传递给物理层，接收方适配器从物理层中提取该以太网帧，解封后上传给网络层：
![[60-Link-layer-and-LAN-Ethernet-frame.png]]
- 前导码：
	- 8字节：前 7个均为 10101010，最后一个为 10101011。前 7字节用于“唤醒”接收适配器，并且将其时钟与发送方时钟进行同步
	- 漂移 drift 问题：发送方适配器传输速率的误差、链路传播和中继器的误差、噪声等等，都会造成接收与发送的不匹配，因此前 7 字节的比特能够锁定发送方适配器的时钟，使得接收方将自己的时钟调到发送端的时钟，从而可以按照发送端的时钟来接收所发送的帧
	- 第 8个字节的最后两位 `11`，用于题型适配器——大的要来了！
- 地址：源 MAC 地址、目标 MAC 地址均为 6字节
	- 接收方收到帧后，检查其中的目标地址，若与本站 MAC 地址相同，或是广播地址，则接收，递交帧中的数据到网络层
	- 否则，适配器丢弃该帧
	- 源 MAC 地址在建立 ARP 表时和接收回应帧时有用
- 类型：
	- 2字节
	- 指出高层协议（大多情况下是 IP，但也支持其它网络层协议 Novell IPX 和 AppleTalk，还有 ARP 这个处于链路层和网络层分界处的协议也有独特的类型编号），使得以太网帧中的负载能够正确提交；
- 数据：
	- 46~1500 字节
		- 46 字节要求 IP 数据报的最小长度不小于 46 字节，因此如果不足 46 字节，需要进行填充
		- 以太网的最大传输单元 MTU=1500 字节，如果 IP 数据报超过1500 字节，就要分片（IPv4允许分片，但 IPv6不允许）
	- 承载IP数据报或其它网络层协议的数据单元
- CRC：在接收方校验
	- 如果没有通过校验，丢弃错误帧
	- 4 字节—— $CRC_{32}$ 的生成多项式

>[! note] 以太网帧的长度
>- 头部长度：8+6+6+2=22 字节
>- 尾部长度：4 字节
>- 数据长度：46~1500 字节
>
>故最小长度 72字节，最大长度 1526字节

#### 以太网服务的特点：无连接、不可靠

- 无连接：帧传输前，发送方和接收方之间没有握手，直接将以太网帧发送到局域网中
	- 类似：This layer-2 connectionless service is analogous to IP’s layer-3 datagram service and UDP’s layer-4 connectionless service.
- 不可靠：接收方适配器不发送 ACK 或 NAK 给发送方
	- 递交给网络层的数据报流可能有 gap
	- 如上层使用像传输层 TCP 协议这样的 RDT 协议，gap 会被补上（重传，尽管以太网不知道这个数据是全新的还是重传的）
	- 否则，应用层就会看到 gap（UDP协议）


#### 以太网技术

>[! note] 眼花缭乱的以太网技术缩写词
> 10BASE-T, 10BASE-2, 100BASE-T, 1000BASE-LX, 10GBASE-T and 40GBASE-T.
> - The first part of the acronym refers to the speed of the standard: 10, 100, 1000, or 10G, for 10 Megabit (per second), 100 Megabit, Gigabit, 10 Gigabit and 40 Gigibit Ethernet, respectively.数字是指该标准提供的速率，单位是 Mbps，最后一个是 Gbps。 
> - “BASE” refers to baseband Ethernet, meaning that the physical media only carries Ethernet traffic; almost all of the 802.3 standards are for baseband Ethernet.BASE 指基带baseband 
> - The final part of the acronym refers to the physical media itself; Ethernet is both a link-layer and a physical-layer specification and is carried over a variety of physical media including coaxial cable, copper wire, and fiber. Generally, a “T” refers to twisted-pair copper wires. 以太网是链路层和物理层的规范，对物理介质也有规定——T 指双绞铜线
> 
> 上面提到了 802.3，它是 IEEE 的 CSMA/CD 以太网工作组，对这些杂乱的标准进行了统一化。
> - 以太网的 MAC 协议：采用二进制退避的 CSMA/CD 协议
> 	- 没有时隙
> 	- NIC 如果侦听到其它 NIC 在发送就不发送：载波侦听 (carrier sense)
> 	- 发送时，适配器当侦听到其它适配器在发送就放弃对当前帧的发送，冲突检测 (collision detection)
> 	- 冲突后尝试重传，重传前适配器等待一个随机时间，随机访问 (random access)
> - ==以太网在低负载和高负载的情况下都较好==。低负载时好是由于CDMA/CD，高负载时好是由于引入了交换机，端口可以并发

- 10BASE-2 和 10BASE-5
	- The early 10BASE-2 and 10BASE-5 standards specify 10 Mbps Ethernet over two types of coaxial cable（同轴电缆）, each limited in length to 500 meters.限制长度不超过 500 米——传输损耗与噪声
	- Longer runs could be obtained by using a ***repeater***(转发器)—a physical layer device that receives a signal on the input side, and regenerates the signal on the output side.（repeater 重新生成信号，保持能量强度）
	- A coaxial cable corresponds nicely to our view of Ethernet as a broadcast medium—all frames transmitted by one interface are received at other interfaces, and Ethernet’s CDMA/CD protocol nicely solves the multiple access problem. Nodes simply attach to the cable, and voila, we have a local area network!

- 10BaseT and 100BaseT
	- 100Mbps 速率也被称之为“fast ethernet”（只是当时算快的）
	- T 代表双绞铜线，后面改进到光纤——代号是 FX, SX, BX
	- 节点连接到 HUB 上：物理上星型，逻辑上总线型，Hub 内部是总线
	- 节点和 HUB 间的最大距离是100m（双绞铜线），光纤则是数千米

![[60-Link-layer-and-LAN-Ethernet-standards.png]]

- ***Gigabit Ethernet***: 
	- an extension to the 10 Mbps and 100 Mbps Ethernet standards.
	- Offering a raw data rate of 40,000 Mbps, 40 Gigabit Ethernet maintains full compatibility with the huge installed base of Ethernet equipment.最大速率 40Gbps，与所有以太网设备兼容
	- The standard for Gigabit Ethernet, referred to as IEEE 802.3z, does the following: 
		- Uses the standard Ethernet frame format and is backward compatible with 10BASE-T and 100BASE-T technologies. This allows for easy integration of Gigabit Ethernet with the existing installed base of Ethernet equipment. 使用标准以太网帧，与 10BAST-T 等保持后向兼容——便于集成和实现该标准
		- Allows for point-to-point links as well as shared broadcast channels. Point-to-point links use switches while broadcast channels use hubs, as described earlier. In Gigabit Ethernet jargon, hubs are called *buffered distributors*.允许点对点链路共享广播信道，并且集线器开始成为带缓存的分配器
		- Uses CSMA/CD for shared broadcast channels. In order to have acceptable efficiency, the maximum distance between nodes must be severely restricted. 使用 CSMA/CD 策略，因此严格限制节点之间的最大距离
		- Allows for full-duplex operation at 40 Gbps in both directions for point-to-point channels. 允许全双工
		- 编码策略：8B10B
	- Initially operating over optical fiber, Gigabit Ethernet is now able to run over category 5 UTP cabling (for 1000BASE-T and 10GBASE-T).支持光纤和双绞铜线

>[!note] 交换机的力量：是否需要新的以太网 MAC 协议？
> As we’ll see shortly, a switch coordinates its transmissions and never forwards more than one frame onto the same interface at any time. Furthermore, modern switches are full-duplex, so that a switch and a node can each send frames to each other at the same time without interference. 
> > 对于基于交换机的星型拓扑，交换机的策略是存储转发分组交换，交换机会协调帧的传输，在任何时候都不会向相同接口转发超过 1 个帧；
> > 
> > 现代交换机是全双工的，因此一台交换机和一个节点能够同时向双方发送帧——不会有干扰。
> 
> In other words, in a switch-based Ethernet LAN there are no collisions and, therefore, ***there is no need for a MAC protocol!***

- Manchester 编码 —— 物理层
	- 在 10BaseT 中使用
	- 每一个 bit 的位时中间有一个信号跳变，传送 1 时信号周期的中间有一个向下的跳变，传送 0 时信号周期的中间有一个向上的跳变
		- 为什么要跳变？为了在电磁波信号中将时钟信号通过一些简单的电路抽取出来——获得时序信息
		- 允许在接收方和发送方节点之间进行时钟同步——自同步的编码方案
		- 其它自同步编码方案：曼彻斯特编码、RZ（归零码）、差分曼彻斯特编码
		- 自同步需要归零，因此周期内不归零的编码方案都不是自同步的
	- 10Mbps，使用20M 带宽，效率50%
	- ![[60-Link-layer-and-LAN-code-exhibit.png]]

- 块编码
	- 100BaseT 中的 4b5b 编码（5 个 bit 代表 4 个 bit）
	- ![[60-Link-layer-and-LAN-ethernet-4b5b.png]]

### switches 链路层交换机

- Hub的碰撞问题
	- 网段 (LAN segments)：可以允许一个站点发送的网络范围
		- 在一个碰撞域，同时只允许一个站点在发送——CSMA
		- 如果有 2 个节点同时发送，则会碰撞
		- 通常拥有相同的前缀，比 IP 子网更详细的前缀
	- Hub 可以级联，所有以 Hub 连到一起的站点处在一个网段/碰撞域（一个碰撞域内一次只能有一个节点发送，两个站点同步发送会导致碰撞，会采用 CSMA/CD 的方式尝试再次重发）
	- 骨干 Hub 将所有网段连到了一起
		- 通过 Hub 可扩展节点之间的最大距离
		- 通过 Hub，不能将 10BaseT 和 100BaseT 的网络连接到一起

#### 交换机的特点

- 接收、存储、转发
	- 链路层设备：扮演主动角色（端口执行以太网协议），与之相对的 Hub 是被动地转发接口的所有数据
	- 对帧进行缓存，根据链路情况进行转发（避免碰撞）
	- 选择性转发：对于到来的帧，检查帧头，根据目标 MAC 地址进行选择性转发
- 接入控制：
	- 当帧需要向某个（些）网段进行转发，需要使用 CSMA/CD 进行接入控制
	- 对每个帧进入的链路使用以太网协议，没有碰撞——每条链路都是一个独立的碰撞域
- 并发性：
	- 通常一个交换机端口一个独立网段，允许多个节点同时发送，并发性强
- 多级结构：
	- 交换机也可以级联，多级结构中通过自学习连接源站点和目标站点
- **透明**性：
	- 主机对交换机的存在可以不关心
	- 通过交换机相联的各节点好像这些站点是直接相联的一样——主机或路由器向另一个主机或路由器直接寻址一个帧，而不是交换机
- 处于链路层：
	- 有 MAC 地址；无 IP 地址
- 即插即用，自学习(self learning)：
	- 交换机表无需手动配置
- 全双工

#### 交换机转发、过滤

- 过滤 filtering
	- 决定一个帧应该转发到某个接口还是丢弃
- 转发 forwarding
	- 决定一个帧被导向哪个接口，并把帧移动到对应接口

- 交换机转发表：实现过滤和转发功能的基础
	- 表项的结构：
		- ![[60-Link-layer-and-LAN-switch-forward-table.png]]
		- MAC 地址、转发该地址的接口、表项存在的时间
		- 基于 MAC 地址转发
	- 转发情况：
		- 当接收到帧，有三种情况：suppose a frame with destination address `DD-DD-DD-DD-DD-DD` arrives at the switch on interface *x*.
			1) 表中没有转发到该地址 (`DD-DD-DD-DD-DD-DD`) 的表项：则向除 *x* 外的所有接口，转发该帧的副本——广播之
			2) 表中存在该地址的表项，但是接口为 *x*：表示帧从包括 `DD-DD-DD-DD-DD-DD` 的网段中到来，无需转发到其他接口——丢弃之
			3) 表中存在该地址的表项，且接口 *y≠x*：转发到 *y* 接口对应的网段
		- 举例：查看上图转发表
			- Suppose that a frame with destination address `62-FE-F7-11-89-A3` arrives at the switch from interface *1*. The switch examines its table and sees that the destination is on the LAN segment connected to interface *1*. ==This means that the frame has already been broadcast on the LAN segment that contains the destination==. The switch therefore filters (that is, discards) the frame. 相同接口的数据报，已经在到达交换机之前在相同网段中进行了广播；
			- Now suppose a frame with the same destination address arrives from interface 2. The switch again examines its table and sees that the destination is in the direction of interface 1; it therefore forwards the frame to the output buffer preceding interface 1.
			- 即，交换机在不同接口间转发以太网帧，是单播的，一一对应的。
	- 如何构造交换机转发表？自学习！
		1) The switch table is *initially empty*.
		2) For each incoming frame received on an interface, the switch stores in its table (1) the MAC address in the frame’s source address field, (2) the interface from which the frame arrived, and (3) the current time. In this manner, the switch records in its table the LAN segment on which the sender resides. If every host in the LAN eventually sends a frame, then every host will eventually get recorded in the table. 存储在表项中的内容：源主机 MAC 地址、帧到达的接口、当前时间。
		3) The switch deletes an address in the table if no frames are received with that address as the source address after some period of time (the ***aging time***). In this manner, if a PC is replaced by another PC (with a different adapter), the MAC address of the original PC will eventually be purged from the switch table.一段时间后没有收到对应源地址的帧，就删除相关表项——保持最新！

#### 链路层交换机提供的高级功能
- ***Elimination of collisions***. In a LAN built from switches (and without hubs), there is no wasted bandwidth due to collisions! The switches buffer frames and never transmit more than one frame on a segment at any one time. As with a router, the maximum aggregate throughput of a switch is the sum of all the switch interface rates. Thus, switches provide a significant performance improvement over LANs with broadcast links. **消除碰撞**。交换机会缓存帧并在同一网段上只传输一个帧（同一时间），消除了 LAN 中的碰撞——没有因碰撞而浪费的带宽，因此类似路由器，其总速率就是所有接口速率之和。
- ***Heterogeneous links***. Because a switch isolates one link from another, the different links in the LAN can operate at different speeds and can run over different media. For example, the uppermost switch in Figure 6.15 might have three1 Gbps 1000BASE-T copper links, two 100 Mbps 100BASE-FX fiber links, and one 100BASE-T copper link. Thus, a switch is ideal for mixing legacy equipment with new equipment. **异质链路**。交换机将链路彼此隔离，不同链路因此得以运行不同的速率或媒体——原有设备和新设备可以混用。 
- ***Management***. In addition to providing enhanced security (see sidebar on Focus on Security), a switch also eases network management. For example, if an adapter malfunctions and continually sends Ethernet frames (called a jabbering adapter), a switch can detect the problem and internally disconnect the malfunctioning adapter. With this feature, the network administrator need not get out of bed and drive back to work in order to correct the problem. Similarly, a cable cut disconnects only that host that was using the cut cable to connect to the switch. In the days of coaxial cable, many a network manager spent hours “walking the line” (or more accurately, “crawling the floor”) to find the cable break that brought down the entire network. Switches also gather statistics on bandwidth usage, collision rates, and traffic types, and make this information available to the network manager. This information can be used to debug and correct problems, and to plan how the LAN should evolve in the future. Researchers are exploring adding yet more management functionality into Ethernet LANs in prototype deployments `[Casado 2007; Koponen 2011]`. **管理功能**。交换机能阻止不停发送帧的适配器——避免洪泛；能检测到失联的适配器——帮助确定断开的链路；能收集带宽使用数据、碰撞率、流量类型——帮助网络管理员调试。

>[!note] 交换机毒化
> When a host is connected to a switch, it typically only receives frames that are intended for it. For example,  when host A sends a frame to host B, and there is an entry for host B in the switch table, then the switch will forward the frame only to host B. If host C happens to be running a sniffer, host C will not be able to sniff this A-to-B frame. Thus, in a switched-LAN environment (in contrast to a broadcast link environment such as 802.11 LANs or hub–based Ethernet LANs), ==it is more difficult for an attacker to sniff frames==.
> > 交换机使得链路中不同网段的节点之间的帧传递是单播的，因此如果其它网段的节点使用嗅探器试图抓包——将会失败，因此相较于传统的广播链路，交换机使得攻击者更难嗅探到包。
> 
> However, because the switch broadcasts frames that have destination addresses that are not in the switch table, the sniffer at C can still sniff some frames that are not intended for C. Furthermore, a sniffer will be able sniff all Ethernet broadcast frames with broadcast destination address FF–FF–FF–FF–FF–FF.
> > 但是，交换机会在转发表中不存在对应表项时，广播收到的帧，并且对于要求广播的帧也会进行广播——此时就会被嗅探到。
> 
> A well-known attack against a switch, called ***switch poisoning***, is to send tons of packets to the switch with many different bogus source MAC addresses, thereby filling the switch table with bogus entries and leaving no room for the MAC addresses of the legitimate hosts. This causes the switch to broadcast most frames, which can then be picked up by the sniffer `[Skoudis 2006]`. 
> > 因此可行的嗅探攻击办法就是“交换机毒化”，伪造大量的不同源 MAC 地址的帧，填满交换机表，从而让交换机不停地广播合法的帧——从而得以嗅探成功。
> 
> As this attack is rather involved even for a sophisticated attacker, switches are significantly less vulnerable to sniffing than are hubs and wireless LANs.

#### 交换机 Vs. 路由器

![[60-Link-layer-and-LAN-switches-routers-hosts.png]]

>[! note] 交换机 vs. 路由器
>- 都是存储转发设备，但层次不同
>	- 交换机：链路层设备（检查链路层头部）
>	- 路由器：网络层设备（检查网络层的头部）
>	- 不过这是传统的分类方式，如今现代交换机的思路仍是“匹配+动作”，但是能够匹配的项从链路层帧、到网络层数据报各自的首部（OpenFlow 能够基于 11 种不同的首部字段进行转发）
>- 都有转发表：
>	- 交换机：维护交换表，按照 MAC 地址转发
>		- 自学习的：即插即用；
>		- 二层设备，速率高
>		- 执行生成树算法：限制广播帧的循环转发，将交换网络的活跃节点拓扑组成一棵生成树
>		- ARP 表项随着站点数量增多而增多——大型交换网络中 ARP 的流量和处理量相当 considerable
>		- 对于广播风暴不提供任何保护措施：如果某主机出现故障，不停地传输广播帧，交换机会忠实地转发，从而导致整个以太网的崩溃
>	- 路由器：路由器维护路由表，执行路由算法
>		- 网络寻址是分层的（MAC 地址是扁平的）：因此即使存在冗余路径，分组也不会通过路由器循环。不过路由表出现错误时，可能会循环，但分组中 TTL 字段避免了无限循环。
>		- 因此分组不会限制在一棵固定的生成树上，而是会动态地选择最小的生成树——路由选择算法的优势！
>		- 对广播分组做限制：设置防火墙
>		- 不是即插即用的，配置网络地址（子网前缀）
>		- 三层设备，速率低：对分组的处理时间更长
>- 何时使用？
>	- 小网络用交换机
>	- 大网络用路由器
>

### 虚拟局域网

> [! note] 传统的扁平结构的缺点
> 前文提到的不论是 MAC 地址的扁平特性、还是交换机基于一张转发表忠实地转发帧，这都表明交换机 LAN 的结构是扁平的。这样的结构存在许多缺点：
> - ***Lack of traffic isolation***. Although the hierarchy localizes group traffic to within a single switch, broadcast traffic (e.g., frames carrying ARP and DHCP messages or frames whose destination has not yet been learned by a self-learning switch) must still traverse the entire institutional network. Limiting the scope of such broadcast traffic would improve LAN performance. Perhaps more importantly, it also may be desirable to limit LAN broadcast traffic for security/privacy reasons. For example, if one group contains the company’s executive management team and another group contains disgruntled employees running Wireshark packet sniffers, the network manager may well prefer that the executives’ traffic never even reaches employee hosts. This type of isolation could be provided by replacing the center switch in Figure 6.15 with a router. We’ll see shortly that this isolation also can be achieved via a switched (layer 2) solution. 广播流量（如 ARP 报文、DHCP 报文、未学习的转发表项）会传播到该交换机所属的所有网段——浪费巨大，限制广播的范围将有效提升 LAN 的性能。并且更重要的是，出于安全性、隐私性原因，部分主机的广播分组应当针对某些特定主机。之前所学，能够提供这种范围划分的设备是路由器，但是路由器的速度较慢，因此交换机提供了 VLAN 功能进行范围划分。
> - ***Inefficient use of switches***. If instead of three groups, the institution had 10 groups, then 10 first-level switches would be required. If each group were small, say less than 10 people, then a single 96-port switch would likely be large enough to accommodate everyone, but this single switch would not provide traffic isolation. 如果在用户分组多而组内用户少时，就会出现每个用户分组都要配备一台交换机，每台交换机都要具有所有用户数量的接口——昂贵，而利用率低。
> - ***Managing users***. If an employee moves between groups, the physical cabling must be changed to connect the employee to a different switch in Figure 6.15. Employees belonging to two groups make the problem even harder. 扁平结构不适用于管理用户。
> 

支持 VLAN 的交换机：
- 允许经一个单一的物理局域网基础设施定义多个虚拟局域网，在一个 VLAN 内的主机彼此能够通信，不在同一个 VLAN 的主机彼此不能通信；
- 在一个基于端口的 VLAN 中，交换机的端口由网络管理员划分成组，每个组构成一个逻辑上的 VLAN，在这个 VLAN 中形成一个逻辑上的广播域。示意图如下：
	- ![[60-Link-layer-and-LAN-VLAN.png]]
	- 端口 2~8 属于 EE 专业，9~15 属于 CS 专业

**新的问题**：如何在两个不同的 VLAN 中交换信息？
- 直接办法：新增一个路由器，将不同 VLAN 连接到不同端口上；
- 更好的办法：将 VLAN 交换机和路由器由网络管理员统一配置，视作一个既有交换机功能、又有路由器功能的单一设备。

**更新的问题**：如何连接相同 VLAN，但是处于不同物理区域的设备？
- 第一种方法：在每台交换机上定义一个相同 VLAN 的端口，然后用电缆连接：
	- ![[60-Link-layer-and-LAN-VLAN-connect.png]]
	- 不过这种方法的弊端是，拓展性不好，因为要有 N 个 VLAN，就要白白占据 N 个端口，在端口有限的设备上难以达到要求。
- 第二种方法：VLAN 干线 trunking 连接
	- ![[60-Link-layer-and-LAN-VLAN-trunking.png]]
	- 每台交换机只开一个特殊端口，用 trunk 连接，由此处理所有 VLAN 中的帧；
	- **新新新的问题**：这种情况下要怎么确定帧属于哪个 VLAN？
		- 802.1Q 出手了！——新新新的以太网帧格式
		- ![[60-Link-layer-and-LAN-802-1Q.png]]
		- 在初始的以太网帧的 Type 字段之前，添加了一个 4 字节的 VLAN 标签——现在首部长度是 26 字节，加上尾部 CRC 字段的 4 字节一共添加了 30 字节的控制信息。
		- VLAN 标签由在 VLAN trunk 发送侧的交换机添加进帧的首部（因此要重新计算 CRC），解析后由 VLAN trunk 接收侧的交换机删除。
		- VLAN 标签本身由一个标签协议控制符（TPID，2 字节，固定值为 81-00H）、一个标签控制信息字段（12 比特）、一个优先权字段（3 比特，类似 IP 数据报的 TOS 字段 type of service 根据不同的服务提供不同的优先级）

## 6.5 链路虚拟化：MPLS

MPLS：多协议标记交换。按照标签 label 来交换分组，而非按照目标 IP 查询路由表进行存储转发，效率更高。

Multiprotocol Label Switching (MPLS) evolved from a number of industry efforts in the mid-to-late 1990s to improve the forwarding speed of IP routers by adopting a key concept from the world of virtual-circuit networks: a fixed-length label. The goal was not to abandon the destination-based IP datagram-forwarding infrastructure for one based on fixed-length labels and virtual circuits, but to augment it by selectively labeling datagrams and allowing routers to forward datagrams based on fixed-length labels (rather than destination IP addresses) when possible. Importantly, these techniques work hand-in-hand with IP, using IP addressing and routing. The IETF unified these efforts in the MPLS protocol [RFC 3031, RFC 3032], effectively blending VC techniques into a routed datagram network.

![[60-Link-layer-and-LAN-MPLS-header.png]]
- Figure 6.28 shows that a link-layer frame transmitted between MPLS-capable devices has a small MPLS header added between the layer-2 (e.g., Ethernet) header and layer-3 (i.e., IP) header. RFC 3032 defines the format of the MPLS header for such links; headers are defined for ATM and frame-relayed networks as well in other RFCs. Among the fields in the MPLS header are the label, 3 bits reserved for experimental use, a single S bit, which is used to indicate the end of a series of “stacked” MPLS headers (an advanced topic that we’ll not cover here), and a time-to-live field.
- It’s immediately evident from Figure 6.28 that an MPLS-enhanced frame can only be sent between routers that are both MPLS capable (since a non-MPLS-capable router would be quite confused when it found an MPLS header where it had expected to find the IP header!). An MPLS-capable router is often referred to as a labelswitched router, since it forwards an MPLS frame by looking up the MPLS label in its forwarding table and then immediately passing the datagram to the appropriate output interface. Thus, the MPLS-capable router need not extract the destination IP address and perform a lookup of the longest prefix match in the forwarding table.

But how does a router know if its neighbor is indeed MPLS capable, and how does a router know what label to associate with the given IP destination?
- ![[60-Link-layer-and-LAN-MPLS-forwarding.png]]
- In the example in Figure 6.29, routers R1 through R4 are MPLS capable. R5 and R6 are standard IP routers. R1 has advertised to R2 and R3 that it (R1) can route to destination A, and that a received frame with MPLS label 6 will be forwarded to destination A. Router R3 has advertised to router R4 that it can route to destinations A and D, and that incoming frames with MPLS labels 10 and 12, respectively, will be switched toward those destinations. Router R2 has also advertised to router R4 that it (R2) can reach destination A, and that a received frame with MPLS label 8 will be switched toward A. Note that router R4 is now in the interesting position of having two MPLS paths to reach A: via interface 0 with outbound MPLS label 10, and via interface 1 with an MPLS label of 8. The broad picture painted in Figure 6.29 is that IP devices R5, R6, A, and D are connected together via an MPLS infrastructure (MPLS-capable routers R1, R2, R3, and R4) in much the same way that a switched LAN or an ATM network can connect together IP devices. And like a switched LAN or ATM network, the MPLScapable routers R1 through R4 do so *without ever touching the IP header of a packet*.

Thus far, the emphasis of our discussion of MPLS has been on the fact that MPLS performs switching based on labels, without needing to consider the IP address of a packet. The true advantages of MPLS and the reason for current interest in MPLS, however, lie not in the potential increases in switching speeds, but rather in the new traffic management capabilities that MPLS enables. As noted above, R4 has two MPLS paths to A. If forwarding were performed up at the IP layer on the basis of IP address, the IP routing protocols we studied in Chapter 5 would specify only a single, least-cost path to A. ==Thus, MPLS provides the ability to forward packets along routes that would not be possible using standard IP routing protocols==. This is one simple form of traffic engineering using MPLS [RFC 3346; RFC 3272; RFC 2702; Xiao 2000], in which a network operator can override normal IP routing and force some of the traffic headed toward a given destination along one path, and other traffic destined toward the same destination along another path (whether for policy, performance, or some other reason).

## 6.6 数据中心网络

数据中心网络的开支：
- The cost of a large data center is huge, exceeding $12 million per month for a 100,000 host data center in 2009 [Greenberg 2009a]. Of these costs, about 45 percent can be attributed to the hosts themselves (which need to be replaced every 3–4 years); 25 percent to infrastructure, including transformers, uninterruptable power supplies (UPS) systems, generators for long-term outages, and cooling systems; 15 percent for electric utility costs for the power draw; and 15 percent for networking, including network gear (switches, routers, and load balancers), external links, and transit traffic costs. (In these percentages, costs for equipment are amortized so that a common cost metric is applied for one-time purchases and ongoing expenses such as power.) While networking is not the largest cost, networking innovation is the key to reducing overall cost and maximizing performance [Greenberg 2009a].

数据中心的物理结构：
- The hosts in data centers, called blades and resembling pizza boxes, are generally commodity hosts that include CPU, memory, and disk storage. The hosts are stacked in racks, with each rack typically having 20 to 40 blades. At the top of each rack, there is a switch, aptly named the Top of Rack (TOR) switch, that interconnects the hosts in the rack with each other and with other switches in the data center. Specifically, each host in the rack has a network interface that connects to its TOR switch, and each TOR switch has additional ports that can be connected to other switches. Today, hosts typically have 40 Gbps or 100 Gbps Ethernet connections to their TOR switches [FB 2019; Greenberg 2015; Roy 2015; Singh 2015]. Each host is also assigned its own data-centerinternal IP address.

数据中心支持的流量：
- ![[60-Link-layer-and-LAN-DataCenter.png]]
- The data center network supports two types of traffic: traffic flowing between external clients and internal hosts and traffic flowing between internal hosts. To handle flows between external clients and internal hosts, the data center network includes one or more border routers, connecting the data center network to the public Internet. The data center network therefore interconnects the racks with each other and connects the racks to the border routers. Figure 6.30 shows an example of a data center network.

### 负载均衡

To support requests from external clients, each application is associated with a publicly visible IP address to which clients send their requests and from which they receive responses. Inside the data center, the external requests are first directed to a load balancer whose job it is to distribute requests to the hosts, balancing the load across the hosts as a function of their current load [Patel 2013; Eisenbud 2016]. A large data center will often have several load balancers, each one devoted to a set of specific cloud applications. Such a load balancer is sometimes referred to as a “layer-4 switch” since it makes decisions based on the destination port number (layer 4) as well as destination IP address in the packet. Upon receiving a request for a particular application, the load balancer forwards it to one of the hosts that handles the application. (A host may then invoke the services of other hosts to help process the request.) The load balancer not only balances the work load across hosts, but also provides a NAT-like function, translating the public external IP address to the internal IP address of the appropriate host, and then translating back for packets traveling in the reverse direction back to the clients. This prevents clients from contacting hosts directly, which has the security benefit of hiding the internal network structure and preventing clients from directly interacting with the hosts.

### 结构

For a small data center housing only a few thousand hosts, a simple network consisting of a border router, a load balancer, and a few tens of racks all interconnected by a single Ethernet switch could possibly suffice. But to scale to tens to hundreds of thousands of hosts, a data center often employs a hierarchy of routers and switches, such as the topology shown in Figure 6.30. At the top of the hierarchy, the border router connects to access routers (only two are shown in Figure 6.30, but there can be many more). Below each access router, there are three tiers of switches. Each access router connects to a top-tier switch, and each top-tier switch connects to multiple second-tier switches and a load balancer. Each second-tier switch in turn connects to multiple racks via the racks’ TOR switches (third-tier switches). All links typically use Ethernet for their link-layer and physical-layer protocols, with a mix of copper and fiber cabling. With such a hierarchical design, it is possible to scale a data center to hundreds of thousands of hosts.

Because it is critical for a cloud application provider to continually provide applications with high availability, data centers also include redundant network equipment and redundant links in their designs (not shown in Figure 6.30). For example, each TOR switch can connect to two tier-2 switches, and each access router, tier-1 switch, and tier-2 switch can be duplicated and integrated into the design [Cisco 2012; Greenberg 2009b]. In the hierarchical design in Figure 6.30, observe that the hosts below each access router form a single subnet. In order to localize ARP broadcast traffic, each of these subnets is further partitioned into smaller VLAN subnets, each comprising a few hundred hosts [Greenberg 2009a].

Although the conventional hierarchical architecture just described solves the problem of scale, it suffers from limited host-to-host capacity [Greenberg 2009b]. To understand this limitation, consider again Figure 6.30, and suppose each host connects to its TOR switch with a 10 Gbps link, whereas the links between switches are 100 Gbps Ethernet links. Two hosts in the same rack can always communicate at a full 10 Gbps, limited only by the rate of the hosts’ network interface controllers. However, if there are many simultaneous flows in the data center network, the maximum rate between two hosts in different racks can be much less. To gain insight into this issue, consider a traffic pattern consisting of 40 simultaneous flows between 40 pairs of hosts in different racks. Specifically, suppose each of 10 hosts in rack 1 in Figure 6.30 sends a flow to a corresponding host in rack 5. Similarly, there are ten simultaneous flows between pairs of hosts in racks 2 and 6, ten simultaneous flows between racks 3 and 7, and ten simultaneous flows between racks 4 and 8. If each flow evenly shares a link’s capacity with other flows traversing that link, then the 40 flows crossing the 100 Gbps A-to-B link (as well as the 100 Gbps B-to-C link) will each only receive 100 Gbps / 40 = 2.5 Gbps, which is significantly less than the 10 Gbps network interface rate. The problem becomes even more acute for flows between hosts that need to travel higher up the hierarchy.

There are several possible solutions to this problem:
- One possible solution to this limitation is to deploy higher-rate switches and routers. But this would significantly increase the cost of the data center, because switches and routers with high port speeds are very expensive. 
- A second solution to this problem, which can be adopted whenever possible, is to co-locate related services and data as close to one another as possible (e.g., in the same rack or in a nearby rack) [Roy 2015; Singh 2015] in order to minimize inter-rack communication via tier-2 or tier-1 switches. But this can only go so far, as a key requirement in data centers is flexibility in placement of computation and services [Greenberg 2009b; Farrington 2010]. For example, a large-scale Internet search engine may run on thousands of hosts spread across multiple racks with significant bandwidth requirements between all pairs of hosts. Similarly, a cloud computing service (such Amazon Web Services or Microsoft Azure) may wish to place the multiple virtual machines comprising a customer’s service on the physical hosts with the most capacity irrespective of their location in the data center. If these physical hosts are spread across multiple racks, network bottlenecks as described above may result in poor performance.
- A final piece of the solution is to provide increased connectivity between the TOR switches and tier-2 switches, and between tier-2 switches and tier-1 switches. For example, as shown in Figure 6.31, each TOR switch could be connected to two tier-2 switches, which then provide for multiple link- and switch-disjoint paths between racks. 
	- ![[60-Link-layer-and-LAN-data-network-topo.png]]
	- In Figure 6.31, there are four distinct paths between the first tier-2 switch and the second tier-2 switch, together providing an aggregate capacity of 400 Gbps between the first two tier-2 switches. Increasing the degree of connectivity between tiers has two significant benefits: there is both increased capacity and increased reliability (because of path diversity) between switches. In Facebook’s data center [FB 2014; FB 2019], each TOR is connected to four different tier-2 switches, and each tier-2 switch is connected to four different tier-1 switches.

A direct consequence of the increased connectivity between tiers in data center networks is that multi-path routing can become a first-class citizen in these networks. Flows are by default multipath flows. A very simple scheme to achieve multi-path routing is Equal Cost Multi Path (ECMP) [RFC 2992], which performs a randomized next-hop selection along the switches between source and destination. Advanced schemes using finer-grained load balancing have also been proposed [Alizadeh 2014; Noormohammadpour 2018]. While these schemes perform multi-path routing at the flow level, there are also designs that route individual packets within a flow among multiple paths [He 2015; Raiciu 2010]

### 数据中心网络的发展趋势

Data center networking is evolving rapidly, with the trends being driven by cost reduction, virtualization, physical constraints, modularity, and customization.

#### Cost Reduction

In order to reduce the cost of data centers, and at the same time improve their delay and throughput performance, as well as ease of expansion and deployment, Internet cloud giants are continually deploying new data center network designs. Although some of these designs are proprietary, others (e.g., [FB 2019]) are explicitly open or described in the open literature (e.g., [Greenberg 2009b; Singh 2015]). Many important trends can thus be identified.

Figure 6.31 illustrates one of the most important trends in data center networking—the emergence of a hierarchical, tiered network interconnecting the data center hosts. This hierarchy conceptually serves the same purpose as a single (very, very!), large crossbar switch that we studied in Section 4.2.2, allowing any host in the data center to communicate with any other host. But as we have seen, this tiered interconnection network has many advantages over a conceptual crossbar switch, including multiple paths from source to destination and the increased capacity (due to multipath routing) and reliability (due to multiple switch- and link-disjoint paths between any two hosts).

The data center interconnection network is comprised of a large number of smallsized switches. For example, in Google’s Jupiter datacenter fabric, one configuration has 48 links between the ToR switch and its servers below, and connections up to 8 tier-2 switches; a tier-2 switch has links to 256 ToR switches and links up to 16 tier-1 switches [Singh 2015]. In Facebook’s data center architecture, each ToR switch connects up to four different tier-2 switches (each in a different “spline plane”), and each tier-2 switch connects up to 4 of the 48 tier-1 switches in its spline plane; there are four spline planes. Tier-1 and tier-2 switches connect down to a larger, scalable number of tier-2 or ToR switches, respectively, below [FB 2019]. For some of the largest data center operators, these switches are being built in-house from commodity, off-the-shelf, merchant silicon [Greenberg 2009b; Roy 2015; Singh 2015] rather than being purchased from switch vendors.

A multi-switch layered (tiered, multistage) interconnection network such as that in Figure 6.31 and as implemented in the data center architectures discussed above is known as Clos networks, named after Charles Clos, who studied such networks [Clos 1953] in the context of telephony switching. Since then, a rich theory of Clos networks has been developed, finding additional use in data center networking and in multiprocessor interconnection networks.

#### Centralized SDN Control and Management

Because a data center is managed by a single organization, it is perhaps natural that a number of the largest data center operators, including Google, Microsoft, and Facebook, are embracing the notion of SDN-like logically centralized control. Their architectures also reflect a clear separation of a data plane (comprised of relatively simple, commodity switches) and a software-based control plane, as we saw in Section 5.5. Due to the immense-scale of their data centers, automated configuration and operational state management, as we encountered in Section 5.7, are also crucial.

#### Virtualization

Virtualization has been a driving force for much of the growth of cloud computing and data center networks more generally. Virtual Machines (VMs) decouple software running applications from the physical hardware. This decoupling also allows seamless migration of VMs between physical servers, which might be located on different racks. Standard Ethernet and IP protocols have limitations in enabling the movement of VMs while maintaining active network connections across servers. Since all data center networks are managed by a single administrative authority, an elegant solution to the problem is to treat the entire data center network as a single, flat, layer-2 network. Recall that in a typical Ethernet network, the ARP protocol maintains the binding between the IP address and hardware (MAC) address on an interface. To emulate the effect of having all hosts connect to a “single” switch, the ARP mechanism is modified to use a DNS style query system instead of a broadcast, and the directory maintains a mapping of the IP address assigned to a VM and which physical switch the VM is currently connected to in the data center network. Scalable schemes that implement this basic design have been proposed in [Mysore 2009; Greenberg 2009b] and have been successfully deployed in modern data centers.

#### Physical Constraints

Unlike the wide area Internet, data center networks operate in environments that not only have very high capacity (40 Gbps and 100 Gbps links are now commonplace) but also have extremely low delays (microseconds). Consequently, buffer sizes are small and congestion control protocols such as TCP and its variants do not scale well in data centers. In data centers, congestion control protocols have to react fast and operate in extremely low loss regimes, as loss recovery and timeouts can lead to extreme inefficiency. Several approaches to tackle this issue have been proposed and deployed, ranging from data center-specific TCP variants [Alizadeh 2010] to implementing Remote Direct Memory Access (RDMA) technologies on standard Ethernet [Zhu 2015; Moshref 2016; Guo 2016]. Scheduling theory has also been applied to develop mechanisms that decouple flow scheduling from rate control, enabling very simple congestion control protocols while maintaining high utilization of the links [Alizadeh 2013; Hong 2012].

#### Hardware Modularity and Customization

Another major trend is to employ shipping container–based modular data centers (MDCs) [YouTube 2009; Waldrop 2007]. In an MDC, a factory builds, within a standard 12-meter shipping container, a “mini data center” and ships the container to the data center location. Each container has up to a few thousand hosts, stacked in tens of racks, which are packed closely together. At the data center location, multiple containers are interconnected with each other and also with the Internet. Once a prefabricated container is deployed at a data center, it is often difficult to service. Thus, each container is designed for graceful performance degradation: as components (servers and switches) fail over time, the container continues to operate but with degraded performance. When many components have failed and performance has dropped below a threshold, the entire container is removed and replaced with a fresh one.

Building a data center out of containers creates new networking challenges. With an MDC, there are two types of networks: the container-internal networks within each of the containers and the core network connecting each container [Guo 2009; Farrington 2010]. Within each container, at the scale of up to a few thousand hosts, it is possible to build a fully connected network using inexpensive commodity Gigabit Ethernet switches. However, the design of the core network, interconnecting hundreds to thousands of containers while providing high host-to-host bandwidth across containers for typical workloads, remains a challenging problem. A hybrid electrical/optical switch architecture for interconnecting the containers is described in [Farrington 2010].

Another important trend is that large cloud providers are increasingly building or customizing just about everything that is in their data centers, including network adapters, switches routers, TORs, software, and networking protocols [Greenberg 2015; Singh 2015]. Another trend, pioneered by Amazon, is to improve reliability with “availability zones,” which essentially replicate distinct data centers in different nearby buildings. By having the buildings nearby (a few kilometers apart), transactional data can be synchronized across the data centers in the same availability zone while providing fault tolerance [Amazon 2014]. Many more innovations in data center design are likely to continue to come.

## 6.7 链路层、网络层、传输层、应用层综合实例

Figure 6.32 illustrates our setting: a student, Bob, connects a laptop to his school’s Ethernet switch and downloads a Web page (say the home page of `www.google.com`). As we now know, there’s a lot going on “under the hood” to satisfy this seemingly simple request.

![[60-Link-layer-and-LAN-Web-request.png]]

### 预备：DHCP、UDP、IP、Ethernet

Let’s suppose that Bob boots up his laptop and then connects it to an Ethernet cable connected to the school’s Ethernet switch, which in turn is connected to the school’s router, as shown in Figure 6.32. The school’s router is connected to an ISP, in this example, comcast.net. In this example, comcast.net is providing the DNS service for the school; thus, the DNS server resides in the Comcast network rather than the school network. We’ll assume that the DHCP server is running within the router, as is often the case.
> - Bob 使用自己的笔电，通过以太网电缆链接到学校的以太网交换机，
> - 这个以太网交换机与学校的路由器相联，
> - 学校路由器与 ISP 连接，ISP 的名字为 `comcast.net`，其为学校提供 DNS 解析服务，所以本地 DNS 服务器在 `comcast.net` 的子网中，
> - DHCP 运行在路由器中，用于动态地为主机分配临时 IP 地址。

When Bob first connects his laptop to the network, he can’t do anything (e.g., download a Web page) without an IP address. Thus, the first network-related action taken by Bob’s laptop is to run the DHCP protocol to obtain an IP address, as well as other information, from the local DHCP server:
> Bob 刚连接到网络时，没有 IP 地址，因此什么事也做不成。于是首先要运行的协议就是 DHCP：

1. The operating system on Bob’s laptop creates a DHCP request message (Section 4.3.3) and puts this message within a UDP segment (Section 3.3) with destination port 67 (DHCP server) and source port 68 (DHCP client). The UDP segment is then placed within an IP datagram (Section 4.3.1) with a broadcast IP destination address (255.255.255.255) and a source IP address of 0.0.0.0, since Bob’s laptop doesn’t yet have an IP address. 
> - Bob 笔电的 OS 创建一个 DHCP 请求信息，其放在 UDP 报文段中，目标端口 67 指向 DHCP 服务器，源端口 68指向 DHCP 客户端；
> - 之后这个 UDP 段存放在 IP 数据报中，然后使用 IP 地址 255.255.255.255 广播，其中源地址是 0.0.0.0（因为这个笔电目前还没有 IP 地址）

2. The IP datagram containing the DHCP request message is then placed within an Ethernet frame (Section 6.4.2). The Ethernet frame has a destination MAC addresses of FF:FF:FF:FF:FF: FF so that the frame will be broadcast to all devices connected to the switch (hopefully including a DHCP server); the frame’s source MAC address is that of Bob’s laptop, 00:16:D3:23:68:8A.
> - 包含 DHCP 请求信息的 IP 数据报被包装在以太网帧中，以太网帧的目标 MAC 地址是 FF:FF:FF:FF:FF:FF（当然也是广播，因为它还不知道 DHCP 服务器的 MAC 地址是什么）
> - 源 MAC 地址就是这台笔电的适配器的地址 00:16:D3:23:68:8A

3. The broadcast Ethernet frame containing the DHCP request is the first frame sent by Bob’s laptop to the Ethernet switch. The switch broadcasts the incoming frame on all outgoing ports, including the port connected to the router.
> - 这个带有 DHCP 请求新的广播以太网帧是 Bob 的笔电发送给以太网交换机的第一个帧，这意味着以太网交换机的交换机转发表中没有对应的表项，因此它会复制、广播给所有其他接口，
> - 于是这个帧能够抵达路由器——其中的 DHCP 服务器会响应，分配一个 IP 地址；

4. The router receives the broadcast Ethernet frame containing the DHCP request on its interface with MAC address 00:22:6B:45:1F:1B and the IP datagram is extracted from the Ethernet frame. The datagram’s broadcast IP destination address indicates that this IP datagram should be processed by upper layer protocols at this node, so the datagram’s payload (a UDP segment) is thus demultiplexed (Section 3.2) up to UDP, and the DHCP request message is extracted from the UDP segment. The DHCP server now has the DHCP request message.
> - 路由器接收到含有 DHCP 请求的广播帧，此处的接收接口是路由器上 MAC 地址为 `00:22:6B:45:1F:1B` 的适配器，
> - 从这个以太网帧中提取出 IP 数据报，得知这个数据报是通过广播到达的，表明这个 IP 数据报应当在此节点交由上层协议进行处理，于是数据报中的负载——UDP 字段解复用并交付给 UDP 协议，
> - UDP 协议会从中提取出 DHCP 请求，得到 Bob 请求 DHCP 分配 IP 地址的请求信息；

5. Let’s suppose that the DHCP server running within the router can allocate IP addresses in the CIDR (Section 4.3.3) block 68.85.2.0/24. In this example, all IP addresses used within the school are thus within Comcast’s address block. Let’s suppose the DHCP server allocates address 68.85.2.101 to Bob’s laptop. The DHCP server creates a DHCP ACK message (Section 4.3.3) containing this IP address, as well as the IP address of the DNS server (68.87.71.226), the IP address for the default gateway router (68.85.2.1), and the subnet block (68.85.2.0/24) (equivalently, the “network mask”). The DHCP message is put inside a UDP segment, which is put inside an IP datagram, which is put inside an Ethernet frame. The Ethernet frame has a source MAC address of the router’s interface to the home network (00:22:6B:45:1F:1B) and a destination MAC address of Bob’s laptop (00:16:D3:23:68:8A).
> - DHCP 服务器以 CIDR 块 68.85.2.0/24 分配 IP 地址——学校的所有 IP 地址都在 `comcast.net` 的地址块中（其地址块为 `68.80.0.0/13` ）
> - 假设 DHCP 服务器分配 68.85.2.101 的 IP 地址给 Bob 的笔电，于是 DHCP 服务器需要生成的响应报文中，应当包含如下地址：
> 	- 分配的 IP 地址：68.85.2.101
> 	- 本地 DNS 服务器的 IP 地址：68.87.71.226
> 	- 默认网关路由器的 IP 地址：68.85.2.1
> 	- 子网块：68.85.2.0/24
> - DHCP 服务器已经生成了包含所需信息的 ACK 响应报文，于是通过 UDP、IP、Ethernet 一层层地重新包装，再发送到以太网中；
> 	- 注意，ACK 响应的以太网帧中，源 MAC 地址是路由器接口的适配器的 MAC 地址 `00:22:6B:45:1F:1B`，
> 	- 目的 MAC 地址是 Bob 的笔电的适配器的 MAC 地址 `00:16:D3:23:68:8A`

6. The Ethernet frame containing the DHCP ACK is sent (unicast) by the router to the switch. Because the switch is self-learning (Section 6.4.3) and previously received an Ethernet frame (containing the DHCP request) from Bob’s laptop, the switch knows to forward a frame addressed to `00:16:D3:23:68:8A` only to the output port leading to Bob’s laptop.
> - 包含 DHCP ACK 的以太网帧有了具体的发送目标，于是单播地从路由器发送给交换机，
> - 交换机是自学习的，因此之前从 DHCP 请求帧中学习到 Bob 笔电的 MAC 地址和来路接口，于是记录了对应的转发表项，因此可以单播地从交换机转发到 Bob 的主机上；

7. Bob’s laptop receives the Ethernet frame containing the DHCP ACK, extracts the IP datagram from the Ethernet frame, extracts the UDP segment from the IP datagram, and extracts the DHCP ACK message from the UDP segment. Bob’s DHCP client then records its IP address and the IP address of its DNS server. It also installs the address of the default gateway into its IP forwarding table (Section 4.1). Bob’s laptop will send all datagrams with destination address outside of its subnet 68.85.2.0/24 to the default gateway. At this point, Bob’s laptop has initialized its networking components and is ready to begin processing the Web page fetch. (Note that only the last two DHCP steps of the four presented in Chapter 4 are actually necessary.)
> - Bob 的笔电收到了包含 DHCP ACK 的以太网帧，从中一层层地从 Ethernet frame、IP datagram、UDP segment 中由 DHCP 客户端提取出 DHCP ACK 信息，
> - 现在，Bob 笔电上的 DHCP 服务器记录下分配到的 IP 地址和解析这个 IP 地址的 DNS 服务器的地址，并且在 IP 转发表中安装默认网关的地址（68.85.2.1），
> - 现在，Bob 笔电上的所有数据报（除了发送给自身所在网段 68.85.2.0/24 的），都将发送给默认网关；

OK，Bob 现在迈出了第一步——初始化好笔电的网络组件（==有了自己的 IP，能够向 DNS 服务器请求网址解析，获知了自己所在的网段和默认转发网关==）。

### 仍在准备：DNS、ARP

When Bob types the URL for `www.google.com` into his Web browser, he begins the long chain of events that will eventually result in Google’s home page being displayed by his Web browser. Bob’s Web browser begins the process by creating a TCP socket (Section 2.7) that will be used to send the HTTP request (Section 2.2) to `www.google.com`. In order to create the socket, Bob’s laptop will need to know the IP address of `www.google.com`. We learned in Section 2.5, that the DNS protocol is used to provide this name-to-IP-address translation service.
> - Bob 在其网络浏览器中键入谷歌的 URL 后，浏览器将会创建一个 TCP socket，用于向谷歌发送 HTTP 请求，
> - 在创建这个 TCP socket 之前，首先需要得知 URL `www.google.com` 背后的 IP 地址究竟是什么——DNS 协议闪亮登场！

8. The operating system on Bob’s laptop thus creates a DNS query message (Section 2.5.3), putting the string “ `www.google.com` ” in the question section of the DNS message. This DNS message is then placed within a UDP segment with a destination port of 53 (DNS server). The UDP segment is then placed within an IP datagram with an IP destination address of 68.87.71.226 (the address of the DNS server returned in the DHCP ACK in step 5) and a source IP address of 68.85.2.101.
> - Bob 的笔电上 OS 创建一个 DNS 请求信息，希望获取 `www.google.com` 的 IP 地址信息，
> - 该报文首先包装在 UDP 段中，其中目标端口 53 指向 DNS 服务器，接着被打包进 IP 数据报，其中目标 IP 地址是 DNS 服务器的地址 68.87.71.226，源 IP 地址是前文从 DHCP 服务器那里分配得到的 68.85.2.101；

9. Bob’s laptop then places the datagram containing the DNS query message in an Ethernet frame. This frame will be sent (addressed, at the link layer) to the gateway router in Bob’s school’s network. However, even though Bob’s laptop knows the IP address of the school’s gateway router (68.85.2.1) via the DHCP ACK message in step 5 above, it doesn’t know the gateway router’s MAC address. In order to obtain the MAC address of the gateway router, Bob’s laptop will need to use the ARP protocol (Section 6.4.1).
> - Bob 的笔电于是将包含 DNS 查询的数据报包装在以太网帧中，帧接着被发送到 Bob 校园的网关路由器，
> - 不过虽然 Bob 已经在 DHCP ACK 报文中得知了网关的 IP 地址，但还不知道它的 MAC 地址，因此为了获得其 MAC 地址，Bob 的笔电需要使用 ARP 协议；

10. Bob’s laptop creates an ARP query message with a target IP address of 68.85.2.1 (the default gateway), places the ARP message within an Ethernet frame with a broadcast destination address (FF:FF:FF:FF:FF:FF) and sends the Ethernet frame to the switch, which delivers the frame to all connected devices, including the gateway router.
> - Bob 的笔电因此创建一个 ARP 查询信息，报文中目标 IP 地址是默认网关的 IP 68.85.2.1，然后将这个 ARP 查询信息包装在以太网帧中广播，最终这个以太网帧将会到达交换机，
> - 交换机会将这个帧发送到所有已连设备，因此网关路由器最终也会收到；

11. The gateway router receives the frame containing the ARP query message on the interface to the school network, and finds that the target IP address of 68.85.2.1 in the ARP message matches the IP address of its interface. The gateway router thus prepares an ARP reply, indicating that its MAC address of `00:22:6B:45:1F:1B` corresponds to IP address 68.85.2.1. It places the ARP reply message in an Ethernet frame, with a destination address of `00:16:D3:23:68:8A` (Bob’s laptop) and sends the frame to the switch, which delivers the frame to Bob’s laptop.
> - 网关路由器在连接到校园网络的接口收到包含 ARP 查询的帧后，从帧的目标 IP 地址得知与对应接口（就是收到 ARP 查询帧的那个接口）匹配，
> - 于是网关路由器会将对应接口的适配器的 MAC 地址 `00:22:6B:45:1F:1B` 放在 ARP 回复报文中，然后送回以太网，向着目标 MAC 地址 `00:16:D3:23:68:8A` 的设备（Bob 的笔电）前进；

12. Bob’s laptop receives the frame containing the ARP reply message and extracts the MAC address of the gateway router (00:22:6B:45:1F:1B) from the ARP reply message.
> - Bob 的笔电收到包含 ARP 响应的信息后，从中提取出网关路由器的 MAC 地址为 `00:22:6B:45:1F:1B`；

13. Bob’s laptop can now (finally!) address the Ethernet frame containing the DNS query to the gateway router’s MAC address. Note that the IP datagram in this frame has an IP destination address of 68.87.71.226 (the DNS server), while the frame has a destination address of `00:22:6B:45:1F:1B` (the gateway router). Bob’s laptop sends this frame to the switch, which delivers the frame to the gateway router.
> - 终于，Bob 的笔电现在能够将包含 DNS 查询信息的以太网帧发送到网关路由器中，
> - 要注意，这个以太网帧的目标 IP 地址是 DNS 服务器的 IP 地址68.81.71.226，而目标 MAC 地址是网关路由的对应接口的适配器的 MAC 地址 `00:22:6B:45:1F:1B`；

### 还在准备：OSPF/RIP、BGP、DNS Server

14. The gateway router receives the frame and extracts the IP datagram containing the DNS query. The router looks up the destination address of this datagram (68.87.71.226) and determines from its forwarding table that the datagram should be sent to the leftmost router in the Comcast network in Figure 6.32. The IP datagram is placed inside a link-layer frame appropriate for the link connecting the school’s router to the leftmost Comcast router and the frame is sent over this link.
> - 网关路由器收到 DNS 请求帧后从中提取出 IP 数据报，检查其中的目标 IP 地址——哦，需要发送到 IP 地址为 68.87.71.226 的地方，于是根据转发表决定数据报应当发送到 `comcast.net` 最左端的路由器，
> ![[60-Link-layer-and-LAN-Web-request.png]]
> - 然后这个数据报重新打包成以太网帧，发送到以太网链路中，注意最长路径匹配原则，网关路由器会发送帧到合理的链路中；

15. The leftmost router in the Comcast network receives the frame, extracts the IP datagram, examines the datagram’s destination address (68.87.71.226) and determines the outgoing interface on which to forward the datagram toward the DNS server from its forwarding table, which has been filled in by Comcast’s intra-domain protocol (such as RIP, OSPF or IS-IS, Section 5.3) as well as the Internet’s inter-domain protocol, BGP (Section 5.4).
> - `comcast.net` 网络中最左端的路由器收到这个 DNS 请求帧，从中提取数据报并了解到目的 IP 地址为 68.87.71.226，于是根据转发表确定转发这个数据报的出口，
> - 而转发表已经根据 `comcast.net` 的域内路由协议（RIP、OSPF、IS-IS）和域间路由协议（BGP）填写完成；

16. Eventually the IP datagram containing the DNS query arrives at the DNS server. The DNS server extracts the DNS query message, looks up the name `www.google.com` in its DNS database (Section 2.5), and finds the DNS resource record that contains the IP address (64.233.169.105) for `www.google.com`. (assuming that it is currently cached in the DNS server). Recall that this cached data originated in the authoritative DNS server (Section 2.5.2) for google.com. The DNS server forms a DNS reply message containing this hostname-to-IP-address mapping, and places the DNS reply message in a UDP segment, and the segment within an IP datagram addressed to Bob’s laptop (68.85.2.101). This datagram will be forwarded back through the Comcast network to the school’s router and from there, via the Ethernet switch to Bob’s laptop.
> - 最终，DNS 请求信息的数据报在 BGP、OSPF 填写的转发表的帮助下，成功到达了 DNS 服务器，
> - DNS 服务器提取其中的 DNS 请求信息——它要 `www.google.com` 的 IP 地址，于是在其 DNS 数据库中查找，如果不久之前有人恰好也访问过 `www.google.com`，那么应当还缓存在 DNS 服务器中，于是能够正确地找到 DNS 资源记录，
> - 如果没有缓存，怎么办？本地 DNS 服务器会依次向根 DNS 服务器、顶级域名服务器、权威域名服务器进行或递归、或迭代的 DNS 查询，但是这里是更深层的 DNS 的工作，暂且不必深究，
> - 于是，DNS 服务器生成了一则 DNS 回应信息，包含了 `www.google.com` 这个 URL 与对应 IP 地址的映射关系，然后依次放入 UDP 段、IP 数据报中发回 Bob 的笔电——和来时的路经相同；

17. Bob’s laptop extracts the IP address of the server `www.google.com` from the DNS message. Finally, after a lot of work, Bob’s laptop is now ready to contact the `www.google.com` server!
> - Bob 的笔电从 DNS 响应报文中提取出 `www.google.com` 的 IP 地址信息，
> - 终于，一番工作后，Bob 的笔电现在终于能连接到名为 `www.google.com` 的服务器；

### 大的终于来了：TCP、HTTP

18. Now that Bob’s laptop has the IP address of `www.google.com` , it can create the TCP socket (Section 2.7) that will be used to send the HTTP GET message (Section 2.2.3) to `www.google.com`. When Bob creates the TCP socket, the TCP in Bob’s laptop must first perform a three-way handshake (Section 3.5.6) with the TCP in `www.google.com`. Bob’s laptop thus first creates a TCP SYN segment with destination port 80 (for HTTP), places the TCP segment inside an IP datagram with a destination IP address of 64.233.169.105 ( `www.google.com` ), places the datagram inside a frame with a destination MAC address of 00:22:6B:45:1F: 1B (the gateway router) and sends the frame to the switch.
> - 现在 Bob 的笔电获悉了 `www.google.com` 的 IP 地址，于是可以创建 TCP socket 来传递 HTTP GET 请求信息，以获取 `www.google.com` 这个网址的内容，
> - Bob 的笔电要创建 TCP socket，就必须首先执行与 `www.google.com` 的经过 TCP 协议的三次握手——
> - 首先创建 TCP SYN 段发送到目标主机的 80 端口，这个 TCP 段将会被打包在目标 IP 地址为 64.233.169.105（即 `www.google.com`）的 IP 数据报中，接着打包在目的 MAC 地址 `00:22:6B:45:1F:1B`（即网关路由器的接收口适配器）的帧中，然后发送给交换机；

19. The routers in the school network, Comcast’s network, and Google’s network forward the datagram containing the TCP SYN toward `www.google.com`, using the forwarding table in each router, as in steps 14–16 above. Recall that the router forwarding table entries governing forwarding of packets over the inter-domain link between the Comcast and Google networks are determined by the BGP protocol (Chapter 5).
> - 校园网络的路由器、`comcast.net` 的网络、Google 的网络会将这个含有 TCP SYN 信息的数据报传递给 `www.google.com`（这里指的是处理这个 socket 建立请求的主机），
> - 每台路由器都会使用到转发表（自然也要经过 OSPF、BGP 的路由协议来填充）；

20. Eventually, the datagram containing the TCP SYN arrives at `www.google.com`. The TCP SYN message is extracted from the datagram and demultiplexed to the welcome socket associated with port 80. A connection socket (Section 2.7) is created for the TCP connection between the Google HTTP server and Bob’s laptop. A TCP SYNACK (Section 3.5.6) segment is generated, placed inside a datagram addressed to Bob’s laptop, and finally placed inside a link-layer frame appropriate for the link connecting `www.google.com` to its first-hop router.
> - 最终，包含 TCP SYN 的数据报抵达了 `www.google.com` ，TCP SYN 的数据将会从数据报中提取、解复用并提交给 80 端口的 socket（用于监听请求信息），
> - 为了在 Bob 的笔电和 Google 的 HTTP 服务器之间建立一个 TCP 连接，HTTP 服务器会生成一个 TCP SYNACK 段再发送回 Bob 的笔电中——作为请求建立连接的响应；

21. The datagram containing the TCP SYNACK segment is forwarded through the Google, Comcast, and school networks, eventually arriving at the Ethernet controller in Bob’s laptop. The datagram is demultiplexed within the operating system to the TCP socket created in step 18, which enters the connected state.
> - 包含 TCP SYNACK 的段再次经过 Google、`comcast.net`、校园网络最终到达 Bob 的笔电的以太网控制器（网卡、适配器），
> - 当数据报被解复用提交给已经创建好的 TCP socket，从而进入了连接建立状态；

22. With the socket on Bob’s laptop now (finally!) ready to send bytes to `www.google.com`, Bob’s browser creates the HTTP GET message (Section 2.2.3) containing the URL to be fetched. The HTTP GET message is then written into the socket, with the GET message becoming the payload of a TCP segment. The TCP segment is placed in a datagram and sent and delivered to `www.google.com` as in steps 18–20 above.
> - Bob 的笔电现在终于能够向 `www.google.com` 发送字节信息，
> - 于是 Bob 的浏览器创建了一个 HTTP GET 请求，其中请求的 URL 为 `www.google.com`，于是提交给刚才建立的 socket 中，现在 GET 请求成为了 TCP 段的负载（上一步是 TCP SYN——用于建立连接），
> - 这个 GET 请求数据报经过之前讨论的步骤，再次抵达 `www.google.com`（三次握手的最后一步，通知 HTTP 服务器连接建立确实成功）；

23. The HTTP server at `www.google.com` reads the HTTP GET message from the TCP socket, creates an HTTP response message (Section 2.2), places the requested Web page content in the body of the HTTP response message, and sends the message into the TCP socket.
> - HTTP 服务器从 TCP socket 中读取 HTTP GET 请求信息，
> - 接着创建 HTTP 响应报文，将被请求的 Web 页面内容放置在 HTTP 响应报文中，然后发送给 TCP socket（在 HTTP 服务器的视角看来，TCP socket 就是一个直达 Bob 的笔电的通道，期间经过的所有路由器、交换机、子网段都不是它要考虑的事情）；

24. The datagram containing the HTTP reply message is forwarded through the Google, Comcast, and school networks, and arrives at Bob’s laptop. Bob’s Web browser program reads the HTTP response from the socket, extracts the html for the Web page from the body of the HTTP response, and finally (finally!) displays the Web page!
> - 包含 HTTP 响应的报文再一次地经过 Google、`comcast.net`、校园网络，抵达了 Bob 的笔电，
> - Bob 的浏览器从 TCP socket 中读取 HTTP 响应，并提取其中的 HTML 文件信息，在本地进行渲染——终于，Bob 能够成功地看到 `www.google.com` 这个网址所对应的页面的内容！

> [! important] 来对上面的全部过程作一个总结
> 1. 在键入 `www.google.com` 后，一共经历了多少种协议？
> 	- DHCP, UDP, IP, Ethernet, ARP, DNS, OSPF, BGP, TCP, HTTP
> 2. Bob 的笔电一共 发出/接收 了多少种报文？
> 	- DHCP request/ACK, UDP segment, IP datagram, Ethernet frame, ARP query/reply, DNS query/reply, TCP SYN/SYNACK/(FIN/FINACK), HTTP GET/response
> 3. 有多少种报文成功抵达了 `www.google.com` 这一主机？
> 	- TCP SYN, (TCP FIN), HTTP GET
> 4. 在 Bob 的笔电和 `www.google.com` 主机之间，完整穿梭了多少次？
> 	- TCP SYN/SYNACK 2 次；HTTP GET/response 2 次；(TCP FIN 拆除连接可能有 4 次）
