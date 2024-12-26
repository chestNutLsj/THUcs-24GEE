>[! warning]
>注意，这部分内容来自 uCore 的实验指导书，但原理与 rCore 类似

## 调度算法框架

调度算法框架实现为一个结构体，其中保存了各个函数指针。通过实现这些函数指针即可实现各个调度算法。结构体的定义如下：

```
struct sched_class{
	//调度类的名字
	const char *name;
	//初始化 run queue
	void (*init)(struct run_queue *rq);
	//把进程放进 run queue，这个是 run queue 的维护函数
	void (*enqueue)(struct run_queue *rq, struct proc_struct *proc);
	//把进程取出 run queue，
	void (*dequeue)(struct run_queue *rq, struct proc_struct *proc);
	//选择下一个要执行的进程
	struct proc_struct *(*pick_next)(struct run_queue *rq);
	//每次时钟中断调用
	void (*proc_tick)(struct run_queue *rq, struct proc_struct *proc);
};
```

所有的进程被组织成一个 `run_queue` 数据结构。这个数据结构虽然没有保存在调度类中，但是是由调度类来管理的。目前 ucore 仅支持单个 CPU 核心，所以只有一个全局的 `run_queue`。

我们在进程控制块中也记录了一些和调度有关的信息：
```
struct proc_struct {
    // ...
    // 表示这个进程是否需要调度
    volatile bool need_resched;
    // run queue的指针
    struct run_queue *rq;
    // 与这个进程相关的run queue表项
    list_entry_t run_link;
    // 这个进程剩下的时间片
    int time_slice;
    // 以下几个都和Stride调度算法实现有关
    // 这个进程在优先队列中对应的项
    skew_heap_entry_t lab6_run_pool;
    // 该进程的Stride值
    uint32_t lab6_stride;
    // 该进程的优先级
    uint32_t lab6_priority;
};
```

前面的几个成员变量的含义都比较直接，最后面的几个的含义可以参见 Stride 调度算法。这也是本次 lab 的实验内容。

结构体 `run_queue` 实现了运行队列，其内部结构如下：
```
struct run_queue {
    // 保存着链表头指针
    list_entry_t run_list;
    // 运行队列中的线程数
    unsigned int proc_num;
    // 最大的时间片大小
    int max_time_slice;
    // Stride调度算法中的优先队列
    skew_heap_entry_t *lab6_run_pool;
};
```

## RR 
有了这些基础，我们就来实现一个最简单的调度算法：Round-Robin 调度算法，也叫时间片轮转调度算法。

时间片轮转调度算法非常简单。它为每一个进程维护了一个最大运行时间片。当一个进程运行够了其最大运行时间片那么长的时间后，调度器会把它标记为需要调度，并且把它的进程控制块放在队尾，重置其时间片。这种调度算法保证了公平性，每个进程都有均等的机会使用 CPU，但是没有区分不同进程的优先级（这个也就是在 Stride 算法中需要考虑的问题）。下面我们来实现以下时间片轮转算法相对应的调度器借口吧！

首先是 `enqueue` 操作。RR 算法直接把需要入队的进程放在调度队列的尾端，并且如果这个进程的剩余时间片为 0（刚刚用完时间片被收回 CPU），则需要把它的剩余时间片设为最大时间片。具体的实现如下：

```
static void
RR_enqueue(struct run_queue *rq, struct proc_struct *proc) {
    assert(list_empty(&(proc->run_link)));
    list_add_before(&(rq->run_list), &(proc->run_link));
    if (proc->time_slice == 0 || proc->time_slice > rq->max_time_slice) {
        proc->time_slice = rq->max_time_slice;
    }
    proc->rq = rq;
    rq->proc_num ++;
}
```

`dequeue` 操作非常普通，将相应的项从队列中删除即可：
```
static void
RR_dequeue(struct run_queue *rq, struct proc_struct *proc) {
    assert(!list_empty(&(proc->run_link)) && proc->rq == rq);
    list_del_init(&(proc->run_link));
    rq->proc_num --;
}
```

`pick_next` 选取队列头的表项，用 `le2proc` 函数获得对应的进程控制块，返回：
```
static struct proc_struct *
RR_pick_next(struct run_queue *rq) {
    list_entry_t *le = list_next(&(rq->run_list));
    if (le != &(rq->run_list)) {
        return le2proc(le, run_link);
    }
    return NULL;
}
```

`proc_tick` 函数在每一次时钟中断调用。在这里，我们需要对当前正在运行的进程的剩余时间片减一。如果在减一后，其剩余时间片为 0，那么我们就把这个进程标记为 “需要调度”，这样在中断处理完之后内核判断进程是否需要调度的时候就会把它进行调度：
```
static void
RR_proc_tick(struct run_queue *rq, struct proc_struct *proc) {
    if (proc->time_slice > 0) {
        proc->time_slice --;
    }
    if (proc->time_slice == 0) {
        proc->need_resched = 1;
    }
}
```

至此我们就实现完了和时间片轮转算法相关的所有重要接口。类似于 RR 算法，我们也可以参照这个方法实现自己的调度算法。本次实验中需要同学们自己实现 Stride 调度算法。

## Stride 

考察 `round-robin` 调度器，在假设所有进程都充分使用了其拥有的 CPU 时间资源的情况下，所有进程得到的 CPU 时间应该是相等的。但是有时候我们希望调度器能够更智能地为每个进程分配合理的 CPU 资源。假设我们为不同的进程分配不同的优先级，则我们有可能希望每个进程得到的时间资源与他们的优先级成正比关系。Stride 调度是基于这种想法的一个较为典型和简单的算法。除了简单易于实现以外，它还有如下的特点：

* 可控性：如我们之前所希望的，可以证明 Stride Scheduling 对进程的调度次数正比于其优先级。

* 确定性：在不考虑计时器事件的情况下，整个调度机制都是可预知和重现的。该算法的基本思想可以考虑如下：
	1. 为每个 runnable 的进程设置一个当前状态 stride，表示该进程当前的调度权。另外定义其对应的 pass 值，表示对应进程在调度后，stride 需要进行的累加值。
	2. 每次需要调度时，从当前 runnable 态的进程中选择 stride 最小的进程调度。
	3. 对于获得调度的进程 P，将对应的 stride 加上其对应的步长 pass（只与进程的优先权有关系）。
	4. 在一段固定的时间之后，回到 2. 步骤，重新调度当前 stride 最小的进程。

可以证明，如果令 `P.pass = BigStride / P.priority` 其中 `P.priority` 表示进程的优先权（大于 1），而 `BigStride` 表示一个预先定义的大常数，则该调度方案为每个进程分配的时间将与其优先级成正比。证明过程我们在这里略去，有兴趣的同学可以在网上查找相关资料。

### 接口
将该调度器应用到 ucore 的调度器框架中来，则需要将调度器接口实现如下：

* `init`
	* 初始化调度器类的信息
	* 初始化当前的运行队列为一个空的容器结构。（比如和 RR 调度算法一样，初始化为一个有序列表）

* `enqueue`
	* 初始化刚进入运行队列的进程 proc 的 stride 属性。 
	* 将 proc 插入放入运行队列中去（注意：这里并不要求放置在队列头部）。
 
* `dequeue` 
	* 扫描整个运行队列，返回其中 stride 值最小的对应进程。  
	* 更新对应进程的 stride 值，即 `pass = BIG_STRIDE / P->priority;  P ->stride += pass`。

* `proc_tick` 
	* 检测当前进程是否已用完分配的时间片。如果时间片用完，应该正确设置进程结构的相关标记来引起进程切换。   
	* 一个 process 最多可以连续运行 `rq.max_time_slice` 个时间片。


### stride 溢出问题

在具体实现时，有一个需要注意的地方：stride 属性的溢出问题，在之前的实现里面我们并没有考虑 stride 的数值范围，而这个值在理论上是不断增加的，在 stride 溢出以后，基于 stride 的比较可能会出现错误。比如假设当前存在两个进程 A 和 B，stride 属性采用 16 位无符号整数进行存储。当前队列中元素如下（假设当前运行的进程已经被重新放置进运行队列中）：

|                  |                  |                               | 
| ---------------- | ---------------- | ----------------------------- |
| A.stride(实际值) | A.stride(理论值) | A.pass(=BigStride/A.priority) |
| 65534            | 65534            | 100                           |
| B.stride(实际值) | B.stride(理论值) | B.pass(=BigStride/B.priority) |
| 65535            | 65535            | 50                            |

此时应该选择 A 作为调度的进程，而在一轮调度后，队列将如下：

|                  |                  |                               |
| ---------------- | ---------------- | ----------------------------- |
| A.stride(实际值) | A.stride(理论值) | A.pass(=BigStride/A.priority) |
| 98               | 65634            | 100                           |
| B.stride(实际值) | B.stride(理论值) | B.pass(=BigStride/B.priority) |
| 65535            | 65535            | 50                            |

可以看到由于溢出的出现，进程间 stride 的理论比较和实际比较结果出现了偏差。我们首先在理论上分析这个问题：
- 令 `PASS_MAX` 为当前所有进程里最大的步进值。
- 则我们可以证明如下结论：对每次 Stride 调度器的调度步骤中，有其最大的步进值 `STRIDE_MAX` 和最小的步进值 `STRIDE_MIN` 之差：`STRIDE_MAX – STRIDE_MIN <= PASS_MAX`

有了该结论，在加上之前对优先级有 `Priority > 1` 限制，我们有 `STRIDE_MAX – STRIDE_MIN <= BIG_STRIDE`, 于是我们只要将 BigStride 取在某个范围之内，即可保证对于任意两个 Stride 之差都会在机器整数表示的范围之内。而我们可以通过其与 0 的比较结果，来得到两个 Stride 的大小关系。

在上例中，虽然在直接的数值表示上 98 < 65535，但是 98 - 65535 的结果用带符号的 16 位整数表示的结果为 99, 与理论值之差相等。所以在这个意义下 98> 65535。基于这种特殊考虑的比较方法，即便 Stride 有可能溢出，我们仍能够得到理论上的当前最小 Stride，并做出正确的调度决定。

## 参考信息

如果有兴趣进一步了解 stride 调度相关内容，可以尝试看看：

- [作者 Carl A. Waldspurger 写这个调度算法的原论文](https://people.cs.umass.edu/~mcorner/courses/691J/papers/PS/waldspurger_stride/waldspurger95stride.pdf)
- [作者 Carl A. Waldspurger 的博士生答辩slide](http://www.waldspurger.org/carl/papers/phd-mit-slides.pdf)

- [南开大学实验指导中对Stride算法的部分介绍](https://nankai.gitbook.io/ucore-os-on-risc-v64/lab6/tiao-du-suan-fa-kuang-jia#stride-suan-fa)

- [NYU OS课关于Stride Scheduling的Slide](https://cs.nyu.edu/~rgrimm/teaching/sp08-os/stride.pdf)

- OSTEP 中也有 stride 的详细讲解

如果有兴趣进一步了解用户态线程实现的相关内容，可以尝试看看：

- [user-multitask in rv64](https://github.com/chyyuu/os_kernel_lab/tree/v4-user-std-multitask)

- [绿色线程 in x86](https://github.com/cfsamson/example-greenthreads)

- [x86版绿色线程的设计实现](https://cfsamson.gitbook.io/green-threads-explained-in-200-lines-of-rust/)

- [用户级多线程的切换原理](https://blog.csdn.net/qq_31601743/article/details/97514081?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control&dist_request_id=&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.control)