## 本节导读

本节我们介绍为何要把标准输入 / 输出用文件来进行抽象，以及如何以文件和文件描述符概念来重新定义标准输入 / 输出，并在进程中把各种文件描述符组织到文件描述符表中，同时将进程对于标准输入输出的访问修改为基于文件抽象的接口实现。这主要是为下一节支持进程间信息传递的管道实现打下基础。

由于管道是基于文件抽象接口来实现的，所以我们将首先对 **一切皆是文件** 的设计思路进行介绍。

## 一切皆是文件

在 UNIX 操作系统之前，大多数的操作系统提供了各种复杂且不规则的设计实现来处理各种 I/O 设备（也可称为 I/O 资源），如键盘、显示器、以磁盘为代表的存储介质、以串口为代表的通信设备等，使得应用程序开发繁琐且很难统一表示和处理 I/O 设备。

随着 UNIX 的诞生，一个简洁优雅的 I/O 设备抽象出现了，这就是 **文件** 。在 UNIX 操作系统中，“**一切皆文件**” (Everything is a file) 是一种重要的设计思想，这种设计思想继承于 Multics 操作系统的 **通用性** 文件的设计理念，并进行了进一步的简化。

在本章中，应用程序访问的 **文件** (File) 就是一系列的字节组合。操作系统管理文件，但操作系统不关心文件内容，只关心如何对文件按字节流进行读写的机制，这就意味着任何程序可以读写任何文件（即字节流），==对文件具体内容的解析是应用程序的任务，操作系统对此不做任何干涉==。例如，一个 Rust 编译器可以读取一个 C 语言源程序并进行编译，操作系统并不会阻止这样的事情发生。

有了文件这样的抽象后，操作系统内核就可把能读写的 I/O 资源按文件来进行管理，并把文件分配给进程，让进程以统一的文件访问接口与 I/O 资源进行交互。在目前和后续可能涉及到的 I/O 硬件设备中，大致可以分成以下几种：

* **键盘设备** 是程序获得字符输入的一种设备，也可抽象为一种==只读性质的文件==，可以从这个文件中读出一系列的字节序列；

* **屏幕设备** 是展示程序的字符输出结果的一种字符显示设备，可抽象为一种==只写性质的文件==，可以向这个文件中写入一系列的字节序列，在显示屏上可以直接呈现出来；

* **串口设备** 
	* 是==获得字符输入和展示程序的字符输出结果的一种字符通信设备，可抽象为一种可读写性质的文件==，可以向这个文件中写入一系列的字节序列传给程序，也可把程序要显示的字符传输出去；
	* 还可以把这个串口设备==拆分成两个文件，一个用于获取输入字符的只读文件和一个传出输出字符的只写文件==。


在 QEMU 模拟的 RISC-V 计算机和 K210 物理硬件上存在虚拟 / 物理串口设备，开发者可通过 QEMU 的串口命令行界面或特定串口通信工具软件来对虚拟 / 物理串口设备进行输入 / 输出操作。由于 RustSBI 直接管理了串口设备，并给操作系统提供了基于串口收发字符的两个 SBI 接口，从而使得操作系统可以很简单地通过这两个 SBI 接口，完成输出或输入字符串的工作。

## 标准输入 / 输出对 `File trait` 的实现

其实我们在第二章就对应用程序引入了基于 **文件** 的标准输出接口 `sys_write` ，在第五章引入了基于 **文件** 的标准输入接口 `sys_read` ；在第六章引入 **文件系统** ，在进程控制块中添加了表示打开文件集合的文件描述符表。

我们提前把标准输出设备在文件描述符表中的文件描述符的值规定为 `1` ，用 `Stdout` 表示；把标准输入设备在文件描述符表中的文件描述符的值规定为 `0`，用 `Stdin` 表示 。现在，我们可以重构操作系统，为标准输入和标准输出实现 `File` Trait，使得进程可以按文件接口与 I/O 外设进行交互：
```
// os/src/fs/stdio.rs

pub struct Stdin;

pub struct Stdout;

impl File for Stdin {
    fn read(&self, mut user_buf: UserBuffer) -> usize {
        assert_eq!(user_buf.len(), 1);
        // busy loop
        let mut c: usize;
        loop {
            c = console_getchar();
            if c == 0 {
                suspend_current_and_run_next();
                continue;
            } else {
                break;
            }
        }
        let ch = c as u8;
        unsafe { user_buf.buffers[0].as_mut_ptr().write_volatile(ch); }
        1
    }
    fn write(&self, _user_buf: UserBuffer) -> usize {
        panic!("Cannot write to stdin!");
    }
}

impl File for Stdout {
    fn read(&self, _user_buf: UserBuffer) -> usize{
        panic!("Cannot read from stdout!");
    }
    fn write(&self, user_buf: UserBuffer) -> usize {
        for buffer in user_buf.buffers.iter() {
            print!("{}", core::str::from_utf8(*buffer).unwrap());
        }
        user_buf.len()
    }
}
```

可以看到，
1. 标准输入文件 `Stdin` 是只读文件，只允许进程通过 `read` 从里面读入，目前每次仅支持读入一个字符，其实现与之前的 `sys_read` 基本相同，只是需要通过 `UserBuffer` 来获取具体将字节写入的位置。
2. 相反，标准输出文件 `Stdout` 是只写文件，只允许进程通过 `write` 写入到里面，实现方法是遍历每个切片，将其转化为字符串通过 `print!` 宏来输出。

## 对标准输入 / 输出的管理

这样，应用程序如果要基于文件进行 I/O 访问，大致就会涉及如下几个操作：

* 打开（open）：进程只有打开文件，操作系统才能返回一个可进行读写的文件描述符给进程，进程才能基于这个值来进行对应文件的读写；

* 关闭（close）：进程基于文件描述符关闭文件后，就不能再对文件进行读写操作了，这样可以在一定程度上保证对文件的合法访问；

* 读（read）：进程可以基于文件描述符来读文件内容到相应内存中；

* 写（write）：进程可以基于文件描述符来把相应内存内容写到文件中；

在本节中，还不会涉及创建文件。当一个进程被创建的时候，内核会默认为其打开三个缺省就存在的文件：

* 文件描述符为 0 的标准输入

* 文件描述符为 1 的标准输出

* 文件描述符为 2 的标准错误输出


在我们的实现中并不区分标准输出和标准错误输出，而是会将文件描述符 1 和 2 均对应到标准输出。实际上，在本章中，标准输出文件就是串口输出，标准输入文件就是串口输入。

这里隐含着有关文件描述符的一条重要的规则：即==进程打开一个文件的时候，内核总是会将文件分配到该进程文件描述符表中 **最小的** 空闲位置==。比如，当一个进程被创建以后立即打开一个文件，则内核总是会返回文件描述符 3 （0~2 号文件描述符已被缺省打开了）。当我们关闭一个打开的文件之后，它对应的文件描述符将会变得空闲并在后面可以被分配出去。

### 创建标准输入 / 输出文件

当新建一个进程的时候，我们需要按照先前的说明为进程打开标准输入文件和标准输出文件：
``` hl:18-25
// os/src/task/task.rs

impl TaskControlBlock {
    pub fn new(elf_data: &[u8]) -> Self {
        ...
        let task_control_block = Self {
            pid: pid_handle,
            kernel_stack,
            inner: Mutex::new(TaskControlBlockInner {
                trap_cx_ppn,
                base_size: user_sp,
                task_cx_ptr: task_cx_ptr as usize,
                task_status: TaskStatus::Ready,
                memory_set,
                parent: None,
                children: Vec::new(),
                exit_code: 0,
                fd_table: vec![
                    // 0 -> stdin
                    Some(Arc::new(Stdin)),
                    // 1 -> stdout
                    Some(Arc::new(Stdout)),
                    // 2 -> stderr
                    Some(Arc::new(Stdout)),
                ],
            }),
        };
        ...
    }
}
```

### 继承标准输入 / 输出文件

此外，在 fork 的时候，**子进程需要完全继承父进程的文件描述符表**来和父进程共享所有文件：
``` hl:8-16,29
// os/src/task/task.rs

impl TaskControlBlock {
    pub fn fork(self: &Arc<TaskControlBlock>) -> Arc<TaskControlBlock> {
        ...
        // push a goto_trap_return task_cx on the top of kernel stack
        let task_cx_ptr = kernel_stack.push_on_top(TaskContext::goto_trap_return());
        // copy fd table
        let mut new_fd_table: Vec<Option<Arc<dyn File + Send + Sync>>> = Vec::new();
        for fd in parent_inner.fd_table.iter() {
            if let Some(file) = fd {
                new_fd_table.push(Some(file.clone()));
            } else {
                new_fd_table.push(None);
            }
        }
        let task_control_block = Arc::new(TaskControlBlock {
            pid: pid_handle,
            kernel_stack,
            inner: Mutex::new(TaskControlBlockInner {
                trap_cx_ppn,
                base_size: parent_inner.base_size,
                task_cx_ptr: task_cx_ptr as usize,
                task_status: TaskStatus::Ready,
                memory_set,
                parent: Some(Arc::downgrade(self)),
                children: Vec::new(),
                exit_code: 0,
                fd_table: new_fd_table,
            }),
        });
        // add child
        ...
    }
}
```

这样，即使我们仅手动为初始进程 `initproc` 打开了标准输入输出，所有进程也都可以访问它们。

## 读写标准输入 / 输出文件

由于有基于文件抽象接口和文件描述符表，之前实现的文件读写系统调用 `sys_read/write` 可以直接用于标准输入 / 输出文件，很好地达到了代码重用的目标。

这样，操作系统通过文件描述符在当前进程的文件描述符表中找到某个文件，无需关心文件具体的类型，只要知道它一定实现了 `File` Trait 的 `read/write` 方法即可。==Trait 对象提供的运行时多态能力会在运行的时候帮助我们定位到符合实际类型的 `read/write` 方法，完成不同类型文件各自的读写==。