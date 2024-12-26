---
publish: "true"
tags:
  - Network
  - 计算机网络原理
date: 2023-10-19
---
## 3.1 概述和传输层服务

### 传输服务和协议
#### 传输层最重要的两个功能
1. 为运行在不同主机上的应用进程提供**逻辑通信**
	- 两个实体如何通过可能丢失和损坏数据的介质进行可靠通信
	- 看上去是通过 socket api 将数据传给另一个进程，实际上报文需要通过层间接口交给传输层，通过两个传输层之间的相互配合交给另一方对等的应用进程
2. 控制传输层实体的传输速率
	- 避免网络拥塞或从拥塞中恢复

#### 传输协议运行在端系统的过程
![[30-Transport-layer-logical-communication.png]]
- 发送方：将应用层的报文分成报文段，然后传递给网络层。网络层路由器在路由报文时，仅作用于网络层字段，不检查传输层及应用层字段
- 接收方：将报文段重组成报文，然后传递给应用层

- 有多个传输层协议可供应用选择
    - Internet：TCP和UDP

### 传输层 vs. 网络层
- 网络层服务（IP协议）：主机之间的逻辑通信
- 传输层服务：将主机-主机的通信细分为进程间的逻辑通信
    - 依赖于网络层的服务（IP协议）
        - 延时（传输、传播、排队等）、带宽（吞吐量，瓶颈链路）
    - 并对网络层的服务进行增强
        - 数据丢失、顺序混乱、加密
        - 如：IP 向上层提供的服务不可靠，TCP 将不可靠变为可靠：RDT。通过 SSL 将 TCP 由不安全变为安全

有些服务是可以加强的：不可靠 -> 可靠；安全    
但有些服务是不可以被加强的：带宽，延迟

> [!example] 类比：东、西2个家庭的通信
> 
> Ann 家的12个小孩给 Bill 家的12个小孩发信
> - 应用层报文 = 信封上的字符
> - 主机 = 家庭
> - 进程 = 小孩们
> - 传输层协议 = Ann 和 Bill
>     - 为家庭小孩提供复用、解复用服务
>     - Ann 将信件复用（打包）发给邮政服务，Bill 从邮政服务收到后进行解复用（拆包分发），当然反过来也可以
> - 网络层协议 = 邮政服务
>     - 家庭-家庭的邮包传输服务
> 
> 可以从上面看出以下几点：
> 1. 传输层协议只工作在端系统中，对报文在网络核心中如何移动并不作任何规定（全部交给网络层去做）
> 2. Ann 和 Bill 可以替换，比如换成另两个孩子——对应于协议的改变，如 TCP、UDP
> 3. Ann 和 Bill 能提供的服务受制于邮政服务——传输层协议的功能受制于网络层提供的服务


### Internet 传输层协议概述
- 可靠的、保序的传输：TCP (Transmission Control Protocol)
    - 多路复用、解复用
    - 拥塞控制
    - 流量控制
    - 面向连接
- 不可靠、不保序的传输：UDP (User Datagram Protocol)
    - 多路复用、解复用
    - 无连接
    - 没有为尽力而为的 IP 服务添加更多的其它额外服务，UDP 本身就是尽力服务型协议
- 都不提供的服务（依赖于网络层）：
    - 延时保证
    - 带宽保证

## 3.2 多路复用与解复用

### 多路复用与多路分解

- 将网络层提供的主机到主机的交付服务，延伸到为运行在主机上的应用程序提供进程到进程的交付服务。

![[30-Transport-layer-multiplex-demultiplex.png]]
- 回顾：一个进程可以有一个或多个 socket，socket 是网络和进程间传递数据的门户；
- 由图可以见得：接收方主机的运输层实际上并没有直接将数据交付给进程，而是将数据交给了一个 socket ；
- socket 有唯一的标识符，用以区分不同进程和进程的不同功能模块。标识符的格式取决于 TCP 还是 UDP 的 socket。

#### 在发送方主机多路复用    
从源主机的多个套接字接收来自多个进程的报文，根据套接字对应的 IP 地址和端口号等信息对报文段封装以首部信息（该头部信息用于以后的解复用）

#### 在接收方主机多路解复用 
根据报文段的头部信息中的 IP 地址和端口号将接收到的报文段发给正确的套接字（和对应的应用进程）

- 主机收到 IP 数据报
    - 每个数据报有源 IP 地址和目标地址
    - 每个数据报承载一个传输层报文段
    - 每个报文段有一个源端口号和目标端口号（特定应用有著名的端口号）
- 主机联合使用 **IP 地址** 和 **端口号** 将报文段发送给合适的套接字

#### 传输层报文段中源与目的端口字段
Socket 有唯一的标识符，每个报文段有特殊字段来指示该报文来自哪个端口、发向哪个端口，这些特殊字段就是：
![[30-Transport-layer-source-destination-port-fields.png]]
- 源端口号字段 (source port number field)；
- 目的端口号字段 (destination port number field)；
- 其它首部字段留待协议使用：TCP 和 UDP 需要的字段不同；

### 无连接(UDP)多路解复用
#### 创建套接字
服务器端：serverSocket 和 sad 指定的端口号捆绑
```python
serverSocket = socket(PF_INET, SOCK_DGRAM, 0);
bind(serverSocket, &sad, sizeof(sad));
``` 

客户端：没有 Bind，ClientSocket 和 OS 为之分配的某个端口号捆绑（客户端使用什么端口号无所谓，客户端主动找服务器）
```python
ClientSocket = socket(PF_INET, SOCK_DGRAM, 0);
```

- UDP socket 用二元组唯一地标识：(destination IP address, destination port)
- 如果两个不同源 IP 地址/源端口号的数据报，但是有相同的目标 IP 地址和端口号，则被定位到相同的目标 UDP 套接字。
- 当主机收到 UDP 报文段：
	- 检查报文段的目标端口号
	- 用该端口号将报文段定位给套接字

> [!example] 例子：UDP socket 传送报文
> 主机 A 中某进程的源 UDP 端口 19157，发向目的主机 B 的 UDP 端口 46428 的进程：
> ![[30-Transport-layer-UDP-socket.png]]
> - 主机 A 中运输层创建一个运输层报文段，其中包括应用程序数据、源端口号 19157、目的端口号 46428 。
> - 接着传输层将得到的报文段传递到网络层，网络层将该报文段封装到一个 IP 数据报中，并尽力地将数据报交付给接收主机；
> - 如果该报文段到达接收主机 B，接收主机运输层就检查该报文段中的目的端口号 46428，并将该报文段交付给端口号 46428 所标识的 socket。
> 
> ==似乎少了什么？源端口号做什么去了？==
> 源端口号作为“返回地址”，当 B 需要回发一个报文段给 A 时，B 到 A 的报文段中的目的端口号便从 A 到 B 的报文中源端口号字段提取得到。

### 面向连接(TCP)的多路复用

- TCP 套接字：四元组本地标识：(源 IP 地址, 源端口号, 目标 IP 地址, 目标端口号)
- 四元组全部用以将报文段定向（分解）到相应的 TCP socket，
- 与 UDP socket 不同的是，两个具有不同源 IP 地址或源端口号的到达 TCP 报文段，将被定向到两个不同的 TCP socket，除非 TCP 报文段携带了初始创建连接的请求

>[!note] 回顾 TCP 客户端-服务器编程
>1. The TCP server application has a “welcoming socket,” that waits for connection establishment requests from TCP clients (see Figure 2.29) on port number 12000. 
>2. The TCP client creates a socket and sends a connection establishment request segment with the lines:
> ```
>clientSocket = socket (AF_INET, SOCK_STREAM) clientSocket.connect ((serverName, 12000)) 
> ```
>
>3. A connection-establishment request is nothing more than a TCP segment with destination port number 12000 and a special connection-establishment bit set in the TCP header (discussed in Section 3.5). The segment also includes a source port number that was chosen by the client. 
>4. When the host operating system of the computer running the server process receives the incoming connection-request segment with destination port 12000, it locates the server process that is waiting to accept a connection on port number 12000. The server process then creates a new socket: 
> ```
>connectionSocket, addr = serverSocket.accept ()
> ```
>
>5. Also, the transport layer at the server notes the following four values in the connection-request segment: 
>	1) the source port number in the segment, 
>	2) the IP address of the source host, 
>	3) the destination port number in the segment, and 
>	4) its own IP address. 
>	The newly created connection socket is identified by these four values; all subsequently arriving segments whose source port, source IP address, destination port, and destination IP address match these four values will be demultiplexed to this socket.
>
>With the TCP connection now in place, the client and server can now send data to each other.


- 服务器能够在一个 TCP 端口上同时支持多个 TCP socket，每个 socket 由其四元组标识
- 当一个 TCP 报文段到达主机时，所有 4 个字段被用来将报文段定向到相应的 socket。

>[!example] TCP socket 传送报文
>主机 C 向服务器 B 发起两个 HTTP 会话，主机 A 向服务器 B 发起一个 HTTP 会话：
> ![[30-Transport-layer-TCP-socket.png]]
>- 主机 A、B、C 都有各自的 IP 地址，
>- 主机 C 为两个 HTTP 连接分配 26145、7532 两个不同的 socket 端口，
>- 主机 A 分配 socket 时与主机 C 不相干，因此也可以创建 ID 为 26145 的socket

### Web Server 与 TCP
- 高性能 Web Server 通常只使用一个进程，但是为每个新客户连接创建一个具有新连接 socket 的新线程，由线程为客户提供服务；
- 在这个场景下，还是根据4元组决定将报文段内容同一个进程下的不同线程关联，解复用到不同线程（与解复用到不同进程相似）

- Web 服务器对持久连接的 HTTP，在持续期间经由同一个服务器 socket 交换 HTTP 报文
    - 非持久连接的 HTTP，则对每个请求有不同的套接字，这种频繁创建和关闭 socket 会严重影响 Web Server 的性能；

## 3.3 无连接传输：UDP

UDP(User Datagram Protocol `[RFC 768]`)：用户数据报协议
- “no frills,” “bare bones”Internet传输协议
- “尽力而为”的服务，报文段可能丢失也可能送到应用进程的报文段乱序
- 仅提供最低限度的 复用/解复用 服务，以便能通过网络层
- 无连接：
    - UDP 发送端和接收端之间**没有握手**
    - 每个 UDP 报文段都被独立地处理
- UDP 被用于：
    - 流媒体（丢失不敏感，速率敏感、应用可控制传输速率）
    - **DNS**
    - SNMP（简单网络管理协议）
- 在 UDP 上若要实现可靠传输：
    - 在应用层增加可靠性
    - 应用特定的差错恢复

### UDP 的优势
- **适合实时应用**：不需要较高的发送速率，而要求较低的报文段发送延迟，能够容忍部分数据丢失；而 TCP 的拥塞控制在此时会导致实时应用的性能变差；
- **适合时延敏感的应用**：如对 DNS 来说，UDP 不会引入建立连接时延；HTTP 通常是建立在 TCP 上的，但是 HTTP/3.0、Google Chrome 的私有 QUIC 协议，是通过 UDP 来提高速率、在应用层保证可靠性；
- **不必维护链接**：因此可以支持的活动用户数量更多、更适合突发情况；
- **分组首部开销小**：UDP 首部开销仅 8 字节，TCP 则是 20 字节，这对数据较小时优势明显；

### 常用应用的协议
![[30-Transport-layer-applications-protocol.png]]

> [!note] SNMP 为什么选用 UDP？
> SNMP 作为网络管理协议，使用 UDP 则能够在网络处于重压状态下运行，而 TCP 的拥塞控制策略会使数据传输难以实现。

>[!note] 流式多媒体应用越来越青睐 TCP
>1. 在拥塞状态下，UDP 的情况会越来越恶化——如果每个人都启动流式高比特率视频而不使用任何拥塞控制手段的话，会导致路由器中有大量分组溢出，最终只有极少量的 UDP 分组能够成功到达目的地。
>2. 无控制的 UDP 发送方导致的高丢包率，将引起 TCP 发送方大大减少它们的速率，从而挤垮 TCP 回话，这个问题相当严重。

### UDP报文段格式

![[30-Transport-layer-UDP-segment-structure.png]]
- 报文段的头部很小，只有四个部分，每个部分 2 字节
	- 源端口号和目的端口号用于进程交付；
	- 长度字段指示了 UDP 报文段中的字节数：头部+应用数据
	- 校验和用来检查报文段在传输过程中是否出现了差错

### UDP 校验和
校验和：EDC (Error Detection and Correction)，差错检测码
- 目标：检测在被传输报文段中的差错（如比特反转），若出错，这个UDP数据报就会被扔掉（表现为丢失）
- 发送方：
    - 将报文段的内容每16bit 切一段，得到一系列长度为 16 比特的段
    - 校验和：将所有 16 比特的段相加，加法中遇到的所有溢出都加回最低位，最后进行反码运算；
    - 发送方将校验和放在 UDP 报文段的校验和字段
- 接收方：
    - 计算接收到的报文段的校验和：
	    - 将收到的校验和字段与报文段全部的 16 比特的段加起来（包括校验和本身）
	    - 如果结果是全 1 串，没有出错；否则至少有一个错误，==整个报文丢弃==或者==警告地发送给应用程序==（校验和没有纠错能力，只能检错）；


> [!example] Internet 校验和的例子：两个16bit 的整数相加
> 
> 注意：当数字相加时，在最高位的进位要回送到最低位，再加到结果上（即最高位的进位数字与末位相加，重新计算得到和）
>
> suppose that we have the following three 16-bit words:
> ![[30-Transport-layer-checksum-example.png]]
> 
> - Note that this last addition had overflow, which was wrapped around. 
> - The 1s complement is obtained by converting all the 0s to 1s and converting all the 1s to 0s. Thus, the 1s complement of the sum 0100101011000010 is 1011010100111101, which becomes the checksum. 
> - At the receiver, all four 16-bit words are added, including the checksum. If no errors are introduced into the packet, then clearly the sum at the receiver will be 1111111111111111. If one of the bits is a 0, then we know that errors have been introduced into the packet.

>[!note] 端到端原则
>在无法确保每条链路的可靠性、也无法确保中途路由器内存的差错检测时，如果端到端的数据传输要求提供服务检测，那么 UDP 必须在端到端基础上在传输层提供差错检测。（即便校验和是最基本的、只能提供检错能力的差错检测机制）
>
>**端到端原则**：This is an example of the celebrated **end-end principle** in system design `[Saltzer 1984]`, which states that since certain functionality (error detection, in this case) must be implemented on an end-end basis: “functions placed at the lower levels may be redundant or of little value when compared to the cost of providing them at the higher level.”

## 3.4 可靠数据传输的原理

Reliable Data Transfer 在应用层、传输层和数据链路层都很重要，是网络 Top 10问题之一。

信道的不可靠特点（只是“尽力而为”）决定了可靠数据传输协议 (Reliable Data Transfer Protocol) 的复杂性。

### 问题描述

![[30-Transport-layer-rdt.png]]
- RDT 协议的功能：为上层实体提供可靠信道的服务抽象
	- 数据可以通过一条可靠的信道进行传输，在可靠信道中传输数据不会导致其损坏或丢失；
	- 所有数据都是按照发送顺序进行交付；
	- TCP 就是 RDT 类型的协议，UDP 是 UDT 类型的协议；
- 因此，RDT 协议是建立在不可靠的信道、不可靠的底层协议上的，实现起来比较困难、复杂。
- 根据上图 b 的语义，我们规定：
	- 调用 `rdt_send()` 函数，上层可以调用数据传输协议的发送方，将要发送的数据交付给位于接收方的高层；
	- `rdt_send()` 函数会通过 `udt_send()` 调用不可靠的信道传输数据，这意味着数据传输中可能被损坏、丢失，但是为了简便目前不考虑底层信道重排序分组的问题；
	- 接收端在分组从信道到达时，调用 `rdt_rcv()` 接收分组，并对分组进行检查，若无问题，则通过 `deliver_data()` 向高层交付数据。

### 构造可靠数据传输协议
#### 工作安排：
- 渐增式地开发可靠数据传输协议 (rdt) 的发送方和接收方（渐增式：从下层可靠、不丢失开始，一步步去掉假设，使下层变得越来越不可靠，从而完善 rdt 协议）
- 只考虑单向数据传输（半双工）。双向数据传输（全双工）只是形式上复杂，并没有触及 RDT 的核心问题，暂时忽略；
- 控制信息是双向流动的，通过 `rdt` 函数交换控制分组，并且都要通过 `udt_send` 发送给对方。
- 双向的数据传输问题实际上是2个单向数据传输问题的综合（两个过程具有对称性）
- 使用有限状态机 (FSM) 来描述发送方和接收方（有限状态机实际上就是描述协议如何工作的一个形式化的描述方案，比语言更加简洁易懂、便于检查）
	- 状态：在该状态时，下一个状态只由下一个事件唯一确定。
	- 节点之间有个状态变迁的边 (edge)连在一起，代表状态 1 变成状态 2 在变迁的这条有限边上有标注。
	- 标注有分子和分母： $\frac{引起状态变化的事件}{状态变迁时采取的动作}$

下面进行渐进式开发： Rdt1.0 --> Rdt2.0 --> Rdt2.1 --> Rdt2.2 --> Rdt 3.0

#### RDT1.0

Rdt1.0：在可靠信道上的可靠数据传输
- 下层的信道是完全可靠的
    - 没有比特出错
    - 没有分组丢失
- 发送方和接收方的FSM
    - ![[30-Transport-layer-rdt10.png]]
    - 发送方和接收方有各自的 FSM ，每个 FSM 都只有一个状态，状态转移就是从自身通过中间态（发送分组和接收分组的动作）后回到自身
    - ==横线上方是表示引起变迁的事件，下方是事件发生时采取的动作==：
	    - The sending side of `rdt` simply accepts data from the upper layer via the `rdt_send(data)` event, creates a packet containing the data (via the action `make_pkt(data)`) and sends the packet into the channel. In practice, the `rdt_send(data)` event would result from a procedure call (for example, to `rdt_send()`) by the upper-layer application.
	    - On the receiving side, `rdt` receives a packet from the underlying channel via the `rdt_rcv(packet)` event, removes the data from the packet (via the action `extract(packet, data)`) and passes the data up to the upper layer (via the action `deliver_data(data)`). In practice, the `rdt_rcv(packet)` event would result from a procedure call (for example, to `rdt_rcv()`) from the lower-layer protocol.


#### RDT2.0
Rdt2.0：去掉一个假设，变为具有比特差错（如0和1的反转）的信道
- 下层信道可能会出错：
    - 分组中的比特可能会翻转，但仍假定分组顺序不变、分组不会丢失
- 怎样确认报文被接收的情况？
    - **确认(ACK，positive acknowledgement)**：接收方显式地告诉发送方分组已被正确接收（send ACK）
    - **否定确认(NAK, negative acknowledgement)**：接收方显式地告诉发送方分组发生了差错（send NAK）
        - 发送方收到 NAK 后，发送方重传分组（之前发送完之后需要保存一个副本）
- **自动重传请求协议** (Automatic Repear reQuest, ARQ)：指基于重传机制的可靠数据传输协议。ARQ 协议提供三种功能处理比特位差错：
    1) 差错检测：用校验和来检测比特流是否出现差错
    2) 接收方反馈：ACK 与 NAK 反馈分组，这些分组只需要一个头部和一个表示位
    3) 重传：受到 NAK 时，重发分组
- rdt2.0 的 FSM：
    - ![[30-Transport-layer-rdt20-fsm.png]]
    - **发送端有两个状态**：等待上层的调用和数据，或者等待接收方的 ACK/NAK 反馈分组，根据反馈报文的不同进行不同的动作——回到等待上层调用，或者重发上一个分组并继续等待反馈报文；
    - *注意:发送方处于等待 ACK/NAK 反馈时，不能接收上层的调用和数据，仅当接收到 ACK 反馈时才会离开等待反馈报文的状态，因为停下来等待，==rdt2.0 及类似行为的协议被称为停等协议==(stop-and-wait)*；
    - **接收端有一个状态**：分组到达时，根据校验和返回一个 ACK/NAK 反馈报文；
- rdt2.0中的新机制：采用差错控制编码进行差错检测
    - 发送方差错控制编码、缓存
    - 接收方使用编码检错
    - 接收方的反馈：控制报文（ACK，NAK）：接收方->发送方
    - 发送方收到反馈相应的动作（发送新的或者重发老的）

#### RDT2.1
Rdt2.0的致命缺陷！
- 如果 ACK/NAK 出错？
    - 发送方不知道接收方发生了什么事情！（既不是 ACK 也不是 NAK）
    - 发送方如何做？
        - 不顾一切地重传？可能重复
        - 不重传？可能死锁（发送方等 ACK，接收方等重发的分组，尬住了）
    - 需要引入新的机制
        - **序号**
- 处理重复：
    - 发送方在每个分组中加入序号
    - 如果 ACK/NAK 出错，发送方重传当前分组
    - 接收方丢弃（不发给上层）重复分组

- 发送方：
    - 在分组中加入序列号
    - 只需要一位即两个序列号（0，1）就足够了
        - 一次只发送一个未经确认的分组（注：若接收方等待的是1号分组，而传来0 号未出错的分组，则接收方传回 ack，与发送方调成同步）
    - 必须检测 ACK/NAK 本身是否出错（需要 EDC）
    - 状态数变成了两倍
        - 必须记住当前分组的序列号为0还是1
- 接收方：
    - 必须检测接收到的分组是否是重复的
        - 状态会指示希望接收到的分组的序号为 0 还是 1
    - 注意：**接收方并不知道发送方是否正确收到了其最后发送的 ACK/NAK**
        - 发送方不对收到的 ack/nak 给确认，没有所谓的确认的确认；
            - 两军悖论：![[30-Transport-layer-two-army-problem.png]]
        - 接收方发送 ack，如果后面接收方收到的是：
            - 老分组 p0？则 ack 错误
            - 下一个分组？P1，ack 正确

![[30-Transport-layer-two-general-problem.png]]
- rdt2.1 的 FSM：
	- ![[30-Transport-layer-rdt21-fsm.png]]
	- 注意到发送方和接收方的状态数都是之前的 2 倍，因为必须要反应出目前发送方正发送的分组或接收方希望收到的分组的序号是 0 还是 1；
	- 接收方在收到失序或正确的分组时，发送肯定确认 ACK；收到错误的分组时，发送否定确认 NAK；

#### RDT2.2
Rdt2.2：无NAK、只有ACK的协议（NAK free）
- 功能同 rdt2.1，但只使用 ACK（ACK 要编号）：
	- 如果收到错误分组（如分组 0），不发送 NAK0 而是发送上一个正确接收的分组的 ACK1，也能实现同样的效果——
	- 发送方收到上一个分组 1 的两个 ACK（这里称作冗余 duplicate ACK），就知道接收方没有正确接收到分组 1 后的分组 0
- 接收方对**最后正确接收的分组发 ACK**，以替代 NAK（==对当前分组的反向确认可由对前一项分组的正向确认代表==，如用 ACK0 代表 NAK1、用 ACK1 代表 NAK0 等等）
    - 接收方必须显式地包含被正确接收分组的序号
- 当收到重复的 ACK（如：再次收到 ACK0）时，发送方与收到 NAK1 采取相同的动作：重传当前分组 0
- NAK free 的意义：
    - 为后面的一次发送多个数据单位做一个准备
    - 一次能够发送多个
    - 每一个的应答都有：ACK，NAK；麻烦
    - 使用对前一个数据单位的ACK，代替本数据单位的nak
    - ==确认信息减少一半，协议处理简单==

- rdt2.2 的 FSM：
	- ![[30-Transport-layer-rdt22-fsm.png]]

#### RDT3.0
Rdt3.0：具有比特差错和分组丢失的信道
- 新的假设：下层信道可能会丢失分组（数据或 ACK 都可能丢失）
    - 会死锁（发送方等待确认，接收方等待分组）
    - Rdt2.2 的机制外，还需增加一种机制处理这种状况：
        - 检验和
        - 序列号
        - ACK
        - 重传
        - ？  ———————— 定时！
- 方法：发送方等待 ACK 一段合理的时间（链路层的 timeout 时间是确定的，传输层 timeout 时间是适应式的（需要**动态地计算**））
    - 发送端**超时重传**：如果到时没有收到ACK->重传
    - 问题：如果分组（或ACK）只是被延迟了：
        - 重传将会导致数据重复（冗余分组），但利用序列号已经可以处理这个问题
        - 接收方必须指明被正确接收的序列号
    - 需要一个倒计数定时器 countdown timer

发送方的步骤：
1) 每次发送一个分组（包括第一次分组和重传分组），都启动一个定时器
2) 响应定时器中断，采取适当的动作——重传
3) 收到 ACK 时，及时终止定时器

- rdt3.0 的 FSM：
	- ![[30-Transport-layer-rdt30-fsm.png]]
	- The sender side of protocol rdt3.0 differs from the sender side of protocol 2.2 in that timeouts have been added. We have seen that the introduction of timeouts adds the possibility of duplicate packets into the sender-to-receiver data stream. However, the receiver in protocol rdt.2.2 can already handle duplicate packets. (Receiver-side duplicates in rdt 2.2 would arise if the receiver sent an ACK that was lost, and the sender then retransmitted the old data). ==Hence the receiver in protocol rdt2.2 will also work as the receiver in protocol rdt 3.0==.

- rdt3.0 的运行情况：
	- ![[30-Transport-layer-rdt30-work-flow.png]]
	- 在图 d 中可见：过早超时（延迟的 ACK）也能够正常工作；但是效率较低，一半的分组和确认是重复的；因此，设置一个**合理的超时时间**也是比较重要的

Rdt3.0的性能
- rdt3.0 停等协议可以工作，但链路容量比较大的情况下（分组全部放完时，分组的第一个比特离接收方还很远），性能很差
    - 链路容量比较大，一次发一个 PDU 的不能够充分利用链路的传输能力（信道明明可容纳很多很多包，每次却只有一个包处于信道中，信道利用率极低）
    - 改进办法：[[#流水线 RDT 协议|使用流水线]]！

> [!example] 例：rdt3.0 的低效之处
> $1Gbps$ 的链路， $15ms$ 端-端传播延时（ $RTT = 30ms$ ），分组大小为 $1kB = 1000Bytes = 8000bits$ ：   
> $$T_{transmit} = \frac{L(分组长度, 比特)}{R(传输速率, bps)} = \frac{8kb/pkt}{10^9 b/sec} = 8\mu{s}$$     
> $$U_{sender} = \frac{L/R}{RTT+L/R} = \frac{{0.008}}{30.008} = 0.00027$$
> - $U_{sender}$ ：利用率 – 忙于发送的时间比例
> - 每 $30ms$ 发送 $1KB$ 的分组 --> $270kbps=33.75kB/s$ 的吞吐量（在 $1Gbps$ 链路上）
> - 瓶颈在于：网络协议限制了物理资源的利用！

#### 经验
在构造 rdt1.0 -> rdt3.0 的过程中，我们学习到，为了在不可靠通道中能够提供可靠数据传输，需要满足以下**五个条件**：
1. 检错、纠错机制——校验和
2. 序号——保证顺序、避免丢失和重复
3. 定时器——避免丢失
4. ACK 和 NAK——表明发送是否成功
5. 重传——万能手段，可以恢复失败的发送

### 流水线 RDT 协议
#### 思路——提高 rdt3.0 的信道利用率

如何提高链路利用率？流水线(pipeline)

![[30-Transport-layer-pipeline-rdt.png]]
- 发送报文，不要停等，无需逐个确认！

![[30-Transport-layer-pipeline-rdt-n.png]]
- 增加 $n$ （如这里从 $n=1$ 变为 $n=3$ ），能提高链路利用率，差不多就是提升了 3 倍；
- 但当达到某个 $n$ ，其 $u=100\%$ 时，无法再通过增加 $n$ ，提高利用率
- 瓶颈由协议转移到 --> 链路带宽（此时可将 $1Gbps$ 链路升级为 $10Gbps$ 链路）

流水线协议（流水线：允许发送方在未得到对方确认的情况下一次发送多个分组）
- ==必须增加序号的范围==：用多个 bit 表示分组的序号（若分组的序号用 $N$ 个比特表示，则整个分组的空间占用是 $2^N$ ）
- 在发送方/接收方要有缓冲区
    - 发送方缓冲：未得到确认，可能需要重传；
    - 接收方缓冲：
	    - 上层用户取用数据的速率 $\neq$ 接收到的数据速率，需要缓冲来对抗速率的差异性；
	    - 接收到的数据可能乱序，排序交付（可靠）
- 两种通用的流水线协议：**回退N步(GBN)** 和 **选择重传(SR)**

#### 滑动窗口

为讲解 GBN 和 SR 协议的差别，先引入**滑动窗口(sliding window)**：

| sending window | receiving window | protocol form |
|:--------------:|:----------------:|:-------------:|
|      = 1       |       = 1        |   stop-wait   |
|      > 1       |       = 1        |      GBN      |
|      > 1       |       > 1        |      SR       |

其中 sw(sending window) > 1 时就是流水线协议。

![[30-Transport-layer-sliding-window-protocol.png]]

几个概念：
- **发送缓冲区**
    - 形式：内存中的一个区域，落入缓冲区的分组可以进行检错重发、超时重发
    - 功能：==用于存放已发送，但是没有得到确认的分组==
    - 必要性：需要重发时可用
- 发送缓冲区的大小：一次最多可以发送多少个未经确认的分组
    - 停止等待协议=1
    - 流水线协议>1，合理的值，不能很大，避免链路拥塞
- 发送缓冲区中的分组
    - 未发送的：落入发送缓冲区的分组，可以连续发送出去；
    - 已经发送出去的、等待对方确认的分组：发送缓冲区的分组只有得到确认才能删除

- **发送窗口**：发送缓冲区内容的一个子集
    - ==存放那些已发送但是未经确认分组==的序号构成的空间
    - 发送窗口的最大值 <= 发送缓冲区的值
    - 一开始：没有发送任何一个分组
        - 后沿 = 前沿
        - 之间为发送窗口的尺寸 = 0
    - 发送窗口的移动——前沿移动
        - 每发送一个分组，前沿前移一个单位
        - 发送窗口前沿移动的极限：（前沿和后沿的距离）不能够超过发送缓冲区
    - 发送窗口的移动——后沿移动
        - 条件：收到老分组的确认
        - 结果：发送窗口缩小，等待接下来的 ACK 
        - 移动的极限：不能够超过前沿
    - 发送窗口滑动过程——相对表示方法（只是为了方便理解）
        - 采用相对移动方式表示，分组不动，窗口向前滑动（实际上真正的滑动过程为窗口不动，分组向前滑动）
        - 可缓冲范围移动，代表一段可以发送的权力

- **接收窗口**(receiving window)：
    - ==接收窗口用于控制哪些分组可以接收==；
        - 只有收到的分组序号**落入接收窗口内才允许接收**
        - 若序号在接收窗口之外，则丢弃；
    - 接收窗口尺寸 $W_{rcv}=1$，则只能顺序接收；
        - 例如：$W_{rcv}＝1$，在 0 的位置；只有 0 号分组可以接收；向前滑动一个，罩在 1 的位置，此时如果来了第 2 号分组，则第 2 号分组为乱序分组，则丢弃，并给出确认——对顺序到来的最高序号的分组给确认——发送 ACK0
    - 接收窗口尺寸 $W_{rcv}>1$，则可以乱序接收（但提交给上层的分组，要按序）
        - 例如：$W_{rcv}=4>1$ 时，此时到来的分组是乱序但是落在接收缓冲区的范围之内，则也进行接收并发出确认，==直到该分组之前的分组都接收到了，接收窗口才进行滑动==，如下图中收到 1，2 分组先不滑动，收到 0 分组后，2 分组前的分组全部收到，此时进行滑动到 3 号分组开始；若第一次接收到的是 0 号分组，则接收缓冲区立即向后滑动到 1 开始）：
        - ![[30-Transport-layer-receiving-window.png]]
    - 接收窗口的滑动和发送确认
        - 滑动：
            - 低序号的分组到来，接收窗口移动；
            - 高序号分组乱序到，缓存但不交付（因为要实现 rdt，不允许失序），不滑动
        - 发送确认：
            - ==接收窗口尺寸=1：发送连续收到的最大的分组确认==（累计确认：这之前的分组都正确收到了）
            - ==接收窗口尺寸>1：收到分组，发送那个分组的确认==（单独确认：只代表正确收到了这个分组）

#### 发送窗口-接收窗口的互动
- 正常情况下的窗口互动
    - 发送窗口
        - 有新的分组落入发送缓冲区范围，发送->前沿滑动
        - 来了老的低序号分组的确认->后沿向前滑动->新的分组可以落入发送缓冲区的范围
    - 接收窗口
        - 收到分组，落入到接收窗口范围内，接收
        - 是低序号，发送确认给对方
    - 发送端上面来了分组->发送窗口滑动->接收窗口滑动->发确认

- 异常情况下 GBN(wr=1)的窗口互动
    - 发送窗口
        - 新分组落入发送缓冲区范围，发送->前沿滑动
        - 超时重发机制让发送端将发送窗口中的所有分组发送出去
        - 来了老分组的重复确认->后沿不向前滑动->新的分组无法落入发送缓冲区的范围（此时如果发送缓冲区有新的分组可以发送）
    - 接收窗口
        - 收到乱序分组，没有落入到接收窗口范围内，抛弃
        - （重复）发送老分组的确认，累计确认

- 异常情况下SR(wr>1)的窗口互动
    - 发送窗口
        - 新分组落入发送缓冲区范围，发送->前沿滑动
        - 超时重发机制让发送端将超时的分组重新发送出去（==不像GBN需要将发送窗口中的所有分组全部重发==）
        - 来了乱序分组的确认->后沿不向前滑动->新的分组无法落入发送缓冲区的范围（此时如果发送缓冲区有新的分组可以发送）
    - 接收窗口
        - 收到乱序分组，落入到接收窗口范围内，接收
        - 发送该分组的确认，单独确认

#### Go-Back-N
- GBN 协议的窗口：
	-  ![[30-Transport-layer-GBN-sender-seq.png]]
	- 基序号 `base`：最早未确认的分组的序号
	- 下一个序号 `nextseqnum`：下一个待发分组的序号
	- `[0,base)`：已发送并确认的分组
	- `[base,nextseqnum)`：已发送待确认的分组
	- `[nextseqnum,base+N)`：窗口内待发送的分组。N 是窗口长度
		- 为什么要限制窗口长度 N？——流量控制
	- `[base+N,buffer_max)`：缓存区中、窗口外的不可发送分组。

- 序号：
	- 由于 GBN 协议窗口大小为 N，因此需要 $\log_{2}N$ 个比特作序号标记分组，该序号存储在分组首部的固定长度字段中；
	- 序号计算使用模 $2^{\log_{2}N}$ 运算，即序号空间中 $2^{\log_{2}N}-1$ 的下一位是 0；

- GBM 的 FSM：
	- ![[30-Transport-layer-GBN-FSM.png]]
	- GBN 发送方必须响应三类事件：
		1) 上层调用：调用 `rdt_send()` 前，发送方检查发送窗口是否已满，未满则可发送，已满则缓存数据、或使用同步机制告知上层（生产者消费者模型）
		2) 收到一个 ACK：GBN 使用累积确认的方式，表明 ACK-n 之前的分组都已收到；
		3) 超时事件：一旦出现超时，重传所有已发送但未确认的分组。上图中仅使用一个定时器，作为最早已发送但未确认的分组的定时器，收到后续 ACK 却仍有未确认分组，定时器就重置，否则停止；
	- GBN 接收方动作：
		- 序号为 n 的分组被接收，且之前的分组业已接收并交付给上层，则发送 ACK-n 并交付；
		- 其它情况则直接丢弃分组 n，并为最近按序接收的分组重传对应 ACK；
	- 优点：==接收方不必缓存任何失序分组==，简单实现；
	- 缺点：丢弃正确的分组导致低效

- GBK 的运行情况：
	- ![[30-Transport-layer-GBN-operation.png]]
	- 由于窗口大小的限制，发送方发送数据包 0 至 3 后，必须等待其中一个或多个数据包得到确认后才能继续发送。当收到每个连续的 ACK（例如 ACK0 和 ACK1）时，窗口向前滑动，发送方可以发送一个新数据包（分别为 pkt4 和 pkt5）。在接收端，数据包 2 丢失，因此数据包 3、4 和 5 被认为不符合顺序而丢弃。
	- 正常情况：
		- ![[30-Transport-layer-GBN.gif]]
	- 发送中丢包：
		- ![[30-Transport-layer-GBN-send-loss.gif]]
		- 当发送过程中的 2 号包发生丢失时，发送方没有收到接收方的 ACK2，于是后面发送的 ACK3, ACK4 全部变成了 ACK1，代表接收方因为丢失了分组 2，所以分组 3 和分组 4 都被丢弃。所以全部返回 ACK1，经过一段时间后，定时器确认超时没有收到 ACK3, ACK4，所以发送方将重新发送。也代表接收方首先只收到了分组 1 及之前的包。
	- 接收中丢包：
		- ![[30-Transport-layer-GBN-receive-loss.gif]]
		- 在此例中，我们可以看到，当接收方接收完消息以后，返回给发送方 ACK0, ACK1, ACK2, ACK3, ACK4，我们假设 ACK2 发生了丢失。根据上一例我们可以知道，如果接收方没有收到分组 2，则后面返回的都是 ACK1，因为本次返回的为 ACK3, ACK4，所以发送方可以判断接收方已经接收到消息，不再进行重复发送。

#### Selective Repeat

GBN 的缺点：在信道差错率较高时，一旦一个分组丢失将引起大量分组重传，最终导致信道中充斥着重传的分组，利用率极低。

- SR 协议的窗口：
	- ![[30-Transport-layer-SR-sender-windows.png]]
	- 发送方仅重传它怀疑未接收到的分组，从而避免不必要的重传；
	- 发送方窗口中可以离散地分布着发送且确认的分组，这一点与 GBN 不同；
	- 接收方确认每个正确接收的分组，无论其是否按序，失序的分组将被缓存直到所有丢失分组都被接收到为止，此时再将一批分组交付给上层应用；
	- ![[30-Transport-layer-SR-sender-receivers-actions.png]]
	- 值得注意的是，在图 3.25 的步骤 2 中，接收方会**重新确认**（而不是忽略）已收到的序列号低于当前窗口基数的数据包。以图 3.23 中的发送方和接收方序列号空间为例，==如果没有从接收方传播到发送方的数据包 send_base 的 ACK，发送方最终将重新发送数据包 send_base，尽管接收方已经收到了该数据包==。如果接收方不确认这个数据包，发送方的窗口就永远不会向前移动！
	- 发送方和接收方对哪些数据包已正确接收，哪些未正确接收的看法并不总是一致的。对于 SR 协议来说，这意味着发送方和接收方的窗口可能不同，这引入了**同步问题**。

>[!important] SR 协议的发送方和接收方窗口同步问题
> ![[30-Transport-layer-SR-sync.png]]
>The lack of synchronization between sender and receiver windows has important consequences when we are faced with the reality of a finite range of sequence numbers. Consider what could happen, for example, with a finite range of four packet sequence numbers, 0, 1, 2, 3, and a window size of three. 
>> 在发送方窗口和接收方窗口处若缺少同步，则会由于有限的序列号的范围而造成严重后果，例如，考虑一个四分组序列的序号 0、1、2、3 以及一个大小为 3 的窗口。
>
>Suppose packets 0 through 2 are transmitted and correctly received and acknowledged at the receiver. At this point, the receiver’s window is over the fourth, fifth, and sixth packets, which have sequence numbers 3, 0, and 1, respectively. Now consider two scenarios. 
>- In the first scenario, shown in Figure 3.27 (a), the ACKs for the first three packets are lost and the sender retransmits these packets. The receiver thus next receives a packet with sequence number 0—a copy of the first packet sent. 
>- In the second scenario, shown in Figure 3.27 (b), the ACKs for the first three packets are all delivered correctly. The sender thus moves its window forward and sends the fourth, fifth, and sixth packets, with sequence numbers 3, 0, and 1, respectively. The packet with sequence number 3 is lost, but the packet with sequence number 0 arrives—a packet containing new data. 
>> 如果分组 0~2 正确地传输并接收，接收方也正常地通知发送方，此时，接收方的窗口会到达第 4、5、6 个分组，它们的序列号各自为 3、0、1，现在考虑以下情景：
>> - 在图 3.27-a 中，最初三个（即第 0、1、2 分组）的 ACK 帧丢失，而发送方超时重传这三个分组，接收方于是就会接收重传中的 0 号分组——它只是最初三个分组中的一个拷贝，是错误的发送分组；
>> - 在图 3.27-b 中，最初三个分组的 ACK 正确传达发送方，发送方于是移动它的发送方窗口并且发送第 4、5、6 个分组，各自序列号是 3、0、1，若 3 号分组在传输中丢失，但是 0 号分组可以正确抵达。
>
>Now consider the receiver’s viewpoint in Figure 3.27, which has a figurative curtain between the sender and the receiver, since the receiver cannot “see” the actions taken by the sender. All the receiver observes is the sequence of messages it receives from the channel and sends into the channel. As far as it is concerned, the two scenarios in Figure 3.27 are identical. ==**There is no way of distinguishing the retransmission of the first packet from an original transmission of the fifth packet**==. 
>> 现在考虑接收方的视角，假想地可以看作接收方与发送方之间有一块幕布，它们无法直接看到互相的动作，所有接收方能够观察到的结果都来自于从通道中接收或发送的消息。因此，图 3.27 中的两种情况在接收方看来却是同一种，它无法区分接收到的 0 号分组是来自于重传的第一个分组，还是来自第五个分组。
>
>Clearly, a window size that is 1 less than the size of the sequence number space won’t work. But how small must the window size be? A problem at the end of the chapter asks you to show that the ==**window size must be less than or equal to half the size of the sequence number space for SR protocols**==.
>> 实际上，SR的窗口大小必须小于等于序号空间大小的一半（发送窗口和接收窗口都是如此）。

- SR 的运行情况：
	- ![[30-Transport-layer-SR-operation.png]]
	- 分组 2 丢失，但是收到分组 3、4、5 时，接收方会缓存它们，直到分组 2 到达再一起交付；
	- 正常情况：
		- ![[30-Transport-layer-SR.gif]]
	- 发送中丢包：
		- ![[30-Transport-layer-SR-send-loss.gif]]
		- 我们同样将分组 2 丢失，但可以看到与 GNB 不同的是，接收方在没有收到分组 2 的情况下，依然返回了 ACK3, ACK4。此时窗口已经由之前的 0-4 变成了 2-6。当所以 ACK1 返回以后，分组 5，分组 6 就已经可以发送。然后在接收方，分组 3456 都已经被缓存，等待分组 2 的计时器超时后，分组 2 将重新发送，然后在接收方的分组 23456 全部变为接收 received 状态。
	- 接收中丢包：
		- ![[30-Transport-layer-SR-receive-loss.gif]]
		- 我们同样将ACK2丢弃，此时发送方的分组已经由0-4到了2-6,在最后2-6的窗口中，分组2会因为ACK2被丢失然后在计时器计时后重新发送一次。==可以看到如果在接收过程中有丢失发生，选择重传SR的效率是不如回退N步GBN的。==

#### GBN协议和SR协议的异同
- 相同之处
    - 发送窗口>1
    - 一次能够可发送多个未经确认的分组
- 不同之处
    - GBN：接收窗口尺寸=1
        - 接收端：只能顺序接收（对顺序接收的最高序号的分组进行确认-累计确认）
            - 对乱序的分组：
                - 丢弃（不缓存）
                - 在接收方不被缓存！
        - 发送端：从表现来看，一旦一个分组没有发成功，如：0,1,2,3,4，假如1未成功，234都发送出去了，要返回1再发送；GB1
            - 只发送ACK：对顺序接收的最高序号的分组
                - 可能会产生重复的ACK
                - 只需记住expectedseqnum；接收窗口=1
                    - 只一个变量就可表示接收窗口
    - SR：接收窗口尺寸>1
        - 接收端：可以乱序接收
        - 发送端：发送0,1,2,3,4，一旦1未成功，2,3,4,已发送，无需重发，选择性发送1

#### 对比 GBN 和 SR 

|      |                  GBN                   |                    SR                    |
|:----:|:--------------------------------------:|:----------------------------------------:|
| 优点 | 简单，所需资源少（接收方一个缓存单元） |          出错时，重传一个代价小          |
| 缺点 |        一旦出错，回退N步代价大         | 复杂，所需要资源多（接收方多个缓存单元） |
- 适用范围
- 出错率低：比较适合GBN，出错非常罕见，没有必要用复杂的SR，为罕见的事件做日常的准备和复杂处理
- 链路容量大（延迟大、带宽大）：比较适合SR而不是GBN，一点出错代价太大

窗口的最大尺寸（若整个的序号空间是 $2^n$ ）
- GBN： $2^n-1$ ，如 $n=2$ 时 $GBN=3$
- SR： $2^{n-1}$ ，如 $n=2$ 时 $SR=2$
    - SR的例子：
        - 接收方看不到二者的区别！
        - 将重复数据误认为新数据

## 3.5 面向连接的传输：TCP

TCP relies on many of the underlying principles discussed in the previous section, including error detection, retransmissions, cumulative acknowledgments, timers, and header fields for sequence and acknowledgment numbers. TCP is defined in RFC 793, RFC 1122, RFC 2018, RFC 5681, and RFC 7323.

### TCP 连接的特点与建立

- **Connection-oriented**: 面向连接
	- TCP is said to be connection-oriented because ==before one application process can begin to send data to another, the two processes must first “handshake” with each other==—that is, they must send some preliminary segments to each other to establish the parameters of the ensuing data transfer. 
	- As part of TCP connection establishment, both sides of the connection will initialize many TCP state variables associated with the TCP connection.

> TCP 是面向连接的协议，在一个进程能够与另一个远程进程发送数据之前，必须首先握手（传输建立提供数据传输的连接的相关参数），连接的两端都会在此过程中初始化许多与 TCP 连接相关的 TCP 状态变量。

- **Logical connection**: 逻辑连接而非物理连接
	- The TCP “connection” is not an end-to-end TDM or FDM circuit as in a circuit switched network. Instead, the “connection” is a ==logical one, with common state residing only in the TCPs in the two communicating end systems==. 
	- Recall that because the ==TCP protocol runs only in the end systems== and not in the intermediate network elements (routers and link-layer switches), the intermediate network elements do not maintain TCP connection state. In fact, ==the intermediate routers are completely oblivious to TCP connections==; they see datagrams, not connections.

> TCP 连接不是物理交换链路中的端到端连接，而是逻辑的、只有两个交流中的端系统的 TCP 协议能够得知的连接——TCP 协议只在端系统上运行，中间网络设备（路由器、链路层交换机等）不会维护 TCP 连接的状态，它们对于 TCP 连接来说是完全透明的。

- **Full-duplex service**: 全双工的
	- A TCP connection provides a full-duplex service: If there is a TCP connection between Process A on one host and Process B on another host, then application-layer data can flow from Process A to Process B at the same time as application-layer data flows from Process B to Process A.

> TCP 连接是全双工的虚拟链接，两端的应用数据都可以同时发送。

- **Point-to-point**: 点到点、端到端
	- A TCP connection is also always point-to-point, that is, between a single sender and a single receiver. 
	- So-called “multicasting” —the transfer of data from one sender to many receivers in a single send operation—is not possible with TCP. With TCP, two hosts are company and three are a crowd!

> TCP 连接中的发送方和接收方是单对单的，并不能实现数据的==多播==。

- 如何建立 TCP 连接？——三次握手
	- Suppose a process running in one host wants to initiate a connection with another process in another host. Recall that ==the process that is initiating the connection is called the client process==, while the other process is called the server process. 
	- The client application process first informs the client transport layer that it wants to establish a connection to a process in the server. By issuing the command `clientSocket.connect((serverName,serverPort))` in Python, TCP in the client then proceeds to establish a TCP connection with TCP in the server. 
	- ==Three-way handshake==: The client first sends a special TCP segment; the server responds with a second special TCP segment; and finally the client responds again with a third special segment. 
		- The first two segments carry no payload (不承载有效载荷), that is, no application-layer data; 
		- the third of these segments may carry a payload. 
		- Because three segments are sent between the two hosts, this connection-establishment procedure is often referred to as a three-way handshake.

> - TCP 连接中想要建立连接的一方是客户端，另一方则是服务端；
> - 三次握手：
> 	- 客户端首先发送特殊的 TCP 段（SYN）以请求建立连接，
> 	- 服务端收到 SYN 请求后回送特殊的 TCP 段以相应（SYNACK），
> 	- 最终客户端在收到 SYNACK 后发送第三个特殊的 TCP 段，
> - 最初的两个 TCP 段没有有效载荷，第三个 TCP 段可能携带请求服务端数据的载荷。

- Send data: After connection established
	- ![[30-Transport-layer-TCP-send-receive-buffers.png]]
	- The client process passes a stream of data through the socket. Once the data is put into the socket, the data is in the hands of TCP running in the client. 客户端通过 socket 传递传输流，一旦数据被放入 socket 中，就意味着客户端上运行的 TCP 将会传递此数据给服务端；
	- As shown in Figure 3.28, ==TCP directs this data to the connection’s **send buffer==**, which is one of the buffers that is set aside during the initial three-way handshake. From time to time, TCP will grab chunks of data from the send buffer and pass the data to the network layer. Interestingly, the TCP specification `[RFC 793]` is not specifying when TCP should actually send buffered data, stating that TCP should “send that data in segments at its own convenience.” (TCP 会先将数据放到连接的发送方缓冲区，并不断地从发送方缓冲区中获取数据并传递给网络层；TCP 不限定何时发送数据，而是应当在它方便的时候以报文段的形式发送数据)
	- The maximum amount of data that can be grabbed and placed in a segment is limited by the **maximum segment size** (MSS). The MSS is typically set by first determining the length of the largest link-layer frame that can be sent by the local sending host (the so-called maximum transmission unit, MTU), and then ==setting the MSS to ensure that a TCP segment (when encapsulated in an IP datagram) plus the TCP/IP header length (typically 40 bytes) will fit into a single link-layer frame==.(能被 TCP 抓取并包装在一个段中的最大数据量由 MSS 限制，MSS 意为最大段长，通常由最大链路层帧的长度设定——MTU，本地发送方的最大传输单元。MSS 的设置需要保证： TCP 报文段+TCP/IP 首部长度≤链路层帧的长度)
		- Both Ethernet and PPP link-layer protocols have an MTU of 1,500 bytes. Thus, a typical value of MSS is 1460 bytes. （以太网和 PPP 链路层协议的 MTU 都是1500字节，因此典型的 MSS 值为 1460字节）
		- Note that the ==MSS is the maximum amount of application-layer data in the segment, not the maximum size of the TCP segment including headers==.（**MSS 是段中能够包含的应用层数据的最大数量，而不是 TCP 段+TCP头的最大长度**）
		- When TCP sends a large file, such as an image as part of a Web page, it typically ==breaks the file into chunks of size MSS== (except for the last chunk, which will often be less than the MSS). Interactive applications, however, often transmit data chunks that are smaller than the MSS; for example, with remote login applications such as ==Telnet and ssh, the data field in the TCP segment is often only one byte==. Because the TCP header is typically 20 bytes (12 bytes more than the UDP header), segments sent by Telnet and ssh may be only 21 bytes in length. ==当 TCP 要发送一个大文件时，通常会将其分解成多个大小为MSS的分组（通常最后一个分片小于MSS）==

- TCP connection composition:
	- A TCP connection consists of buffers, variables, and a socket connection to a process in one host, and another set of buffers, variables, and a socket connection to a process in another host.
	- As mentioned earlier, ==no buffers or variables are allocated to the connection in the network elements== (routers, switches, and repeaters) between the hosts.

> TCP 连接包含缓冲区、变量、连接到主机上一个进程的 socket；在连接中没有为网络中的中间设备设置缓冲区或变量（指的是抽象层次上，或 TCP 的视角中，实际上如路由器等中间设备当然具有缓冲区）

- 可靠的、按顺序的字节流：
    - 没有报文边界，报文界限靠应用进程自己区分
- 管道化（流水线）：
    - TCP 拥塞控制和流量控制设置窗口大小
- 有流量控制：
    - 发送方不会淹没接收方

### 段结构

![[30-Transport-layer-TCP-segment-structure.png]]
- As with UDP, the header includes **source and destination port numbers**, which are used for multiplexing/demultiplexing data from/to upper-layer applications. 
- Also, as with UDP, the header includes a checksum field.
> 类似 UDP，TCP 也有源和目的端口号，也包含校验和域。

- The 32-bit **sequence number field** and the 32-bit **acknowledgment number field** are used by the TCP sender and receiver in implementing a reliable data transfer service. 
> 注：这里第二行的“序号”不是前面讲的 PDU 的序号（不是分组号），而是==对字节计数的序号——body 部分的第一个字节在整个字节流中的偏移量 offset==（第 $i$ 个 MSS 的第一个字节在字节流中的位置，初始的序号称为 $X$ ， $X$ 在建立连接时两个进程商量好，第 $n$ 个的序号为 $X+n*MSS$ ）

- The 16-bit **receive window field** is used for flow control. It is used to indicate the number of bytes that a receiver is willing to accept.
> ==16 位的接收窗口域，用于流量控制——告知接收方能够接收的字节数==。

- The 4-bit **header length field** specifies the length of the TCP header in 32-bit words. The TCP header can be of variable length due to the TCP options field. (Typically, the options field is empty, so that the length of the ==typical TCP header is 20 bytes==.)
> ==4 位的头部长度域：只是 TCP 首部的长度，单位是 32 位的字==。
> TCP 首部可以是变长的，因为 options 域不定，但若不添加 options，则通常是 20 字节。

- The **optional and variable-length options field** is used when a sender and receiver negotiate the maximum segment size (MSS) or as a window scaling factor for use in high-speed networks. A time-stamping option is also defined. See RFC 854 and RFC 1323 for additional details. 
> 可选的变长域是用于当发送方和接收方协商 MSS 或在高速网络中使用的窗口伸缩因子时。
> 时间戳 option 也在此定义。

- The **flag field** contains 6 bits. ==标记域有 6 比特==：
	- The ACK bit is used to indicate that the value carried in the acknowledgment field is valid; that is, the segment contains an acknowledgment for a segment that has been successfully received. ==ACK 位用于指示确认域的值是合法的，即该段包含着对之前成功接收的 TCP 段的确认==。
	- The RST, SYN, and FIN bits are used for connection setup and teardown. ==SYN、RST、FIN 三个位用于 TCP 连接的建立、重置、拆除==。
	- The CWR and ECE bits are used in explicit congestion notification. ==CWR 和 ECE 位用于显式拥塞通知==。 
	- Setting the PSH bit indicates that the receiver should pass the data to the upper layer immediately. ==PSH 位指示接收方应当将数据立即传递给上层协议==。
	- Finally, the URG bit is used to indicate that there is data in this segment that the sending-side upperlayer entity has marked as “urgent.” The location of the last byte of this urgent data is indicated by the 16-bit urgent data pointer field. TCP must inform the receiving-side upper-layer entity when urgent data exists and pass it a pointer to the end of the urgent data. (In practice, the PSH, URG, and the urgent data pointer are not used. However, we mention these fields for completeness.) ==URG 位指示该段中有部分数据被发送方的上层协议实体标记为“紧迫的”。这个“紧迫”数据的最后一个字节的位置由 16位的紧迫数据指针域指示==。TCP 必须在紧迫数据存在时，通知接收方的上层协议实体并传递其指针给紧迫数据的末端。

#### TCP序号，确认号
- 序号(sequence number)：报文段首字节在字节流中的编号
    - ![[30-Transport-layer-TCP-segments-seq-num.png]]
    - Suppose that a process in Host A wants to send a stream of data to a process in Host B over a TCP connection. ==The TCP in Host A will implicitly number each byte in the data stream==. 主机 A 中的 TCP 会隐式地为数据流中的每个字节做序号标记。
    - Suppose that the data stream consists of a file consisting of 500,000 bytes, that the MSS is 1,000 bytes, and that the first byte of the data stream is numbered 0. As shown in Figure 3.30, TCP constructs 500 segments out of the data stream. The first segment gets assigned sequence number 0, the second segment gets assigned sequence number 1,000, the third segment gets assigned sequence number 2,000, and so on. 假设一个数据流有 500,000字节，而 MSS 为1000字节，数据流的第一个字节的序号为 0。像图 3.30中展示的那样，TCP 构造了500个段以存放数据流，第一个段被分配的序列号为 0，第二个为 1000，第三个为 2000，以此类推。
    - Each sequence number is inserted in the sequence number field in the header of the appropriate TCP segment. ==每个序列号都插入在 TCP 段首部的序列号域中==。

- 确认号(ack number)：**期望从另一方收到的下一个字节的序号**
    - TCP 是全双工的，因此发送方和接收方的角色可能互换，因此==设置此“确认号域”提示对方要发什么数据==；
    - 举例：Suppose that Host A has received all bytes numbered 0 through 535 from B and suppose that it is about to send a segment to Host B. Host A is waiting for byte 536 and all the subsequent bytes in Host B’s data stream. So Host A puts 536 in the acknowledgment number field of the segment it sends to B. 考虑这样的场景：主机 A 收到了序号0~535的来自主机 B 的所有段，并且希望发送一个段告知主机 B——它正在等待 B 的数据流中第 536个字节及之后的数据，因此 A 发送的确认段的确认号为536。
    - 举例：Suppose that Host A has received one segment from Host B containing bytes 0 through 535 and another segment containing bytes 900 through 1,000. For some reason Host A has not yet received bytes 536 through 899. In this example, ==Host A is still waiting for byte 536 (and beyond) in order to re-create B’s data stream==. Thus, A’s next segment to B will contain 536 in the acknowledgment number field. Because TCP only acknowledges bytes up to the first missing byte in the stream, TCP is said to provide **cumulative acknowledgments**. 考虑这样的场景：主机 A 从主机 B 处接收到两个 TCP 段，分别是 0~535字节和 900~1000字节的两个段，而中间 536~899字节的数据丢失，此时 A 仍在期待收到536字节及之后的内容，于是 A 向 B 发送的下一个 TCP 段（注意不一定是确认段，因为完全可以捎带地确认）中确认号是 536。==TCP 只会确认最后一个收到的字节，因此称 TCP 为累积确认==。
    - 举例：The TCP RFCs do not impose any rules here and leave the decision up to the programmers implementing a TCP implementation. There are basically two choices: either (1) the receiver immediately discards out-of-order segments (which, as we discussed earlier, can simplify receiver design), or (2) the receiver keeps the out-of-order bytes and waits for the missing bytes to fill in the gaps. Clearly, the latter choice is more efficient in terms of network bandwidth, and is the approach taken in practice. RFC 中没有对 TCP 中失序段的处理方式的规定，这取决于实现者自己的抉择——通常有两种：一是接收方立即丢弃失序段；二是接收方保留失序段，并等待丢失字节完成填充。第二种方法对于网络带宽的利用更加高效。


> [!example] Telnet: 序号与确认号的使用案例
> 
> ![[30-Transport-layer-telnet-seq-ack-num.png]]
> A 向 B 发起 Telnet 回话，A 作客户，B 为服务器。客户端 A 输入的每个字符都被发送到远程主机 B 上，B 会将该字符回显到客户屏幕，从而确保远程主机 B 确实接收到用户的数据，即用户 A 输入的字符被传输了两次：
> 
> 1. A(User types 'C') --> B(host ACKs receipt of 'C', echoes back 'C'): Seq = $42$ , ACK = $79$ , data = 'C';
> 2. B(host ACKs receipt of 'C', echoes back 'C') --> A(host ACKs receipt of echoed 'C'): Seq = $79$ , ACK = $43$ , data = 'C';
> 3. A(host ACKs receipt of echoed 'C') --> B: Seq = $43$ , ACK = $80$ , data = ... 
> 4. ...
> 
> 需要注意的是：第二个报文段由服务器回送给客户，有两个作用
> - 为该服务器所收到的数据提供确认。ACK=43 即代表 42 及之前的数据都已收到，同时回送的数据'C'也验证了确实收到 Seq=42 的报文；
> - 注意到第二个报文段的 Seq 为 79，这是服务器向客户端发送的第一个报文的序号；对客户到服务器的报文的确认存放在服务器到客户的报文段中，称之为捎带 piggybacked。
> 
>另外：Telnet 协议逐渐被 SSH 替代，是因为 Telnet 传送数据时不加密。

### TCP往返延时(RTT)和超时
- Q：怎样设置TCP超时定时器？
    - 比 RTT 要长，但 RTT 是变化的——取决于信道拥塞和端系统负载变化
    - 太短：太早超时，引起不必要的重传
    - 太长：对报文段丢失反应太慢，过于消极
- Q：怎样估计 RTT？逐个报文检查 RTT 开销过大不现实
    - SampleRTT：测量从报文段发出到收到确认的时间，得到往返延时
        - 仅为已发送但尚未确认的报文段估计 SampleRTT
        - 如果有重传，忽略此次测量，仅为传输一次的报文段测量SampleRTT
    - SampleRTT 会变化，
        - Because of this fluctuation, any given SampleRTT value may be atypical. In order to estimate a typical RTT, it is therefore natural to take some sort of average of the SampleRTT values.
        - ==TCP maintains an average, called **EstimatedRTT**, of the SampleRTT values==. TCP 通过维护一个称为 `估计 RTT` 的平均值作为 `采样 RTT` 值。

- EstimatedRTT（滤波算法）的计算方式
    - $\text{EstimatedRTT} = (1-α) \times \text{EstimatedRTT} + α \times \text{SampleRTT}$ 
    - 推荐值： $\alpha = 0.125$
    - EstimatedRTT 是一个 SampleRTT 值的加权平均，其对最近样本赋予的权值比旧样本的权值更高，过去样本的影响呈指数衰减。统计学上称之为**指数加权移动平均** (Exponential Weighted Moving Average)
    - ![[30-Transport-layer-sampleRTT-estimatedRTT.png]]

- 另一个尺度：RTT 的离散程度：
	- $\text{DevRTT}=(1-β) \times \text{DevRTT} + β \times |\text{SampleRTT-EstimatedRTT}|$
	- DevRTT 是对 SampleRTT 偏离 EstimatedRTT 的程度的估算
	- β 推荐值为 0.25

- 设置超时
    - EstimtedRTT + 安全边界时间 (safety margin)
    - It is desirable to set the timeout equal to the EstimatedRTT plus some margin. ==The margin should be large when there is a lot of fluctuation in the SampleRTT values; it should be small when there is little fluctuation==. The value of DevRTT should thus come into play here. All of these considerations are taken into account in TCP’s method for determining the retransmission timeout interval: 通常设置超时时间为 `估计RTT` + `边界时间`，边界时间需要在 `采样RTT` 波动较大时变大，在波动较小时变小。反映波动程度的就是 DevRTT，因此 TCP 综合考虑后设置的超时间隔为：
    - $\text{TimeoutInterval} = \text{EstimatedRTT} + 4\times\text{DevRTT}$ 
    - ==推荐的初始超时间隔为 1s，出现超时后将加倍，而只要收到报文段就重新更新 EstimatedRTT 和 TimeoutInterval==.


### 可靠数据传输

- TCP 在 IP 不可靠的尽力服务的基础上建立了 RDT 服务
    - TCP 确保一个进程从接收缓存中取出的数据流是无损坏、无间隙、非冗余、按序的数据流。
    - 管道化的报文段
        - GBN or SR
    - 累积确认，期望从另一方收到的下一个字节的序号，对顺序到来的最后一个字节给予确认（像GBN）
    - 单个重传定时器（像GBN），只和最老的段相关联，一旦超时就将最老的段重发一遍（像SR）
    - 是否可以接受乱序的，没有规范
- 通过以下事件触发重传
    - 超时（只重发那个最早的未确认段：SR）
    - 重复的确认（又收到3个冗余确认）（又叫快速重传）
        - 例子：收到了ACK50，之后又收到3个ACK50
- 首先考虑简化的TCP发送方：
    - 忽略重复的确认
    - 忽略流量控制和拥塞控制

#### TCP 发送方事件

```
/* 简化版：假设发送方不受 TCP 流量或拥塞控制的限制，来自上方的数据长度小于 MSS，且数据传输仅为单向传输 */ 

NextSeqNum=InitialSeqNumber
SendBase=InitialSeqNumber

loop (forever) {
	switch(event) 
		event: data received from application above
			create TCP segment with sequence number NextSeqNum 
			if (timer currently not running) 
				start timer
			pass segment to IP 
			NextSeqNum=NextSeqNum+length(data)
			break; 
		event: timer timeout 
			retransmit not-yet-acknowledged segment with smallest sequence number
			start timer 
			break; 
		event: ACK received, with ACK field value of y 
			if (y > SendBase) { 
				SendBase=y
				if (there are currently any not-yet-acknowledged segments) 
					start timer 
				} 
			break; 
	} /* end of loop forever */
```

- 从应用层接收数据：
    - 用 nextseq 创建报文段并交付给 IP 
    - 如果定时器还没有运行，在传递给 IP 报文时，启动之
        - ==定时器与最早未确认的报文段关联==
        - 定时器的过期间隔：TimeOutInterval
- 超时：
    - 重传未确认的最小序号的报文段
    - 重新启动定时器
- 收到确认：
    - On the occurrence of this event, TCP compares the ACK value `y` with its variable `SendBase`. The TCP state variable `SendBase` is the sequence number of the oldest unacknowledged byte. (Thus SendBase–1 is the sequence number of the last byte that is known to have been received correctly and in order at the receiver.) 收到确认段时，TCP 比较其中的确认值和自身的变量 `SendBase`，`SendBase` 是最早未确认的段的序号。
    - As indicated earlier, TCP uses ==cumulative acknowledgments==, so that *y* acknowledges the receipt of all bytes before byte number *y*. 
    - If `y > SendBase`, then the ACK is acknowledging one or more previously unacknowledged segments. Thus the sender updates its `SendBase` variable; it also restarts the timer if there currently are any not-yet-acknowledged segments. 收到确认段后，TCP 会通过累计确认的方式更新 `SendBase` 的值，并且==如果当前还有任一未确认段时，重启定时器==。

#### 三种 TCP 重传情况分析

![[30-Transport-layer-TCP-retransmission-1.png]]
Figure 3.34 depicts the first scenario, in which Host A sends one segment to Host B. 
- Suppose that this segment has sequence number 92 and contains 8 bytes of data. After sending this segment, Host A waits for a segment from B with acknowledgment number 100. Although the segment from A is received at B, ==the acknowledgment from B to A gets lost==.
- In this case, the timeout event occurs, and Host A retransmits the same segment. Of course, when Host B receives the retransmission, it observes from the sequence number that ==the segment contains data that has already been received. Thus, TCP in Host B will discard the bytes in the retransmitted segment==.
> 主机 A 向主机 B 发送 92 号开始的 8 个字节的数据，即 `[92,99]` 这 8 个字节，期待收到 B 的确认段的确认号为 100，然而虽然数据成功被 B 接收，但==确认段丢失，于是超时事件发生，主机 A 重传同一个段，当 B 收到重传时，发现它已经完成过接收，于是丢弃这个重传段并重发确认段==。

![[30-Transport-layer-TCP-retransmission-2.png]]
In a second scenario, shown in Figure 3.35, Host A sends two segments back to back. The first segment has sequence number 92 and 8 bytes of data, and the second segment has sequence number 100 and 20 bytes of data. 
- Suppose that both segments arrive intact at B, and B sends two separate acknowledgments for each of these segments. The first of these acknowledgments has acknowledgment number 100; the second has acknowledgment number 120. 
- Suppose now that ==neither of the acknowledgments arrives at Host A before the timeout==. When the timeout event occurs, Host A resends the first segment with sequence number 92 and restarts the timer. ==As long as the ACK for the second segment arrives before the new timeout, the second segment will not be retransmitted==.
> 主机 A 向主机 B 连续发送了 92 号开始的 8 字节段和 100 号开始的 20 字节段，这两个段都到达了 B 并且 B 为这两个段都发送了确认段，第一个段的确认序号是 100，第二个确认段的序号是 120。假设这两个确认段都在超时前到达主机 A，于是主机 A 由于超时事件又重传了第一个段（92 号开始的那个）并重启计时器。只要第二个段（确认号 120）在又一次超时前到达，那么就不会启动第二个段的重发。

![[30-Transport-layer-retransmission-3.png]]
In a third and final scenario, suppose Host A sends the two segments, exactly as in the second example. 
- The acknowledgment of the first segment is lost in the network, but just ==before the timeout event, Host A receives an acknowledgment with acknowledgment number 120==. 
- Host A therefore knows that Host B has received everything up through byte 119; so Host A does not resend either of the two segments. This scenario is illustrated in Figure 3.36.
> 与之前类似，但是主机 B 发送给主机 A 的两个确认段（100 和 120）中，前者丢失，而后者到达，且都未超过超时间隔，那么由于累积确认，主机 A 会认为两个段都成功到达。

#### 超时间隔加倍

Whenever the timeout event occurs, TCP retransmits the not-yet-acknowledged segment with the smallest sequence number, as described above. 

But each time TCP retransmits, it sets the next timeout interval to twice the previous value, rather than deriving it from the last EstimatedRTT and DevRTT.

> 无论何时发生超时，TCP 会立即重传最早未确认的段，但每一次重传，TCP 都会设置下一个超时间隔为之前的两倍，而不是由超时发生前的 `估计RTT` 和 `方差RTT` 重新推导。

>[!example] 超时间隔加倍的举例
>For example, suppose `TimeoutInterval` associated with the oldest not yet acknowledged segment is 0.75 sec when the timer first expires. TCP will then retransmit this segment and set the new expiration time to 1.5 sec. If the timer expires again 1.5 sec later, TCP will again retransmit this segment, now setting the expiration time to 3.0 sec.
>
>Thus, ==the intervals grow exponentially after each retransmission==.
> > 超时间隔以指数形式增加。 
>
>However, ==whenever the timer is started after either of the two other events (that is, data received from application above, and ACK received)==, the TimeoutInterval is derived from the most recent values of `EstimatedRTT` and `DevRTT`.
> > 然而，这只是连续对同一个段超时才会加倍，一旦有新的事件发生，如应用层发来新数据、确认段到达，超时间隔就会从 `估计RTT` 和 `方差RTT` 中重新计算，而不是加倍。

==超时重传提供了一种形式受限的拥塞控制，可以处理由于网络拥塞导致的排队时间过长，并且由于重发时间的延长也一定程度上降低了信道中重传报文的数量==。

#### 快速重传

- 超时周期往往太长，
    - 因此等待超时重传会导致分组的端到端延时过长。
- 通过冗余 ACK 来检测报文段丢失
    - 发送方通常连续发送大量报文段
    - 如果报文段频繁丢失（不论发送方报文段还是 ACK 报文段），通常会引起多个重复的 ACK
    - 更详细的 ACK 生成策略如下表：

| 接收方的事件                                                     | TCP 接收方动作                                                                                                                           |
| ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| 所期望序号的报文段按序到达。所有在期望序号之前的数据都已经被确认 | 延迟的 ACK（提高效率，少发一个 ACK）。对另一个按序报文段的到达最多等待 500ms。如果下一个报文段在这个时间间隔内没有到达，则发送一个 ACK。 |
| 有期望序号的报文段到达。另一个按序报文段等待发送 ACK             | 立即发送单个累积 ACK，以确认两个按序报文段。                                                                                             |
| 比期望序号大的报文段乱序到达。检测出数据流中的间隔，出现报文丢失 | 立即发送重复的 ACK，指明下一个期待字节的序号                                                                                             |
| 能部分或完全填充接收数据间隔的报文段到达                         | 若该报文段起始于间隔 (gap)的低端，则立即发送 ACK（给确认并反映下一段的需求）。                                                           |

- 如果发送方收到同一报文段的 3 个冗余 ACK：
    - 快速重传：==在定时器过时之前重发报文段==
    - 它假设跟在被确认的数据后面的数据丢失了
        - 第一个ACK是正常的；
        - 收到第二个该段的ACK，表示接收方收到一个该段后的乱序段；
        - 收到第3，4个该段的ack，表示接收方收到该段之后的2、3个乱序段，有非常大的可能性为段丢失了

![[30-Transport-layer-TCP-fast-retransmit.png]]

```
/* 快速重传版：注意看事件——ACK被接收 */ 

NextSeqNum=InitialSeqNumber
SendBase=InitialSeqNumber

loop (forever) {
	switch(event) 
		event: data received from application above
			create TCP segment with sequence number NextSeqNum 
			if (timer currently not running) 
				start timer
			pass segment to IP 
			NextSeqNum=NextSeqNum+length(data)
			break; 
		event: timer timeout 
			retransmit not-yet-acknowledged segment with smallest sequence number
			start timer 
			break; 
		event: ACK received, with ACK field value of y 
			if (y > SendBase) { 
				SendBase=y 
				if (there are currently any not yet acknowledged segments) 
					start timer 
				}else {/* a duplicate ACK for already ACKed segment */ 
					increment number of duplicate ACKs received for y 
					if (number of duplicate ACKS received for y==3) 
					/* TCP fast retransmit */ 
					resend segment with sequence number y 
					} 
				break; 
	} /* end of loop forever */
```

>[!note] 为什么要等到 3 个冗余 ACK 才重传？而不是 1 个冗余 ACK 就重传？
>Suppose packets n, n+1, and n+2 are sent, and that packet n is received and ACKed. If packets n+1 and n+2 are reordered along the end-to-end-path (i.e., are received in the order n+2, n+1) then the receipt of packet n+2 will generate a duplicate ack for n and would trigger a retransmission under a policy of waiting only for second duplicate ACK for retransmission. By waiting for a triple duplicate ACK, it must be the case that two packets after packet are correctly received, while n+1 was not received.
>
>==The designers of the triple duplicate ACK scheme probably felt that waiting for two packets (rather than 1) was the right tradeoff between triggering a quick retransmission when needed==, but not retransmitting prematurely in the face of packet reordering.

#### TCP: GBN or SR?

- Recall that TCP acknowledgments are cumulative and correctly received but out-of-order segments are not individually ACKed by the receiver. 
- Consequently, as shown in Figure 3.33, the TCP sender need only maintain the smallest sequence number of a transmitted but unacknowledged byte (`SendBase`) and the sequence number of the next byte to be sent (`NextSeqNum`). In this sense, TCP looks a lot like a GBN-style protocol. 
- But there are some striking differences between TCP and Go-Back-N. Many TCP implementations will buffer correctly received but out-of-order segments. 

>[!example] TCP 也有 SR 风格
>Consider also what happens when the sender sends a sequence of segments 1, 2, . . . , N, and all of the segments arrive in order without error at the receiver. 
>
>Further suppose that the acknowledgment for packet n < N gets lost, but ==the remaining N - 1 acknowledgments arrive at the sender before their respective timeouts==. In this example, GBN would retransmit not only packet n, but also all of the subsequent packets n + 1, n + 2, . . . , N. ==**TCP, on the other hand, would retransmit at most one segment**==, namely, segment n. 
>
>Moreover, TCP would not even retransmit segment n if the acknowledgment for segment n + 1 arrived before the timeout for segment n. 

A proposed modification to TCP, the so-called selective acknowledgment `[RFC 2018]`, allows a TCP receiver to acknowledge out-of-order segments selectively rather than just cumulatively acknowledging the last correctly received, in-order segment. When combined with selective retransmission—skipping the retransmission of segments that have already been selectively acknowledged by the receiver— ==TCP looks a lot like our generic SR protocol==. Thus, TCP’s error-recovery mechanism is probably best categorized as a hybrid of GBN and SR protocols.

### 流量控制

目的：==接收方控制发送方，不让发送方发送的太多、太快，超过了接收方的处理能力==，以至于让接收方的接收缓冲区溢出

>[!warning] 不要混淆流量控制与拥塞控制
>- 流量控制是由接收方控制发送方，根据接收方缓冲区接收数据的能力提示发送方发送报文的速度，从而达到速度匹配，对于发送方来说是**被动地**听从接收方的指令。
>- 拥塞控制是发送方通过监测信道，由于 IP 网络的拥塞而**主动地**减缓发送报文的速度。
>
>这二者采取的动作和结果非常相似，但概念和针对的情形截然不同。

TCP流量控制
- 通过让发送方维护接收窗口 `receive window` 这一变量，使发送方得知接收方还有多少可用的缓存空间。由于 TCP 是全双工的，因此 the sender at each side of the connection maintains a distinct `receive window`.
- 试分析主机 A 通过 TCP 向主机 B 发送一个大文件：
	- 主机 B 为该连接分配的接收缓存大小 —— `RcvBuffer`
	- 主机 B 的高层进程不时从该缓存中读取数据，其中以 `LastByteRead` 表示从缓存中读取的数据流的最后一字节的编号，以 `LastByteRcvd` 表示网络中到达并放入接收缓存中数据流的最后一个字节的编号。可见有关系：`LastByteRcvd-LastByteRead <= RcvBuffer`
	- 接收窗口有如下关系：
		- `rwnd=RcvBuffer-(LastByteRcvd-LastByteRead)`
		- ![[30-Transport-layer-TCP-flow-control-rwnd.png]]
		- `rwnd` 是动态变化的。
	- 主机 B 通过把当前的 `rwnd` 值放入发送给主机 A 的确认段的[[30-Transport-layer-TCP-segment-structure.png|接收窗口字段]]中，通知主机 A 该连接的接收缓存中还有多少空间。初始时 `rwnd=RcvBuffer`
	- 主机 A 轮流跟踪两个变量 `LastByteSent` 和 `LastByteAcked`，==二者之差就是 A 发送到连接中但未确认的报文段==。通过控制这个差值不大于 `rwnd`，就可以保证 B 的接收缓存不会溢出。即 A 在连接整个过程中都要保证 `LastByteSent-LastByteAcked<=rwnd`
	- 此时，要注意一种特殊情况：
		- B 的接收缓存满时，`rwnd=0`，且 B 没有任何数据需要发送给 A（即 B 向 A 发送的最后一个报文段是 `rwnd=0` 的 ACK 确认），
		- 那么即使 B 中缓存区已经清空，A 仍然不会发送报文段，A 被阻塞在此！（只有当 B 向 A 发送报文段时才会更新 `rwnd` 从而 A 恢复发送）
		- 要解决这一问题，==TCP 要求 B 的接收窗口为 0 时，A 继续发送只有一个字节的报文段，从而保证接收方不会停止发送 ACK==；

![[TCP-flow-control.mp4]]

### 连接管理

连接
- TCP 连接的建立会显著增加延迟
- 常见网络攻击如 SYN 洪泛就是攻击了 TCP 连接管理的弱点
- 连接的本质：
    - 双方知道要和对方通信
    - 为这次通信建立好缓冲区，准备好资源
    - 一些控制变量需要做置位，两者需要互相告知自己的初始序号，初始化的RcvBuffer

#### “三次握手”建立连接

在正式交换数据之前，发送方和接收方握手建立通信关系：
- 同意建立连接（双方都知道对方愿意建立连接）
- 同意连接参数

“三次握手”的详细步骤：
![[30-Transport-layer-TCP-connection-3-handshake.png]]
1. Step 1. The client-side TCP first sends a special TCP segment to the server-side TCP. ==This special segment contains no application-layer data. But one of the flag bits in the segment’s header, the SYN bit, is set to 1==. For this reason, this special segment is referred to as a **SYN segment**. In addition, the client randomly chooses an initial sequence number (`client_isn`) and puts this number in the sequence number field of the initial TCP SYN segment. This segment is encapsulated within an IP datagram and sent to the server. There has been considerable interest in properly randomizing the choice of the `client_isn` ==in order to avoid certain security attacks== `[RFC 4987]`.
> 第一步：发送方 TCP 发送一个特殊的 TCP 段给接收方 TCP，这个特殊段中不包含任何应用层的数据，而且其标记位中的 SYN 位置为 1，因此称为 SYN 段。另外，客户端随机地选择一个初始序号（`client_isn`）并放入初始的 SYN 段的[[30-Transport-layer-TCP-segment-structure.png|序号域]] 中，这里之所以随机地选择初始序号是为了安全考量。

2. Step 2. Once the IP datagram containing the TCP SYN segment arrives at the server host, the server extracts the TCP SYN segment from the datagram, ==allocates the TCP buffers and variables to the connection==, and sends a connection-granted segment to the client TCP. This connection-granted segment also contains no application-layer data. However, it does contain three important pieces of information in the segment header. 
	- First, the SYN bit is set to 1. 
	- Second, the acknowledgment field of the TCP segment header is set to `client_isn+1`.
	- Finally, the server chooses its own initial sequence number (`server_isn`) and puts this value in the sequence number field of the TCP segment header. ==The connection-granted segment is referred to as a **SYNACK segment==**. 
> 第二步：一旦包含 SYN 段的 IP 数据报到达接收方，接收方从数据报中提取 SYN 段并为连接分配接收方相关的缓冲区和变量，然后向客户端 TCP 发送连接确认段。连接确认段中也没有应用层数据，但是有三个关键信息：1) SYN 位仍置为 1；2) 确认段的确认域设置为 `client_isn+1`；3) 选择一个序号作为 `server_isn` 放入确认段的序号域。
> 这个连接确认段称为 SYNACK 段。

3. Step 3. ==Upon receiving the SYNACK segment, the client also allocates buffers and variables to the connection==. The client host then sends the server yet another segment; this last segment acknowledges the server’s connection-granted segment (the client does so by putting the value `server_isn+1` in the acknowledgment field of the TCP segment header). **The SYN bit is set to zero**, since the connection is established. This third stage of the three-way handshake may carry client-to server data in the segment payload.
> 第三步：当收到 SYNACK 段，客户端为连接分配发送方缓冲区和变量，客户端接着发送给服务端新的段——新段确认了 SYNACK 这个段（通过设置确认号为 `server_isn+1`），并且设置 SYN 标记为0，并且可能包含应用层的数据负载。

>[!warning] Server allocates buffers too early.
>We’ll see in Chapter 8 that the allocation of these buffers and variables before completing the third step of the three-way handshake makes TCP vulnerable to a denial-of-service attack known as SYN flooding.
>> 由于服务端为连接分配缓冲区过早，这衍生出攻击服务器的一种方式——SYN 洪泛。

==经过三次握手后，连接被建立，之后的报文交换中 SYN 标记位始终是 0==。

TCP 3次握手中的捎带：（变化的初始序号+双方确认对方的序号）
- 第一次：client-->server：客户端一方的初始序号 $x$
- 第二次：server-->client：发送确认 $ACKnum=x+1$ 和服务器一方的初始序号 $y$ （捎带）
- 第三次：client-->server：发送确认 $ACKnum=y+1$ 和客户端data（捎带）

>[!note] 为什么是三次握手而不是两次？
>![[30-Transport-layer-two-way-handshake-false.png]]
>
>三次握手通过变化的初始序号+双方确认对方的序号，解决半连接和接收老数据问题。
>
>![[30-Transport-layer-three-way-handshake-right.png]]
>1. 二次握手：可能发送半连接（只在服务器维护了连接）；三次握手：客户端在第三次握手拒绝连接请求
>2. 二次握手：老的数据被当成新的数据接收了；三次握手：未建立连接（无半连接），故将发来的数据丢掉（扔掉：连接不存在，没建立起来；连接的序号不在当前连接的范围之内）
>3. 若一个数据滞留时间足够长导致在TCP第二次连接（两个三次握手后）到来，这个数据包大概率也会被丢弃，因为初始序号seq不一样，而seq又与时间有关（时钟周期的第k位）

#### “四次挥手”关闭连接

- “对称释放，并不完美”

![[30-Transport-layer-four-way-handwave.png]]
TCP：关闭连接（连接释放）：**4 次挥手**
- 客户端，服务器分别关闭它自己这一侧的连接 
	- 客户端发送 FIN bit = 1 的特殊 TCP 段
	- 服务器一旦接收到 FIN 报文，用 ACK 回应，等待一段时间后发送服务器自己的 FIN 报文段
	- 客户端接到来自服务器的 FIN 报文段，发送 ACK 确认；同时客户端从收到服务器的 FIN 报文开始，等待一段时间后关闭自己的连接
- 可以处理同时的 FIN 交换

>[!question] 为什么连接的时候是三次握手，关闭的时候却是四次握手
>
>答：因为当 Server 端收到 Client 端的 SYN 连接请求报文后，可以直接发送 SYN+ACK 报文。其中 ACK 报文是用来应答的，SYN 报文是用来同步的。但是关闭连接时，当 Server 端收到 FIN 报文时，很可能并不会立即关闭 SOCKET，所以只能先回复一个 ACK 报文，告诉 Client 端，“你发的 FIN 报文我收到了”。只有等到 Server 端所有的报文都发送完了，我才能发送 FIN 报文，因此不能一起发送。故需要四步握手。

#### TCP 连接的生命周期

![[30-Transport-layer-TCP-client-lifetime.png]]
- The client TCP begins in the CLOSED state. 
	- The application on the client side initiates a new TCP connection. 
	- This causes TCP in the client to send a SYN segment to TCP in the server. After having sent the SYN segment, the client TCP enters the `SYN_SENT` state. 
	- While in the `SYN_SENT` state, the client TCP waits for a segment from the server TCP that includes an acknowledgment for the client’s previous segment and has the SYN bit set to 1. 
	- Having received such a segment, the client TCP enters the ESTABLISHED state. While in the ESTABLISHED state, the TCP client can send and receive TCP segments containing payload (that is, application-generated) data.

- Suppose that the client application decides it wants to close the connection. (Note that the server could also choose to close the connection.) 
	- This causes the client TCP to send a TCP segment with the FIN bit set to 1 and to enter the `FIN_WAIT_1` state. While in the `FIN_WAIT_1` state, the client TCP waits for a TCP segment from the server with an acknowledgment. 
	- When it receives this segment, the client TCP enters the `FIN_WAIT_2` state. While in the `FIN_WAIT_2` state, the client waits for another segment from the server with the FIN bit set to 1; after receiving this segment, the client TCP acknowledges the server’s segment and enters the `TIME_WAIT` state. 
	- The `TIME_WAIT` state lets the TCP client ==resend the final acknowledgment== in case the ACK is lost. The time spent in the `TIME_WAIT` state is implementation-dependent, but typical values are 30 seconds, 1 minute, and 2 minutes. After the wait, the connection formally closes and all resources on the client side (including port numbers) are released.

![[30-Transport-layer-TCP-connection-server-lifetime.png]]

>[!note] 如果收到报文段要求的端口不匹配？
>Let’s consider what happens when a host receives a TCP segment whose port numbers or source IP address do not match with any of the ongoing sockets in the host. 
>
>For example, suppose a host receives a TCP SYN packet with destination port 80, but the host is not accepting connections on port 80 (that is, it is not running a Web server on port 80).
>> 如果服务端收到一个 SYN 分组，其目标端口是 80，但是服务端不接收 80 端口的连接：
>
>- Then the host will send a special reset segment to the source. This TCP segment has the RST flag bit (see Section 3.5.2) set to 1. 
>- Thus, when a host sends a reset segment, it is telling the source “I don’t have a socket for that segment. Please do not resend the segment.” 
>- When a host receives a UDP packet whose destination port number doesn’t match with an ongoing UDP socket, the host sends a special ICMP datagram, as discussed in Chapter 5.
>
>> 服务端将会发送一个特殊的重置段给源端，重置段中 RST 标记位置为1；当服务端发送 RST 段时，意味着告知源端“我没有为那个段准备 socket，请重发合适的段”；当服务端收到不匹配的 UDP socket 端口号的分组时，服务端会发送一个 ICMP 数据报。

#### SYN cookies

THE SYN FLOOD ATTACK We’ve seen in our discussion of TCP’s three-way handshake that a server allocates and initializes connection variables and buffers in response to a received SYN. The server then sends a SYNACK in response, and awaits an ACK segment from the client. If the client does not send an ACK to complete the third step of this 3-way handshake, eventually (often after a minute or more) the server will terminate the half-open connection and reclaim the allocated resources. 

This TCP connection management protocol sets the stage for a classic Denial of Service (DoS) attack known as the SYN flood attack. ==In this attack, the attacker(s) send a large number of TCP SYN segments, without completing the third handshake step. With this deluge of SYN segments, the server’s connection resources become exhausted as they are allocated (but never used!) for half-open connections; legitimate clients are then denied service==. Such SYN flooding attacks were among the first documented DoS attacks `[CERT SYN 1996]`. Fortunately, an effective defense known as SYN cookies `[RFC 4987]` are now deployed in most major operating systems.

***SYN cookies*** work as follows:

- When the server receives a SYN segment, it does not know if the segment is coming from a legitimate user or is part of a SYN flood attack. So, ==instead of creating a half-open TCP connection for this SYN, the server creates an initial TCP sequence number that is a complicated function (hash function) of source and destination IP addresses and port numbers of the SYN segment, as well as a secret number only known to the server==. This carefully crafted initial sequence number is the so-called “cookie.” The server then sends the client a SYNACK packet with this special initial sequence number. **Importantly, the server does not remember the cookie or any other state information corresponding to the SYN**. 
- A legitimate client will return an ACK segment. When the server receives this ACK, it must verify that the ACK corresponds to some SYN sent earlier. But how is this done if the server maintains no memory about SYN segments? As you may have guessed, it is done with the cookie. Recall that for a legitimate ACK, the value in the acknowledgment field is equal to the initial sequence number in the SYNACK (the cookie value in this case) plus one (see Figure 3.39). The server can then run the same hash function using the source and destination IP address and port numbers in the SYNACK (which are the same as in the original SYN) and the secret number. ==If the result of the function plus one is the same as the acknowledgment (cookie) value in the client’s SYNACK, the server concludes that the ACK corresponds to an earlier SYN segment and is hence valid. The server then creates a fully open connection along with a socket==. 
- On the other hand, if the client does not return an ACK segment, then the original SYN has done no harm at the server, since the server hasn’t yet allocated any resources in response to the original bogus SYN.

## 3.6 拥塞控制原理

丢包通常是网络拥塞时路由器缓存区溢出而引起，因此==分组重传往往是拥塞的预兆，但重传并不能解决拥塞反而会进一步加重拥塞==。

### 拥塞原因与代价

拥塞：
- 非正式的定义：“太多的数据需要网络传输，超过了网络的处理能力”
- 与流量控制不同，流量控制是我到你的问题，拥塞控制是网络的问题
- 拥塞的表现：
    - 分组丢失（路由器缓冲区溢出）
    - 分组经历比较长的延迟（在路由器的队列中排队）

#### Case 1: Two Senders, a Router with Infinite Buffers

![[30-Transport-layer-congestion-case-1.png]]
- 2个发送端，2个接收端，共享 1 个单跳路由，路由器具备无限大的缓冲
- 主机 A 和 B 的应用输入连接中的数据速率为 $\lambda_{in}$ Bytes/s  
- 传输层协议简单封装即发送，不做差错恢复（没有重传）、流量控制、拥塞控制

![[30-Transport-layer-congestion-case-1-performance.png]]
- 左图是每连接的吞吐量 $\lambda_{out}$ 与连接发送速率 $\lambda_{in}$ 的关系，发送速率处于 0~R/2 时二者相等，一旦发送速率超过 R/2，吞吐量不再增加；
- 无论 A 和 B 将发送速率提升到多高，各自的吞吐量也不会超过 R/2
- 右图显示，当吞吐量接近链路容量时，时延快速增大：当进入的速率 $\lambda_{in}$ 接近链路链路带宽 $R$ 时，延迟增大（此时流量强度趋于1）


#### Case 2: Two Senders and a Router with Finite Buffers
![[30-Transport-layer-congestion-case-2.png]]
- 1个路由器，有限的缓冲: 缓冲区满时到达的分组会被丢弃
- 为了可靠，分组丢失时，发送端会重传
    - 应用层的发送速率 $\lambda_{in}$
    - 传输层的发送速率 $\lambda_{in}^{'} = \lambda_{in} + \lambda_{resend}$ ，$\lambda_{in}^{'}$ 称为供给载荷 offered load
    -  $\lambda_{in}^{'} \geq \lambda_{in}$，等号当且仅当 A 和 B 的发送速率都不超过 R/2 时才成立，
    - 一旦发送速率 $\lambda_{in}>R/2$ 时，缓存区溢出会导致重传分组，==重传一旦产生就会逐渐累积，情况渐渐恶化==。
![[30-Transport-layer-congestion-case-2-performance.png]]

1. First, consider the unrealistic case that Host A is able to somehow determine whether or not a buffer is free in the router and thus ==sends a packet only when a buffer is free==. In this case, no loss would occur, $\lambda_{in}$ would be equal to $\lambda_{in}^{'}$, and the throughput of the connection would be equal to $\lambda_{in}$. This case is shown in Figure 3.46(a). ***Note that the average host sending rate cannot exceed R/2 under this scenario, since packet loss is assumed never to occur***. 
> 首先考虑一种不现实的情况——主机 A 能够得知路由器缓冲区是否空闲，并且只在空闲时才发送分组。这时，不会发生丢失，$\lambda_{in}=\lambda_{in}^{'}$，吞吐量等于 $\lambda_{in}$ 
> 这个情况显示在图 3.46-a 中，此时主机发送速率不能超过 $\frac{R}{2}$，才不会导致分组丢失的发生。

2. Consider the case that the sender ==retransmits only when a packet is known for certain to be lost==. In this case, the offered load, $\lambda_{in}^{'}$ (the rate of original data transmission plus retransmissions), equals R/2. 
	- According to Figure 3.46 (b), at this value of the offered load, the rate at which data are delivered to the receiver application is $\lambda_{out}=R/3$. 
	- Thus, out of the 0.5R units of data transmitted, 0.333R bytes/sec (on average) are original data and 0.166R bytes/sec (on average) are retransmitted data. 
	- We see here another cost of a congested network — ***the sender must perform retransmissions in order to compensate for dropped (lost) packets due to buffer overflow.***
> - 考虑发送方只在分组确认丢失时才重传的情况，此时 $\lambda_{in}^{'}=\frac{R}{2}$，即数据传送速率+重传速率不超过链路带宽的一半。
> - 如图 3.46-b 所示，在这种给定的速率情况下，接收方应用能够获取数据的速率为 $\lambda_{out}=\frac{R}{3}$。
> - 也即，每 0.5R 的数据传输速率中，有 0.333R 的速率用于原始数据传输，而 0.167R 的速率用于重传数据的传输。
> - 这就是另一种拥塞网络的损耗之处——发送方必须执行重传，以补偿由于缓冲区溢出而丢失的分组。

3. Consider the case that the ==sender may time out prematurely and retransmit a packet that has been delayed in the queue but not yet lost==. In this case, both the original data packet and the retransmission may reach the receiver.
	- The work done by the router in forwarding the retransmitted copy of the original packet was wasted, as the receiver will have already received the original copy of this packet.
	- Here then is yet another cost of a congested network—***unneeded retransmissions by the sender in the face of large delays may cause a router to use its link bandwidth to forward unneeded copies of a packet***. 
	- Figure 3.46 (c) shows the throughput versus offered load when each packet is assumed to be forwarded (on average) twice by the router. Since each packet is forwarded twice, the throughput will have an asymptotic value of R/4 as the offered load approaches R/2
> - 考虑发送方可能会过早地超时，并且重传由于延迟而排队的未丢失分组，这种情况中原始数据分组和重传分组可能都会到达接收方；
> - 路由器转发的原始分组的重传拷贝是浪费的，因此这是另一种不必要的拥塞网络的开销，这将会导致链路带宽用于不必要的转发行为，从而进一步导致网络拥塞情况的恶化；
> - 图 3.46-c 中显示了这种负载下的吞吐量——当每个分组都被路由器转发两次时，吞吐量将会渐进地接近 $\frac{R}{4}$ 

- 此时随着 $\lambda_{in}$ 的增大，$\lambda_{out}$ 也增大，但是由于超时重传的比例越来越大， $\lambda_{out}$ 越来越小于 $\lambda_{in}$
    - 输出比输入少原因：1）重传的丢失分组；2）没有必要重传的重复分组

拥塞的“代价”：
- 延时大
- 为了达到一个有效输出，网络需要做更多的泵入（重传）
- 没有必要的重传，链路中包括了多个分组的拷贝
    - 是那些没有丢失，经历的时间比较长（拥塞状态）但是超时的分组，重发没有必要，对本就拥塞的网络雪上加霜（**网络加速变坏，非线性**）
    - 降低了的“goodput”

#### Case 3: Four Senders, Routers with Finite Buffers, and Multihop Paths

![[30-Transport-layer-congestion-case-3.png]]

- 4个发送端
- 多重路径，多跳连接，所有主机 $\lambda_{in}$ 相同，所有链路容量都是 $R$
- 超时/重传
- Let’s consider the connection from Host A to Host C, passing through routers R1 and R2. The A–C connection shares router R1 with the D–B connection and shares router R2 with the B–D connection. 
	- For extremely small values of $\lambda_{in}$, buffer overflows are rare (as in congestion cases 1 and 2), and the throughput approximately equals the offered load. 
	- For slightly larger values of $\lambda_{in}$, the corresponding throughput is also larger, since more original data is being transmitted into the network and delivered to the destination, and overflows are still rare. Thus, for small values of $\lambda_{in}$, an increase in $\lambda_{in}$ results in an increase in $\lambda_{in}$.
> 考虑从主机 A 到主机 C 的连接，这条连接穿过了路由器 R1 和 R2，且与 D-B 的连接共享路由器 R1、与 B-D 的连接共享路由器 R2。
> - 对很小的输入速率 $\lambda_{in}$，几乎不会发生缓冲区溢出，吞吐量与所提供的负载相同。
> - 但随着 $\lambda_{in}$ 的略微增大，相关的吞吐量在缓冲区不溢出的前提下同步增加。

- Having considered the case of extremely low traffic, let’s next examine the case that $\lambda_{in}$ / $\lambda_{in}^{'}$ is extremely large. Consider router R2. 
	- The A–C traffic arriving to router R2 (which arrives at R2 after being forwarded from R1) can have an arrival rate at R2 that is at most R, the capacity of the link from R1 to R2, regardless of the value of $\lambda_{in}$. 
	- If $\lambda_{in}^{'}$ is extremely large for all connections (including the B–D connection), then the arrival rate of B–D traffic at R2 can be much larger than that of the A–C traffic. Because the A–C and B–D traffic must ==compete at router R2 for the limited amount of buffer space==, the amount of A–C traffic that successfully gets through R2 (that is, is not lost due to buffer overflow) becomes smaller and smaller as the offered load from B–D gets larger and larger.
	- In the limit, as the offered load approaches infinity, an empty buffer at R2 is immediately filled by a B–D packet, and the throughput of the A–C connection at R2 goes to zero. These considerations give rise to the offered load versus throughput tradeoff shown in Figure 3.48.
> 然而一旦 $\lambda_{in}$ / $\lambda_{in}^{'}$ 大到一定程度时，再次考虑路由器 R2 的情况:
> - A-C 的流量在到达路由器 R2 时可能达到速率最大值 R，即链路带宽，而无论 $\lambda_{in}$ 具体的值是多少；
> - 如果 $\lambda_{in}^{'}$ 对所有连接都相当大，于是路由器 R2 处 B-D 连接的流量到达速率可能远比 A-C 连接的流量到达速率要大，因为 A-C 连接和 B-D 连接必须在路由器 R2 处竞争有限的缓冲空间，如果不考虑缓冲区的溢出，那么随着 B-D 连接的供给载荷 $\lambda_{in}^{'}$ 增大时，A-C 连接的流量逐渐减小；
> - 在这种限制中，如果 B-D 连接的供给载荷趋于无穷大，那么路由器 R2 的缓冲区会立即被 B-D 连接的分组填满，从而 A-C 连接的吞吐量趋于 0，如下图 3.48 所示：

![[30-Transport-layer-congestion-case-3-performance.png]]

In the high-traffic scenario outlined above, ==whenever a packet is dropped at a second-hop router, the work done by the first-hop router in forwarding a packet to the second-hop router ends up being “wasted.”== The network would have been equally well off (more accurately, equally bad off) if the first router had simply discarded that packet and remained idle. 
> 在上述高速率流量的情境中，无论何时分组在第二跳路由器中丢失，前一跳路由器所做的工作都相当于浪费，这样比在第一跳路由器就丢失的情况更加糟糕！

More to the point, the transmission capacity used at the first router to forward the packet to the second router could have been much more profitably used to transmit a different packet. (For example, ==when selecting a packet for transmission, it might be better for a router to give priority to packets that have already traversed some number of upstream routers==.) 
> 更关键的是，第一跳路由器用于转发分组给第二跳路由器的传输能力本应更经济地用于传输不同的分组——例如，当选择一个分组进行传输时，对于路由器来说更佳策略是对分组给定优先级，这个优先级由之前的上游路由器给定。

So here we see yet another cost of dropping a packet due to congestion—***when a packet is dropped along a path, the transmission capacity that was used at each of the upstream links to forward that packet to the point at which it is dropped ends up having been wasted***.
> 这就是分组丢失的另一种对网络拥塞的影响——上游传输能力被浪费了。

| 情况   | 拥塞的代价                                                                         |
| ------ | ---------------------------------------------------------------------------------- |
| Case 1 | 分组到达速率接近链路容量时，分组的排队时延相当大                                   |
| Case 2 | 发送方必须执行重传以补偿因缓存溢出而丢弃的分组                                     |
| Case 3 | 当分组丢失时，任何“关于这个分组的上游传输能力”都被浪费了，严重时导致整个网络死锁。 | 

### 拥塞控制方法

- **端到端拥塞控制**（自身判断）
    - 没有来自网络层的显式反馈
    - 端系统根据网络行为的观察，如反组丢失频率和延迟，推断是否有拥塞
    - TCP 采用的方法：==超时==或 ==3 次冗余确认==，推断出网络拥塞的迹象

- **网络辅助的拥塞控制**（路由器反馈）
    - 路由器显式地提供给端系统以网络拥塞状态相关的信息
	    - 单个 bit 置位，显示有拥塞 (SNA, DECbit, TCP/IP ECN, ATM 等体系结构)
	    - 显式提供发送端可以采用的速率也可以，只是实现起来更复杂
	- 拥塞信息从网络反馈给发送方的两种方式：
		- ![[30-Transport-layer-congestion-feedback.png]]
		1. 采用阻塞分组 choke packet 的形式，网络路由器直接反馈信息给发送方
		2. 路由器标记或更新从发送方流向接收方的分组中的某个字段来指示拥塞的产生。一旦收到一个标记分组后，接收方就会向发送方通知该网络拥塞指示，这种通知至少经过一个完整的往返时间。

> [!example] ATM ABR 拥塞控制（网络辅助的拥塞控制）
> 
> ATM网络数据交换的单位叫 信元（一个小分组，53字节：5字节的头部+48字节的数据载荷）。
> 
> 信元每个分组数据量较小，在每个交换节点存储-转发耽误的时间比分组交换小，且时间固定，便于调度。ATM网络集合了分组交换和线路交换的特性。
> 
> ATM虽然不是大众常用网络，但是在一些专用场合用处较大
> 
> ATM网络有很多模式，其中一个模式是ABR: available bit rate:
> - 提供“弹性服务” 
> - 如果发送端的路径“轻载”不发生拥塞
>     - 发送方尽可能地使用可用带宽（“市场调控”）
> - 如果发送方的路径拥塞了
>     - 发送方限制其发送的速度到一个最小保障速率上（“配给制”）
> 
> ATM网络中的信元可以分为2种：大部分是数据信元，另一种为间或地插入到数据信元当中的资源管理信元。
> 
> RM(资源管理)信元:
> - 由发送端发送，在数据信元中间隔插入
> - RM信元中的比特被交换机设置（“网络辅助”）
>     - NI bit: no increase in rate （轻微拥塞）速率不要增加了，轻微拥塞时设置为1
>     - CI bit: congestion indication 拥塞指示，拥塞时设置为1
> - 发送端发送的RM信元被接收端返回，接收端不做任何改变
> - 在RM信元中的2个字节ER(explicit rate)字段
>     - 拥塞的交换机可能会降低信元中ER的值
>     - 发送端发送速度因此是最低的可支持速率
> - 数据信元中的EFCI bit：被拥塞的交换机设置成1
>     - 如果在管理信元RM前面的数据信元EFCI被设置成了1，接端在返回的RM信元中设置CI bit
> 
> 总结：网络提供一些信息，包括一些标志位的置位以及字段 （为两主机间的通信提供多大的带宽）

## 3.7 TCP拥塞控制

- TCP拥塞控制采用==端到端的拥塞控制机制==
    - 因为 IP 层不向端系统提供显式的网络拥塞反馈（这只是传统的路由器功能，后文提及的 ECN 机制和 DataCenterTCP 中就采用了显式网络拥塞反馈）
        - 路由器的负担较轻
        - 符合网络核心简单的TCP/IP架构原则，网络核心提供最小的服务集
    - 端系统根据自身得到的信息，判断是否发生拥塞，从而采取动作：增加 or 降低发送速率

- 拥塞控制的几个问题
    - 如何限制向连接发送流量的速率？
    - 如何检测拥塞的情况？
        - 轻微拥塞
        - 严重拥塞
    - 采用何种算法改变发送速率？

### TCP 发送方如何限制向连接发送流量的速率？
[[#流量控制]] 

TCP 拥塞控制机制跟踪拥塞窗口 `cwnd`，其对 ==TCP 发送方能向网络中发送流量的速率进行了限制==：`LastByteSent - LastByteAcked <= min(cwnd,rwnd)`
- 以关注拥塞控制为目的，假设 TCP 连接方的接收缓存足够大，从而忽略 `rwnd` 的限制，使发送方未被确认的数据量仅受限于 `cwnd`
- **通过限制发送方窗口中未被确认的数据量，因此间接地限制发送方的发送速率**
	- 忽略丢包和发送时延，==在每个往返时间 RTT 的起始点，根据 `cwnd` 的限制，发送方的发送速率为 `cwnd/RTT` byte/sec，因此通过调整 `cwnd` 的值可以限定发送速率的大小==。

- `cwnd` 是动态的，是感知到的网络拥塞程度的函数
    - 超时或者 3 个冗余 ACK，`cwnd` 下降
        - 超时：`cwnd` 降为 1 MSS，进入 SlowStart 阶段然后再倍增到 `cwnd/2`（每个 RTT），从而进入 CongestionAvoidance 阶段
        - 3 个冗余 ACK：`cwnd` 降为 `cwnd/2`, CongestionAvoidance 阶段
    - 否则（正常收到 ACK，没有发送以上情况）：`cwnd` 跃跃欲试（上升）
        - SS 阶段（慢启动阶段）：每经过一个 RTT 进行 `cwnd` 的加倍
        - CA 阶段（拥塞避免阶段）：每经过一个 RTT 进行 1 个 MSS 的线性增加

==TCP 拥塞控制和流量控制的联合动作==：
- 发送端控制发送但是未确认的量同时也不能够超过接收窗口的空闲尺寸（接收方在返回给发送方的报文中捎带），满足流量控制要求
	- $SendWin = \min(cwnd, rwnd)$
	- 同时满足 ***拥塞控制*** 和 ***流量控制*** 要求

### 发送方如何探测到拥塞？

- **丢包事件**：超时或接收到 3 个冗余 ACK
- 若 TCP 持续收到未确认分组的 ACK，则根据收到确认的速率动态地调整拥塞窗口的大小
- 某个段超时了（丢失事件）：拥塞
    - 超时时间到，某个段的确认没有来
    - 原因1：网络拥塞（某个路由器缓冲区没空间了，被丢弃） 概率大
    - 原因2：出错被丢弃了（各级错误，没有通过校验，被丢弃） 概率小。此时不应该降低发送速率
    - 一旦超时，就认为拥塞了，有一定误判（原因2，但概率小），但是总体控制方向是对的

- 有关某个段的3次重复ACK：轻微拥塞
    - ![[30-Transport-layer-tcp-congestion-perception.png]]
    - 段的第1个 ACK，正常，确认绿段，期待红段
    - 段的第2个重复 ACK，意味着红段的后一段收到了——蓝段乱序到达
    - 段的第2、3、4个 ACK 重复，意味着红段的后第2、3、4个段收到了——橙段乱序到达，同时红段丢失的可能性很大（后面3个段都到了，红段都没到）
    - 网络这时还能够进行一定程度的传输，拥塞但情况要比第一种好

TCP 关于拥塞的原则：
1. 一个丢失的报文段意味着拥塞，因此当丢失报文段时应当降低 TCP 发送方的速率
2. 一个确认报文段指示该网络正在向接收方交付发送方的报文段，因此对先前未确认报文段的确认到达时，应当增加发送方速率
3. 带宽周期性检测：The TCP sender increases its transmission rate to probe for the rate that at which congestion onset begins, backs off from that rate, and then to begins probing again to see if the congestion onset rate has changed. TCP 发送方递增其传输速率以探测到何种速率时拥塞会发生，然后退回到拥塞前的速率并继续探测——查看拥塞速率是否有所变化——这样能够是的 TCP 连接尽可能地利用链路带宽。

### TCP 拥塞控制：策略概述
#### 慢启动
- 连接刚建立， `cwnd` 设置为根据 MSS 计算的较小值
	- 如：$MSS = 1460bytes$ ，$RTT = 200msec$
	- 则可计算得初始速率为 $MSS/RTT = 58.4kbps$ ，这是一个很小的速率，可用带宽可能远远大于它，应当尽快着手增速，到达希望的速率
- 当连接开始时，指数性（后一次速率是前一次的2倍）增加发送速率，直到发生丢失的事件
	- ![[30-Transport-layer-tcp-slow-start.png]]
	- 启动初值很低
	- 但是增长速度很快（指数增加，SS 时间很短，长期来看可以忽略）
	- ***注意：每个报文段被确认，就对该报文段加 1 个 MSS，即指数增长，每一个 RTT，`cwnd` 加倍***

==何时停止指数增长==？三种策略：
1. 出现超时指示的丢包事件，意味着大概率出现了拥塞。
	- ==TCP 发送方将 `cwnd` 重置为 1 个 MSS 并重新开始慢启动==
	- 将第二个状态变量 `ssthresh` 设置为 `cwnd/2`

2. 直接与 `ssthresh` 的值相关联
	- 当检测到拥塞时，`ssthresh` 设置为 `cwnd/2`，之后再次达到或超过 `ssthresh` 时，不宜继续使用翻番的 `cwnd` 的值；
	- 结束慢启动转移到拥塞避免模式；

3. 如果检测到 3 个冗余 ACK，则执行快速重传并进入快速恢复状态

TCP 的 FSM：
- ![[30-Transport-layer-TCP-congestion-control-fsm.png]]

#### 拥塞避免

On entry to the ***congestion-avoidance*** state, the value of `cwnd` is approximately half its value when congestion was last encountered — congestion could be just around the corner! Thus, rather than doubling the value of `cwnd` every RTT, TCP adopts a more conservative approach and ***increases the value of `cwnd` by just a single MSS every RTT***. 
> 当进入拥塞避免状态时，拥塞窗口的设置为上一次遇到拥塞时拥塞窗口的一半，接着 `cwnd` 的值不再每过一个 RTT 就翻番，而是每个 RTT 都仅递增 1 个 MSS 大小。

A common approach is for the TCP sender to increase `cwnd` by MSS bytes ==(MSS/cwnd) whenever a new acknowledgment arrives==. 
- For example, if MSS is 1,460 bytes and `cwnd` is 14,600 bytes, then 10 segments are being sent within an RTT. 
- Each arriving ACK (assuming one ACK per segment) ==increases the congestion window size by 1/10 MSS==, and thus, the value of the congestion window will have increased by one MSS after ACKs when all 10 segments have been received.
> 一种常见的 TCP 发送方递增 `cwnd` 的方法是每收到一个 ACK 就递增 `MSS/cwnd` 个字节的大小，这样会在收到所有已发送段的 ACK 后，`cwnd` 仅增加了 1 个 MSS 大小。
> 
> *注意这里并没有违背每个 RTT 都增加1个 MSS 的规则—— `cwnd` 的大小意味着 1RTT 内能够发送多少个 TCP 段*。

But when should congestion avoidance’s linear increase (of 1 MSS per RTT) end? TCP’s congestion-avoidance algorithm behaves the same when a timeout occurs as in the case of slow start: ==The value of `cwnd` is set to 1 MSS, and the value of `ssthresh` is updated to half the value of ` cwnd ` when the loss event occurred==. 
> 拥塞避免的线性增长何时结束？与慢启动状态类似，一旦再次发生超时，就将 `cwnd` 置为 1MSS 大小，并且拥塞阈值 `ssthresh` 置为之前 `cwnd` 的一半。

Recall, however, that a loss event also can be triggered by a triple duplicate ACK event.In this case, the network is continuing to deliver some segments from sender to receiver (as indicated by the receipt of duplicate ACKs). So TCP’s behavior to this type of loss event should be less drastic than with a timeout-indicated loss: ==TCP halves the value of `cwnd` (adding in 3 MSS for good measure to account for the triple duplicate ACKs received) and records the value of `ssthresh` to be half the value of `cwnd` when the triple duplicate ACKs were received==. The fast-recovery state is then entered.
> 另外，丢包事件还可能由 3 个冗余 ACK 的事件触发，此时拥塞情况其实并没有超时那样严重，因此 TCP 的行为也相对不那么激进：
> - TCP 将 `cwnd` 的值缩为之前的一半（注意超时事件则置为 1MSS 大小）（*注意，对于 TCP Reno 来说，`cwnd` 的值应为 `old_cwnd/2 + 3`，这里 +3 的原因在于收到了 3 个冗余 ACK*）
> - 将 `ssthresh` 置为收到 3 个冗余 ACK 时 `cwnd` 的一半；
> - 接下来进入快速恢复阶段；

#### 快速恢复

In fast recovery, ==the value of `cwnd` is increased by 1 MSS for every duplicate ACK received for the missing segment== that caused TCP to enter the fast-recovery state. 
> 快速恢复阶段，`cwnd` 的值在每收到一个冗余 ACK 时递增 1 个 MSS。

Eventually, 
- ==when an ACK arrives for the missing segment, TCP enters the congestion-avoidance state== after deflating `cwnd`. 
- If a timeout event occurs, fast recovery transitions to the slow-start state after performing the same actions as in slow start and congestion avoidance: The value of `cwnd` is set to 1 MSS, and the value of `ssthresh` is set to half the value of `cwnd` when the loss event occurred.
> 当丢失段的 ACK 到达时，TCP 进入在缩小 `cwnd` 后进入拥塞避免状态；
> 当超时事件发生时，快速恢复状态在执行与慢启动和拥塞避免中相同的动作后转进到慢启动状态——将 `cwnd` 置为 1MSS 大小，将 `ssthresh` 设置为丢失时间发生时的 `cwnd` 的一半大小。

>[!example] 不同版本的 TCP 如何处理拥塞？
> 
> - 早期版本的 TCP Tahoe，只要发生丢包事件（超时 or 3 冗余）都无条件地进入慢启动阶段；
> - Reno 版则综合了快速回复这一特性；
> 
> ![[30-Transport-layer-TCP-congestion-window-tahoe-reno.png]]
> 
> In this figure, the threshold is initially equal to 8 MSS. 
> - For the first eight transmission rounds, Tahoe and Reno take identical actions. The congestion window climbs exponentially fast during slow start and hits the threshold at the fourth round of transmission. 
> - The congestion window then climbs linearly until a triple duplicate - ACK event occurs, just after transmission round 8. 
> - Note that the congestion window is `12*MSS` when this loss event occurs. The value of `ssthresh` is then set to `0.5*cwnd = 6*MSS`. 
> - Under TCP Reno, the congestion window is set to `cwnd = 9*MSS` and then grows linearly. 
> - Under TCP Tahoe, the congestion window is set to `1 MSS` and grows exponentially until it reaches the value of `ssthresh` (which is now set to 6), at which point it grows linearly.

### AIMD：线性增、乘性减少

Ignoring the initial slow-start period when a connection begins and assuming that losses are indicated by triple duplicate ACKs rather than timeouts, TCP’s congestion control consists of ***linear (additive) increase in `cwnd` of 1 MSS per RTT and then a halving (multiplicative decrease) of `cwnd` on a triple duplicate-ACK event***. For this reason, TCP congestion control is often referred to as an additive-increase, multiplicative-decrease (AIMD) form of congestion control. 
> 忽略当连接开始时的慢启动阶段，假设所有丢包都由于 3 个冗余 ACK 而不是超时，TCP 的拥塞控制策略可以用 `每 RTT 增加 1MSS 的线性增加策略、每 3个冗余 ACK 事件发生则减半 cwnd 的乘性减少策略` 来概括。

![[30-Transport-layer-AMID.png]]

AIMD congestion control gives rise to the “saw tooth” behavior shown in Figure 3.53, which also nicely illustrates our earlier intuition of TCP “probing” for bandwidth—TCP linearly increases its congestion window size (and hence its transmission rate) until a triple duplicate-ACK event occurs. It then decreases its congestion window size by a factor of two but then again begins increasing it linearly, probing to see if there is additional available bandwidth.
> AIMD 拥塞控制窗口的大小曲线类似一个锯齿形，这很好地描述了早期阶段 TCP 探测带宽、线性避免拥塞直到冗余 ACK 发生这样的策略。

### 总结：TCP 拥塞控制

- 当 `cwnd < ssthresh` ，发送端处于慢启动阶段(slow-start)，窗口指数性增长。
- 当 `cwnd > ssthresh` ，发送端处于拥塞避免阶段(congestion-avoidance)，窗口线性增长。
- 当收到三个冗余 ACKs 时，
	- Reno 策略：`ssthresh = cwnd/2`，`cwnd = ssthresh+3`
	- Tahoe 策略：`ssthresh = cwnd/2`，`cwnd = 1`
- 当超时事件发生时，`ssthresh = cwnd/2`，`cwnd = 1`，进入 SS 阶段

TCP 发送端拥塞控制

| 事件                                  | 状态         | TCP 发送端行为                                                   | 解释                                             |
| ------------------------------------- | ------------ | ---------------------------------------------------------------- | ------------------------------------------------ |
| 以前没有收到 ACK 的 data 被 ACKed     | 慢启动(SS)   | cwnd = cwnd + MSS <br> If (cwnd >ssthresh) <br>    状态变成 “CA” | 每一个 RTT，cwnd 加倍                            |
| 以前没有收到 ACK 的 data 被 ACKed     | 拥塞避免(CA) | cwnd = cwnd + MSS*(MSS/cwnd)                                     | 加性增加，每一个 RTT 对 cwnd 加一个1MSS          |
| 通过收到3个重复的 ACK，发现丢失的事件 | SS or CA     | ssthresh = cwnd/2 <br> cwnd = ssthresh+3 <br> 状态变成“CA”       | 快速重传，实现乘性的减。<br> cwnd 没有变成1MSS。 |
| 超时                                  | SS or CA     | ssthresh = cwnd/2 <br> cwnd = 1MSS <br> 状态变成“SS”             | 进入 slow start                                  | 
| 重复的 ACK                             | SS or CA     | 对被 ACKed 的 segment，增加重复 ACK 的计数                            | cwnd and ssthresh 不变                       |

### TCP Cubic

Given TCP Reno’s additive-increase, multiplicative-decrease approach to congestion control, one might naturally wonder whether this is the best way to “probe” for a packet sending rate that is just below the threshold of triggering packet loss. 
> TCP Reno 的策略是最佳吗？

Indeed, cutting the sending rate in half (or even worse, cutting the sending rate to one packet per RTT as in an earlier version of TCP known as TCP Tahoe) and then increasing rather slowly over time may be overly cautious. If the state of the congested link where packet loss occurred hasn’t changed much, then perhaps it’s better to more quickly ramp up the sending rate to get close to the pre-loss sending rate and only then probe cautiously for bandwidth. 
> 将发送速率砍半再线性增加的策略未免过于保守。如果拥塞链路的状态没有多大变化，那么快速地增加发送速率接近之前丢包时的速率，再小心地探索发送带宽，可能更佳。

This insight lies at the heart of a flavor of TCP known as TCP CUBIC `[Ha 2008, RFC 8312]`. TCP CUBIC differs only slightly from TCP Reno. Once again, the congestion window is increased only on ACK receipt, and the slow start and fast recovery phases remain the same. 
> TCP CUBIC 策略大部分与 Reno 策略一致。

CUBIC only changes the congestion avoidance phase, as follows: 
- Let $W_{max}$ be size of TCP’s congestion control window when loss was last detected, and let $K$ be the future point in time when TCP CUBIC’s window size will again reach $W_{max}$, assuming no losses. Several tunable CUBIC parameters determine the value $K$, that is, how quickly the protocol’s congestion window size would reach $W_{max}$. 
- CUBIC ==increases the congestion window as a function of cube of the distance between the current time==, $t$, and $K$. Thus, when $t$ is further away from $K$, the congestion window size increases are much larger than when $t$ is close to $K$. That is, CUBIC quickly ramps up TCP’s sending rate to get close to the pre-loss rate, $W_{max}$, and only then probes cautiously for bandwidth as it approaches $W_{max}$. 
- When $t$ is greater than $K$, the cubic rule implies that CUBIC’s congestion window increases are small when $t$ is still close to $K$ (which is good if the congestion level of the link causing loss hasn’t changed much) but then increases rapidly as $t$ exceeds $K$ (which allows CUBIC to more quickly find a new operating point if the congestion level of the link that caused loss has changed significantly). 

![[30-Transport-layer-TCP-cubic.png]]
Under these rules, the idealized performance of TCP Reno and TCP CUBIC are compared in Figure 3.54, adapted from `[Huston 2017]`. We see the slow start phase ending at $t_0$. Then, when congestion loss occurs at $t_1$, $t_2$, and $t_3$, CUBIC more quickly ramps up close to $W_{max}$ (thereby enjoying more overall throughput than TCP Reno). ==We can see graphically how TCP CUBIC attempts to maintain the flow for as long as possible just below the (unknown to the sender) congestion threshold==. 
> TCP CUBIT 策略能够尽力维持略低于拥塞发生时的发送速率，从而提高平均吞吐率，在后文可以看到 Reno 的线性策略，使得平均吞吐率仅 $\frac{3}{4}\times \frac{W_{max}}{RTT}$

Note that at $t_3$, the congestion level has presumably decreased appreciably, allowing both TCP Reno and TCP CUBIC to achieve sending rates higher than $W_{max}$. 
> 在拥塞解除后，Reno 策略一开始增长速度线性是快于 CUBIC 策略的，但很快 CUBIC 策略就会反超，并更快地到达下一个拥塞点。

TCP CUBIC has recently gained wide deployment. While measurements taken around 2000 on popular Web servers showed that nearly all were running some version of TCP Reno `[Padhye 2001]`, more recent measurements of the 5000 most popular Web servers shows that nearly 50% are running a version of TCP CUBIC `[Yang 2014]`, which is also the default version of TCP used in the Linux operating system.

### 宏观视角的 TCP Reno 吞吐量

对估算作出的假设：
- TCP 的平均吞吐量（平均发送速率）是拥塞窗口 w 和当前 RTT 的函数
    - 忽略慢启动阶段，因为这一阶段的持续时间非常短
    - 假设发送端总有数据传输
- 当前窗口长度是 w 字节，且当前往返时间是 RTT 秒时，TCP 的发送速率大概是 w/RTT
- TCP 通过每经过一个 RTT 将 w 增加一个 MSS 探测出带宽，直到丢包事件发生为止，当一个丢包事件发生时，用 W 表示 w 的值。
- 假设连接持续期间 RTT 和 W 保持稳定，那么 TCP 的传输速率将在 W/2RTT 和 W/RTT 之间变化

TCP 稳态动态性模型：
1. 速率增长至 W/RTT 时，网络丢弃来自连接的分组
2. 发送速率减半，进而每过一个 RTT 就发送速率增加 MSS/RTT，直到再次增长到 W/RTT 为止，这一过程不断重复
3. 而 TCP 吞吐量在这两个极值之间线性增长，于是平均吞吐量 $Throughput_{avg} = \frac{3}{4}\times \frac{W}{RTT}$

### 高带宽下的 TCP 策略研究
> 这个话题仅是一个拓展性的讨论，并没有考试价值。

The need for continued evolution of TCP can be illustrated by considering the high-speed TCP connections that are needed for grid- and cloud-computing applications. For example, consider a TCP connection with 1,500-byte segments and a 100 ms RTT, and suppose we want to send data through this connection at 10 Gbps. Following `[RFC 3649]`, we note that using the TCP throughput formula above, in order to achieve a 10 Gbps throughput, the average congestion window size would need to be 83,333 segments. That’s a lot of segments, leading us to be rather concerned that one of these 83,333 in-flight segments might be lost. What would happen in the case of a loss? Or, put another way, what fraction of the transmitted segments could be lost that would allow the TCP congestion-control algorithm specified in Figure 3.52 still to achieve the desired 10 Gbps rate? In the homework questions for this chapter, you are led through the derivation of a formula relating the throughput of a TCP connection as a function of the loss rate (L), the round-trip time (RTT), and the maximum segment size (MSS):

$$
\text{average throughput of a connection}=\frac{1.22\times MSS}{RTT\sqrt{L}}
$$

Using this formula, we can see that in order to achieve a throughput of 10 Gbps, today’s TCP congestion-control algorithm can only tolerate a segment loss probability of 2 · 10–10 (or equivalently, one loss event for every 5,000,000,000 segments)—a very low rate. This observation has led a number of researchers to investigate new versions of TCP that are specifically designed for such high-speed environments; see [Jin 2004; Kelly 2003; Ha 2008; RFC 7323] for discussions of these efforts.

### TCP公平性
- 公平性目标：如果 $K$ 个 TCP 会话分享一个链路带宽为 $R$ 的瓶颈，每一个会话的有效带宽为 $R/K$ —— 即每条连接都得到相同份额的链路带宽。
> 瓶颈链路： By bottleneck link, we mean that for each connection, all the other links along the connection’s path are not congested and have abundant transmission capacity as compared with the transmission capacity of the bottleneck link.

> [!example] AIMD 策略是公平的吗？
> ![[30-Transport-layer-TCP-fairness-1.png]]
> Let’s consider the simple case of two TCP connections sharing a single link with transmission rate R, as shown in Figure 3.54. 
> - Assume that the two connections have the same MSS and RTT (so that if they have the same congestion window size, then they have the same throughput), 
> - that they have a large amount of data to send, 
> - and that no other TCP connections or UDP datagrams traverse this shared link. 
> - Also, ignore the slow-start phase of TCP and assume the TCP connections are operating in CA mode (AIMD) at all times.
> > 对公平性探讨作出一些理想性的假设。
> 
> ![[30-Transport-layer-TCP-fairness-2.png]]
> Figure 3.55 plots the throughput realized by the two TCP connections. ==If TCP is to share the link bandwidth equally between the two connections, then the realized throughput should fall along the 45-degree arrow (equal bandwidth share)== emanating from the origin. Ideally, ***the sum of the two throughputs should equal R***. So the goal should be to have the achieved throughputs fall somewhere near the intersection of the equal bandwidth share line and the full bandwidth utilization line in Figure 3.55.
> > 若 TCP 公平性可以保证，那么吞吐量曲线应该是从远点向 45°方向辐射，并且吞吐量总和为 R。
> 
> Suppose that the TCP window sizes are such that at a given point in time, 
> - connections 1 and 2 realize throughputs indicated by point A in Figure 3.55. Because the amount of link bandwidth jointly consumed by the two connections is less than R, ==no loss will occur, and both connections will increase their window== by 1 MSS per RTT as a result of TCP’s congestion-avoidance algorithm. Thus, the joint throughput of the two connections proceeds along a 45-degree line (equal increase for both connections) starting from point A. 
> - Eventually, the link bandwidth jointly consumed by the two connections will be greater than R, and eventually packet loss will occur. Suppose that connections 1 and 2 experience packet loss when they realize throughputs indicated by point B. 
> - Connections 1 and 2 then decrease their windows by a factor of two. The resulting throughputs realized are thus at point C, halfway along a vector starting at B and ending at the origin. Because the joint bandwidth use is less than R at point C, the two connections again increase their throughputs along a 45-degree line starting from C. 
> - Eventually, loss will again occur, for example, at point D, and the two connections again decrease their window sizes by a factor of two, and so on. 
> > 在诸多理想条件的加持下，连接 1 和 2 在全带宽利用率曲线附近来回波动，即从宏观视角而言，AIMD 策略确实做到了公平。
> 
> You should convince yourself that the bandwidth realized by the two connections eventually fluctuates along the equal bandwidth share line. You should also convince yourself that ==the two connections will converge to this behavior regardless of where they are in the two-dimensional space==! Although a number of idealized assumptions lie behind this scenario, it still provides an intuitive feel for why TCP results in an equal sharing of bandwidth among connections.
> 
> In practice, these idealized conditions are typically not met, and ***client-server applications can thus obtain very unequal portions of link bandwidth***. In particular, it has been shown that when multiple connections share a common bottleneck, ==those sessions with a smaller RTT are able to grab the available bandwidth at that link more quickly as it becomes free== (that is, open their congestion windows faster) and thus will enjoy higher throughput than those connections with larger RTTs `[Lakshman 1997]`.
> > 实践中这些理想条件并不总能达成，通常较小 RTT 的连接能在链路空闲时更快地抢到可用带宽，从而享有更高的吞吐量。


- 公平性和UDP
    - 多媒体应用通常不是用TCP
        - 应用发送的数据速率希望不受拥塞控制的节制，而宁愿在网络拥塞时出现丢包
    - 使用UDP：
        - 音视频应用泵出数据的速率是恒定的，忽略数据的丢失
    - 研究领域：TCP 友好性
        - 由于 UDP 会压制 TCP 流量——UDP 不与其他连接合作，也不适时地调整传输速率，一股脑地发送报文到链路中，而不管是否拥塞，
        - 因此如何开发网络拥塞机制以组织 UDP 流量的压制行为，是网络研究的热点。

- 公平性和并行TCP连接
    - 另一个公平性的重要问题：主机间可以打开多个并行的 TCP 连接，具有多个连接的应用事实上地占用了链路中的较大比例带宽（而不是前文的理想性地假设，应用公平地平分链路带宽）
    - 例如：Web 浏览器中，带宽为 R 的链路已经支持了9个连接 
        - 如果新的应用要求建1个 TCP 连接，获得带宽 R/10
        - 如果新的应用要求建11个 TCP 连接，获得带宽 R/2

### 显式拥塞通告：网络辅助拥塞控制

Since the initial standardization of slow start and congestion avoidance in the late 1980’s `[RFC 1122]`, TCP has implemented the form of end-end congestion control that we studied in Section 3.7.1: a TCP sender receives no explicit congestion indications from the network layer, and instead infers congestion through observed packet loss. 

More recently, extensions to both IP and TCP `[RFC 3168]` have been proposed, implemented, and deployed that allow the network to explicitly signal congestion to a TCP sender and receiver. In addition, a number of variations of TCP congestion control protocols have been proposed that infer congestion using measured packet delay. We’ll take a look at both network-assisted and delay-based congestion control in this section. 
> [RFC 3168]中提出、实现了允许网络显式通知 TCP 发送方和接收方拥塞情况的扩展，这一扩展涉及 TCP 和 IP 两大协议。此后大量的 TCP 拥塞控制策略的变体被提出，不过大多数都是基于网络支持和延后分组策略的实现。

#### Explicit Congestion Notification

![[30-Transport-layer-ECN.png]]
Explicit Congestion Notification `[RFC 3168]` is the form of network-assisted congestion control performed within the Internet. As shown in Figure 3.55, ==both TCP and IP are involved==. At the network layer, two bits (with four possible values, overall) in the Type of Service field of the IP datagram header (which we’ll discuss in Section 4.3) are used for ECN. 
> 网络层中使用 IP 数据报首部的服务类型字段中的两个 bit 来实现 ECN。

One setting of the ECN bits is used by a router to indicate that it (the router) is experiencing congestion. This congestion indication is then carried in the marked IP datagram to the destination host, which then informs the sending host, as shown in Figure 3.55.
> ECN 拥塞指示被 IP 数据报发送给目的主机，再由目的主机通知发送主机。

`RFC 3168` does not provide a definition of when a router is congested; that decision is a configuration choice made possible by the router vendor, and decided by the network operator. However, the intuition is that the congestion indication bit can be set to signal the onset of congestion to the send before loss actually occurs.
> ECN 标志是否设立有路由器和网络操作员决定，直觉上设置 ECN 标志，可以在丢包事件实际发生前告知发送端拥塞现象发生。

A second setting of the ECN bits is used by the sending host to inform routers that the sender and receiver are ECN-capable, and thus capable of taking action in response to ECN-indicated network congestion. 
> 发送方主机对 ECN 标志的另一种选择是通知路由器——发送方和接收方都是能够对 ECN 通知采取行动的。

As shown in Figure 3.55, when the TCP in the receiving host receives an ECN congestion indication via a received datagram, the TCP in the receiving host informs the TCP in the sending host of the congestion indication by setting the ECE (Explicit Congestion Notification Echo) bit (see [[#段结构|Figure 3.29]]) in a receiver-to-sender TCP ACK segment. 
> TCP 接收方收到带有 ECN 标记的数据报时，会通过 ECE 标志回送告知 TCP 发送方——拥塞发生了。

The TCP sender, in turn, reacts to an ACK with a congestion indication by halving the congestion window, as it would react to a lost segment using fast retransmit, and sets the CWR (Congestion Window Reduced) bit in the header of the next transmitted TCP sender-to-receiver segment.
> TCP 发送方对具有 ECE 指示的 ACK 做出响应——减少拥塞窗口到之前的一半，并设置下一个发送的报文以 CWR 比特。

Other transport-layer protocols besides TCP may also make use of network layer-signaled ECN. 
- The Datagram Congestion Control Protocol (DCCP) `[RFC 4340]` provides a low-overhead, congestion-controlled UDP-like unreliable service that utilizes ECN. 
- DCTCP (Data Center TCP) `[Alizadeh 2010, RFC 8257]` and DCQCN (Data Center Quantized Congestion Notification) `[Zhu 2015]` designed specifically for data center networks, also makes use of ECN. 
- Recent Internet measurements show increasing deployment of ECN capabilities in popular servers as well as in routers along paths to those servers `[Kühlewind 2013]`

#### Delay-based Congestion Control

Recall from our ECN discussion above that ==a congested router can set the congestion indication bit to signal congestion onset to senders before full buffers cause packets to be dropped at that router==. This allows senders to decrease their sending rates earlier, hopefully before packet loss, thus avoiding costly packet loss and retransmission. ==A second congestion-avoidance approach takes a delay-based approach to also proactively detect congestion onset before packet loss occurs==. 

In TCP Vegas `[Brakmo 1995]`, the sender measures the RTT of the source-to-destination path for all acknowledged packets. Let $RTT_{min}$ be the minimum of these measurements at a sender; this occurs when the path is uncongested and packets experience minimal queueing delay. If TCP Vegas’ congestion window size is `cwnd`, then the uncongested throughput rate would be $cwnd/RTT_{min}$. The intuition behind TCP Vegas is that if the actual sender-measured throughput is close to this value, the TCP sending rate can be increased since (by definition and by measurement) the path is not yet congested. However, if the actual sender-measured throughput is significantly less than the uncongested throughput rate, the path is congested and the Vegas TCP sender will decrease its sending rate. Details can be found in `[Brakmo 1995]`
> TCP Vegas 的策略是测量不拥塞、不排队时最小的 RTT，以此得出不拥塞时吞吐率将是 $cwnd/RTT_{min}$，因此只要吞吐率在这个值附近，表明链路不拥塞，如果远小于这个值，则拥塞发生。

TCP Vegas operates under the intuition that TCP senders should “Keep the pipe just full, but no fuller” `[Kleinrock 2018]`. “Keeping the pipe full” means that links (in particular the bottleneck link that is limiting a connection’s throughput) are kept busy transmitting, doing useful work; “but no fuller” means that there is nothing to gain (except increased delay!) if large queues are allowed to build up while the pipe is kept full.

The BBR congestion control protocol `[Cardwell 2017]` builds on ideas in TCP Vegas, and incorporates mechanisms that allows it compete fairly (see [[#TCP公平性]]) with TCP non-BBR senders. `[Cardwell 2017]` reports that in 2016, Google began using BBR for all TCP traffic on its private B4 network `[Jain 2013]` that interconnects Google data centers, replacing CUBIC. It is also being deployed on Google and YouTube Web servers. Other delay-based TCP congestion control protocols include TIMELY for data center networks `[Mittal 2015]`, and Compound TCP (CTPC) `[Tan 2006]` and FAST `[Wei 2006]` for high-speed and long distance networks.

## 3.8 传输层功能的演进

Our discussion of specific Internet transport protocols in this chapter has focused on UDP and TCP — the two “work horses” of the Internet transport layer. However, as we’ve seen, three decades of experience with these two protocols has identified circumstances in which ==neither is ideally suited, and so the design and implementation of transport layer functionality has continued to evolve==. 

We’ve seen a rich evolution in the use of TCP over the past decade. In Sections 3.7.1 and 3.7.2, we learned that in addition to “classic” versions of TCP such as TCP Tahoe and Reno, there are now several newer versions of TCP that have been developed, implemented, deployed, and are in significant use today. These include TCP CUBIC, DCTCP, CTCP, BBR, and more. Indeed, measurements in `[Yang 2014]` indicate that CUBIC (and its predecessor, BIC `[Xu 2004]`) and CTCP are more widely deployed on Web servers than classic TCP Reno; we also saw that BBR is being deployed in Google’s internal B4 network, as well as on many of Google’s public-facing servers. And there are ==many (many!) more versions of TCP==!
- There are versions of TCP specifically designed for use over wireless links, over high-bandwidth paths with large RTTs, for paths with packet re-ordering, and for short paths strictly within data centers. 
- There are versions of TCP that implement different priorities among TCP connections competing for bandwidth at a bottleneck link, and for TCP connections whose segments are being sent over different source-destination paths in parallel. 
- There are also variations of TCP that deal with packet acknowledgment and TCP session establishment/closure differently than we studied in Section 3.5.6. 

Indeed, it’s probably not even correct anymore to refer to “the” TCP protocol; perhaps ==the only common features of these protocols is that they use the TCP segment format== that we studied in Figure 3.29, and that they should compete “fairly” amongst themselves in the face of network congestion! For a survey of the many flavors of TCP, see `[Afanasyev 2010]` and `[Narayan 2018]`.

### QUIC: Quick UDP Internet Connections

If the transport services needed by an application don’t quite fit either the UDP or TCP service models—perhaps an application ==needs more services than those provided by UDP but does not want all of the particular functionality that comes with TCP==, or may want different services than those provided by TCP—application designers can always “roll their own” protocol at the application layer.
> 在应用层定制个性化的协议。

This is the approach taken in the QUIC (Quick UDP Internet Connections) protocol `[Langley 2017, QUIC 2020]`. Specifically, QUIC is a new application-layer protocol designed from the ground up to improve the performance of transport-layer services for secure HTTP. QUIC has already been widely deployed, although is still in the process of being standardized as an Internet RFC `[QUIC 2020]`. Google has deployed QUIC on many of its public-facing Web servers, in its mobile video streaming YouTube app, in its Chrome browser, and in Android’s Google Search app. With more than 7% of Internet traffic today now being QUIC `[Langley 2017]`, we’ll want to take a closer look. 
> QUIC 协议是新兴的、用于提升安全 HTTP 的性能的应用层协议。现在已经被广泛部署，尽管其标准仍在 RFC 的讨论中。

Our study of QUIC will also serve as a nice culmination of our study of the transport layer, as QUIC uses many of the approaches for reliable data transfer, congestion control, and connection management that we’ve studied in this chapter. 

![[30-Transport-layer-QUIC-HTTP3.png]]

As shown in Figure 3.58, QUIC is an application-layer protocol, ==using UDP as its underlying transport-layer protocol==, and is designed to interface above specifically to a simplified but evolved version of HTTP/2. In the near future, HTTP/3 will natively incorporate QUIC `[HTTP/3 2020]`. 
> QUIC 使用 UDP 作为其传输层的基础，并且提供给上层的简化版 HTTP/2 协议一些接口。HTTP/3 将会吸收 QUIC 协议并进一步构建新的 HTTP 标准。

Some of QUIC’s major features include: 
- ***Connection-Oriented and Secure***. Like TCP, QUIC is a connection-oriented protocol between two endpoints. ==This requires a handshake between endpoints to set up the QUIC connection state==. Two pieces of connection state are the source and destination connection ID. All QUIC packets are encrypted, and as suggested in Figure 3.58, QUIC combines the handshakes needed to establish connection state with those needed for authentication and encryption, thus providing faster establishment than the protocol stack in Figure 3.58 (a), where multiple RTTs are required to first establish a TCP connection, and then establish a TLS connection over the TCP connection.
> QUIC 是面向连接并且安全的。其需要握手来建立端到端的 QUIC 连接状态，QUIC 连接状态中包含源和目标的连接 ID。所有的 QUIC 分组都是加密的。QUIC 将建立连接所需的握手和加密所需的握手结合起来，而 TCP 需要多个 RTT 来建立 TCP 连接，然后再 TCP 连接上再建立 TLS 连接。

- ***Streams***. QUIC allows several different application-level “streams” to be multiplexed through a single QUIC connection, and once a QUIC connection is established, new streams can be quickly added. A stream is an abstraction for the reliable, in-order bi-directional delivery of data between two QUIC endpoints. In the context of HTTP/3, there would be a different stream for each object in a Web page. Each connection has a connection ID, and each stream within a connection has a stream ID; both of these IDs are contained in a QUIC packet header (along with other header information). Data from multiple streams may be contained within a single QUIC segment, which is carried over UDP. The Stream Control Transmission Protocol (SCTP) `[RFC 4960, RFC 3286]` is an earlier reliable, message-oriented protocol that pioneered the notion of multiplexing multiple application-level “streams” through a single SCTP connection. We’ll see in Chapter 7 that SCTP is used in control plane protocols in 4G/5G cellular wireless networks. 

![[30-Transport-layer-HTTP11-vs-HTTP3.png]]

- ***Reliable***, TCP-friendly congestion-controlled data transfer. ==As illustrated in Figure 3.59 (b), QUIC provides reliable data transfer to each QUIC stream separately. Figure 3.59 (a) shows the case of HTTP/1.1 sending multiple HTTP requests, all over a single TCP connection==. Since TCP provides reliable, in-order byte delivery, this means that the multiple HTTP requests must be delivered in-order at the destination HTTP server. Thus, if bytes from one HTTP request are lost, the remaining HTTP requests can not be delivered until those lost bytes are retransmitted and correctly received by TCP at the HTTP server—the so-called [[20-Application-layer#HTTP/1.1 and HOL problem|HOL blocking problem]] that we encountered earlier in Section 2.2.5. Since QUIC provides a reliable in-order delivery on a per-stream basis, a lost UDP segment only impacts those streams whose data was carried in that segment; HTTP messages in other streams can continue to be received and delivered to the application. QUIC provides reliable data transfer using acknowledgment mechanisms similar to TCP’s, as specified in `[RFC 5681]`.

QUIC’s congestion control is based on TCP NewReno `[RFC 6582]`, a slight modification to the TCP Reno protocol that we studied in Section 3.7.1. QUIC’s Draft specification `[QUIC-recovery 2020]` notes “Readers familiar with TCP’s loss detection and congestion control will find algorithms here that parallel well-known TCP ones.” Since we’ve carefully studied TCP’s congestion control in Section 3.7.1, we’d be right at home reading the details of QUIC’s draft specification of its congestion control algorithm!

In closing, it’s worth highlighting again that QUIC is an application-layer protocol providing reliable, congestion-controlled data transfer between two endpoints. The authors of QUIC `[Langley 2017]` stress that this means that changes can be made to QUIC at “application-update timescales,” that is, much faster than TCP or UDP update timescales.