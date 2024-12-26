## 网桥

### 网桥工作原理

### 生成树算法

![[80-Switching-bridges-parallel-links.png]]

To increase reliability, redundant links can be used between bridges. In the example of `Fig.4-35`, there are two links in parallel between a pair of bridges. This design ensures that if one link is cut, the network will not be partitioned into two sets of computers that cannot talk to each other.
> 为了增强可靠性，网桥中使用冗余链路，如上图所示，B1 和 B2 之间有平行的两条链路，这种设计能够在一条链路损坏时，另一条保持可用。

However, this redundancy introduces some additional problems because it creates loops in the topology. An example of these problems can be seen by looking at how a frame sent by A to a previously unobserved destination is handled in `Fig.4-35`. Each bridge follows the normal rule for handling unknown destinations, which is to flood the frame. Call the frame from A that reaches bridge B1 frame F0. The bridge sends copies of this frame out all of its other ports. We will only consider the bridge ports that connect B1 to B2 (though the frame will be sent out the other ports, too). Since there are two links from B1 to B2, two copies of the frame will reach B2. They are shown in `Fig.4-35` as F1 and F2.
> 然而，这种冗余引入了新的问题——拓扑环路，上图中即是一个例子：节点 A 向 B1 发送了一个帧 F0，而网桥 B1 中并没有该帧目的地址相关的表项，于是会向所有端口进行复制、转发该帧，因此分别创建 F1 和 F2 向 网桥 B2 进行发送。

Shortly thereafter, bridge B2 receives these frames. However, it does not (and cannot) know that they are copies of the same frame, rather than two different frames sent one after the other. So bridge B2 takes F1 and sends copies of it out all the other ports, and it also takes F2 and sends copies of it out all the other ports. This produces frames F3 and F4 that are sent along the two links back to B1. Bridge B1 then sees two new frames with unknown destinations and copies them again. This cycle goes on forever.
> 很快，网桥 B2 收到了 F1 和 F2 两个帧，但 B2 并不能区分它们是同一个帧的不同副本，而是当作两个新的帧进行转发，若恰巧 B2 中也没有目的地址相关的表项，于是 B2 就会向所有端口转发——F1 的复制品 F4 又转发回 B1，F2 的复制品 F3 也转发回 B1。那么 B1 收到这两个复制品时会发生什么呢？它仍然未获取目的地址的相关信息，于是继续转发——这个循环将永远持续下去！

The solution to this difficulty is for the bridges to communicate with each other and overlay the actual topology with a spanning tree that reaches every bridge. In effect, some potential connections between bridges are ignored in the interest of constructing a fictitious loop-free topology that is a subset of the actual topology.
> 解决这个问题的办法就是网桥之间进行通信，建立一棵能够到达所有网桥的生成树来覆盖掉实际的物理拓扑结构。在构造虚拟的无环拓扑结构时将网桥之间可能的潜在连接进行强制性的忽略，因此消除了循环转发的问题。

![[80-Switching-spanning-tree.png]]

For example, in `Fig.4-36` we see five bridges that are interconnected and also have stations connected to them. Each station connects to only one bridge. There are some redundant connections between the bridges so that frames will be forwarded in loops if all of the links are used. This topology can be thought of as a graph in which the bridges are the nodes and the point-to-point links are the edges. The graph can be reduced to a spanning tree, which has no cycles by definition, by dropping the links shown as dashed lines in `Fig.4-36`. Using this spanning tree, there is exactly one path from every station to every other station. Once the bridges have agreed on the spanning tree, all forwarding between stations follows the spanning tree. Since there is a unique path from each source to each destination, loops are impossible.
> 上图是五个互联的网桥，其中一些网桥连接两个邻居及以上，因此如果所有链路都使用的话必然出现转发环路，而通过网桥之间协定好一棵支撑树，将部分链路暂停使用，就可以避免环路的出现——任一网桥到其它网桥，都只有单一的路径。

To build the spanning tree, the bridges run a distributed algorithm. Each bridge periodically broadcasts a configuration message out all of its ports to its neighbors and processes the messages it receives from other bridges, as described next. These messages are not forwarded, since their purpose is to build the tree, which can then be used for forwarding.
> 为了建立支撑树，所有网桥都运行分布式的算法——周期性地从所有端口广播配置信息，发送给它的邻居并从邻居接收信息。注意，这个信息并不会被转发，因为它只用于配置生成树。

The bridges must first choose one bridge to be the root of the spanning tree. To make this choice, they each include an identifier based on their MAC address in the configuration message, as well as the identifier of the bridge they believe to be the root. MAC addresses are installed by the manufacturer and guaranteed to be unique worldwide, which makes these identifiers convenient and unique. ==The bridges choose the bridge with the lowest identifier to be the root==. After enough messages have been exchanged to spread the news, all bridges will agree on which bridge is the root. In `Fig.4-36`, bridge B1 has the lowest identifier and becomes the root.
> 网桥们首先需要抉择出作为支撑树的树根的网桥，这依赖于 MAC 地址实现——MAC 地址最低者作为树根，然后将该信息传递给所有网桥。上图中 B1 是 MAC 地址最低者，于是被公认为树根。

Next, a tree of shortest paths from the root to every bridge is constructed. In `Fig.4-36`, bridges B2 and B3 can each be reached from bridge B1 directly, in one hop that is a shortest path. Bridge B4 can be reached in two hops, via either B2 or B3. To break this tie, the path via the bridge with the lowest identifier is chosen, so B4 is reached via B2. Bridge B5 can be reached in two hops via B3. To find these shortest paths, bridges include the distance from the root in their configuration messages. Each bridge remembers the shortest path it finds to the root. The bridges then turn off ports that are not part of the shortest path. 
> 然后，从树根建立一棵最短路径树，通往所有网桥。上图中，B1 直接连接到 B2 和 B3，因此直连的链路被选择；而 B4 可以通过 B2 或 B3 在两跳内到达，选择其中最小标识符的路径，于是 B4 通过 B2 到达；而 B5 只有通过 B3 才能在两跳内到达，因此 B3——B4、B4——B5 之间的链路被选择屏蔽，关掉各自的对应端口。最终，每个网桥都能确定到达根网桥的最短路径。

Although the tree spans all the bridges, not all the links (or even bridges) are necessarily present in the tree. This happens because turning off the ports prunes some links from the network to prevent loops. Even after the spanning tree has been established, the algorithm continues to run during normal operation to automatically detect topology changes and update the tree.
> 建立生成树的过程中必然会删除一些边，并且在网桥运行期间也需要周期性地运行这个算法，以便自动探寻到拓扑的变化并及时更新树。

## 网络设备总览

![[80-Switching-network-devices.png]]

**Network Devices:** Network devices, also known as networking hardware, are physical devices that allow hardware on a computer network to communicate and interact with one another. For example ***Repeater, Hub, Bridge, Switch, Routers, Gateway, Brouter, and NIC***, etc.

### 中继器

***Repeater***:
- A repeater operates at the physical layer.
- *Its job is to regenerate the signal over the same network before the signal becomes too weak or corrupted to extend the length to which the signal can be transmitted over the same network*.
- An important point to be noted about repeaters is that they not only amplify the signal but also regenerate it. When the signal becomes weak, they copy it bit by bit and regenerate it at its star topology connectors connecting following the original strength.
- It is a 2-port device. A signal appearing on one cable is cleaned up, amplified, and put out on another cable.
- Repeaters do not understand frames, packets, or headers. They understand the symbols that encode bits as volts.

> - 中继器**工作在物理层**，其**直接作用在信号**之上，通过**逐位**地**重新生成信号**(而不是直接增强信号强度)来抵消信号传播过程中的电路衰减损耗；
> - 中继器的视角中，没有帧、分组、头等概念，只是通过电平的变化对位进行编码；
> - 中继器是一种两端口设备；

### 集线器

***Hub***:
- A hub is a ==basically multi-port repeater==. A hub connects multiple wires coming from different branches, for example, the connector in star topology which connects different stations. Hubs differ from repeaters in that they do not (usually) amplify the incoming signals and are designed for multiple input lines, but the differences are slight. Like repeaters, hubs are physical layer devices that do not examine the link layer addresses or use them in any way.
- Hubs cannot filter data, so data packets are sent to all connected devices. In other words, ==the collision domain of all hosts connected through Hub remains one==.
- Frames arriving on any of the lines are sent out on all the others. If two frames arrive at the same time, they will collide, just as on a coaxial cable.
- All the lines coming into a hub must operate at the same speed.
- Also, *they do not have the intelligence to find out the best path for data packets* which leads to inefficiencies and wastage. 

> - 集线器其实就是中继器的多端口强化版——能够连接多条电路；
> - 集线器不会隔离冲突域，即连接到一台集线器上的设备既处于同一广播域、又处于同一冲突域；
> - 集线器会从任一端口接收数据帧，然后向其余所有端口转发之，因此当两个数据帧同时到达集线器时，就会发生碰撞；
> - 集线器当然不具备路由选择功能；


更多种类的集线器：
* ***Active Hub***: These are the hubs that have their power supply and can clean, boost, and relay the signal along with the network. It serves both as a repeater as well as a wiring center. These are used to extend the maximum distance between nodes.
* ***Passive Hub***: These are the hubs that collect wiring from nodes and power supply from the active hub. These hubs relay signals onto the network without cleaning and boosting them and can’t be used to extend the distance between nodes.
* ***Intelligent Hub***: It works like an active hub and includes remote management capabilities. They also provide flexible data rates to network devices. It also enables an administrator to monitor the traffic passing through the hub and to configure each port in the hub.

> - 主动集线器：能够主动地对信号选择清除、增强、转发等操作，因此既作为 repeater 又作为 wiring center(星型拓扑)；
> - 被动集线器：转发来自主动集线器的信号，没有清除和增强的能力，不能延长节点间的长度；
> - 智能集线器：主动集线器+远程管理员可控制，提供动态的数据速率，并且允许管理员监测流量。

### 网桥

***Bridge***:
- A bridge operates at the data link layer.
- A bridge is a repeater, ==with add on the functionality of filtering content by reading the MAC addresses of the source and destination==.
- It is also used for interconnecting two LANs working on the same protocol. ==A bridge connects two or more LANs==. Like a hub, a modern bridge has multiple ports, usually enough for 4 to 48 input lines of a certain type. Unlike in a hub, each port is isolated to be its own collision domain; if the port has a full-duplex point-to-point line, the CSMA/CD algorithm is not needed.
- When a frame arrives, the bridge extracts the destination address from the frame header and looks it up in a table to see where to send the frame. For Ethernet, this address is the 48-bit destination address (MAC address). 
- The bridge only outputs the frame on the port where it is needed and can forward multiple frames at the same time. 
> - 网桥是链路层设备，是在中继器的基础上增加了依据 MAC 地址进行内容过滤的能力；
> - 网桥能够连接多个 LAN，并且每个端口都自成冲突域，即连接到网桥中同一端口的设备处于同一冲突域，连接到不同端口的设备则处于不同冲突域，因此如果连接到端口的电路是全双工的，那么就不必使用 CSMA/CD 算法；
> - 当数据帧到达时，网桥从中提取 MAC 地址，并进行查转发表、转发到特定段口的动作；（当然，如果转发表中没有对应目的地址的表项，就会执行广播）
> - 网桥只能隔离冲突域，而不能隔离广播域，因此存在广播风暴 (broadcast storm) 问题；

- Bridges offer much better performance than hubs, and the isolation between bridge ports also means that the input lines may run at different speeds, possibly even with different network types. A common example is a bridge with ports that connect to 10-, 100-, and 1000-Mbps Ethernet.
- Buffering within the bridge is needed to accept a frame on one port and transmit the frame out on a different port. If frames come in faster than they can be retransmitted, the bridge may run out of buffer space and have to start discarding frames. For example, if a gigabit Ethernet is pouring bits into a 10-Mbps Ethernet at top speed, the bridge will have to buffer them, hoping not to run out of memory. This problem still exists even if all the ports run at the same speed because more than one port may be sending frames to a given destination port.
> - 网桥的隔离性能够支持不同端口运行不同速率、甚至不同协议的链路；
> - 网桥需要缓冲空间，如果数据帧到达速率比转发速率更快，缓冲空间就会耗尽并不得不丢弃帧；
> - 即使所有端口都以相同的速率运行，缓冲空间耗尽的问题仍然存在，因为有可能大量端口都向同一个端口转发；

- Bridges were originally intended to be able to join different kinds of LANs, for example, an Ethernet and a Token Ring LAN. However, this never worked well because of differences between the LANs. Different frame formats require copying and reformatting, which takes CPU time, requires a new checksum calculation, and introduces the possibility of undetected errors due to bad bits in the bridge’s memory. Different maximum frame lengths are also a serious problem with no good solution. Basically, frames that are too large to be forwarded must be discarded. So much for transparency.
> - 网桥尽管能够连接多种 LAN，例如以太网和令牌环 LAN，但这会引发潜在的问题：诸如不同的帧格式需要不同的处理方法、校验和计算、网桥内存出错而引发差错；
> - 大到难以转发的帧通常会被网桥丢弃；

- Two other areas where LANs can differ are security and quality of service. Some LANs have link-layer encryption, for example 802.11, and some do not, for example Ethernet. Some LANs have quality of service features such as priorities, for example 802.11, and some do not, for example Ethernet. Consequently, when a frame must travel between these LANs, the security or quality of service expected by the sender may not be able to be provided. For all of these reasons, modern bridges usually work for one network type, and routers, which we will come to soon, are used instead to join networks of different types.
> - 不同 LAN 的差异还体现在安全性和服务质量 QoS 上，例如 802.11 (无线局域网)要求链路加密和 QoS 特性，而以太网不要求；
> - 因此，当数据帧必须在不同 LAN 间穿过时，安全性或 QoS 难以得到预期的保证，于是现代网桥通常只服务于一种网络类型，通过路由器来连接不同网络；

#### 网桥的分类

* ***Transparent Bridges*** These are the bridge in which the stations are completely unaware of the bridge’s existence i.e. whether or not a bridge is added or deleted from the network, reconfiguration of the stations is unnecessary. These bridges make use of two processes i.e. bridge forwarding and bridge learning.
* **Source Routing Bridges** In these bridges, routing operation is performed by the source station and the frame specifies which route to follow. The host can discover the frame by sending a special frame called the discovery frame, which spreads through the entire network using all possible paths to the destination.
* More：[Network bridge - Wikipedia](https://en.wikipedia.org/wiki/Network_bridge?useskin=vector#Shortest_Path_Bridging)

### 交换机

***Switch***:
- Switches are ==modern bridges by another name==. The differences are more to do with marketing than technical issues, but there are a few points worth knowing. Bridges were developed when classic Ethernet was in use, so they tend to join relatively few LANs and thus have relatively few ports. The term "switch" is more popular nowadays. Also, modern installations all use point-to-point links, such as twisted-pair cables, so individual computers plug directly into a switch and thus the switch will tend to have many ports. Finally, "switch" is also used as a general term. With a bridge, the functionality is clear. On the other hand, a switch may refer to an Ethernet switch or a completely different kind of device that makes forwarding decisions, such as a telephone switch.
- The switch can perform error checking before forwarding data, which makes it very efficient as it does not forward packets that have errors and forward good packets selectively to the correct port only. In other words, the switch divides the collision domain of hosts, but the broadcast domain remains the same. 

> - 交换机其实就是网桥换了个马甲——更多的区别在市场上，而非技术上；
> - 不过现代交换机已经与传统交换机有了很大的区别，如今更加智能、高级、可靠：提供差错检验，以及早丢弃错误的帧避免白白转发；提供冲突域隔离，但不提供广播域隔离；

#### 交换机的种类

1. ***Unmanaged switches***: These switches have a simple plug-and-play design and do not offer advanced configuration options. They are suitable for small networks or for use as an expansion to a larger network.
2. ***Managed switches***: These switches offer advanced configuration options such as VLANs, QoS, and link aggregation. They are suitable for larger, more complex networks and allow for centralized management.
3. ***Smart switches***: These switches have features similar to managed switches but are typically easier to set up and manage. They are suitable for small- to medium-sized networks.
4. ***Layer 2 switches***: These switches operate at the Data Link layer of the OSI model and are responsible for forwarding data between devices on the same network segment.
5. ***Layer 3 switches***: These switches operate at the Network layer of the OSI model and can route data between different network segments. They are more advanced than Layer 2 switches and are often used in larger, more complex networks.
6. ***PoE switches***: These switches have Power over Ethernet capabilities, which allows them to supply power to network devices over the same cable that carries data.
7. ***Gigabit switches***: These switches support Gigabit Ethernet speeds, which are faster than traditional Ethernet speeds.
8. ***Rack-mounted switches***: These switches are designed to be mounted in a server rack and are suitable for use in data centers or other large networks.
9. ***Desktop switches***: These switches are designed for use on a desktop or in a small office environment and are typically smaller in size than rack-mounted switches.
10. ***Modular switches***: These switches have modular design, which allows for easy expansion or customization. They are suitable for large networks and data centers.

### 路由器

**Routers**:
- A router is a device like a switch that ==routes data packets based on their IP addresses==.
- The router is mainly a Network Layer device.
- Routers normally connect LANs and WANs and have a dynamically updating routing table based on which they make decisions on routing the data packets.
- The router divides the broadcast domains of hosts connected through it.
> - 路由器与交换机通过 MAC 地址转发不同，其通过 IP 地址进行转发；
> - 路由器是网络层设备，能够连接不同 LAN 和 WAN，并且能够动态地更新路由表以作出当下最合适的分组转发决策；
> - 路由器能够对广播域进行划分，因此其能够抑制广播风暴；

![[80-Switching-network-devices-1.png]]

### 网关

**Gateway**
- A gateway, as the name suggests, is a passage to connect two networks that may work upon different networking models. They work as messenger agents that take data from one system, interpret it, and transfer it to another system.
- Gateways are also called protocol converters and can operate at any network layer. Gateways are generally more complex than switches or routers.
> - 网关，能够连接两个可能运行不同网络模型/协议的网络，其作为信息代理，从一个系统获取数据，解释、翻译后转发向另一个系统；
> - 网关也称为协议转换器，能够运行在任意 OSI 层，其比交换机和路由器更加复杂；

### 桥接路由器

**Brouter**:
- It is also known as the bridging router is a device that ==combines features of both bridge and router==.
- It can work either at the data link layer or a network layer. Working as a router, it is capable of routing packets across networks and working as the bridge, it is capable of filtering local area network traffic. 

> 其实就是三层交换机

### 网卡

**NIC**:
- NIC or network interface card is a network adapter that is used to connect the computer to the network. It is installed in the computer to establish a LAN.
- It has a unique id that is written on the chip, and it has a connector to connect the cable to it. The cable acts as an interface between the computer and the router or modem.
- NIC card is a layer 2 device which means that it works on both the physical and data link layers of the network model. 

> - 网卡就是安装在计算机内部的网络适配器，能够同 LAN 建立连接；
> - 每个网卡都有出厂时写入的唯一 ID（即 MAC 地址）
> - 网卡是链路层设备；

## VLAN

