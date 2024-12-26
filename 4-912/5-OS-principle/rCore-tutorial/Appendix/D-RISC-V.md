## 如何生成汇编代码
```
# 通常办法，生成的汇编代码有比较冗余的信息
# 生成缺省debug模式的汇编
$cargo rustc -- --emit asm
$ls target/debug/deps/<crate_name>-<hash>.s
# 生成release模式的汇编
$cargo rustc --release -- --emit asm
$ls target/release/deps/<crate_name>-<hash>.s
# 在rcore-tutorial-v3中的应用的汇编代码生成举例
$cd user
$cargo  rustc --release --bin hello_world -- --emit asm
$find ./target -name "hello_world*.s"

#生成更加干净的汇编代码
#基于 cargo-show-asm(https://github.com/pacak/cargo-show-asm)的办法
如果没用安装这个cargo asm子命令，就安装它
$cargo install cargo-show-asm
#在rcore-tutorial-v3中的应用的汇编代码生成举例
$cd user
$cargo  asm  --release --bin hello_world

Compiling user_lib v0.1.0 (/home/chyyuu/thecodes/rCore-Tutorial-v3/user)
 Finished release [optimized + debuginfo] target(s) in 0.10s

     .section .text.main,"ax",@progbits
             .globl  main
             .p2align        1
             .type   main,@function
     main:

             .cfi_sections .debug_frame
             .cfi_startproc
             addi sp, sp, -64
             .cfi_def_cfa_offset 64

             sd ra, 56(sp)
             sd s0, 48(sp)
             .cfi_offset ra, -8
             .cfi_offset s0, -16
             addi s0, sp, 64
             .cfi_def_cfa s0, 0

             auipc a0, %pcrel_hi(.L__unnamed_1)
             addi a0, a0, %pcrel_lo(.LBB0_1)

             sd a0, -64(s0)
             li a0, 1

             sd a0, -56(s0)
             sd zero, -48(s0)

             auipc a0, %pcrel_hi(.L__unnamed_2)
             addi a0, a0, %pcrel_lo(.LBB0_2)

             sd a0, -32(s0)
             sd zero, -24(s0)

             addi a0, s0, -64

             call user_lib::console::print
             li a0, 0
             ld ra, 56(sp)
             ld s0, 48(sp)
             addi sp, sp, 64
             ret
```

## 参考信息

- [RISC-V Assembly Programmer’s Manual](https://github.com/riscv/riscv-asm-manual/blob/master/riscv-asm.md)

- [RISC-V Low-level Test Suits](https://github.com/riscv/riscv-tests)

- [CoreMark®-PRO comprehensive, advanced processor benchmark](https://github.com/RISCVERS/coremark-pro)

- [riscv-tests的使用](https://stackoverflow.com/questions/39321554/how-do-i-use-the-riscv-tests-suite)

- [RISC-V instructions](http://www.robalni.org/riscv/instructions.html)

## 硬件信息

- [Registers & ABI](https://five-embeddev.com/quickref/regs_abi.html)
    
- [Interrupt](https://five-embeddev.com/quickref/interrupts.html)
    
- [ISA & Extensions](https://five-embeddev.com/quickref/isa_ext.html)
    
- [Toolchain](https://five-embeddev.com/quickref/tools.html)
    
- [Control and Status Registers (CSRs)](https://five-embeddev.com/quickref/csrs.html)
    
- [Accessing CSRs](https://five-embeddev.com/quickref/csrs-access.html)
    
- [Assembler & Instructions](https://five-embeddev.com/quickref/instructions.html)
    

## 指令集规范

- [User-Level ISA, Version 1.12](https://five-embeddev.com/riscv-isa-manual/latest/riscv-spec.html)
    
- [4 Supervisor-Level ISA, Version 1.12](https://five-embeddev.com/riscv-isa-manual/latest/supervisor.html)
    
- [Vector Extension](https://five-embeddev.com/riscv-v-spec/draft/v-spec.html)
    
- [RISC-V Bitmanip Extension](https://five-embeddev.com/riscv-bitmanip/draft/bitmanip.html)
    
- [External Debug](https://five-embeddev.com/riscv-debug-spec/latest/riscv-debug-spec.html)
    
- [ISA Resources](https://five-embeddev.com/riscv-isa-manual/)