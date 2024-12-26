---
url: https://blog.csdn.net/abc123lzf/article/details/109289567
title: (25 条消息) x86 保护模式——全局描述符表 GDT 详解_gdt 字符_A__Plus 的博客 - CSDN 博客
date: 2023-07-20 17:04:19
tag: 
summary: 
---
## 1. GDT 作用

GDT 全称 [Global](https://so.csdn.net/so/search?q=Global&spm=1001.2101.3001.7020) Descriptor Table，是 x86 保护模式下的一个重要数据结构，在保护模式下，GDT 在内存中有且只有一个。GDT 的数据结构是一个描述符数组，每个描述符 8 个字节，可以存放在内存当中任意位置：  

![](1689843859453.png)

  
其中，`addr`相当于 GDT 的基地址，GDT 的总长度（单位字节）为 GDT 界限。

在实模式中，CPU 通过段地址和段偏移量寻址。其中段地址保存到段寄存器，包含：CS、SS、DS、ES、FS、GS。段偏移量可以保存到 IP、BX、SI、DI 寄存器。在汇编代码`mov ds:[si], ax`中，会将 AX 寄存器的数据写入到物理内存地址`DS * 16 + SI`中。

而在保护模式下，也是通过段寄存器和段偏移量寻址，但是此时段寄存器保存的数据意义不同了。  
此时的 CS 和 SS 寄存器后 13 位相当于 GDT 表中某个描述符的索引，即**段选择子**。第 2 位存储了 TI 值（0 代表 GDT，1 代表 LDT），第 0、1 位存储了当前的特权级（CPL）。  

![](1689843860942.png)

  
例如在保护模式下执行汇编代码`mov ds:[si], ax`的大致步骤如下：

1.  首先 CPU 需要查找 GDT 在内存中位置，GDT 的位置从 GDTR 寄存器中直接获取
2.  然后根据 DS 寄存器得到目标段描述符的物理地址
3.  计算出描述符中的段基址的值加上 SI 寄存器存储的偏移量的结果，该结果为目标物理地址
4.  将 AX 寄存器中的数据写入到目标物理地址

#### 2 - GDTR 寄存器

CPU 切换到保护模式前，需要准备好 GDT 数据结构，并执行`LGDT`指令，将 GDT 基地址和界限传入到 GDTR 寄存器。

GDTR 寄存器长度为 6 字节（48 位），前两个字节为 GDT 界限，后 4 个字节为 GDT 表的基地址。所以说，GDT 最多只能拥有 8192 个描述符（`65536 / 8`）。  
[外链图片转存失败, 源站可能有防盗链机制, 建议将图片保存下来直接上传 (img-AXzy19kY-1603970086921)(#pic_center)]

![](1689843861311.png)

一旦切换到保护模式，一般不会更改 GDTR 寄存器的内容。

#### 3 - GDT 段描述符结构

一个 GDT 段描述符占用 8 个字节，包含三个部分：

*   段基址（32 位），占据描述符的第 16～39 位和第 55 位～63 位，前者存储低 16 位，后者存储高 16 位
*   段界限（20 位），占据描述符的第 0～15 位和第 48～51 位，前者存储低 16 位，后者存储高 4 位。
*   段属性（12 位），占据描述符的第 39～47 位和第 49～55 位，段属性可以细分为 8 种：`TYPE`属性、`S`属性、`DPL`属性、`P`属性、`AVL`属性、`L`属性、`D/B`属性和`G`属性。

![](1689843862416.png)

下面介绍各个属性的作用：

##### S 属性

S 属性存储了描述符的类型

*   S = 0 S=0 S=0 时，该描述符对应的段是系统段（System Segment）。
*   S = 1 S=1 S=1 时，该描述符对应的段是数据段（Data Segment）或者代码段（Code Segment）

##### TYPE 属性

TYPE 属性存储段的类型信息，该属性的意义随着 S 属性不同而不同。  
当 S = 1 S=1 S=1 （该段为数据段或代码段）时，需要分为两种情况：

*   当 TYPE 属性第三位为 0 时，代表该段为数据段，第 0～2 位的作用为：
    
    <table><thead><tr><th>位</th><th>作用</th><th>值为 0 时</th><th>值为 1 时</th></tr></thead><tbody><tr><td>2</td><td>段的增长方向</td><td>向上增长</td><td>向下增长（例如栈段）</td></tr><tr><td>1</td><td>段的写权限</td><td>只读</td><td>可读可写</td></tr><tr><td>0</td><td>段的访问标记</td><td>该段未被访问过</td><td>该段已被访问过</td></tr></tbody></table>
    
    （第 0 位对应描述符的第 43 位，第 1 位对应第 42 位，以此类推）
*   当 TYPE 属性第三位为 1 时，代表该段为代码段，第 0～2 位的作用为：
    
    <table><thead><tr><th>位</th><th>作用</th><th>值为 0 时</th><th>值为 1 时</th></tr></thead><tbody><tr><td>2</td><td>一致代码段标记</td><td>不是一致代码段</td><td>是一致代码段</td></tr><tr><td>1</td><td>段的读权限</td><td>只能执行</td><td>可读、可执行</td></tr><tr><td>0</td><td>段的访问标记</td><td>该段未被访问过</td><td>该段已被访问过</td></tr></tbody></table>
    
    一致代码段的 “一致” 意思是：当 CPU 执行`jmp`等指令将 CS 寄存器指向该代码段时，如果当前的特权级低于该代码段的特权级，那么当前的特权级会被延续下去（简单的说就是可以被低特权级的用户直接访问的代码），如果当前的特权级高于该代码段的特权级，那么会触发常规保护错误（可以理解为内核态下不允许直接执行用户态的代码）。  
    如果不是一致代码段并且该代码段的特权级**不等于**（高于和低于都不行）当前的特权级，那么会引发常规保护错误。

当 S = 0 S=0 S=0 （该段为系统段）时：

<table><thead><tr><th>TYPE 的值（16 进制）</th><th>TYPE 的值（二进制）</th><th>解释</th></tr></thead><tbody><tr><td>0x1</td><td>0 0 0 1</td><td>可用 286TSS</td></tr><tr><td>0x2</td><td>0 0 1 0</td><td>该段存储了局部描述符表（LDT）</td></tr><tr><td>0x3</td><td>0 0 1 1</td><td>忙的 286TSS</td></tr><tr><td>0x4</td><td>0 1 0 0</td><td>286 调用门</td></tr><tr><td>0x5</td><td>0 1 0 1</td><td>任务门</td></tr><tr><td>0x6</td><td>0 1 1 0</td><td>286 中断门</td></tr><tr><td>0x7</td><td>0 1 1 1</td><td>286 陷阱门</td></tr><tr><td>0x9</td><td>1 0 0 1</td><td>可用 386TSS</td></tr><tr><td>0xB</td><td>1 0 1 1</td><td>忙的 386TSS</td></tr><tr><td>0xC</td><td>1 1 0 0</td><td>386 调用门</td></tr><tr><td>0xE</td><td>1 1 1 0</td><td>386 中断门</td></tr><tr><td>0xF</td><td>1 1 1 1</td><td>386 陷阱门</td></tr></tbody></table>

（其余值均为未定义）

##### DPL 属性

DPL 属性占 2 个比特，记录了访问段所需要的特权级，特权级范围为 0～3，越小特权级越高。

##### P 属性

P 属性标记了该段是否存在：

*   P = 0 P=0 P=0 时，该段在内存中不存在
*   P = 1 P=1 P=1 时，该段在内存中存在

尝试访问一个在内存中不存在的段会触发段不存在错误（#NP）

##### AVL 属性

AVL 属性占 1 个比特，该属性的意义可由操作系统、应用程序自行定义。  
Intel 保证该位不会被占用作为其他用途。

##### L 属性

该属性仅在 IA-32e 模式下有意义，它标记了该段是否为 64 位代码段。  
当 L = 1 L=1 L=1 时，表示该段是 64 位代码段。  
如果设置了 L 属性为 1，则必须保证 D 属性为 0。

##### D/B 属性

D/B 属性中的 D/B 全称 Default operation size / Default stack pointer size / Upper bound。  
该属性的意义随着段描述符是代码段（Code Segment）、向下扩展数据段（Expand-down Data Segment）还是栈段（Stack Segment）而有所不同。

*   代码段（S 属性为 1，TYPE 属性第三位为 1）  
    如果对应的是代码段，那么该位称之为 D 属性（D flag）。如果设置了该属性，那么会被视为 32 位代码段执行；如果没有设置，那么会被视为 16 位代码段执行。
    
*   栈段（被 SS 寄存器指向的数据段）  
    该情况下称之为 B 属性。如果设置了该属性，那么在执行堆栈访问指令（例如`PUSH`、`POP`指令）时采用 32 位堆栈指针寄存器（ESP 寄存器），如果没有设置，那么采用 16 位堆栈指针寄存器（SP 寄存器）。
    
*   向下扩展的数据段  
    该情况下称之为 B 属性。如果设置了该属性，段的上界为 4GB，否则为 64KB。
    

##### G 属性

G 属性记录了段界限的粒度：

*   G = 0 G=0 G=0 时，段界限的粒度为字节
*   G = 1 G=1 G=1 时，段界限的粒度为 4KB

例如，当 G = 0 G=0 G=0 并且描述符中的段界限值为 10000 10000 10000，那么该段的界限为 10000 字节，如果 G = 1 G=1 G=1，那么该段的界限值为 40000KB。

所以说，当 G = 0 G=0 G=0 时，一个段的最大界限值为 1MB（因为段界限只能用 20 位表示， 2 20 = 1048576 2^{20}=1048576 220=1048576），最小为 1 字节（段的大小等于段界限值加 1）。  
当 G = 1 G=1 G=1 时，最大界限值为 4GB，最小为 4KB。

在访问段（除栈段）时，如果超出了段的界限，那么会触发常规保护错误（#GP）  
如果访问栈段超出了界限，那么会产生堆栈错误（#SS）

#### 案例学习

实现保护模式下打印红色的 Hello, world（不依赖操作系统和 BIOS 中断服务）。

首先定义单个描述符的数据结构，用 NASM 汇编宏可以表示为

```
; 描述符
; usage: Descriptor Base, Limit, Attr
;        Base:  dd
;        Limit: dd (low 20 bits available)
;        Attr:  dw (lower 4 bits of higher byte are always 0)
%macro Descriptor 3
	dw	%2 & 0FFFFh				; 段界限 1				(2 字节)
	dw	%1 & 0FFFFh				; 段基址 1				(2 字节)
	db	(%1 >> 16) & 0FFh			; 段基址 2				(1 字节)
	dw	((%2 >> 8) & 0F00h) | (%3 & 0F0FFh)	; 属性 1 + 段界限 2 + 属性 2		(2 字节)
	db	(%1 >> 24) & 0FFh			; 段基址 3				(1 字节)
%endmacro
```

定义段属性常量：

```
DA_DR		EQU	90h	; 存在的只读数据段类型值
DA_DRW		EQU	92h	; 存在的可读写数据段属性值
DA_DRWA		EQU	93h	; 存在的已访问可读写数据段类型值
DA_C		EQU	98h	; 存在的只执行代码段属性值
DA_CR		EQU	9Ah	; 存在的可执行可读代码段属性值
DA_CCO		EQU	9Ch	; 存在的只执行一致代码段属性值
DA_CCOR		EQU	9Eh	; 存在的可执行可读一致代码段属性值
```

定义三个描述符：

*   负责打印字符串的代码段
    
    ```
    DESC_CODE: Descriptor 0, CODE32_LEN - 1, DA_C + DA_32
    ```
    
    此处段基址暂时设置为 0，在实模式下动态设置
*   存放需要打印的字符串（`Hello, world`字符串）的数据段
    
    ```
    DESC_DATA: Descriptor 0, STRING_LEN - 1, DA_DR
    ```
    
    同样设置为 0，在实模式下动态设置
*   显存映射到内存的数据段（固定在 0xB8000，如果不懂可以百度），并且该段要设置成可写
    
    ```
    DESC_VIDEO: Descriptor 0xB8000, 0xFFFF, DA_DRW
    ```
    

定义 GDTR 寄存器的数据：

```
GdtLen equ $ - DESC_GDT  ; GDT表的长度
GdtPtr dw GdtLen  ; GDT界限
       dd 0       ; GDT基址，暂时设置为0，需要动态设置
```

定义段选择子：

```
DataSelector equ DESC_DATA - DESC_GDT
CodeSelector equ DESC_CODE - DESC_GDT
VideoSelector equ DESC_VIDEO - DESC_GDT
```

因为 TI 和 DPL 都为 0，且正好占据 3 位，所以只需要将描述符的地址减去基址就可以得到索引了（等同于左移 3 位）

然后是实模式下的初始化代码，主要完成三件事：

1.  初始化段描述符
2.  初始化 GDT 的基址，并存放到 GDTR 寄存器
3.  切换到保护模式（打开 A20 地址线，将 CR0 寄存器第 0 位设置为 1），并跳转到负责打印字符串的代码段。

代码如下：

```
%include "pm.inc"

org 0x7C00
jmp _main_16

[SECTION .gdt]
DESC_GDT: Descriptor 0, 0, 0   ;该描述符仅用来计算下面三个描述符的地址
DESC_VIDEO: Descriptor 0xB8000, 0xFFFF, DA_DRW
DESC_DATA: Descriptor 0, STRING_LEN - 1, DA_DR
DESC_CODE: Descriptor 0, CODE32_LEN - 1, DA_C + DA_32

GdtLen equ $ - DESC_GDT
GdtPtr dw GdtLen
       dd 0

DataSelector equ DESC_DATA - DESC_GDT
CodeSelector equ DESC_CODE - DESC_GDT
VideoSelector equ DESC_VIDEO - DESC_GDT

[SECTION .s16]
[BITS 16]
_main_16:
	; 初始化DS/ES/SS/SP寄存器
	mov ax, cs
    mov ds, ax
    mov es, ax
    mov ss, ax
    mov sp, 0x7C00
 	; 初始化段描述符
	call near _init_desc

	; 初始化GDT基址
	xor eax, eax
    mov ax, ds
    shl eax, 4
    add eax, DESC_GDT
    mov dword [GdtPtr + 2], eax

	lgdt [GdtPtr]

	cli
	in al, 0x92
    or al, 00000010b
    out 0x92, al

	mov eax, cr0
	or eax, 1
	mov cr0, eax
	; 跳转到代码段
	jmp dword CodeSelector:0

_init_desc:
	xor eax, eax
	mov ax, cs
	shl eax, 4
	add eax, _main_32

	mov di, DESC_CODE
	call near _init_desc_base_address

	xor eax, eax
	mov ax, cs
	shl eax, 4
	add eax, STRING

	mov di, DESC_DATA
	call near _init_desc_base_address
	ret

_init_desc_base_address:
	mov word [di + 2], ax
	shr eax, 16
	mov byte [di + 4], al
    mov byte [di + 7], ah
	ret
```

最后是打印字符串的 32 位代码段：

```
[SECTION .s32]
[BITS 32]
_main_32:
    mov ax, VideoSelector
    mov gs, ax
    mov esi, 0xA0
 
    mov ax, DataSelector
    mov ds, ax
    mov edi, 0

    mov ecx, STRING_LEN

    print_loop:
        mov al, ds:[edi]
        mov ah, 0xC
        mov word gs:[esi], ax
        add esi, 2
        inc edi
        loop print_loop

    jmp $

CODE32_LEN equ $ - _main_32

[SECTION .s32]
[BITS 32]
STRING: db 'Hello, world'
STRING_LEN equ $ - STRING
```

完整代码：

`pm.inc`

```
DA_DR		EQU	90h	; 存在的只读数据段类型值
DA_DRW		EQU	92h	; 存在的可读写数据段属性值
DA_DRWA		EQU	93h	; 存在的已访问可读写数据段类型值
DA_C		EQU	98h	; 存在的只执行代码段属性值
DA_CR		EQU	9Ah	; 存在的可执行可读代码段属性值
DA_CCO		EQU	9Ch	; 存在的只执行一致代码段属性值
DA_CCOR		EQU	9Eh	; 存在的可执行可读一致代码段属性值

; 宏 ------------------------------------------------------------------------------------------------------
;
; 描述符
; usage: Descriptor Base, Limit, Attr
;        Base:  dd
;        Limit: dd (low 20 bits available)
;        Attr:  dw (lower 4 bits of higher byte are always 0)
%macro Descriptor 3
	dw	%2 & 0FFFFh				; 段界限 1				(2 字节)
	dw	%1 & 0FFFFh				; 段基址 1				(2 字节)
	db	(%1 >> 16) & 0FFh			; 段基址 2				(1 字节)
	dw	((%2 >> 8) & 0F00h) | (%3 & 0F0FFh)	; 属性 1 + 段界限 2 + 属性 2		(2 字节)
	db	(%1 >> 24) & 0FFh			; 段基址 3				(1 字节)
%endmacro
```

`boot.asm`：

```
%include "pm.inc"

org 0x7C00
jmp _main_16

[SECTION .gdt]
DESC_GDT: Descriptor 0, 0, 0
DESC_VIDEO: Descriptor 0xB8000, 0xFFFF, DA_DRW
DESC_DATA: Descriptor 0, STRING_LEN - 1, DA_DR
DESC_CODE: Descriptor 0, CODE32_LEN - 1, DA_C + DA_32

GdtLen equ $ - DESC_GDT
GdtPtr dw GdtLen
       dd 0

DataSelector equ DESC_DATA - DESC_GDT
CodeSelector equ DESC_CODE - DESC_GDT
VideoSelector equ DESC_VIDEO - DESC_GDT

[SECTION .s16]
[BITS 16]
_main_16:
	mov ax, cs
    mov ds, ax
    mov es, ax
    mov ss, ax
    mov sp, 0x7C00

	call near _init_desc

	; 设置段基址
	xor eax, eax
    mov ax, ds
    shl eax, 4
    add eax, DESC_GDT
    mov dword [GdtPtr + 2], eax

	lgdt [GdtPtr]

	cli
	in al, 0x92
    or al, 00000010b
    out 0x92, al

	mov eax, cr0
	or eax, 1
	mov cr0, eax

	jmp dword CodeSelector:0

_init_desc:
	xor eax, eax
	mov ax, cs
	shl eax, 4
	add eax, _main_32

	mov di, DESC_CODE
	call near _init_desc_base_address

	xor eax, eax
	mov ax, cs
	shl eax, 4
	add eax, STRING

	mov di, DESC_DATA
	call near _init_desc_base_address
	ret

_init_desc_base_address:
	mov word [di + 2], ax
	shr eax, 16
	mov byte [di + 4], al
    mov byte [di + 7], ah
	ret

[SECTION .s32]
[BITS 32]
_main_32:
    mov ax, VideoSelector
    mov gs, ax
    mov esi, 0xA0
 
    mov ax, DataSelector
    mov ds, ax
    mov edi, 0

    mov ecx, STRING_LEN

    print_loop:
        mov al, ds:[edi]
        mov ah, 0xC  ;设置成红色
        mov word gs:[esi], ax
        add esi, 2
        inc edi
        loop print_loop

    jmp $

CODE32_LEN equ $ - _main_32

[SECTION .s32]
[BITS 32]
STRING: db 'Hello, world'
STRING_LEN equ $ - STRING

times 290 db 0
dw 0xAA55
```

编译：

```
NASM -f bin -o boot.com boot.asm
```

生成 IMG 软盘文件镜像：

```
dd if=boot.com of=boot.img bs=512 count=1
dd if=/dev/zero of=/tmp/empty.img bs=512 count=2880
dd if=/tmp/empty.img of=boot.img skip=1 seek=1 bs=512 count=2879
```

使用 VMWare 虚拟机，添加软盘设备并启动，运行结果：  

![](1689843862774.png)