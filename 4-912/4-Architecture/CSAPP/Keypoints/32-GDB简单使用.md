## 开始和停止
- quit
	- 退出GDB
- run
	- 运行程序（在此给出命令行参数）
- kill
	- 停止程序

## 断点
- break func_name
	- 在函数 func_name 入口处设置断点
- break * 0x400540
	- 在地址 0x400540 处设置断点
- delete 1
	- 删除断点 1
- delete 
	- 删除所有断点

## 执行
- stepi
	- 执行 1 条指令
- stepi 4
	- 执行 4 条指令
- nexti
	- 类似于 stepi，但是以函数调用为单位
- continue
	- 继续执行
- finish
	- 运行到当前函数返回

## 检查代码
- disas
	- 反汇编当前函数
- disas func_name
	- 反汇编指定函数
- disas 0x400544
	- 反汇编位于地址 0x400544 附近的函数
- disas 0x400540,0x40054d
	- 反汇编指定地址范围内的代码
- print /x $rip
	- 以十六进制输出程序计数器的值

## 检查数据
- print $rax
	- 以十进制输出%rax 内容
- print /x $rax
	- 十六进制输出%rax
- print /t $rax
	- 二进制输出%rax
- print 0x100
	- 输出 0x100 的十进制表示
- print /x 555
	- 输出 555 的十六进制
- print /x ($rsp+8)
	- 按十六进制输出%rsp+8
- print * (long * ) 0x7fffffffe818
	- 输出位于 0x7fffffffe818 地址的长整数
- print * (long * ) ($rsp+8)
	- 输出位于%rsp+8 地址的长整数
- x/2g 0x7fffffffe818
	- 检查从地址 0x7fffffffe818 开始的双 8 字节的字
- x/20bfunc_name
	- 检查函数的前 20 个字节

## 有用的信息
- info frame
	- 有关当前栈帧的信息
- info registers
	- 所有寄存器的值
- help
	- 获取 GDB 的相关信息