---
publish: "true"
tags:
  - Network
  - 计算机网络原理
date: 2023-10-16
---
## 本章内容：
- 应用层协议原理；
- 各类协议：Web and HTTP、FTP、Email 的 SMTP/POP3/IMAP、DNS；
- P2P 应用；
- CDN；
- TCP、UDP 套接字编程

### 目标：
- 网络应用的原理：网络应用协议的概念和实现方面
    - 传输层的服务模型
    - 客户-服务器模式
    - 对等模式(peer-to-peer)
    - 内容分发网络
- 网络应用的实例：互联网流行的应用层协议
    - HTTP、FTP、SMTP/POP3/IMAP、DNS
- 编程：网络应用程序
    - Socket API

一些网络应用的例子：E-mail、Web、文本消息、远程登录、P2P文件共享、即时通信、多用户网络游戏、流媒体（YouTube，Hulu，Netflix）、Internet电话、实时电视会议、社交网络、搜索……

### 创建一个新的网络应用
- 编程
    - 在不同的端系统上运行
    - 通过网络基础设施提供的服务，应用进程彼此通信
    - 如Web：Web服务器软件与浏览器软件通信
- 网络核心中没有应用层软件
    - 网络核心没有应用层功能
    - 网络应用只在端系统上存在，快速网络应用开发和部署

## 2.1 应用层协议原理

### 网络应用程序体系结构
- 客户-服务器模式（C/S:client/server）
- 对等模式（P2P:Peer To Peer）
- 混合体：客户-服务器和对等体系结构

![[20-Application-layer-cs-p2p.png]]
客户-服务器（C/S）体系结构：服务器是中心，资源在服务器，客户端请求服务器传回资源
- 服务器：
    - 一直运行
    - ==固定的 IP 地址和周知的端口号（约定）==，让客户端可以找到
    - 扩展性：服务器场
        - 数据中心进行扩展
        - 扩展性差，随访问用户的增加在达到一定阈值后性能急剧下降而非正常平滑下降
    - 可靠性差：若服务器宕机，客户端就享受不到它的服务
- 客户端：
    - 主动与服务器通信
    - 与互联网有间歇性的连接
    - 可能是动态IP地址
    - 不直接与其它客户端通信

对等体（P2P）体系结构
- （几乎）没有一直运行的服务器
- 任意端系统之间可以进行通信
- **每一个节点既是客户端又是服务器**
    - 自扩展性-新peer节点带来新的服务能力，当然也带来新的服务请求，性能可以维持在一定程度
- 参与的主机间歇性连接且可以改变IP地址
    - 难以管理
    - 安全性、性能、可靠性等受到挑战
- 例子：Gnutella，(P2P 协助下载加速器)迅雷，(文件共享)BitTorrent，(因特网电话和视频会议)Skype

C/S和P2P体系结构的混合体
- Napster: _Napster_ was a peer-to-peer file sharing application that originally launched on June 1, 1999 with an emphasis on digital audio file distribution.
    - 文件搜索：集中
        - 主机在中心服务器上注册其资源，主机上线时报告IP及其拥有的资源
        - 主机向中心服务器查询资源位置，在获取位置后向目标主机请求资源
    - 文件传输：P2P
        - 任意Peer节点之间
- 即时通信
    - 在线检测：集中
        - 当用户上线时，向中心服务器注册其IP地址
        - 用户与中心服务器联系，以找到其在线好友的位置
    - 两个用户之间聊天：P2P

### 进程通信
- 进程：在主机上运行的应用程序
- 在同一个主机内，使用进程间通信机制通信（操作系统定义）
- 不同主机，通过交换报文(Message)来通信
    - ==使用OS提供的通信服务==
    - ==按照应用协议交换报文==
        - 借助传输层提供的服务
    - 注意：P2P 架构的应用也有客户端进程和服务器进程之分，在每个对话上对等体如果首先发起请求则是客户端，如果接收请求则是服务器(P2P 的每个设备既是客户端又是服务端)

客户端(client)进程：发起通信的进程，主动；
服务器(server)进程：等待连接的进程，被动。

**分布式进程通信需要解决的问题**
- 问题1：==进程标示和寻址问题==（服务用户），使客户端能够找到服务器
- 问题2：==传输层-应用层提供什么样的服务==（服务）
    - 位置：层间界面的服务访问点SAP（TCP/IP：socket）
    - 形式：应用程序接口API（TCP/IP：socket API）
- 问题3：==如何使用传输层提供的服务，实现应用进程之间的报文交换，实现应用==（用户使用服务）
    - 定义应用层协议：报文格式，解释，时序等
    - 编制程序，使用OS提供的API，调用网络基础设施提供通信服务传报文，实现应用时序等；

#### 问题1：对进程进行编址(addressing)——主机IP、TCP or UDP、端口号
- 进程为了接收报文，必须有一个标识即：SAP（发送也需要标示）
    - 主机：唯一的32位IP地址
        - 仅仅有 IP 地址不能够唯一标示一个进程——在一台端系统上有很多应用进程在运行
    - 所采用的传输层协议：TCP or UDP
    - **端口号**(Port Numbers)：传输层引入端口号来区分进程（TCP、UDP各 $16bit$ 即 $2^{16}=65536$ 个端口号）
- 一些知名应用所用协议和端口号的例子（惯例）：
    - HTTP：TCP 80； Mail：TCP 25； ftp：TCP 2
- 一个进程：用IP+port标示端节点（IP上的某个TCP端口）
- 本质上，一对主机进程之间的通信由2个端节点构成

#### 问题2：传输层提供的服务

<mark style="background: #FFB8EBA6;">需要穿过层间的信息</mark>
- **层间接口必须要携带的信息**
    - 要传输的==报文==（对于本层来说：SDU）
    - ==谁传的==（发送方）：对方的应用进程的标示：IP+TCP/UDP端口
    - ==传给谁==（接收方）：对方的应用进程的标示：对方的IP+TCP/UDP端口号
- **传输层实体**（tcp或者udp实体）根据这些信息进行TCP报文段/UDP数据报的封装
    - 源端口号，目标端口号，数据等
    - 将IP地址往下交IP实体，用于封装IP数据报：源IP，目标IP

<mark style="background: #FFB8EBA6;">层间信息的代表</mark>
- 如果Socket API每次传输报文，都携带如此多的信息，太繁琐易错，不便于管理
- 用个==代号标示通信的双方或者单方：socket==，可以综合在一起，减小信息量，提高效率
- 就像 OS 打开文件返回的文件描述符一样
    - 对文件描述符的操作，就是对文件的操作

>[!warning] 对传输层 socket 控制有限
>应用程序开发者可以控制 socket 在应用层端的一切，但是对该套接字的传输层一端几乎没有控制权。
>应用程序开发者可以控制的传输层权限仅有：
>1. 选择什么传输层协议
>2. 设定个别的传输层参数，如最大缓存、最大报文段长度等


- **TCP socket**：
    - TCP服务，两个进程之间的通信需要之前要建立连接
        - 两个进程通信会持续一段时间，通信关系稳定
    - 可以用一个整数表示两个应用实体之间的通信关系，本地标示 *注：C语言定义一个整型变量int socket_fd用来存放socket，所以==socket本质上是一个整数，这个整数在TCP中就代表了一个包括源IP和端口号、目标IP和端口号的四元组，在UDP中就代表源IP和源端口号的二元组==*
    - 穿过层间接口的信息量最小
    - TCP socket：源IP，源端口，目标IP，目标端口

**TCP之上的套接字(socket)**
- 对于使用面向连接服务（TCP）的应用而言，套接字是四元组的一个具有**本地意义的标示**（==只有自己操作系统知道，即只有自己的应用层和传输层知道，网络层以下和对方都不知道==。建立连接时操作系统返回一个整数代表双方的IP和端口信息，发送时就可以查一张表确定双方的IP和端口信息，接受时也可以通过建立起来的socket的表找到这个socket的对应值）
    - 四元组：(源IP, 源port, 目标IP, 目标port)
    - 唯一的指定了一个会话（2个进程之间的会话关系）
    - 应用使用这个标示，与远程的应用进程通信
    - 不必在每一个报文的发送都要指定这四元组
    - 就像使用操作系统打开一个文件，OS 返回一个文件描述符一样，以后使用这个文件描述符，而不是使用这个文件的目录名、文件名
    - 简单，便于管理
![[20-Application-layer-tcp-socket.png]]


**层间信息代码**
- UDP socket：
    - UDP服务，两个进程之间的通信需要之前无需建立连接
        - 每个报文都是独立传输的
        - 前后报文可能给不同的进程
    - 因此，==只能用一个整数表示本应用实体的标示==
        - 因为这个报文可能传给另外一个分布式进程
    - 穿过层间接口的信息大小最小，便于管理
    - UDP socket：本IP，本端口
    - 但是传输报文时：必须要提供对方IP，port（*注：传输报文时，==TCP实际上传两样：报文+socket；UDP实际上传三样：报文+socket+目标地址信息（IP+port）==，因为在TCP中socket已经包含目标地址信息*）
        - 接收报文时：传输层需要上传对方的IP，port

**UDP之上的套接字(socket)**
- 对于使用无连接服务（UDP）的应用而言，套接字是二元组的一个具有本地意义的标示
    - 二元组：IP，port（源端指定）
    - UDP套接字指定了应用所在的一个端节点(end point)
    - 在发送数据报时，采用创建好的本地套接字（标示ID），就不必在发送每个报文中指明自己所采用的ip和port
    - 但是在发送报文时，必须要指定对方的ip和udp port（构成另外一个端节点）

<img src="http://knight777.oss-cn-beijing.aliyuncs.com/img/image-20210722162951507.png" style="zoom:80%"/>

**套接字(Socket)**
- 进程向套接字发送报文或从套接字接收报文
- 套接字 <-> 门户
    - 发送进程将报文推出门户，发送进程依赖于传输层设施在另外一侧的门将报文交付给接受进程
    - 接收进程从另外一端的门户收到报文（依赖于传输层设施）

#### 问题3：如何使用传输层提供的服务实现应用
- 定义应用层协议：报文格式，解释，时序等
- 编制程序，通过API调用网络基础设施提供通信服务传报文，解析报文，实现应用时序等

### 应用层协议
- 定义了：==运行在不同端系统上的应用进程如何相互交换报文==，与网络交互相关，是==应用的一部分==，又叫应用实体（注：**实体**指的是仅仅和网络交互有关的这部分内容）
    - 交换的报文类型：请求和应答报文
    - 各种报文类型的语法：报文中的各个字段及其描述
    - 字段的语义：即字段取值的含义
    - 进程何时、如何发送报文及对报文进行响应的规则
- 应用协议仅仅是应用的一个组成部分
    - Web应用：HTTP协议，web客户端，web服务器，HTML
- 公开协议：
    - 由RFC文档定义
    - 允许互操作
    - 如HTTP，SMTP
- 专用（私有）协议：
    - 协议不公开
    - 如：Skype

### 需要的传输层功能
1. **数据丢失率**
	- 有些应用则要求100%的==可靠数据传输==（如文件）
	- 有些应用（如音频）==能容忍一定比例以下的数据丢失==
2. **延迟**
	- 一些应用出于有效性考虑，对数据传输有严格的时间限制，如多媒体应用等对延迟较为敏感
	- Internet 电话、交互式游戏
	- 延迟、延迟差
3. **吞吐量**（吞吐量取决于瓶颈链路）
	- ==带宽敏感应用==：需要最小限度的吞吐，从而使得应用能够有效运转(仍然主要是多媒体应用，如因特网电话若接受的吞吐量还不到一半，是几乎没什么用的)
	- 弹性应用：能充分利用可供使用的吞吐量、带宽的应用（当然也不会嫌吞吐量过多）
4. **安全性**
	- 机密性
	- 完整性
	- 可认证性（鉴别）

常见应用对传输服务的要求

|    应用    | 数据丢失率 |                   吞吐                   | 时间敏感性 | 
|:----------:|:----------:|:----------------------------------------:|:----------:|
|  文件传输  |  不能丢失  |                   弹性                   |     不     |
|   email    |  不能丢失  |                   弹性                   |     不     |
|  Web文档   |  不能丢失  |                   弹性                   |     不     |
| 实时音视频 |  容忍丢失  | 音频：5kbps-1Mbps<br>视频：100kbps-5Mbps | 是，100ms  |
| 存储音视频 |  容忍丢失  |                   同上                   |  是，几秒  |
| 交互式游戏 |  容忍丢失  |              几kbps-10kbps               | 是，100ms  |
|  即时讯息  |  不能丢失  |                   弹性                   |  是和不是  |

### Internet传输层提供的服务
- **TCP服务**：两者之间能够交互协调
    - 包括==面向连接服务==和==可靠数据传输服务==
    - 流量控制：发送方不会淹没接受方
    - 拥塞控制：感知路径上的拥塞程度，当网络出现拥塞时，能抑制发送方
    - **不能提供的服务**：时间保证、最小吞吐保证和安全
    - 面向连接：
	    - 应用层数据报文开始流动之前，要求在客户端进程和服务器进程之间建立连接——交换传输层控制信息。
	    - 通过握手过程建立客户和服务器间的连接，提醒它们应对接下来的大量分组；
	    - 连接是全双工的，并且在应用程序结束报文发送时，必须拆除连接
	    - 连接仅仅体现在端系统上，在网络核心中没有主机对通信关系的维护
- **UDP服务**：
    - 不可靠数据传输
    - 不提供的服务：可靠，流量控制、拥塞控制、时间、带宽保证、建立连接、报文到达的顺序
    - 仅提供最小的、尽力的服务
    - 如果网络情况较好，UDP 可以比 TCP 更快地传送数据，但若情况糟糕，UDP 的可靠性、传输速度都要大打折扣

>[!note] 为什么要有 UDP？UDP 存在的必要性
>- 能够**区分不同的进程**，而 IP 服务不能
>	- 在 IP 提供的主机到主机端到端功能的基础上，区分了主机的应用进程
>- **无需建立连接**，省去了建立连接时间，适合事务性的应用
>- **不做可靠性的工作**，例如检错重发，适合那些对实时性要求比较高而对正确性要求不高的应用
>	- 因为为了实现可靠性（准确性、保序等），必须付出时间代价（检错重发）
>- 没有拥塞控制和流量控制，**应用能够按照设定的速度发送数据**
>	- 而在 TCP 上面的应用，应用发送数据的速度和主机向网络发送的实际速度是不一致的，因为有流量控制和拥塞控制

Internet 应用及其应用层协议和传输协议

|     应用     |         应用层协议         | 下层的传输协议 | 
|:------------:|:--------------------------:|:--------------:|
|    email     |      SMTP [RFC 2821]       |      TCP       |
| 远程终端访问 |      Telnet [RFC 854]      |      TCP       |
|     Web      |      HTTP [RFC 2616]       |      TCP       |
|   文件传输   |       FTP [RFC 959]        |      TCP       |
|    流媒体    | 专用协议（如RealNetworks） |    TCP或UDP    |
| Internet电话 |  专用协议（如Net2Phone）   |    TCP或UDP    |

>[!note] 安全 TCP
>- TCP & UDP
>	- 都没有加密
>	- 明文通过互联网传输，甚至密码
>	- 此时需要在 TCP 之上加一个安全套接字层 (SSL)加强 TCP 的安全性，为 App 提供安全的通信服务，包括服务器端的认证、客户端的认证、私密传输、报文的完整性
>- SSL
>	- 在 TCP 上面实现，提供加密的 TCP 连接
>	- 私密性
>	- 数据完整性
>	- 端到端的鉴别
>- SSL 在应用层
>	- 应用采用 SSL 库，SSL 库使用 TCP 通信
>- SSL socket API
>	- 应用通过 API 将明文交给 socket，SSL 将其加密在互联网上传输

## 2.2 Web and HTTP

Web是一种应用，HTTP是支持Web应用的协议。

一些术语
- **Web页**：由一些**对象**组成
- 对象可以是HTML文件、JPEG图像、Java小程序、声音剪辑文件等
- Web页含有一个**基本的HTML文件**，该基本HTML文件又包含若干对象的引用（链接）
- 通过**URL**（通用资源定位符）对每个对象进行引用
    - **访问协议（HTTP、FTP等等），用户名，口令字，端口等；**
    - 匿名访问时可以不提供用户名和口令
- URL 格式: `Prot://user:psw@www.someSchool.edu:port/someDept/pic.gif`
	- 其中 Prot 为 协议名，
	- user: psw 为 用户: 口令，
	- `www.someSchool.edu` 为 主机名，
	- port 为 端口，
	- someDept/pic.gif 为文件路径名

### HTTP概况
- Web 的应用层协议：Web 的核心，由两个程序实现——客户程序和服务器程序；
- 客户/服务器模式
    - 客户：在建立TCP连接的基础上，请求、接收和显示Web对象的浏览器，如IE浏览器、Google浏览器、360浏览器等
    - 服务器：对请求进行响应，发送对象的Web服务器，如Apache服务器、RRS服务器等

HTTP ==定义了 Web 客户向 Web 服务器请求 Web 页面的方式，以及服务器向客户传送 Web 页面的方式==：
![[20-Application-layer-HTTP-request-response.png]]

- HTTP 使用 TCP 作为支撑运输协议:
    - 客户发起一个与服务器的TCP连接（建立套接字），端口号为80
    - 服务器接受客户的 TCP 连接，建立起连接后通过套接字接口访问TCP
    - 在浏览器（HTTP客户端）与Web服务器（HTTP服务器server）交换HTTP报文（应用层协议报文）
    - TCP连接关闭
>[!note] socket 监听端口
>初始时 Web 服务器有一个 socket 作为监听描述符监听端口并在新请求过来时创建与之对应的新的 socket，则在 n 个客户端请求连接时又产生 n 个 socket 作为各个连接的已连接描述符，负责服务器与各个客户端之间的连接状态，而第一个守候 socket 继续等待新的请求并连通

- HTTP 是无状态协议 stateless protocol!
    - 服务器向客户发送被请求的文件，但不维护关于客户的任何信息
    - If a particular client asks for the same object twice in a period of a few seconds, the server does not respond by saying that it just served the object to the client; instead, the server resends the object, as it has ==completely forgotten what it did earlier==.

>[!example] 不同版本的 HTTP
>- The original version of HTTP is called HTTP/1.0 and dates back to the early 1990’s [RFC 1945]. 
>- As of 2020, the majority of HTTP transactions take place over HTTP/1.1 [RFC 7230].
>- However, increasingly browsers and Web servers also support a new version of HTTP called HTTP/2 [RFC 7540]. At the end of this section, we’ll provide an introduction to HTTP/2.

>[!warning] 维护状态的协议很复杂！
>- 必须维护历史信息（状态）
>- 如果服务器/客户端死机，它们的状态信息可能不一致，二者的信息必须是一致
>- 无状态的服务器能够支持更多的客户端

### HTTP连接
- 非持久HTTP
    - 最多只有一个对象 (所谓对象，就是请求-响应对)在 TCP 连接上发送
    - 下载多个对象需要多个TCP连接
    - HTTP/1.0使用非持久连接
- 持久HTTP
    - 多个对象可以在一个（在客户端和服务器之间的）TCP连接上传输
    - HTTP/1.1默认使用持久连接

>[!tip] 连接关闭否？
>非持久 HTTP 在每次正式发送和响应请求前都要先建立 TCP 连接，在报文发送完毕后 TCP 连接即关闭；而持久 HTTP 在报文发送完成后连接不关闭，可继续发送和接收报文。

#### 非持久HTTP连接
- 假设用户输入 URL：`www.someSchool.edu/someDept/home.index`，其中包含文本和10 个 jpeg 图像的引用
- 则在非持久HTTP连接的情况下，随时间顺序，客户端和服务器通信如下：
	1. HTTP 客户端在端口号 80 发起一个到服务器 `www.someSchool.edu` 的连接；
	2. 位于主机 `www.someSchool.edu` 的 HTTP 服务器在 80 号端口监听，接受连接请求并告知客户端成功与否，**客户端和服务器各有自己的 socket 与该连接相关联**；
	3. HTTP 客户端经自己的 socket，向 TCP 的套接字发送 HTTP 请求报文，报文指明客户端需要的对象 someDepartment/home.index
	4. HTTP 服务器接从其 socket 中收到请求报文，在自己主机中（RAM or 磁盘里）检索出被请求的对象，将对象封装在一个**响应报文**中，并通过其 socket 向客户端发送
	5. HTTP 服务器上的进程通知 TCP 断开该连接，==实际上直到 TCP 确认客户端完整收到报文才真正地中断连接==
	6. HTTP 客户端收到包含 html 文件的响应报文，并关闭 TCP 连接。然后对 html 文件进行检查，找到10 个图像引用对象（即还需向服务器发送 10 个图像的请求）
	7. 对 10 个 jpeg 对象，重复1-6步

>[!tip] web 真的是逐条接收连接吗？
>实际上，web 可以对多个连接并行访问，即解析 html 文件后得到 10 个图像资源的引用，这时同时对多个图像资源建立对应的连接进行访问。现代 web 客户端一般提供 5~10 个并行的 TCP 连接。
>
>当然可以手动设置并行数为 1，此时就是串行了。

>[!warning] HTTP 与 web 如何解释 html 无关
>HTTP 规范仅定义在 HTTP 客户程序和 HTTP 服务器程序之间的通信协议。如何解释 HTML 文件，是 web 自己要做的事。

**响应时间模型**
- 往返时间 RTT (round-trip time)：一个小的分组从客户端到服务器，再回到客户端的时间（传输时间忽略，但是==传播时间==不忽略）
	- 不要忘记，分组在传播过程中，可能会在路由器和交换机上排队，因此有==排队时延==
	- 分组在节点还要进行比特级检错，因此还有==分组处理时延==
- 响应时间：共 2RTT + 传输时间
	- 一个 RTT 用来发起 TCP 连接
	- 一个 RTT 用来 HTTP 请求并等待 HTTP 响应
	- 文件传输时间

![[20-Application-layer-request-receive-time-calc.png]]
- 三次握手：
	1. 客户向服务器发送一个小 TCP 报文段
	2. 服务器用一个小 TCP 报文段作出确认和响应
	3. 客户向服务器返回确认
- 一旦第三部分向服务器确认的报文被接收，服务器就在该 TCP 连接上开始发送 html 文件
#### 持久HTTP
- **非持久 HTTP 的缺点**：
    - 每个对象要 2 个 RTT
    - 每个对象都要占用 1 个 TCP 连接，由于操作系统必须为每个 TCP 连接分配资源，这样开销未免太大

- **持久 HTTP**
    - 服务器在发送响应后，仍保持TCP连接
    - 在相同客户端和服务器之间的后续请求和响应报文通过相同的连接进行传送
    - 客户端在遇到一个引用对象的时候，就可以尽快发送向该对象的请求
    - 持久HTTP也分为两种：
        - 非流水方式(non-pipeline)的持久HTTP：
            - 客户端**只能在收到前一个响应后才能发出新的请求**，一次只有一个请求
            - 每个引用对象花费一个RTT
        - 流水方式(pipeline)的持久HTTP：
            - HTTP/1.1的默认模式
            - 客户端**遇到一个引用对象就立即产生一个请求**，而非在收到前一个请求的响应后才产生新的请求，最后对象依次回来，不必等待之前请求的回答
            - 所有引用（小）对象只花费一个RTT是可能的

>[!note] HTTP/2 的改进
>HTTP/2 允许在相同连接中多个请求和回答交错，并增加了在该连接中优化 HTTP 报文请求和回答的机制。

### HTTP 报文格式

- 两种类型的 HTTP 报文：**请求**、**响应**

#### HTTP 请求报文

- HTTP请求报文：
    - ASCII（人能阅读）
```
# 请求行 (request line,每个请求报文的第一行,包含GET，POST，HEAD，PUT，DELETE等方法选项)
GET /somedir/page.html HTTP/1.1     # 第一个是接口请求：GET是请求资源行为；POST是上传表单行为；HEAD是只取HTTP头部；PUT是存放资源；DELETE是删除资源，搜索引擎从头部提取描述信息建立索引或用于维护。第二个是目录和文件，主机名因为已经建立连接所以可以忽略。第三个是协议和版本号
# 首部行 (header line,后面的行都是首部行)
Host: www.someschool.edu  #首部名:首部值。Host表示主机域名
User-agent: Mozilla/4.0   #User-agent表示用户代理的程序，浏览器的第几个版本
Connection: close         #表示连接状态开启还是关闭
Accept-language: fr       #发送资源的语言版本，fr表示法语
(一个额外的换行回车符)        # 换行回车符表示报文结束
# Entity body
--- snip ---
```

![[20-Application-layer-HTTP-request-message.png]]
- GET 方法时 Entity body 为空；
- POST 方法时 Entity body 处存放要提交给服务器检查的表单；如向 Google 搜索“计算机网络”这一条目，Entity body 就存放这个条目，Google 搜索引擎服务器检索后将数据返还给请求方。

**提交表单输入**
- Post方式：
    - 网页通常包括表单输入
    - 包含在**实体主体**(entity body)中的输入被提交到服务器
- URL方式：
    - 方法：GET
    - 输入通过请求行的 URL 字段上载（以**参数**形式上传）
    - 如 `http://www.baidu.com/s?wd=xx+yy+zzz&cl=3` ，`?` 后面的 wd，cl 为参数并通过 `&` 表示并列连接，而XX+YY+zzz，3为参数值

**方法类型**
- HTTP/1.0
    - GET
    - POST
    - HEAD
        - 要求服务器在响应报文中不包含请求对象 -> 多用来进行故障跟踪
- HTTP/1.1
    - GET，POST，HEAD
    - PUT
        - 将实体主体中的文件上载==提交到URL字段规定的路径==，==通常用于网页的维护修改==
    - DELETE
        - 删除URL字段规定的文件

#### HTTP响应报文
```
# 状态行(status line,协议版本、状态码和相应状态信息)
HTTP/1.1 200 OK #状态码200和状态信息OK，表示请求一切正常
# 首部行
Connection close #发送报文后将关闭连接
Date: Thu, 06 Aug 1998 12:00:15 GMT #报文发送的时间
Server: Apache/1.3.0 (Unix) #服务器类型与服务器操作系统
Last-Modified: Mon, 22 Jun 1998 10:00:00 GMT #对象最后修改时间，对本地客户端和网络缓存服务器至关重要
Content-Length: 6821 #被发送对象的字节长度
Content-Type: text/html #对象类型

# Entity body，如请求的HTML文件
<data>
```
注：TCP 只负责传输报文，报文字节流的结构需要 HTTP 进行判断，从而提取出首部与其他部分

![[20-Application-layer-response-message.png]]

**HTTP 响应状态码**：位于服务器->客户端的响应报文中的首行，以下是部分示例：
- 200 OK
    - 请求成功，请求对象包含在响应报文的后续部分
- 301 Moved Permanently
    - 请求的对象已经被永久转移了；新的 URL 在响应报文的 `Location:` 首部行中指定
    - 客户端软件自动用新的URL去获取对象
- 400 Bad Request
    - 一个通用的差错代码，==表示该请求不能被服务器解读==
- 404 Not Found
    - 请求的文档在该服务器上没有找到
- 505 HTTP Version Not Supported

### 维护用户-服务器状态：cookies
HTTP 本身的无状态简化了服务器的设计和压力，允许开发同时处理成千上万的 TCP 连接的高性能 Web 服务器。
但 Web 站点需要识别用户——提高用户使用体验——大多数主要的门户网站使用 cookies 改造无状态的 HTTP 协议。

#### 举例
![[20-Application-layer-cookies.png]]
- 4个组成部分：
    1) 在HTTP响应报文中有一个cookie的首部行
    2) 在HTTP请求报文含有一个cookie的首部行
    3) 在用户端系统中保留有一个cookie文件，由用户的浏览器管理
    4) 在Web站点有一个后端数据库
- 例子：
    - Susan 总是用同一个 PC 使用 Internet Explore 上网
    - 她要访问一个使用了 Cookie 的电子商务网站 Amazon，此前已经访问过 eBay；
    - 当最初的 HTTP 请求到达 Amazon 服务器时，该 Web 站点产生一个唯一的 ID，并以此作为索引在它的后端数据库中产生一个表项存储——代表该 ID 与用户 Susan 绑定；
    - 接下来 Amazon 服务器使用包含 `Set-cookie:` 首部行的 HTTP 响应报文告知 Susan 的浏览器应该设置的识别码（这里是 1678）
    - Susan 的浏览器接收到响应报文时，从 `Set-cookie` 中得知 Amazon 给出的识别码，进行设置后再次访问 Amazon 时，此时请求报文的首部行 `Cookie:1678` 就可以使 Amazon 对请求进行查询——这是来自 Susan 的请求；
    - 如果 Susan 在一定时间（如一周）后再次访问，同样会利用这个 cookie 进行查询 Susan 的信息——名字、邮箱、收货地址、信用卡号等——这就是电子购物商场的实现原理；

Cookies能带来什么：
用户登录验证、购物车、推荐、用户的其他状态（Web e-mail）

#### 如何维持状态
- 协议端节点：在多个事务上，发送端和接收端维持状态
- cookies：http 报文携带状态信息
- 通过 cookie 在 HTTP 之上建立一个用户与服务器的会话层，在该会话层内服务器对用户进行标识；

#### Cookies与隐私
- Cookies允许站点知道许多关于用户的信息
- 可能将它知道的东西卖给第三方
- 使用重定向和cookie的搜索引擎还能知道用户更多的信息
    - 如通过某个用户在大量站点上的行为，了解其个人浏览方式的大致模式
- 广告公司从站点获得信息

### Web 缓存（代理服务器 proxy server）
- 目标：不访问原始服务器，就满足客户的请求，对客户端来说速度快，对服务器和网络来说负载压力更小

![[20-Application-layer-web-cache.png]]
- Web 缓存具有自己的磁盘存储空间，在**存储空间中保存最近请求过的对象的副本**；
- 用户设置浏览器——使 HTTP 优先通过缓存访问 Web。

As an example, suppose a browser is requesting the object `http://www.someschool.edu/campus.gif`. Here is what happens: 
1. The browser establishes a TCP connection to the Web cache and sends an HTTP request for the object to the Web cache.
2. The Web cache checks to see if it has a copy of the object stored locally. If it does, the Web cache returns the object within an HTTP response message to the client browser.
3. If the Web cache does not have the object, the Web cache opens a TCP connection to the origin server, that is, to `www.someschool.edu`. The Web cache then sends an HTTP request for the object into the cache-to-server TCP connection. After receiving this request, the origin server sends the object within an HTTP response to the Web cache.
4. When the Web cache receives the object, it ==stores a copy== in its local storage and sends a copy, within an HTTP response message, to the client browser (over the existing TCP connection between the client browser and the Web cache).

- **缓存既是客户端又是服务器**
- 通常缓存是由 ISP 安装（大学、公司、居民区 ISP）
- 为什么要使用 Web 缓存？
    - 降低客户端的请求响应时间，提升速度
    - 可以大大减少一个机构内部网络与 Internet 接入链路上的流量，降低负载
    - 互联网大量采用了缓存：可以使较弱的 ICP(Internet Cache Protocol) 也能够有效提供内容

> [!example] 缓存示例
> 
>  1.更快的接入链路
> ![[20-Application-layer-bottleneck-between-internet-barely-institutional-network.png]]
> 假设：
> - 平均对象大小为 $100kb$
> - 机构内浏览器对原始服务器的平均请求率为 $15请求/s$
> - 则平均到浏览器的速率为 $1.5Mbps$
> - 若机构内部路由器到原始服务器再返回到路由器的的延时（Internet延时）为 $2s$
> - 若接入链路带宽为 $1.54Mbps$
> 
> 结果
> - LAN 的流量强度 $\frac{La}{r} = 15\%$ （按局域网内部带宽为 $1Gbps$ 计算）
> - 接入链路上的流量强度 $= 1.5Mbps / 1.54Mbps = 99\%$ ，排队延时较大，其他延时可以忽略不计（*注：排队延时公式 $t_{queue}=\frac{I}{1-I}\cdot\frac{L}{R}$，其中 $I$ 为流量强度）*
> - 总延时 = LAN 延时 + 接入延时 + Internet 延时 = $ms + minutes + 2s$
> - 注：若升级接入链路带宽到 $154Mbps$ ，则流量强度降低到 $0.99\%$ ，则接入延时从分钟级降低为毫秒级。但是==代价非常大：增加接入链路带宽非常昂贵==！  
>
> 2. 配置本地缓存
> ![[20-Application-layer-inetnet-webcache-institutional-network.png]]
> 假设：
> - 平均对象大小为 $100kb$
> - 机构内浏览器对原始服务器的平均请求率为 $15请求/s$
> - 则平均到浏览器的速率为 $1.5Mbps$
> - 机构内部路由器到原始服务器再返回到路由器的的延时（Internet延时）为 $2s$
> - 接入链路带宽为 $1.54Mbps$
> 
> 代价：web缓存（廉价！）
> 
> 计算链路利用率，有缓存的延迟：
> - 假设缓存命中率 $0.4$，即 $0.4$ 的可能性直接在本地访问， $0.6$ 的可能性需要通过外网拉取对象
> - 接入链路利用率：
>     - $60\%$的请求采用接入链路
> - 经过接入链路到达浏览器的数据速率 $= 0.6*1.50 Mbps = 0.9 Mbps$
>     - 利用率 $= 0.9/1.54 = 0.58$
> - 总体延迟（加权平均）：
>     - $= 0.6 * (从原始服务器获取对象的延迟) + 0.4 * (从缓存获取对象的延迟)$  
>     $= 0.6 * (2.01 secs) + 0.4 * (10 msecs)$  
>     $\approx 1.2 secs$  
>     - 比安装 $154Mbps$ 链路还来得小（而且比较便宜!）

### 条件GET方法(conditional-GET)
- 众所周知：只要使用缓存，就存在缓存一致性问题——缓存的对象副本可能是陈旧的，服务器中的数据可能已经更新。
- HTTP 协议的条件 GET 方法，可以验证缓存中的对象是否最新：如果缓存中的对象拷贝是最新的，就不要封装并发送整个对象，只用发送头部

如何使用条件 GET？
- 缓存：在 HTTP 请求中使用 GET 方法，并且增加一个首部行 `If-modified-since:` —— 指定缓存拷贝的日期：

```
If-modified-since: <date>
```

- 服务器：
    - 如果缓存拷贝陈旧没有变化，则响应报文不包含对象：
```
	HTTP/1.0 304 Not Modified
```

- 如果缓存拷贝的原对象已经被修改，则响应报文包含对象：
```
	HTTP/1.0 200 OK
	<data>
```

>[!example] 条件 GET 方法举例
>To illustrate how the conditional GET operates, let’s walk through an example. 
>
>First, on the behalf of a requesting browser, a proxy cache sends a request message to a Web server: 
> ```
> GET /fruit/kiwi. gif
> HTTP/1.1 Host: www.exotiquecuisine.com 
> ```
> 
>Second, the Web server sends a response message with the requested object to the cache: 
> ```
> HTTP/1.1 200 OK 
> Date: Sat, 3 Oct 2015 15:39:29 
> Server: Apache/1.3.0 (Unix) 
> Last-Modified: Wed, 9 Sep 2015 09:23:24 
> Content-Type: image/gif 
> 
> (data data data data data ...)
>```
>
>The cache forwards the object to the requesting browser but also caches the object locally. Importantly, the cache also stores the last-modified date along with the object. Third, one week later, another browser requests the same object via the cache, and the object is still in the cache. Since this object may have been modified at the Web server in the past week, the cache performs an up-to-date check by issuing a conditional GET. Specifically, the cache sends:
> ```
> GET /fruit/kiwi. gif HTTP/1.1 
> Host: www.exotiquecuisine.com 
> If-modified-since: Wed, 9 Sep 2015 09:23:24
> ```
>Note that the value of the `If-modified-since:` header line is exactly equal to the value of the `Last-Modified:` header line that was sent by the server one week ago. This conditional GET is telling the server to send the object only if the object has been modified since the specified date. Suppose the object has not been modified since 9 Sep 2015 09:23:24. Then, fourth, the Web server sends a response message to the cache: 
> ```
> HTTP/1.1 304 Not Modified 
> Date: Sat, 10 Oct 2015 15:39:29 
> Server: Apache/1.3.0 (Unix)
> 
> (empty entity body) 
> ```
> We see that in response to the conditional GET, the Web server still sends a response message but does not include the requested object in the response message. Including the requested object would only waste bandwidth and increase user-perceived response time, particularly if the object is large. Note that this last response message has 304 Not Modified in the status line, which tells the cache that it can go ahead and forward its (the proxy cache’s) cached copy of the object to the requesting browser.

### HTTP/2
HTTP/2 `[RFC 7540]`, standardized in 2015, was the first new version of HTTP since HTTP/1.1, which was standardized in 1997. Since standardization, HTTP/2 has taken off, with over 40% of the top 10 million websites supporting HTTP/2 in 2020 `[W3Techs]`. Most browsers—including Google Chrome, Internet Explorer, Safari, Opera, and Firefox—also support HTTP/2. 

The primary goals for HTTP/2 are to 
- ==reduce perceived (感知到的) latency== by enabling request and response multiplexing over a single TCP connection, 
- ==provide request prioritization and server push==, 
- and ==provide efficient compression of HTTP header fields==. 

==HTTP/2 does not change HTTP methods, status codes, URLs, or header fields. Instead, HTTP/2 changes how the data is formatted and transported between the client and server==.

#### HTTP/1.1 and HOL problem
To motivate the need for HTTP/2, recall that HTTP/1.1 uses persistent TCP connections, allowing a Web page to be sent from server to client over a single TCP connection. By having only one TCP connection per Web page, the number of sockets at the server is reduced and each transported Web page gets a fair share of the network bandwidth (as discussed below). ==But developers of Web browsers quickly discovered that sending all the objects in a Web page over a single TCP connection has a **Head of Line (HOL) blocking** problem==. 
> 仅通过单个 TCP 连接传输一个网页的所有对象有行首阻塞问题。

To understand HOL blocking, consider a Web page that includes an HTML base page, a large video clip near the top of Web page, and many small objects below the video. Further suppose there is a low-to-medium speed bottleneck link (for example, a low-speed wireless link) on the path between server and client. ==Using a single TCP connection, the video clip will take a long time to pass through the bottleneck link, while the small objects are delayed as they wait behind the video clip==; that is, the video clip at the head of the line blocks the small objects behind it. 
> 类比 OS 中的长进程占用了短进程的执行时间，使得短进程不得不等待很久才会得到执行。长消息也是如此，如果加载一个包含视频、文字的网站，先传送视频的话，就会极大地延长网页打开的时间。

#### How HTTP/1.1 and TCP deal with HOL?
HTTP/1.1 browsers typically work around this problem by opening multiple parallel TCP connections, thereby having objects in the same web page sent in parallel to the browser. This way, the small objects can arrive at and be rendered in the browser much faster, thereby reducing user-perceived delay. TCP congestion control, discussed in detail in Chapter 3, also provides browsers an unintended incentive (预期外的激励，指不能预先确定的) to use multiple parallel TCP connections rather than a single persistent connection. Very roughly speaking, TCP congestion control aims to give each TCP connection sharing a bottleneck link an equal share of the available bandwidth of that link; so if there are n TCP connections operating over a bottleneck link, then each connection approximately gets 1/nth of the bandwidth. By opening multiple parallel TCP connections to transport a single Web page, the browser can “cheat” and grab a larger portion of the link bandwidth. Many HTTP/1.1 browsers open up to six parallel TCP connections not only to circumvent(避免，绕过) HOL blocking but also to obtain more bandwidth. 
> HTTP/1.1 和 TCP 通过打开多个连接、并发地进行数据传输来避免 HOL 问题，但这并不能根治，而且多连接策略意味着会极大地加大服务器的负载。

#### The purpose of HTTP/2
One of the primary goals of HTTP/2 is to get rid of (or at least reduce the number of) parallel TCP connections for transporting a single Web page. This not only reduces the number of sockets that need to be open and maintained at servers, but also allows TCP congestion control to operate as intended. But with only one TCP connection to transport a Web page, HTTP/2 requires carefully designed mechanisms to avoid HOL blocking.

#### HTTP/2 Framing
The HTTP/2 solution for HOL blocking is to ***break each message into small frames, and interleave the request and response messages on the same TCP connection***. 
> HTTP/2 的解决办法就是将整块的信息打碎成多个小片段，在同一个 TCP 连接中交错地请求、响应——完成传输；
> 类比 OS 的 RR 调度策略，给每个进程都分配固定的执行时间，执行完一段就让渡 CPU 给其他进程。

To understand this, consider again the example of a Web page consisting of one large video clip and, say, 8 smaller objects. Thus the server will receive 9 concurrent requests from any browser wanting to see this Web page. For each of these requests, the server needs to send 9 competing HTTP response messages to the browser. Suppose all frames are of fixed length, the video clip consists of 1000 frames, and each of the smaller objects consists of two frames. ==With frame interleaving, after sending one frame from the video clip, the first frames of each of the small objects are sent==. Then after sending the second frame of the video clip, the last frames of each of the small objects are sent. Thus, all of the smaller objects are sent after sending a total of 18 frames. If interleaving were not used, the smaller objects would be sent only after sending 1016 frames. Thus the HTTP/2 framing mechanism can significantly decrease user-perceived delay. 

The ability to break down an HTTP message into independent frames, interleave them, and then reassemble them on the other end is the single most important enhancement of HTTP/2. ==The framing is done by the framing sub-layer(组帧子层) of the HTTP/2 protocol==. When a server wants to send an HTTP response, the response is processed by the framing sub-layer, where it is broken down into frames. ==The header field of the response becomes one frame, and the body of the message is broken down into one for more additional frames==. The frames of the response are then interleaved by the framing sub-layer in the server with the frames of other responses and sent over the single persistent TCP connection. As the frames arrive at the client, they are first reassembled into the original response messages at the framing sub-layer and then processed by the browser as usual. 

==Similarly, a client’s HTTP requests are broken into frames and interleaved==. In addition to breaking down each HTTP message into independent frames, the framing sublayer also binary encodes the frames. ==Binary protocols are more efficient to parse, lead to slightly smaller frames, and are less error-prone==.

#### Response Message Prioritization and Server Pushing
Message prioritization allows developers to customize the relative priority of requests to better optimize application performance. 

As we just learned, the framing sub-layer organizes messages into parallel streams of data destined to the same requestor. When a client sends concurrent requests to a server, it can ==prioritize the responses it is requesting by assigning a weight between 1 and 256 to each message==. **The higher number indicates higher priority**. Using these weights, the server can send first the frames for the responses with the highest priority. In addition to this, ==the client also states each message’s **dependency on other messages** by specifying the ID of the message on which it depends==. 
> 另外，HTTP/2也支持对请求进行优先级划分，根据权重的不同分配不同的服务器响应时间；客户端会评估每条信息对其它信息的依赖度——如果一条信息被大量其他信息所依赖，那么通常来说它具有更重要的地位。

Another feature of HTTP/2 is ***the ability for a server to send multiple responses for a single client request***. That is, in addition to the response to the original request, the server can push additional objects to the client, without the client having to request each one. This is possible since the HTML base page indicates the objects that will be needed to fully render the Web page. So ==instead of waiting for the HTTP requests for these objects, the server can analyze the HTML page, identify the objects that are needed, and send them to the client before receiving explicit requests for these objects==. Server push eliminates the extra latency due to waiting for the requests.
> HTTP/2 的另一个特点是可以在一个客户端请求中，服务器预先发送可能请求的多个资源。将被动化为主动，更方便服务器进行策略优化。

### HTTP/3
***QUIC***, discussed in Chapter 3, is a new “transport” protocol that is implemented in the application layer over the bare-bones(简陋的) UDP protocol. 

QUIC has several features that are desirable for HTTP, such as 
- message multiplexing (interleaving), 
- per-stream flow control, 
- and low-latency connection establishment. 

HTTP/3 is yet a new HTTP protocol that is designed to operate over QUIC. As of 2020, HTTP/3 is described in Internet drafts and has not yet been fully standardized. Many of the HTTP/2 features (such as message interleaving) are subsumed(纳入、归入) by QUIC, allowing for a simpler, streamlined design for HTTP/3.

## 2.3 FTP（文件传输协议）

### 工作原理
- 向远程主机上**传输文件**或从远程主机**接收文件**
- 客户/服务器模式
    - 客户端：发起传输的一方
    - 服务器：远程主机
- `[RFC 959]`
- ftp 端口号：21用于控制命令，20用于数据传输。

![[20-Application-layer-FTP.png]]

### FTP 特点

**控制连接**（FTP 客户端与 FTP 服务器建立起的控制性的 TCP 连接）与**数据连接**分开：
- FTP 客户端与 FTP 服务器通过端口21联系，并使用 TCP 为传输协议
- 客户端通过控制连接获得身份确认 *<mark style="background: #FFB8EBA6;">注：FTP用户名和口令采用明文传输，容易被抓包</mark>*
- 客户端通过控制连接发送命令浏览远程目录并要求服务器将某个文件下载给客户端
- ==收到一个文件传输命令时，服务器主动打开一个到客户端的数据连接==（端口号20）
- 一个文件传输完成后，服务器关闭连接
- 服务器==打开第二个TCP数据连接用来传输另一个文件==
- 控制连接：通过带外 (“out of band”)传送 
	- *注：“带内”传数据，“带外”传指令、控制信息*
- FTP **服务器维护用户的状态信息**：
	- 当前路径、用户帐户与控制连接对应；
	- **有状态**（HTTP 无状态，通过 SSL 打了一个补丁）

### FTP命令、响应
- 命令样例：（在控制连接上以ASCII文本方式传送）
    - USER username
    - PASS password
    - LIST：请服务器返回远程主机当前目录的文件列表
    - RETR filename：从远程主机的当前目录检索下载文件(gets)
    - STOR filename：向远程主机的当前目录存放文件 (puts) 
	    - *注：客户端向服务器发送东西——上载；服务器向客户端发送东西——下载。都默认以客户端的角度来讲。*
- 返回码样例：（状态码和状态信息（同HTTP））
    - 331 Username OK, password required
    - 125 data connection already open; transfer starting
    - 425 Can’t open data connection
    - 452 Error writing file

## 2.4 EMail（电子邮件）

### 3个主要组成部分
- 用户代理(user agent)：发送、接收电子邮件的客户端软件 
    - 又名“邮件阅读器”
    - 撰写、编辑和阅读邮件
    - 如Outlook、Foxmail
    - 输出和输入邮件保存在服务器上
    - *注：Web 应用的用户代理：Web 浏览器；FTP 的用户代理：FTP 的客户端软件*
- 邮件服务器(mail server)
- 简单邮件传输协议：SMTP（Simple Mail Transfer Protocol, SMTP 是发送的协议，EMail 还有拉取的协议包括 POP3、IMAP、HTTP 等等）

![[20-Application-layer-email-system.png]]
A typical message starts its journey in the sender’s user agent, then travels to the sender’s mail server, and then travels to the recipient’s mail server, where it is deposited in the recipient’s mailbox.

### EMail: 邮件服务器
- 邮箱中管理和维护发送给用户的邮件
- 如果发送失败（不论是自身原因还是对方未接收的原因），邮件都会在 mail server 的**报文队列**中保持，排队等待后续尝试（邮件服务器通常设置成一段时间间隔发一次）
- 邮件服务器之间通过 SMTP 协议发送 email 报文
    - 客户：发送方邮件服务器
    - 服务器：接收端邮件服务器
    - 每台邮件服务器上既运行 SMTP 客户端，也运行 SMTP 服务端。

> [!example] 举例：Alice 给 Bob 发送报文
> ![[20-Application-layer-alice-send-msg-bob.png]]
> 1) Alice 使用用户代理撰写邮件并发送给 `bob@someschool.edu`
> 2) Alice的用户代理将邮件**发送**到她自己的邮件服务器；邮件放在报文队列中（SMTP协议）
> 3) SMTP的客户端打开到Bob邮件服务器的TCP连接
> 4) SMTP客户端通过TCP连接**传输**Alice的邮件
> 5) Bob的邮件服务器将邮件放到Bob的邮箱
> 6) Bob调用他的用户代理从他自己的邮件服务器**拉取**并阅读邮件（POP3、IMAP、HTTP等协议）
> 
> Alice的用户代理 --发送--> Alice的邮件服务器 --传输--> Bob的邮件服务器 --拉取--> Bob的用户代理    
> **“两推一拉”**

> [!note] SMTP has no intermediate mail server!
> It is important to observe that SMTP does not normally use intermediate mail servers for sending mail, even when the two mail servers are located at opposite ends of the world. If Alice’s server is in Hong Kong and Bob’s server is in St. Louis, the TCP connection is a direct connection between the Hong Kong and St. Louis servers. 
> > SMTP 没有中间服务器，无论多么远，都会直接建立 TCP 连接、发送邮件。除非接收方服务器宕机，邮件将会保存在发送方的服务器中并等待重发。
> 
> In particular, ==if Bob’s mail server is down, the message remains in Alice’s mail server and waits for a new attempt== — the message does not get placed in some intermediate mail server.

### EMail：SMTP 
`[RFC 2821]`, `[RFC 5321]`

- 使用TCP在客户端和服务器之间传送报文，**端口号为25**
- 直接传输：从发送方服务器到接收方服务器

- 命令/响应交互
    - 命令：ASCII文本
    - 响应：状态码和状态信息
- **报文必须为7位ASCII码**（所有的字节范围为0-127，包括邮件内容）

> [!note] SMTP, a legacy technology.
> For example, it restricts the body (not just the headers) of all mail messages to simple 7-bit ASCII. This restriction made sense in the early 1980s when transmission capacity was scarce and no one was e-mailing large attachments or large image, audio, or video files. 
> > SMTP 限制其报文在首部和主体部分的编码都是 ASCII，这在过去带宽有限时很有帮助，但如今已经远远不能提供足够的服务——例如在多媒体资源中，会将多媒体资源都编码成 ASCII 再发送，这意味着接收方需要从 ASCII 编码的信息中再还原为二进制的多媒体文件，才可以使用。
> 
> But today, in the multimedia era, the 7-bit ASCII restriction is a bit of a pain — ==it requires binary multimedia data to be encoded to ASCII before being sent over SMTP==; and it requires the corresponding ASCII message to be decoded back to binary after SMTP transport. Recall from Section 2.2 that ==HTTP does not require multimedia data to be ASCII encoded before transfer==.

>[!example] 简单的 SMTP 交互
>The ASCII text lines prefaced with C: are exactly the lines the client sends into its TCP socket, and the ASCII text lines prefaced with S: are exactly the lines the server sends into its TCP socket.The following transcript begins as soon as the TCP connection is established. 
> ```
> S: 220 hamburger.edu   
> C: HELO crepes.fr   
> S: 250 Hello crepes.fr, pleased to meet you   
> C: MAIL FROM: <alice@crepes.fr>   
> S: 250 alice@crepes.fr... Sender ok   
> C: RCPT TO: <bob@hamburger.edu>   
> S: 250 bob@hamburger.edu ... Recipient ok   
> C: DATA 
> S: 354 Enter mail, end with "." on a line by itself   
> C: Do you like ketchup?   
> C: How about pickles?   
> C: .   
> S: 250 Message accepted for delivery   
> C: QUIT   
> S: 221 hamburger.edu closing connection 
> ```
> As part of the dialogue, the client issued five commands: 
> 1. `HELO` (an abbreviation for HELLO), 
> 2. `MAIL FROM`, 
> 3. `RCPT TO`, 
> 4. `DATA`,
> 5. and `QUIT`. 
> 
> These commands are self-explanatory. The client also sends a line consisting of a single period, which indicates the end of the message to the server. (In ASCII jargon, each message ends with CRLF. CRLF, where CR and LF stand for carriage return (回车) and line feed (换行，没错这是两个键), respectively.)
> 
> We mention here that ==SMTP uses persistent connections==: If the sending mail server has several messages to send to the same receiving mail server, it can send all of the messages over the same TCP connection. ==For each message, the client begins the process with a new MAIL FROM: crepes. fr, designates the end of message with an isolated period, and issues QUIT only after all messages have been sent==.
> （对于每封邮件，客户端都以新的 `MAIL FROM：crepes.fr` 开始处理，用一个孤立的句号指定邮件结束，并在所有邮件发送完毕后才发出 QUIT。）

- 预先准备：建立连接——First, the client SMTP (running on the sending mail server host) has TCP establish a connection to port 25 at the server SMTP (running on the receiving mail server host). If the server is down, the client tries again later.
- **传输的 3 个阶段**
    - ==握手==：SMTP clients and servers introduce themselves before transferring information. During this SMTP handshaking phase, the SMTP client indicates the e-mail address of the sender (the person who generated the message) and the e-mail address of the recipient.
    - ==传输报文==：
	    - SMTP can count on the reliable data transfer service of TCP to get the message to the server without errors.
	    - The client then repeats this process over the same TCP connection if it has other messages to send to the server.
    - ==关闭==：

#### SMTP：总结
- SMTP 使用**持久连接**，**使用 TCP 可靠数据传输服务**
- SMTP要求报文（首部和主体）为7位ASCII编码
- SMTP 服务器使用 `CRLF.CRLF` 决定报文的尾部
- SMTP与HTTP比较：
    - HTTP：拉协议(pull)——内容提供者在 Web 服务器上装载信息，用户使用 HTTP 从中拉取信息，尤其是 TCP 连接发起自想接收文件的主机；
    - SMTP：推协议(push)——邮件服务器把文件推向另一个邮件服务器，尤其是该 TCP 连接发起自要发送文件的主机；
    - 二者都是 ASCII 形式的命令/响应交互、状态码，但 SMTP 要求必须是 ASCII 编码，而 HTTP 限制较小；
    - HTTP：每个对象封装在各自的响应报文中，一个响应报文至多一个对象
    - SMTP：多个对象包含在一个报文中

### 邮件报文格式

- The header lines and the body of the message are separated by a blank line (that is, by CRLF). 
- `[RFC 5322]` specifies the exact format for mail header lines as well as their semantic interpretations. 
- As with HTTP, each header line contains readable text, consisting of a keyword followed by a colon followed by a value. 
- Some of the keywords are required and others are optional. Every header must have a `From:` header line and a `To:` header line; a header may include a `Subject:` header line as well as other optional header lines. 
- It is important to note that these header lines are ==different from the SMTP commands== we studied in Section 2.3.1 (even though they contain some common words such as “from” and “to”). The commands in that section were part of the SMTP handshaking protocol; the header lines examined in this section are part of the mail message itself.

```
From: alice@crepes.fr 
To: bob@hamburger.edu 
Subject: Searching for the meaning of life.

<body> # only in ASCII 
```

#### base64 和 MIME 扩展
若邮件包括中文字符，（一个中文字符包括两个字节），若两个字节都不在 ASCII 码的范围之内，就需要进行 base64编码（编码：定义一个映射关系，将一串不在 ASCII 码范围之内的字节映射到一串更长的字节，其中每个字节都在 ASCII 码的范围之内，最常见的是 base64编码），进行 MIME 扩展

- MIME：多媒体邮件扩展（multimedia mail extension），RFC 2045, 2056
- 在报文首部用额外的行申明MIME内容类型
```SMTP
From: alice@crepes.fr 
To: bob@hamburger.edu 
Subject: Picture of yummy crepe. 
MIME-Version: 1.0                   # MIME版本
Content-Transfer-Encoding: base64   # 编码方式 
Content-Type: image/jpeg            # 多媒体数据类型、子类型和参数申明

base64 encoded data .....           # 编码好的数据
.........................           # 编码好的数据
......base64 encoded data           # 编码好的数据
```

### 邮件访问协议
- SMTP：仅传送邮件到接收方的邮件服务器，而不是邮箱客户端
![[20-Application-layer-email-protocols-between-communicating-entities.png]]
- Now let’s consider the path an e-mail message takes when it is sent from Alice to Bob. We just learned that at some point along the path the e-mail message needs to be deposited in Bob’s mail server. This could be done simply by having Alice’s user agent send the message directly to Bob’s mail server. 
- However, ==typically the sender’s user agent does not dialogue directly with the recipient’s mail server==. Instead, as shown in Figure 2.16, Alice’s user agent uses SMTP or HTTP to deliver the e-mail message into her mail server, then Alice’s mail server uses SMTP (as an SMTP client) to relay the e-mail message to Bob’s mail server. 
- Why the two-step procedure? Primarily because without relaying through Alice’s mail server, Alice’s user agent doesn’t have any recourse to an unreachable destination mail server. By having Alice first deposit the e-mail in her own mail server, ==Alice’s mail server can repeatedly try to send the message to Bob’s mail server, say every 30 minutes, until Bob’s mail server becomes operational==. (And if Alice’s mail server is down, then she has the recourse of complaining to her system administrator!)
> 邮件发送需要分为两步，第一步使用 SMTP 或 HTTP 将邮件发送到自己的 SMTP 服务器，第二步使用 SMTP 将邮件从自己的服务器发向目的方的服务器。

- 但是，似乎忘了什么...SMTP 是个 push protocol，它是没法从邮件服务器获得邮件的！
- **邮件访问协议**：从邮件服务器拉取、访问邮件
    - POP：邮局访问协议(Post Office Protocol) `[RFC 1939]`
        - 用户身份确认（代理<-->服务器）并下载
    - IMAP：Internet 邮件访问协议(Internet Mail Access Protocol) `[RFC 1730]`
        - 比POP3具备更多特性（更复杂），包括远程目录的维护（远程将报文从一个邮箱搬到另一个邮箱）
        - 在服务器上处理存储的报文
    - HTTP：Hotmail，Yahoo! Mail等
        - 方便

>[!note] 被替代的 POP3 和越发强大的 HTTP
>注意到，上面第八版的 Top-down 课本里插图是仅出现了 IMAP 和 HTTP 两种邮件拉取协议。而第七版的插图是这样：
> ![[20-Application-layer-email-pull-protocols-7th.png]]
>这是因为 POP3 已经广泛地被 IMAP 替代了。第八版书中这样写到：
>Today, there are two common ways for Bob to retrieve his e-mail from a mail server. 
>- If Bob is using Web-based e-mail or a smartphone app (such as ==Gmail==), then the user agent will use HTTP to retrieve Bob’s e-mail. This case requires Bob’s mail server to have an HTTP interface as well as an SMTP interface (to communicate with Alice’s mail server). 
>- The alternative method, typically used with mail clients such as ==Microsoft Outlook==, is to use the Internet Mail Access Protocol (IMAP) defined in `RFC 3501`. 
>
>Both the HTTP and IMAP approaches allow Bob to manage folders, maintained in Bob’s mail server. Bob can move messages into the folders he creates, delete messages, mark messages as important, and so on.

#### POP3协议
POP3 begins when the user agent (the client) opens a TCP connection to the mail server (the server) on port 110. With the TCP connection established, POP3 progresses through three phases: authorization, transaction, and update.
1. During the first phase, authorization, the user agent sends a username and a password (in the clear) to authenticate the user. 
2. During the second phase, transaction, the user agent retrieves messages; also during this phase, the user agent can mark messages for deletion, remove deletion marks, and obtain mail statistics. 
3. The third phase, update, occurs after the client has issued the quit command, ending the POP3 session; at this time, the mail server deletes the messages that were marked for deletion.

- **用户确认阶段**
    - 客户端命令：
        - user：申明用户名
        - pass：口令
    - 服务器响应
        - +OK(后跟服务器到客户的数据)
        - -ERR
- **事物处理阶段**，客户端：
    - list：报文号列表
    - retr：根据报文号检索报文
    - dele：删除
    - quit
```
telnet mailServer 110 #通过110端口连接到指定邮件服务器
# 用户确认阶段
S: +OK POP3 server ready 
C: user bob 
S: +OK         #如果服务器验证错误会发送-ERR，下面密码也一样
C: pass hungry 
S: +OK user successfully logged on
# 事务处理阶段
C: list     #列出所有报文及其长度
S: 1 498  
S: 2 912 
S: . 
C: retr 1 
S: <message 1 contents>
S: . 
C: dele 1   # 下载并删除模式，如果没有dele指令则下载但不删除。这取决于用户代理程序配置的方式
C: retr 2 
S: <message 2 contents>
S: . 
C: dele 2 
C: quit 
S: +OK POP3 server signing off
```

#### POP3与IMAP
- POP3：本地管理文件夹
    - 先前的例子使用“下载并删除”模式（一共有两种模式：下载并删除、下载并保留）。
        - 如果改变客户机，Bob不能阅读邮件
    - “下载并保留”：不同客户机上为报文的拷贝，在其他邮件客户端仍能阅读邮件
    - POP3 在会话中是无状态的
    - 但无论是否保存，POP3 都是在本地邮件服务器保存的，没有给用户提供任何远程访问和管理能力
- IMAP：远程管理邮件文件夹
    - IMAP 服务器将每个报文与一个文件夹联系起来；when a message first arrives at the server, it is associated with the recipient’s INBOX folder.
    - 允许用户用目录来组织报文
    - IMAP also provides commands that allow users to search remote folders for messages matching specific criteria.
    - IMAP在会话过程中保留用户状态：
        - 目录名、报文 ID 与目录名之间映射
	- Another important feature of IMAP is that it has commands that **permit a user agent to obtain components of messages**. For example, a user agent ==can obtain just the message header of a message== or just one part of a multipart MIME message. This feature is ==useful when there is a low-bandwidth connection== (for example, a slow-speed modem link) between the user agent and its mail server. With a low-bandwidth connection, the user may not want to download all of the messages in its mailbox, particularly avoiding long messages that might contain, for example, an audio or video clip.

#### HTTP 也能用作邮件访问

More and more users today are sending and accessing their e-mail through their Web browsers. Hotmail introduced Web-based access in the mid 1990s. Now Web-based e-mail is also provided by Google, Yahoo!, as well as just about every major university and corporation. ==With this service, the user agent is an ordinary Web browser, and the user communicates with its remote mailbox via HTTP==. 

When a recipient, such as Bob, wants to access a message in his mailbox, the e-mail message is sent from Bob’s mail server to Bob’s browser using the HTTP protocol rather than the POP3 or IMAP protocol. When a sender, such as Alice, wants to send an e-mail message, the e-mail message is sent from her browser to her mail server over HTTP rather than over SMTP. ==Alice’s mail server, however, still sends messages to, and receives messages from, other mail servers using SMTP==.

## 2.5 DNS (Domain Name System)

域名解析系统(DNS)不是一个给人用的应用，而是一个给其他应用使用的应用，提供**域名到 IP 地址的转换**，供应用使用。如 Web 应用中，用户输入 URL，Web 浏览器调用 DNS 的解析功能，得到域名对应的 IP 地址

### DNS 大致介绍
#### DNS的必要性
- IP地址标识主机、路由器（Everything over IP）（IP地址用于**标识**、**寻址**）
- 但 IP 地址不好记忆（IPv4是一个4字节即32bit 的数字；如果是 IPv6的话是一个16字节128bit 的数字），不便人类使用（没有人类语言的语义）
- 人类一般倾向于使用一些有意义的字符串来标识Internet上的设备
    - 例如：
        - `qzheng@ustc.edu.cn` 所在的邮件服务器
        - `www.ustc.edu.cn` 所在的 web 服务器
- 存在着 “字符串”——IP地址 的转换的必要性
- 人类用户提供要访问机器的“字符串”名称
- 由 DNS(Domain Name System) 负责转换成为二进制的网络地址（IP 地址）

#### DNS 的历史
- ARPANET的名字解析解决方案
    - 主机名：没有层次的一个字符串（全部在一个平面）。当时的节点比较少，问题不大
    - 存在着一个（集中）维护站：维护着一张 主机名-IP 地址 的映射文件：Hosts.txt。原因同上，一台设备集中式解决的负载不够大
    - 每台主机定时从维护站取文件
- ARPANET解决方案的问题
    - 当网络中主机数量很大时
        - 没有层次的主机名称很难分配
        - 文件的管理、发布、查找都很麻烦

#### DNS 是什么？
The DNS is 
1) a ***distributed database*** implemented in a hierarchy of DNS servers, and 
2) an application-layer protocol that ==allows hosts to query the distributed database==.

The DNS servers are often UNIX machines running the Berkeley Internet Name Domain (BIND) software `[BIND 2020]`. 

The DNS protocol **runs over UDP and uses port 53**.

#### DNS 系统需要解决的问题
- 问题1：如何命名设备
    - 用有意义的字符串：好记，便于人类用使用
    - 解决一个平面命名的重名问题：**层次化命名**
- 问题2：如何完成名字到IP地址的转换
    - **分布式的数据库**维护（一个节点维护一小个范围）和响应名字查询
- 问题3：如何维护：增加或者删除一个域，需要在域名系统中做哪些工作

### DNS 提供的服务
DNS is commonly employed by other application-layer protocols, including HTTP and SMTP, to translate user-supplied hostnames to IP addresses. 

#### Most important: translate hostname to IP-address
As an example, consider what happens when a browser (that is, an HTTP client), running on some user’s host, requests the URL `www.someschool.edu/index.html`. In order for the user’s host to be able to send an HTTP request message to the Web server `www.someschool.edu` , the user’s host must first obtain the IP address of `www.someschool.edu`. This is done as follows: 
1. The ==same user machine runs the client side== of the DNS application. 
2. The browser extracts the hostname, `www.someschool.edu` , from the URL and passes the hostname to the client side of the DNS application. 
3. The ==DNS client sends a query containing the hostname to a DNS server==. 
4. The DNS client eventually receives a reply, which includes the IP address for the hostname. 
5. ==Once the browser receives the IP address from DNS, it can initiate a TCP connection to the HTTP server== process located at port 80 at that IP address.

#### Host aliasing
DNS provides a few other important services in addition to translating hostnames to IP addresses:
- **Host aliasing**. A host with a complicated hostname can have one or more alias names. For example, a hostname such as `relay1.west-coast.enterprise.com` could have, say, two aliases such as `enterprise.com` and `www.enterprise.com`. In this case, the hostname `relay1.west-coast.enterprise.com` is said to be a canonical hostname (规范主机名). Alias hostnames, when present, are typically more mnemonic (好记的) than canonical hostnames. DNS can be invoked by an application to obtain the canonical hostname for a supplied alias hostname as well as the IP address of the host.
> 主机别名，即为难记的主机名提供简单的别名；另外，还可以为不同的主机（即有不同的 IP），提供相同的 URL，从而实现负载均衡。

#### Mail Server aliasing
- **Mail server aliasing**. For obvious reasons, it is highly desirable that e-mail addresses be mnemonic. For example, if Bob has an account with Yahoo Mail, Bob’s e-mail address might be as simple as `bob@yahoo.com`. However, the hostname of the Yahoo mail server is more complicated and much less mnemonic than simply yahoo. com (for example, the canonical hostname might be something like `relay1.west-coast.yahoo.com`). ==DNS can be invoked by a mail application to obtain the canonical hostname for a supplied alias hostname as well as the IP address of the host==. In fact, the MX record (see below) permits a company’s mail server and Web server to have identical (aliased) hostnames; for example, a company’s Web server and mail server can both be called `enterprise.com`.
#### Load distribution
- **Load distribution**. DNS is also used to perform load distribution among replicated(冗余的) servers, such as replicated Web servers. Busy sites, such as `cnn.com`, are replicated over multiple servers, with each server running on a different end system and each having a different IP address. For replicated Web servers, ==a set of IP addresses is thus associated with one alias hostname. The DNS database contains this set of IP addresses. When clients make a DNS query for a name mapped to a set of addresses, the server responds with the entire set of IP addresses, but rotates the ordering of the addresses within each reply==. Because a client typically sends its HTTP request message to the IP address that is listed first in the set, DNS rotation distributes the traffic among the replicated servers. DNS rotation is also used for e-mail so that multiple mail servers can have the same alias name. Also, content distribution companies such as Akamai have used DNS in more sophisticated ways `[Dilley 2002]` to provide Web content distribution.


### DNS 名字空间(The DNS Name Space)
#### DNS域名结构
- 一个层面命名设备会有很多重名
- DNS采用层次树状结构的命名方法
- Internet根被划为几百个**顶级域**(top lever domains)
	- 通用的(generic)
		- .com ; .edu ; .gov ; .int ; .mil ; .net ; .org ; .firm ; .hsop ; .web ; .arts ; .rec ;
	- 国家的(countries)
		- .cn ; .us ; .nl ; .jp
- 每个（子）域下面可划分为若干子域(subdomains)，如每个顶级域分为若干二级域（也可以不分），每个二级域分为若干个三级域（也可以不分）等等。
- 在这棵倒着生长的树上，树叶是主机

![[20-Application-layer-DNS-namespace.png]]

#### 域名(Domain Name)
- 从本域往上，直到树根
	- 中间使用“.”间隔不同的级别
	- 例如：`ustc.edu.cn` ； `auto.ustc.edu.cn` ；`www.auto.ustc.edu.cn`
	- 域的域名：可以用于表示一个域
	- 主机的域名：一个域上的一个主机
- 域名的管理
    - 一个域管理其下的子域
        - `.jp` 被划分为 `ac.jp` `co.jp`
        - `.cn` 被划分为 `edu.cn` `com.cn`
    - 创建一个新的域，必须征得它所属域的同意
- 域与物理网络无关，如国内某个大学的某个子网可能是欧洲的某台服务器在维护
    - **域遵从组织界限，而不是物理网络**
        - 一个域的主机可以不在一个网络
        - 一个网络的主机不一定在一个域
    - 域的划分是逻辑的，而不是物理的

### 解析域名——名字服务器(Name Server)

- 区域(zone) —— 分布式管理
    - 区域的划分有区域管理者自己决定
    - 将DNS名字空间划分为互不相交的区域，每个区域都是树的一部分
    - 名字服务器：
        - 每个区域都有一个（权威）名字服务器：维护着它所管辖区域的权威信息(authoritative record)
        - 名字服务器允许被放置在区域之外，以保障可靠性

名字空间划分为若干区域：Zone
![[20-Application-layer-DNS-zone.png]]

**权威 DNS 服务器**：
- 组织机构的 DNS 服务器，提供组织机构服务器（如 Web 和 mail）可访问的主机和 IP 之间的映射
- 组织机构可以选择实现自己维护或由某个服务提供商来维护

**顶级域(TLD)服务器**：
- 负责顶级域名（如 com，org，net，edu 和 gov）和所有国家级的顶级域名（如 cn，uk，fr，ca，jp）
- *注：DNS 根名字服务器维护的是顶级域名的根，如 `ustc.edu.cn.` 的最后一个点，TLD 服务器维护的是顶级域名，如上面的 `.cn`，即 TLD 服务器是根服务器的下面一级，顶级域名是根的下面一级*
- Network solutions 公司维护 `com` TLD 服务器
- Educause 公司维护 `edu` TLD 服务器

### DNS 的工作机理
[[#Most important translate hostname to IP-address|From the perspective of the invoking application]] in the user’s host, DNS is a black box providing a simple, straightforward translation service. But in fact, the **black box** that implements the service is complex, consisting of a large number of DNS servers distributed around the globe, as well as an application-layer protocol that specifies how the DNS servers and querying hosts communicate.
> 用户视角下的 DNS 就是一个黑箱，只需要提供 URL，就会得到 IP。

#### 分布式、层次数据库

>[!note] DNS 不采用集中式(整个网络使用一个 DNS 服务器)数据库的原因
>The problems with a centralized design include: 
>- ***A single point of failure***. If the DNS server crashes, so does the entire Internet! 
>- ***Traffic volume***. A single DNS server would have to handle all DNS queries (for all the HTTP requests and e-mail messages generated from hundreds of millions of hosts). 
>- ***Distant centralized database***. A single DNS server cannot be “close to” all the querying clients. If we put the single DNS server in New York City, then all queries from Australia must travel to the other side of the globe, perhaps over slow and congested links. This can lead to significant delays. 
>- ***Maintenance***. The single DNS server would have to keep records for all Internet hosts. Not only would this centralized database be huge, but it would have to be updated frequently to account for every new host.
>总之，集中式数据库既不合适、也不可行。

**No single DNS server has all of the mappings for all of the hosts in the Internet. Instead, the mappings are distributed across the DNS servers**.

上面这句话是对 DNS 分布式机制的最好诠释。

#### 三类 DNS 服务器
DNS 分为三类：root DNS servers, top-level domain (TLD) DNS servers, and authoritative DNS servers：
![[20-Application-layer-DNS-hierarchy.png]]
To understand how these three classes of servers interact, suppose a DNS client wants to determine the IP address for the hostname `www.amazon.com`. 
1. To a first approximation, the following events will take place. The client first contacts one of the ==root servers, which returns IP addresses for TLD servers for the top-level domain `com`==. 
2. The client then contacts one of these TLD servers, which returns the IP address of an authoritative server for `amazon.com`. 
3. Finally, the client contacts one of the authoritative servers for `amazon.com`, which returns the IP address for the hostname `www.amazon.com`.

> 用户->根 DNS 服务器，获得顶级域名的IP->TLD 服务器，获得权威域名的IP->目的主机的IP

> [!note] More about DNS hierarchy
> - **Root DNS servers**. There are more than 1000 root servers instances scattered all over the world, as shown in Figure 2.18. These root servers are copies of 13 different root servers, managed by 12 different organizations, and coordinated through the Internet Assigned Numbers Authority `[IANA 2020]`. The full list of root name servers, along with the organizations that manage them and their IP addresses can be found at `[Root Servers 2020]`. ==Root name servers provide the IP addresses of the TLD servers==. 
> - **Top-level domain (TLD) servers**. For each of the top-level domains—top-level domains such as `com`, `org`, `net`, `edu`, and `gov`, and all of the country top-level domains such as `uk`, `fr`, `ca`, and `jp` —there is TLD server (or server cluster). The company Verisign Global Registry Services maintains the TLD servers for the `com` top-level domain, and the company Educause maintains the TLD servers for the `edu` top-level domain. The network infrastructure supporting a TLD can be large and complex; see `[Osterweil 2012]` for a nice overview of the Verisign network. See `[TLD list 2020]` for a list of all top-level domains. ==TLD servers provide the IP addresses for authoritative DNS servers==. 
> - **Authoritative DNS servers**. ==Every organization with publicly accessible hosts (such as Web servers and mail servers) on the Internet must provide publicly accessible DNS records that map the names of those hosts to IP addresses==. An organization’s authoritative DNS server houses these DNS records. 
> 	- An organization can choose to implement its own authoritative DNS server to hold these records; 
> 	- alternatively, the organization can pay to have these records stored in an authoritative DNS server of some service provider. 
> 	- Most universities and large companies implement and maintain their own primary and secondary (backup) authoritative DNS server.
> 
> ![[20-Application-layer-DNS-root.png]]

#### 本地 DNS 服务器
There is another important type of DNS server called the **local DNS server**. A local DNS server does not strictly belong to the hierarchy of servers but is nevertheless central to the DNS architecture. Each ISP—such as a residential ISP or an institutional ISP—has a local DNS server (also called a default name server). 

==When a host connects to an ISP, the ISP provides the host with the IP addresses of one or more of its local DNS servers==(typically through **DHCP**, which is discussed in Chapter 4). You can easily determine the IP address of your local DNS server by accessing network status windows in Windows or UNIX. 
> 本地域名服务器不在上述 DNS 层次中，实际上是由 ISP 服务商提供给本地用户使用的。当用户的主机连接到 ISP，会通过 DHCP 获得临时 IP 地址，然后可以通过本地域名服务器加快解析到外部网络的速度。

A host’s ==local DNS server is typically “close to” the host==. For an institutional ISP, the local DNS server may be on the same LAN as the host; for a residential ISP, it is typically separated from the host by no more than a few routers. ==When a host makes a DNS query, the query is sent to the local DNS server, which acts a proxy, forwarding the query into the DNS server hierarchy, as we’ll discuss in more detail below==.

#### DNS 查询实例
![[20-Application-layer-DNS-interaction.png]]
Let’s take a look at a simple example. Suppose the host `cse.nyu.edu` desires the IP address of `gaia.cs.umass.edu`. Also suppose that NYU’s local DNS server for `cse.nyu.edu` is called `dns.nyu.edu` and that an authoritative DNS server for `gaia.cs.umass.edu` is called `dns.umass.edu`. 
1. As shown in Figure 2.19, the host `cse.nyu.edu` first sends a DNS query message to its local DNS server, `dns.nyu.edu`. The query message contains the hostname to be translated, namely, `gaia.cs.umass.edu`. 
2. The local DNS server forwards the query message to a root DNS server. The root DNS server takes note of the edu suffix and returns to the local DNS server a list of IP addresses for TLD servers responsible for `edu`. 
3. The local DNS server then resends the query message to one of these TLD servers. The TLD server takes note of the `umass.edu` suffix and responds with the IP address of the authoritative DNS server for the University of Massachusetts, namely, `dns.umass.edu`. 
4. Finally, the local DNS server resends the query message directly to `dns.umass.edu`, which responds with the IP address of `gaia.cs.umass.edu`. 

Note that in this example, in order to obtain the mapping for one hostname, ==eight DNS messages were sent: four query messages and four reply messages==!

Our previous example assumed that the TLD server knows the authoritative DNS server for the hostname. In general, this is not always true. Instead, ==the TLD server may know only of an intermediate DNS server, which in turn knows the authoritative DNS server for the hostname==. For example, suppose again that the University of Massachusetts has a DNS server for the university, called `dns.umass.edu`. Also suppose that each of the departments at the University of Massachusetts has its own DNS server, and that each departmental DNS server is authoritative for all hosts in the department. In this case, when the intermediate DNS server, `dns.umass.edu`, receives a query for a host with a hostname ending with `cs.umass.edu`, it returns to `dns.nyu.edu` the IP address of `dns.cs.umass.edu`, which is authoritative for all hostnames ending with `cs.umass.edu`. The local DNS server `dns.nyu.edu` then sends the query to the authoritative DNS server, which returns the desired mapping to the local DNS server, which in turn returns the mapping to the requesting host. ==In this case, a total of 10 DNS messages are sent!== 

#### 递归查询和迭代查询
![[20-Application-layer-DNS-interaction.png]]
The example shown in Figure 2.19 makes use of both recursive queries and iterative queries. 
- The query sent from `cse.nyu.edu` to `dns.nyu.edu` is a recursive query, since the query asks `dns.nyu.edu` to obtain the mapping on its behalf. 
- However, the subsequent three queries are iterative since all of the replies are directly returned to `dns.nyu.edu`. 

In theory, any DNS query can be iterative or recursive. For example, Figure 2.20 shows a DNS query chain for which all of the queries are recursive. 
![[20-Application-layer-recursive-queries-DNS.png]]
In practice, the queries typically follow the pattern in Figure 2.19: ==The query from the requesting host to the local DNS server is recursive, and the remaining queries are iterative==.

>[!note] 为什么查询要分两种？
>递归查询的缺陷：
>- 问题：根服务器的负担太重
>- 解决：迭代查询

#### DNS caching

上文中可以看到，为了请求一个地址，要进行多次的、重复的 DNS 查询，这可以通过设立 DNS 缓存来优化查询时间和请求次数。

In a query chain, when a DNS server receives a DNS reply (containing, for example, a mapping from a hostname to an IP address), it can ==cache the mapping in its local memory==. 

Because hosts and mappings between hostnames and IP addresses are by no means permanent, ==DNS servers discard cached information after a period of time== (often set to two days).

### DNS 记录
共同实现 DNS 分布式数据库的所有 DNS 服务器存储了资源记录 Resource Record，RR 提供主机名到 IP 地址的映射。

- 资源记录 (resource records，缩写 RR)
    - ==作用：维护 域名-IP 地址的映射关系==（还有其它如 别名-规范名、邮件服务器别名-邮件服务器正规名）
    - 位置：Name Server 的分布式数据库中
- 资源记录格式：
    - 四元组：(Name, Value, Type, TTL)
    - TTL (time to live)：生存时间，决定了资源记录应当从缓存中删除的时间（权威，缓冲记录）
	    - 若 ttl 很长趋于无限大，则指的是权威记录；
	    - 若 ttl 为有限值，则为缓冲记录，需要过了 ttl 这么长的时间后将资源记录删除，
	    - 一般是其他区域的域名和 IP 地址的关系，需要暂时缓存（为了性能），超过时限后删除（为了一致性，防止域名改变后本权威名字服务器还保留错误的老旧域名））
    - Value 和 Name 取决于 Type：
    - Type 类别：本资源记录的类型，用不同的值来标识
        - Type=A （是什么）
            - Name 为主机名；Value 为 IP 地址
            - 如：`relay1.bar.foo.com, 145.37.93.126, A` 
        - Type=NS（在哪里）
            - Name 中放子域的域名（如 foo. com ）
            - Value 为该域名的权威服务器的主机名
            - 用于沿着查询链来路由 DNS 查询
            - 如：`foo.com, dns.foo.com, NS`
        - Type=CNAME
            - Name 为规范名字的别名
            - value 为规范名字
            - 如：`foo.com, relay1.bar.foo.com, CNAME`
        - Type=MX
            - Name 中为邮件服务器的别名
            - Value 为 Name 对应的邮件服务器的规范名字

>[!note] 资源记录的类型分别对应什么情况的主机名？
>1. If a DNS server is authoritative for a particular hostname, then the DNS server will contain a Type A record for the hostname. (Even if the DNS server is not authoritative, it may contain a Type A record in its cache.)
>2. If a server is not authoritative for a hostname, then the server will contain a Type NS record for the domain that includes the hostname; it will also contain a Type A record that provides the IP address of the DNS server in the Value field of the NS record. 
>
>As an example, suppose an `edu` TLD server is not authoritative for the host `gaia.cs.umass.edu`. Then this server will contain a record for a domain that includes the host `gaia.cs.umass.edu`, for example, (`umass.edu`, `dns.umass.edu`, NS). The edu TLD server would also contain a Type A record, which maps the DNS server `dns.umass.edu` to an IP address, for example, (`dns.umass.edu`, `128.119.40.111`, A).

### DNS 报文
- DNS协议：**查询和响应报文的报文格式相同**，通过标识位(flags)加以区分
- 报文首部
    - 标识符 (identification/ID)：16 位。
	    - 使用 ID 号，通过查询 ID 和响应 ID 的比对，Name server 可以同时维护相当多的查询，而非等待该 ID 查询完之后再进行下一个查询
    - flags:
        - 查询/应答
        - 希望递归
        - 递归可用
        - 应答为权威

![[20-Application-layer-DNS-message-format.png]]
1. The first 12 bytes is the header section, which has a number of fields. 
	- The first field is a 16-bit number that identifies the query. This identifier is copied into the reply message to a query, allowing the client to match received replies with sent queries. 
	- There are a number of flags in the flag field. 
		- A 1-bit query/reply flag indicates whether the message is a query (0) or a reply (1).
		- A 1-bit authoritative flag is set in a reply message when a DNS server is an authoritative server for a queried name. 
		- A 1-bit recursion-desired flag is set when a client (host or DNS server) desires that the DNS server perform recursion when it doesn’t have the record. 
		- A 1-bit recursion-available field is set in a reply if the DNS server supports recursion. 
	- In the header, there are also four number-of fields. ==These fields indicate the number of occurrences of the four types of data sections that follow the header==. 

2. The ==question section contains information about the query that is being made==. This section includes 
	1) a ==name field== that contains the name that is being queried, and 
	2) a ==type field== that indicates the type of question being asked about the name—for example, a host address associated with a name (Type A) or the mail server for a name (Type MX)

3. In a reply from a DNS server, the ==answer section contains the resource records for the name that was originally queried==. 
	- Recall that in each resource record there is the Type (for example, A, NS, CNAME, and MX), the Value, and the TTL. 
	- A reply can return multiple RRs in the answer, since a hostname can have multiple IP addresses (for example, for replicated Web servers, as discussed earlier in this section). 

4. The ==authority section contains records of other authoritative servers==. 

5. The ==additional section contains other helpful records==. 
	- For example, the answer field in a reply to an MX query contains a resource record providing the canonical hostname of a mail server. The additional section contains a Type A record providing the IP address for the canonical hostname of the mail server.


### 维护问题：如何在 DNS 中新增一个域
- 在上级域的名字服务器中增加两条记录，指向这个新增的子域的域名 和域名服务器的 IP 地址
- 在新增子域的名字服务器上运行名字服务器，负责本域的名字解析：名字->IP 地址

>[!example] 例子：在 com 域中建立一个“Network Utopia”
>1. 到注册登记机构 resigtrar 注册域名 `networkutopia.com`
>2. 需要向该机构提供权威 DNS 服务器（基本的、和辅助的）的名字和 IP 地址, 登记机构在 com TLD 服务器中插入两条 RR 记录：
>	- ( `networkutopia.com`, `dns1.networkutopia.com` , NS) 新增子域的域名->维护这个新增子域的权威服务器的域名
>	- ( `dns1.networkutopia.com` , `212.212.212.1`, A) 维护这个新增子域的名字服务器的域名->维护这个新增子域的名字服务器的 IP 地址
>3. 在 networkutopia. com 的权威服务器中确保有
>	- 用于 Web 服务器的 `www.networkuptopia.com` 的类型为 A 的记录
>	- 用于邮件服务器 `mail.networkutopia.com` 的类型为 MX 的记录
>
>现在，这个 Web 站点就可以访问了！让我们来试试：
>Suppose Alice in Australia wants to view the Web page `www.networkutopia.com`. 
>- As discussed earlier, her host will first send a DNS query to her local DNS server. 
>- The local DNS server will then contact a TLD com server. (The local DNS server will also have to contact a root DNS server if the address of a TLD com server is not cached.) 
>- This TLD server contains the Type NS and Type A resource records listed above, because the registrar had these resource records inserted into all of the TLD com servers. 
>- The TLD `com` server sends a reply to Alice’s local DNS server, with the reply containing the two resource records. 
>- The local DNS server then sends a DNS query to `212.212.212.1`, asking for the Type A record corresponding to `www.networkutopia.com`. 
>- This record provides the IP address of the desired Web server, say, `212.212.71.4`, which the local DNS server passes back to Alice’s host. 
>- Alice’s browser can now initiate a TCP connection to the host `212.212.71.4` and send an HTTP request over the connection

### 攻击DNS
- DDoS攻击
    - 对根服务器进行流量轰炸攻击：发送大量ping
        - 没有成功
        - 原因1：根目录服务器配置了流量过滤器，防火墙
        - 原因2：Local DNS 服务器缓存了 TLD 服务器的 IP 地址，因此无需查询根服务器
    - 向TLD服务器流量轰炸攻击：发送大量查询
        - 可能更危险
        - 效果一般，大部分 DNS 缓存了 TLD
- 重定向攻击
    - 中间人攻击
        - 截获查询，伪造回答，从而攻击某个（DNS 回答指定的 IP）站点
    - DNS 污染 (poison)
        - 发送伪造的应答给 DNS 服务器，希望它能够缓存这个虚假的结果
    - 技术上较困难：分布式截获和伪造
- 利用 DNS 基础设施进行 DDoS
    - 伪造某个 IP 进行查询，攻击这个目标 IP
    - 查询放大，响应报文比查询报文大
    - 效果有限

## 2.6 P2P应用

相比于C/S模式，P2P可扩展性高（所有的对等方都是服务器），不会出现服务器宕机就整个无法使用的情况，但是可管理性差

### P2P 架构
- 没有（或极少）一直运行的服务器
- 任意端系统都可以直接通信
- 利用 peer 主机的服务能力，每个对等方能够向任何其他对等方重新发送它已经收到的该文件的任何部分，从而在分发过程中协助服务器
- Peer节点间歇上网，每次IP地址都有可能变化
- 例子：
    - 文件分发 (BitTorrent)
    - 流媒体 (KanKan)：不需要专业的大型服务器，而是许多 peer 节点相互服务，很容易扩展到几百上千万的用户量级
    - VoIP (Skype)

### 文件分发：C/S vs P2P
问题：**从一台服务器分发文件（大小 $F$ ）到 $N$ 个 peer 用时几何**？
- Peer 节点的上下载能力是有限的资源
- 不妨假设：每个客户端上载带宽为 $u_i$ ，下载带宽为 $d_i$ ，服务器的上载带宽为 $u_s$
- ![[20-Application-layer-file-distribution.png]]

#### 文件分发时间：**C/S模式**
- 服务器传输：都是由服务器发送给 peer，服务器必须顺序传输（上载） $N$ 个文件拷贝:
	- 发送 $1$ 个copy： $\frac{F}{u_{s}}$ 
	- 则发送 $N$ 个 copy： $N\times\frac{F}{u_{s}}$
- 客户端：每个客户端必须下载一个文件拷贝 注：C/S模式都是通过服务器的服务来获取文件，所以每个客户端的上载能力无关紧要
	- $d_{min}$ 为客户端最小的下载速率
	- 下载带宽最小的客户端下载的时间： $\frac{F}{d_{min}}$，
	- 因此最少的分发时间为：$\frac{F}{d_{min}}$
- 则采用 C/S 模式将一个 $F$ 大小的文件分发给 $N$ 个客户端耗时： $$D_{c/s} \geq \max\left(N\times \frac{F}{u_{s}}, \frac{F}{d_{min}}\right)$$ 
- 对于足够大的 N，C/S 模式的分发时间取决于 $N\times \frac{F}{u_{s}}$ 

#### 文件分发时间：**P2P模式**
- 服务器传输：最少需要上载一份拷贝
	- 发送 $1$ 个拷贝的时间： $\frac{F}{u_{s}}$
- 客户端：每个客户端必须下载一个拷贝
	- 最小下载带宽的客户耗时： $\frac{F}{d_{min}}$
- 客户端：所有客户端总体下载量： $N\times F$
	- 除了服务器可以上载，其他所有的 peer 节点都可以上载，因此
	- 最大上载带宽是： $u_s+\sum\limits_{i=1}^{N}{u_i}$
- 则采用 P2P 方法将一个 $F$ 大小的文件分发给 $N$ 个客户端耗时的下界： $$D_{P2P} \geq \max\left(\frac{F}{u_{s}}, \frac{F}{d_{min}}, \frac{N\times F}{u_{s} + \sum\limits_{i=1}^{N}{u_{i}}}\right)$$  
- 分子随着 $N$ 线性变化，每个节点需要下载，整体下载量随着 $N$ 增大…… 分母也是如此，随着 peer 节点的增多每个 peer 也带了服务能力  

![[20-Application-layer-p2p-vs-cs.png]]
Figure 2.23 compares the minimum distribution time for the client-server and P2P architectures assuming that all peers have the same upload rate u. In Figure 2.23, we have set $\frac{F}{u_{s}} = 1 hour$, $u_s = 10u_i$, and $d_{min} \ge u_{s}$. Thus, a peer can transmit the entire file in one hour, the server transmission rate is 10 times the peer upload rate, and (for simplicity) the peer download rates are set large enough so as not to have an effect. 

- We see from Figure 2.23 that for the ==client-server architecture==, the distribution time increases linearly and without bound as the number of peers increases. 
- However, for the P2P architecture, the minimal distribution time is not only always less than the distribution time of the client-server architecture; it is also less than one hour for any number of peers N. 

Thus, applications with the P2P architecture can be self-scaling. **This scalability is a direct consequence of peers being redistributors as well as consumers of bits**.
> P2P 中的 peer 主机不仅是服务器文件分发的消费者，也是再生产者，会传递地发送给其它 peer 主机。

### P2P 文件共享的问题与解决方案
- 两大问题：
    - 如何定位所需资源
    - 如何处理对等方的加入与离开
- 可能的方案
    - 集中
    - 分散
    - 半分散

#### P2P 结构分类
- 非结构化 P2P：peer 节点之间组成边变成邻居关系，构成一个逻辑上的覆盖网 (overlay)
	- 覆盖网络是一个图，peer 节点之间的关系是任意连接的，构成的覆盖网是随意的、随机的 
	- 如果 X 和 Y 之间有一个 TCP 连接，则二者之间存在一条边
	- 所有活动的对等方和边就是覆盖网络边并不是物理链路
	- 给定一个对等方，通常所连接的节点少于 10个

- 集中化目录：最初的“Napster”设计
	- ![[20-Application-layer-P2P-centralized.png]]
		1. Alice 在其笔记本电脑上运行 P2P 客户端程序
		2. 间歇性地连接到 Internet，每次从其 ISP 得到新的 IP 地址
		3. 请求“双截棍. MP3”
		4. 应用程序显示其他有“双截棍. MP3” 拷贝的对等方
		5. Alice 选择其中一个对等方，如 Bob.
		6. 文件从 Bob’s PC 传送到 Alice 的笔记本上：HTTP
		7. 当 Alice 下载时，其他用户也可以从 Alice 处下载
		8. 注：Alice的对等方既是一个Web客户端，也是一个瞬时Web服务器
	- 集中式目录中存在的问题：单点故障、性能瓶颈、侵犯版权 
	- *文件传输是分散的，而定位内容则是高度集中的*


- 完全分布式（查询洪泛(flooding)）：所有节点构成一个overlay，没有中心服务器；开放文件共享协议；许多Gnutella客户端实现了Gnutella协议（类似HTTP有许多的浏览器）  
	- Gnutella：协议：向所有邻居发送查询报文，所有邻居向它们各自的所有邻居再发送查询报文……
		- 在已有的TCP连接上发送查询报文
		- 对等方转发查询报文
		- 以反方向返回查询命中报文
		- *可扩展性：限制范围的洪泛查询（如 ttl：每过 5 跳若还没查到自动停止；再比如记忆化搜索：让中转节点记住已经转发过该查询报文，下次收到时不再转发）*
		- ![[20-Application-layer-P2P-gnutella.png]]
		
	- Gnutella：对等方加入——建立覆盖网
		- 1.对等方X必须首先发现某些已经在覆盖网络中的其他对等方：使用可用对等方列表
			- 自己维持一张对等方列表（经常开机的对等方的IP）联系维持列表的Gnutella站点（安装软件时得到配置文件即对等方列表）
		- 2.X接着试图与该列表上的对等方建立TCP连接，直到与某个对等方Y建立连接
		- 3.X向Y发送一个Ping报文，Y转发该Ping报文
		- 4.所有收到Ping报文的对等方以Pong报文响应
			- IP地址、共享文件的数量及总字节数
		- 5.X收到许多Pong报文，然后它能建立其他TCP连接
		- 当一个对等方暂时离开时，先通知其他对等方，其他节点从网络中再挑一个节点补充，用以维持网络强度，以满足 Gnutella 网络正常运行的最低要求

![[20-Application-layer-P2P-kazaa.png]]
- 混合式（利用不匀称性：KaZaA。组内集中式，组长和组长之间分布式）
	- 每个对等方要么是一个组长，要么隶属于一个组长
		- 对等方与其组长之间有TCP连接
		- 组长对之间有TCP连接
	- 组长跟踪其所有的孩子的内容
	- 若查询的东西组内没有，组长与其他组长联系
		- 转发查询到其他组长
		- 获得其他组长的数据拷贝
	- KaZaA：查询
		- 每个文件有一个散列标识码（唯一Hash，上载时赋予）和一个描述符（描述这个文件是干什么的）
		- 客户端向其组长发送关键字查询
		- 组长用匹配进行响应：
			- 对每个匹配：元数据、散列标识码和IP地址
		- 如果组长将查询转发给其他组长，其他组长也以匹配进行响应
		- 客户端选择要下载的文件
			- 向拥有文件的对等方发送一个带散列标识码的HTTP请求
	- KaZaA小技巧
		- 请求排队
			- 限制并行上载的数量
			- 确保每个被传输的文件从上载节点接收一定量的带宽
		- 激励优先权
			- 鼓励用户上载文件
			- 加强系统的扩展性
		- 并行下载
			- 从多个对等方下载同一个文件的不同部分
				- HTTP的字节范围首部
				- 更快地检索一个文件

![[20-Application-layer-P2P-DHT.png]]
- DHT (Distributed Hash Table) ：
	- 基于分布式散列表的 P2P。peer 节点之间可以构成一个有序的覆盖网，如环、树等等
        - 哈希表
        - DHT方案
        - 环形DHT以及覆盖网络
        - Peer波动

### BitTorrent
BitTorrent is a popular P2P protocol for file distribution `[Chao 2011]`. In BitTorrent lingo, ==the collection of all peers participating in the distribution of a particular file is called a **torrent**==.

- Peers in a torrent download equal-size chunks of the file from one another, with a typical chunk size of 256 KBytes.
- When a peer first joins a torrent, it has no chunks. Over time it accumulates more and more chunks.
- While it downloads chunks it also uploads chunks to other peers. Once a peer has acquired the entire file, it may (selfishly) leave the torrent, or (altruistically) remain in the torrent and continue to upload chunks to other peers.
- Also, any peer may leave the torrent at any time with only a subset of chunks, and later rejoin the torrent.

![[20-Application-layer-BitTorrent.png]]
Each torrent has an infrastructure node called a **tracker**. ==When a peer joins a torrent, it registers itself with the tracker and periodically informs the tracker that it is still in the torrent==. In this manner, the tracker keeps track of the peers that are participating in the torrent. A given torrent may have fewer than ten or more than a thousand peers participating at any instant of time.

> [!example] 例：P2P 文件分发 - BitTorrent
> 
> 当 Alice 加入洪流后，tracker 随机地从洪流中选出 peer 的一个子集（假定 50 个 peer 的子集），并将这 50 个 peer 的 IP 地址发送给 Alice，Alice 持有这张列表并试图与其中的 peer 建立 TCP 连接。若 Alice 成功地与 peer 创建连接，称其为**邻近 peer**。
> 
> 随着时间的推移，其中一些对等点可能会离开，其他对等点（在最初的 50 个对等点之外）可能会尝试与 Alice 建立 TCP 连接。因此，一个对等点的邻近对等点会随着时间的推移而波动。
> 
> 在任何给定时间，每个 peer 都将拥有文件中的一个分块子集，不同的 peer 拥有不同的子集。Alice 会定期（通过 TCP 连接）向每个相邻的对等节点询问它们拥有的数据块列表。如果 Alice 有 L 个不同的邻居，她将获得 L 个数据块列表。有了这些信息，Alice 就可以（再次通过 TCP 连接）请求获得她目前没有的数据块。
> 
> 每个文件块用 0 或 1 标识是否自己具备该文件块（该方法称为 map）。每个节点都有一个 bit map ，标记自己对用该文件的拥有情况。所有的 peer 节点在该洪流中定期地泛洪/交换 bit map，各个节点就知道了其他节点的情况。  
> 
> 那么，Alice 应该向她的邻居请求哪些块呢？她又应当向哪些向她请求块的邻居发送块？
> 1. 最稀缺优先策略解决第一个问题。
> 	- 其思路是从她没有的数据块中确定在她的邻居中最稀有的数据块（即在她的邻居中重复拷贝最少的数据块），然后首先请求这些最稀有的数据块。
> 	- 通过这种方式，最稀有的数据块可以更快地重新分配，从而（大致）均衡洪流中每个数据块的副本数量。
> 
> 2. 对换策略解决第二个问题。
> 	- 其基本思想是，Alice 优先考虑目前以最高速率向其提供数据的邻居。具体来说，对于每个邻居，Alice 不断测量其接收比特的速率，并确定以最高速率向其提供比特的四个对等方。然后，她向这四个对等网络发送数据块。
> 	- 每隔 10 秒，她会重新计算速率，并在可能的情况下修改四个对等点的集合。用 BitTorrent 的行话说，这四个对等点是疏通的 unchoked。
> 	- 重要的是，每隔 30 秒，她还会随机选择一个额外的邻居并向其发送数据块。我们称随机选择的邻居为鲍勃。用 BitTorrent 的行话说，Bob 是乐观的无标记者。由于爱丽丝正在向鲍勃发送数据，她可能成为鲍勃的前四名上传者之一，在这种情况下，鲍勃将开始向爱丽丝发送数据。如果鲍勃向爱丽丝发送数据的速度足够快，鲍勃就会反过来成为爱丽丝的四大上传者之一。
> 	- 换句话说，每隔 30 秒，爱丽丝会随机选择一个新的交易伙伴，并与该伙伴开始交易。如果双方都对交易感到满意，就会把对方列入自己的前四名名单，并继续进行交易，直到其中一方找到更好的伙伴为止。==这样做的结果是，能以兼容速率上传的对等点往往会找到彼此==。
> 	- 随机邻居选择还允许新的 peer 获得数据块，这样他们就有东西可以交易了。
> 	- 除了这 5 个 peer（4 个 "顶级 top"peer 和 1 个试探 peer）外，所有其他邻接 peer 都会被 "掐死 choked "，也就是说，它们不会从 Alice 处接收到任何数据块。
> 
> The incentive mechanism for trading just described is often referred to as tit-for-tat(一报还一报) `[Cohen 2003]`. It has been shown that this incentive scheme can be circumvented `[Liogkas 2006; Locher 2006; Piatek 2008]`. Nevertheless, the BitTorrent ecosystem is wildly successful.

## 2.7 CDN

### 视频流服务
- 视频流量：占据着互联网大部分的带宽
    - Netflix，YouTube：占据37%，16%的ISP下行流量
    - 约1B YouTube 用户，约75Million Netflix 用户
- 挑战：规模巨大-如何服务者 1Billion 用户?
    - 单个超级服务器无法提供服务
- 挑战：
    - 规模性：用户规模大
    - 异构性：不同用户拥有不同的能力，需求也不一样
	    - 有线接入和移动用户；
	    - 带宽丰富和受限用户；
- 解决方案：分布式的，应用层面的基础设施：用CDN解决

#### 多媒体：视频
- 视频：固定速度显示的图像序列
    - e.g. 24 images/sec
- 网络视频特点：
    - 高码率：10 倍于音频，高的网络带宽需求
    - 可以被压缩——比特率用来衡量视频图像的质量
    - 90%以上的网络流量是视频
- 数字化图像：像素的阵列
    - 每个像素被若干 bits 表示 
- 编码：使用图像内和图像间的冗余来降低编码的比特数（压缩）
    - 空间冗余（图像内）
        - 空间编码例子：对于一幅大面积紫色的图片，不是发送N个相同的颜色值，而是仅仅发送2各值：颜色（紫色）和重复的个数（N个）
    - 时间冗余（相邻的图像间）
        - 时间编码例子：不是发送第i+1帧的全部编码，而仅仅发送和帧i差别的地方
- 不同的压缩标准：
    - CBR(constant bit rate)：以固定速率编码
    - VBR(variable bit rate): 视频编码速率随时间的变化而变化
    - 例子：
        - MPEG1 (CD-ROM) 1.5 Mbps
        - MPEG2 (DVD) 3-6 Mbps
        - MPEG4 (often used in Internet, < 1 Mbps)

#### 存储视频的流化服务（streaming）   
简单场景：每下载几秒就播放（缓冲） 
![[20-Application-layer-CDN-streaming.png]]
- video server (stored video) --> Internet --> client 
- 在 HTTP 流媒体中，视频只是作为一个普通文件存储在 HTTP 服务器中，并带有一个特定的 URL。当用户想观看视频时，客户端会与服务器建立 TCP 连接，并对该 URL 发出 HTTP GET 请求。
- 然后，服务器在底层网络协议和流量条件允许的情况下，通过 HTTP 响应信息快速发送视频文件。在客户端，字节被收集到客户端应用程序缓冲区中。一旦缓冲区中的字节数超过预定的阈值，客户端应用程序就会开始播放--具体来说，流媒体视频应用程序会定期从客户端应用程序缓冲区中抓取视频帧，解压缩后将其显示在用户屏幕上。
- 因此，==视频流应用程序在显示视频的同时，也在接收和缓冲与视频后几部分相对应的帧==。

HTTP 流的缺陷：所有客户收到相同编码的视频——不论它们的设备如何。

#### DASH
DASH —— 解决不同客户端、不同网络需求和能力的问题
- DASH：Dynamic, Adaptive Streaming over HTTP
- 服务器：
    - 将视频文件分割成多个块
    - 每个块独立存储，编码于不同码率（8-10 种），码率指的就是比特率，对应不同的质量水平
    - 提供告示文件 (manifest file)：描述该文件是什么，它的描述信息，切成了多少块，不同块的 URL，每块视频持续的范围，有哪些编码版本等等
- 客户端：
    - 先获取告示文件
    - 周期性地测量服务器到客户端的带宽
    - 查询告示文件，在一个时刻请求一个块，通过 HTTP GET 请求报文，一次选择一个不同的块。HTTP 头部指定字节范围
        - 客户动态地请求来自不同版本的视频段数据块，这取决于客户的带宽。如果带宽足够，选择最大码率的视频块
        - 会话中的不同时刻，可以切换请求不同的编码块（取决于当时的可用带宽、客户端的需求）

- “智能”客户端：客户端自适应决定
    - 什么时候去请求块（不至于缓存挨饿，或者溢出）
    - 请求什么编码速率的视频块（当带宽够用时，请求高质量的视频块）
    - 哪里去请求块、哪些服务器去请求（可以向离自己近的服务器发送URL，或者向高可用带宽的服务器请求）

### 视频流输出
前面解决了客户如何从互联网获得视频流的方法。那么，问题在于视频服务提供者如何分发视频内容？—— 若现在只有一个或很少服务器，并发数较大，如何解决？（即：服务器如何通过网络向上百万用户同时流化视频内容（上百万视频内容））

#### Mega-server
- 选择1：单个的、大的超级服务中心“mega-server”。其缺点非常明显：
    - 服务器到客户端路径上跳数较多，瓶颈链路的带宽小导致停顿（服务器只能优化自身，无法优化整个网络）
    - 网络同时充斥着同一个视频的多个拷贝，效率低（付费高、带宽浪费、效果差）
    - 如果服务器单点故障，那就不能分发任何视频流
    - 周边网络的拥塞
    - 评述：相当简单，但是这个方法不可扩展

#### CDN
- 选择 2：通过 CDN
	- 全网部署缓存节点 (server)，**预先存储服务内容**，在 CDN 节点中存储内容的多个拷贝 
		- e.g. Netflix stores copies of MadMen
	- 就近为用户提供服务
		- ISP 购买 CDN 运营商的服务，加速对用户服务的传输质量，当用户请求时，==通过域名解析的重定向，找到离它最近、服务质量最好的拷贝节点，由那些节点提供服务==，
		- 如果网络路径拥塞则可能选择不同的拷贝，来提高用户体验：内容加速服务）
    - 部署策略：
	    1. enter deep：深入
	        - 通过在遍及全球的接入 ISP 中部署服务器集群来深入到 ISP 的接入网中
	        - 更接近用户，数量多，离用户近，服务质量高，但数量多管理困难
	        - Akamai，1700个位置
	    2. bring home：邀请做客
	        - 通过在少量关键位置建造大集群来邀请 ISP 做客，这些集群通常在因特网交换点 IXP 中
	        - 维护和管理开销较低，但端用户时延和吞吐量不够理想
	        - Limelight 等多数 CDN 公司采用

### CDN 存放副本的策略
一旦集群部署就位，CDN 就会在集群间复制内容。CDN 不会在每个集群中放置每个视频的副本，因为有些视频很少被观看，或者只在某些国家流行。事实上，许多 CDN 并不向集群推送视频，而是采用简单的拉取策略：
- 如果客户端向未存储视频的集群请求视频，集群就会检索视频（从中央存储库或其他集群）并在本地存储一份副本，同时将视频流传输给客户端。
- 与网络缓存类似，当群集的存储空间满时，它会删除不常被请求的视频。

### CDN 操作
When a browser in a user’s host is instructed to retrieve a specific video (identified by a URL), the CDN must intercept (截获) the request so that it can 
1) determine a suitable CDN server cluster for that client at that time, and 
2) redirect the client’s request to a server in that cluster.

==Most CDNs take advantage of DNS to intercept and redirect requests==. 

>[!example] 用户如何通过 CDN 访问视频资源？
>Suppose a content provider, NetCinema, employs the third-party CDN company, KingCDN, to distribute its videos to its customers. On the NetCinema Web pages, each of its videos is assigned a URL that includes the string “video” and a unique identifier for the video itself; for example, *Transformers 7* (变形金刚 7) might be assigned `http://video.netcinema.com/6Y7B23V`. Six steps then occur, as shown in Figure 2.25:
> ![[20-Application-layer-CDN-operate.png]]
>1. The user visits the Web page at NetCinema.
>2. When the user clicks on the link `http://video.netcinema.com/6Y7B23V`, the user’s host sends a DNS query for `video.netcinema.com`. 
>3. The user’s Local DNS Server (LDNS) relays the DNS query to an authoritative DNS server for NetCinema, which observes the string “video” in the hostname `video.netcinema.com`. To “hand over” the DNS query to KingCDN, instead of returning an IP address, the NetCinema authoritative DNS server returns to the LDNS a hostname in the KingCDN’s domain, for example, `a1105.kingcdn.com`. 
>4. From this point on, the DNS query enters into KingCDN’s private DNS infrastructure. The user’s LDNS then sends a second query, now for `a1105.kingcdn.com`, and KingCDN’s DNS system eventually returns the IP addresses of a KingCDN content server to the LDNS. It is thus here, within the KingCDN’s DNS system, that the CDN server from which the client will receive its content is specified.
>5. The LDNS forwards the IP address of the content-serving CDN node to the user’s host. 
>6. Once the client receives the IP address for a KingCDN content server, it establishes a direct TCP connection with the server at that IP address and issues an HTTP GET request for the video. If DASH is used, the server will first send to the client a manifest file with a list of URLs, one for each version of the video, and the client will dynamically select chunks from the different versions.

### 集群选择策略
任何 CDN 部署的核心都是**集群选择策略**，即动态将客户导向 CDN 内的服务器集群或数据中心的机制。CDN 通过客户端的 DNS 查询得知客户端 LDNS 服务器的 IP 地址。在得知该 IP 地址后，CDN 需要根据该 IP 地址选择一个合适的群集。CDN 通常采用专有的集群选择策略。有以下几种策略：
1. **将客户端分配到地理位置最近的集群**。
	- 利用商业地理位置数据库，每个 LDNS IP 地址都会映射到一个地理位置。当收到来自某个 LDNS 的 DNS 请求时，CDN 会选择地理位置上最近的集群。
	- 对于大部分客户来说，这种解决方案的效果还算不错。但是，对于某些客户来说，这种解决方案可能效果不佳，因为就网络路径的长度或跳数而言，地理位置上最近的集群可能并不是网络长度或跳数最近的集群。
	- 所有基于 DNS 的方法都有一个固有的问题，即有些终端用户被配置为使用远程定位的 LDNS，在这种情况下，LDNS 的位置可能远离客户端的位置。
	- 此外，这种简单的策略忽略了互联网路径的延迟和可用带宽随时间的变化，总是将相同的集群分配给特定的客户端。

2. **根据当前的流量条件为客户端确定最佳集群**。
	- CDN 可以对其集群和客户端之间的延迟和损耗性能进行定期的实时测量。例如，CDN 可以让其每个集群定期向全球所有 LDNS 发送探测信息（如 ping 消息或 DNS 查询）。
	- 这种方法的一个缺点是，许多 LDNS 被配置为不响应此类探测。

> [!example] CDN 例子：Netflix
> ![[20-Application-layer-CDN-netflix.png]]


## 2.8 Socket 编程

A typical network application consists of a pair of programs—a client program and a server program—residing in two different end systems. When these two programs are executed, a client process and a server process are created, and these processes communicate with each other by reading from, and writing to, **sockets**.

There are **two types of network applications**. 
1. One type is an implementation whose operation is specified in a protocol standard, such as an RFC or some other standards document; 
	- such an application is sometimes referred to as “open,” since the rules specifying its operation are known to all.
	- For such an implementation, the client and server programs must conform to the rules dictated by the RFC. For example, the client program could be an implementation of the client side of the HTTP protocol.
2. The other type of network application is a proprietary network application. 
	- In this case, the client and server programs employ an application-layer protocol that has not been openly published in an RFC or elsewhere. 
	- A single developer (or development team) creates both the client and server programs, and the developer has complete control over what goes in the code.

>[!warning] 应该使用什么端口号？
>- Recall also that when a client or server program implements a protocol defined by an RFC, it should use the well-known port number associated with the protocol; 
>- conversely, when developing a proprietary application, the developer must be careful to avoid using such well-known port numbers. 
>
>常用端口号列表：[TCP/UDP端口列表 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/TCP/UDP%E7%AB%AF%E5%8F%A3%E5%88%97%E8%A1%A8)
>- 20/21，FTP 协议，数据/控制端口
>- 22， SSH 端口
>- 23， Telnet 端口
>- 25， SMTP 端口
>- 53， DNS 端口
>- 56， 远程访问协议
>- 80， HTTP 端口
>- 110，POP3 端口
>- 143，IMAP 端口
>- 443，HTTPS 端口
>- 546/547， DHCPv6 客户端/服务器

应用进程使用传输层提供的服务才能够交换报文，实现应用协议，实现应用    
    TCP/IP：应用进程使用Socket API访问传输服务     
    地点：界面上的SAP(Socket) 方式：Socket API    
目标：学习如何构建能借助sockets进行通信的C/S应用程序     
socket：分布式应用进程之间的门，传输层协议提供的端到端服务接口     

<img src="http://knight777.oss-cn-beijing.aliyuncs.com/img/image-20210724104344583.png" />

2种传输层服务的socket类型：
- TCP：可靠的、字节流的服务
- UDP：不可靠（数据UDP数据报）服务

套接字Socket：应用进程与端到端传输协议（TCP或UDP）之间的门户。socket是一个整数，TCP的socket（4元组）之于进程正如句柄之于文件，对于句柄的操作（读、写）就是对于文件的操作，对于这个socket的操作就是对于会话上的两个应用进程之间的操作

TCP服务：从一个进程向另一个进程可靠地传输字节流（从应用程序的角度：TCP在客户端和服务器进程之间
提供了可靠的、字节流（管道）服务）

### UDP 套接字编程

UDP：在客户端和服务器之间没有连接
- 没有握手，socket只和本地的IP和端口号相捆绑
- 发送端在每一个报文中明确地指定目标的IP地址和端口号
- 服务器必须从收到的分组中提取出发送端的IP地址和端口号
  
UDP传送的数据可能乱序，也可能丢失

进程视角看UDP服务：UDP为客户端和服务器提供不可靠的字节组的传送服务

**[有关UDP socket编程，详见这个视频3:45开始](https://www.bilibili.com/video/BV1JV411t7ow?p=21&vd_source=485cdaef2e99160b22fe3c01315013e0&t=225.7)**

<img src="http://knight777.oss-cn-beijing.aliyuncs.com/img/image-20210724132456460.png" />

> [!example]
> 样例：
> 1. C客户端（UDP）
> ```c
> // client.c
> 
> void main(int argc, char *argv[])
> { 
>     struct sockaddr_in sad; // structure to hold an IP address
>     int clientSocket; // socket descriptor
>     struct hostent *ptrh; // pointer to a host table entry 
> 
>     char Sentence[128]; 
>     char modifiedSentence[128];
> 
>     host = argv[1]; 
>     port = atoi(argv[2]);
> 
>     clientSocket = socket(PF_INET, SOCK_DGRAM, 0); // 创建客户端socket，没有连接到服务器
> 
>     /* determine the server's address */
>         memset((char *)&sad, 0, sizeof(sad)); // clear sockaddr structure
>         sad.sin_family = AF_INET; // set family to Internet
>         sad.sin_port = htons((unsigned short)port); 
>         ptrh = gethostbyname(host);
>             /* Convert host name to IP address */
>         memcpy(&sad.sin_addr, ptrh->h_addr, ptrh->h_length);
>     
>     gets(Sentence) // Get input stream from user
> 
>     addr_len = sizeof(struct sockaddr); 
>     n = sendto(clientSocket, Sentence, strlen(Sentence)+1, (struct sockaddr *) &sad, addr_len); // Send line to server
> 
>     n = recvfrom(clientSocket, modifiedSentence, sizeof(modifiedSentence), (struct sockaddr *) &sad, &addr_len); // Read line from server
> 
>     printf("FROM SERVER: %s\n", modifiedSentence);
> 
>     close(clientSocket); // Close connection
> }
> 
> ```
> 
> 2. C服务器（UDP）
> ```c
> // server.c
> 
> void main(int argc, char *argv[])
> { 
>     struct sockaddr_in sad; // structure to hold an IP address
>     struct sockaddr_in cad;
>     int serverSocket; // socket descriptor
>     struct hostent *ptrh; // pointer to a host table entry
> 
>     char clientSentence[128]; 
>     char capitalizedSentence[128]; 
> 
>     port = atoi(argv[1]);
> 
>     /*
>         Create welcoming socket at port
>             &
>         Bind a local address
>     */
>     serverSocket = socket(PF_INET, SOCK_DGRAM, 0);   
>         memset((char *)&sad,0,sizeof(sad)); // clear sockaddr structure 
>         sad.sin_family = AF_INET; // set family to Internet
>         sad.sin_addr.s_addr = INADDR_ANY; // set the local IP address 
>         sad.sin_port = htons((unsigned short)port); // set the port number
>         bind(serverSocket, (struct sockaddr *) &sad, sizeof(sad));
>     while(1):
>     { 
>         n = recvfrom(serverSocket, clientSentence, sizeof(clientSentence), 0, (struct sockaddr *) &cad, &addr_len); // Receive messages from clients 
>         
>         /* capitalize Sentence and store the result in capitalizedSentence */
>         
>         n = sendto(serverSocket, capitalizedSentence, strlen(capitalizedSentence)+1, (struct sockaddr *) &cad, &addr_len); // Write out the result to socket
>     } // End of while loop, loop back and wait for another client connection
> }
> 
> ```

### TCP套接字编程的工作流程
- 服务器首先运行，等待连接建立    
    1.服务器进程必须先处于运行状态
    - 创建 欢迎socket，返回一个整数
    - 和本地端口捆绑，正式成为 欢迎socket(welcome socket)
    - 在 欢迎socket 上阻塞式等待接收用户的连接（调用socket api的另外一个函数——accept函数 接收连接请求，若无来自客户端的请求，就一直等待，阻塞着不往下走，故称为：阻塞式）
- 客户端主动和服务器建立连接：    
    2.创建客户端本地套接字（隐式捆绑到本地port *注：隐式：不一定要调用，不调用的话客户端返回的整数默认和当前没有用的一个端口相捆绑*）
    - 指定服务器进程的IP地址和端口号，与服务器进程连接（调用connect函数，阻塞式）
- 3.当与客户端连接请求到来时
    - 服务器接受来自用户端的请求，解除阻塞式等待（同意连接建立时客户端的connect函数也会解除阻塞，返回一个有效值），返回一个新的socket（与欢迎socket不一样，为connection socket），与客户端通信
    - 允许服务器与多个客户端通信
    - 使用源IP和源端口来区分不同的客户端
- 4.连接API调用有效时，客户端P与服务器建立了TCP连接

两个重要的结构体：
1. 数据结构sockaddr_in
```c
'''IP地址和port捆绑关系的数据结构（标识进程的端节点）'''

struct sockaddr_in
{
    short sin_family; //AF_INET地址簇，给一个常量代表TCP/IP的协议族
    unsigned short sin_port; //port
    struct in_addr sin_addr; //IP address, unsigned long
    char sin_zero[8]; //align对齐
}
```

|    变量     |  含义  |
|:-----------:|:------:|
| sin_family  | 地址簇 |
|  sin_port   | 端口号 |
|  sin_addr   | IP地址 |
| sin_zero[8] |  对齐  |

2. 数据结构hostent
```c
'''域名和IP地址的数据结构'''

struct hostent
{
    char *h_name;
    char **h_aliases;
    int h_addrtype;
    int h_length; //地址长度
    char **h_addr_list;;
    #define h_addr h_addr_list[0]
}

/*h_addr_list作为调用域名解析函数时的参数。返回后，将IP地址拷贝到sockaddr_in的IP地址部分*/
```

|        变量        |       含义       |
|:------------------:|:----------------:|
|     `*h_name`      |     主机域名     |
|   `**h_aliases`    | 主机的一系列别名 | 
|     `h_length`     |  IP 地址的长度   |
| `**h_addr_list[i]` |  IP 地址的列表   |

**[有关TCP socket编程，详见这个视频21:20开始](https://www.bilibili.com/video/BV1JV411t7ow?p=20&vd_source=485cdaef2e99160b22fe3c01315013e0&t=1280.2)**

> C/S模式的应用样例：
> 1) 客户端从标准输入装置读取一行字符，发送给服务器
> 2) 服务器从socket读取字符
> 3) 服务器将字符转换成大写，然后返回给客户端
> 4) 客户端从socket中读取一行字符，然后打印出来
> 
> 实际上，这里描述了C-S之间交互的动作次序

<img src="http://knight777.oss-cn-beijing.aliyuncs.com/img/image-20210724125259562.png" />

> [!example]
> 样例：
> 1. C客户端（TCP）
> ```c
> // client.c
> 
> void main(int argc, char *argv[]) // 两个参数，第一个指明服务器端域名，第二个指明服务器端守候的端口号（注：argv[0]为应用程序名，这里为"client.c"）
> {
>     struct socketaddr_in sad; // structure to hold an IP address of server
>     int clientSocket; // socket descriptor
>     struct hostent *ptrh; //pointer to a host table entry
> 
>     char Sentence[128];
>     char modifiedSentence[128];
> 
>     host = argv[1]; // 第一个参数放主机域名
>     port = atoi(argv[2]); // 第二个参数放置端口号，将字符串转变为整型
> 
>     /*
>         Create client socket,
>             connect to server
>     */
>     clientSocket = socket(PF_INET, SOCK_STREAM, 0);
>         memset((char *) &sad, 0, sizeof(sad)); // clear socketaddr structure
>         sad.sin_family = AF_INET; // set family to Internet
>         sad.sin_port = htons((unsigned short)port);
>         ptrh = gethostbyname(host); // 调用解析器，得到一个结构体的指针，指针中相应放置IP地址
>         // convert host name to IP address
>         memcpy(&sad.sin_addr, ptrh->h_addr, ptrh->h_length); // copy IP address to sad.sin_addr
>         connect(clientSocket, (struct socketaddr *) &sad, sizeof(sad));
> 
>     gets(Sentence); // Get input stream from user
> 
>     n = write(clientSocket, Sentence, strlen(Sentence)+1); // Send line to server
> 
>     n = read(clientSocket, modifiedSentence, sizeof(modifiedSentence)); // Read line from server
> 
>     printf("FROM SERVER: %s\n", modifiedSentence);
> 
>     close(clientSocket); // Close connection;
> }
> 
> ```
> 
> 2. C服务器（TCP）
> ```c
> // server.c
> 
> void main(int argc, char *argv[])
> {
>     struct socketaddr_in sad; // structure to hold an IP address of server
>     struct socketaddr_in cad; // client
>     int welcomeSocket, connectionSocket; // socket descriptor
>     struct hostent *ptrh; // pointer to a host table entry
> 
>     char clientSentence[128];
>     char capitalizedSentence[128];  
> 
>     port = atoi(argv[1]);
> 
>     /*
>         Create welcoming socket at port 
>             &
>         Bind a local address
>     */
>     welcomeSocket = socket(PF_INET, SOCK_STREAM, 0);
>         memset((char *) &sad, 0, sizeof(sad)); // clear socketaddr structure
>         sad.sin_family = AF_INET; // set family to  Internet
>         sad.sin_addr.s_addr = INADDR_ANY; // set the local IP address
>         sad.sin_port = htons((unsigned short)port); // set the port number
>     bind(welcomeSocket, (struct sockaddr *) &sad, sizeof(sad));
> 
>     /* Specify the maximum number of clients that can be queued */
>     listen(welcomeSocket, 10) // 队列长度为10，超过10个连接建立请求就拒绝
> 
>     while(1):
>     {
>         connectionSocket = accept(welcomeSocket, (struct sockaddr *) &cad, &alen); // Wait on welcoming socket for contact by a client
> 
>         n = read(connectionSocket, clientSentence, sizeof(clientSentence));
> 
>         /* capitalize Sentence and store the result in capitalizedSentence */
> 
>         n = write(connectionSocket, capitalizedSentence, strlen(capitalizedSentence)+1) // Write out the result to socket
> 
>         close(connectionSocket);
>     } // End of while loop, loop back and wait for another client connection
> }
> 
> ```

## 2.9 小结

- 应用程序体系结构
    - 客户-服务器（C/S）
    - P2P
    - 混合
- 应用程序需要的服务品质描述：
    - 可靠性、带宽、延时、安全
- Internet传输层服务模式
    - 可靠的、面向连接的服务：TCP
    - 不可靠的数据报：UDP
- 流行的应用层协议:
    - HTTP
    - FTP
    - SMTP, POP, IMAP
    - DNS
- Socket编程

更重要的：学习协议的知识
- 应用层协议报文类型：请求/响应报文：
    - 客户端请求信息或服务
    - 服务器以数据、状态码进行响应
- 报文格式：
    - 首部：关于数据信息的字段
    - 数据：被交换的信息
- 控制报文 vs. 数据报文
    - 带内、带外
- 集中式 vs. 分散式
- 无状态 vs. 维护状态
- 可靠的 vs. 不可靠的报文传输
- 在网络边缘处理复杂性

*一个协议定义了在两个或多个通信实体之间交换报文的格式和次序、以及就一条报文传输和接收或其他事件采取的动作*

