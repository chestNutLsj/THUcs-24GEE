## 静态存储器存储原理

### 结构

### 特点

- 速度快
- 存储密度低，单位面积存储容量小
- 数据入/出共用管脚
- 能耗高
- 成本高

![[31-Cache-compare.png]]

## 高速缓冲存储器（Cache）概述

### 利用局部性原理

使用高速缓冲存储器 Cache 来提高 CPU 对存储器的平均访问速度。

**时间局部性**：
- 最近被访问的信息很可能还要被访问。
- 将最近被访问的信息项装入到 Cache 中。


**空间局部性**：
- 最近被访问的信息临近的信息也可能被访问。
- 将最近被访问的信息项临近的信息一起装入到 Cache 中。

### 参数

- 块（Line）：数据交换的最小单位
- 命中（Hit）：在较高层次中发现要访问的内容
	- 命中率（Hit Rate）：命中次数/访问次数
	- 命中时间：访问在较高层次中数据的时间
- 缺失（Miss）：需要在较低层次中访问块
	- 缺失率（Miss Rate）：1-命中率
	- 缺失损失（Miss Penalty）：替换较高层次数据块的时间 + 将该块交付给处理器的时间
- 命中时间<<缺失损失
- 平均访问时间=HR * 命中时间 + (1-HR) * 缺失损失

参数典型数值
- 块大小：4~128 Bytes
- 命中时间：1~4 周期
- 失效损失：
	- 访问时间：6~10 个周期
	- 传输时间：2~22 个周期
- 命中率：80%~99%
- Cache 容量：1KB~256KB

![[31-Cache-structure.png]]

### 要解决的问题

1. 地址和 Cache 行之间的映射关系：
	- 如何根据主存地址得到 Cache 中的数据？

2. 数据之间一致性：
	- Cache 中的内容是否已经是主存对应地址的内容？ 

3. 数据交换的粒度：
	- Cache 中的内容与主存内容以多大的粒度交换？ 

4. Cache 内容装入和替换策略
	- 如何提高 Cache 的命中率？

## Cache 的地址映射

### 直接映射

![[31-Cache-direct-mapped.png]]

![[31-Cache-direct-mapped-cache.png]]

**特点**
1. 主存的字块只可以和固定的 Cache 字块对应，方式直接，利用率低。
2. 标志位较短，比较电路的成本低。 如果主存空间有 2^m 块，Cache 中字块有 2^c 块，则标志位只要有 m-c 位。且 仅需要比较一次。
3. 利用率低，命中率低，效率较低

### 多路组相联

![[31-Cache-set-associative-cache.png]]

![[31-Cache-set-associate.png]]
![[31-Cache-2-way.png]]

**特点**：
1. 前两种方式的折衷方案。组内为全相连，组间为直接映射。
2. 集中了两个方式的优点。成本也不太高。
3. 是常用的方式

![[31-Cache=4-way.png]]
### 全相联

![[31-Cache-fully-associative-cache.png]]

特点：
1. 主存的字块可以和 Cache 的任何字块对应，利用率高，方式灵活。
2. 标志位较长，比较电路的成本太高。如果主存空间有 2^m 块，则标志位要有 m 位。同时，如果 Cache 有 n 块，则需要有 n 个比较电路。 使用成本太高

![[31-Cache-fully.png]]

## 一致性保证

### 写回策略

写直达（Write through）
- 强一致性保证，效率低
- 在 Cache 中命中：同时修改 Cache 和对应的主存内容
- 没有在 Cache 中命中：
	- 写分配（Write allocate）
	- 非写分配（not Write allocate ）

- 拖后写（Write back）
	- 弱一致性保证，替换时再写主存
		- 主动替换
		- 被动替换
	- 通过监听总线上的访问操作来实现
	- 实现复杂，效率比较高

![[31-Cache-write-back-vs-write-through.png]]

### 多核 cache 一致性保证策略 MESI

要保证本地 cache 的数据，其它核 cache 的数据，内存的数据有一个一致的视图
- 修改态（M）：处于这个状态的 cache 块中的数据已经被修改过，和主存中对应的数据已不同，只能从 cache 中读到正确的数据
- 独占态（E）：处于本状态的 cache 块的数据和主存中对应的数据块内容相同，而且在其它 cache 中没有副本
- 共享态（S）：处于本状态的 cache 块的数据和主存中对应的数据块内容相同，而且可能在其它 cache 中有该块的副本
- 无效态（I）：处于本状态的 cache 块中尚未装入数据

![[31-Cache-MESI.png]]

## 提高存储访问的性能

平均访问时间 = 命中时间 x 命中率 + 缺失损失 x 缺失率
- 提高命中率
- 缩短缺失时的访问时间
- 提高 Cache 本身的速度

### Cache 缺失的原因

- 必然缺失（Compulsory Miss）
	- 开机或者是进程切换（冷启动，需要热身）
	- 首次访问数据块
- 容量缺失（Capacity Miss）
	- 活动数据集超出了 Cache 的大小
- 冲突缺失（Conflict Miss）
	- 多个内存块映射到同一 Cache 块
	- 某一 Cache 组块已满，但空闲的 Cache 块在其他组
- 无效缺失
	- 其他进程修改了主存数据

对策：
- 必然缺失
	- 世事总有缺憾
	- 如果程序访问存储器的次数足够多，也就可以忽略了（程序先热个身）
	- 策略 —— 预取
- 容量缺失
	- 出现在 Cache 容量太小的时候
	- 增加 Cache 容量，可缓解缺失现象
- 冲突缺失
	- 两块不同的内存块映射到相同的 Cache 块
	- 对直接映射的 Cache，这个问题尤其突出
		- 增加 Cache 容量有助于缓解冲突
		- 增加相联的路数有助于缓解冲突

### 影响命中率的因素

- Cache 容量
	- 大容量可以提高命中率，但是贵
- Cache 块大小
	- 选择多大的行，还真是个问题
- 地址映射方式
	- 多路组相联，但到底多少路呢？
- 替换算法
	- 替换哪行出去呢？
- 多级 Cache
	- 给用户更多的选择

![[31-Cache-hit-rate-vs-capacity.png]]

![[31-Cache-hit-rate-vs-cache-size-1.png]]
- 一般来说，数据块较大可以更好地利用空间局部性,
- 但是：
	- 数据块大意味着缺失损失的增大：需要花费更长的时间来装入数据块
	- 若块大小相对 Cache 总容量来说太大的话，命中率将降低：Cache 块数太少

![[31-Cache-factors.png]]

### 替换策略

直接映射
- 主存中的一块只能映射到 Cache 中唯一的一个位置
- 定位时，不需要选择，只需替换

全相联映射
- 主存中的一块可以映射到 Cache 中任何一个位置

N 路组相联映射
- 主存中的一块可以选择映射到 Cache 中 N 个位置


全相联映射和 N 路组相联映射的失效处理
- 从主存中取出新块
- 为了腾出 Cache 空间，需要替换出一个 Cache 块
- 不唯一，则需要选择应替出哪块
	- 最近最少使用 LRU
		- 满足程序局部性要求
		- 有较高命中率
		- 硬件实现复杂
	- 先进先出 FIFO
		- 满足时间局部性
		- 实现比较简单
	- 随机替换 RAND
		- 实现简单
		- 命中率也不太低

### 多级 cache

采用两级或更多级 cache 来提高命中率
- 增加 Cache 层次
- 增加了用户的选择
- 将 Cache 分解为指令 Cache 和数据 Cache
	- 指令流水的现实要求（缓解结构冲突）
	- 根据具体情况，选用不同的组织方式、容量

![[31-Cache-multilevel.png]]
