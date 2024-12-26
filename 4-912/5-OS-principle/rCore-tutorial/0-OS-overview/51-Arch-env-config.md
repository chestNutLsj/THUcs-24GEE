## 安装 Qemu
```
sudo pacman -S qemu-full
```

## Clone rCore
```
git clone https://github.com/rcore-os/rCore-Tutorial-v3.git
cd rCore-Tutorial-v3/os
make run
```

### 正确配置的输出
```
❯ make run  
(rustup target list | grep "riscv64gc-unknown-none-elf (installed)") || rustup target add riscv64gc-unk  
nown-none-elf  
riscv64gc-unknown-none-elf (installed)  
cargo install cargo-binutils  
   Updating crates.io index  
    Ignored package `cargo-binutils v0.3.6` is already installed, use --force to override  
rustup component add rust-src  
info: component 'rust-src' is up to date  
rustup component add llvm-tools-preview  
info: component 'llvm-tools-preview' for target 'x86_64-unknown-linux-gnu' is up to date  
Platform: qemu  
  Compiling os v0.1.0 (/home/senjl/MyLearning/computer_science/operating_system/thu-rcore-2023/rCore-T  
utorial-v3/os)  
warning: unused import: `sip`  
 --> src/trap/mod.rs:14:10  
  |  
14 |     sie, sip, sscratch, sstatus, stval, stvec,  
  |          ^^^  
  |  
  = note: `#[warn(unused_imports)]` on by default  
  
warning: `os` (bin "os") generated 1 warning  
   Finished release [optimized + debuginfo] target(s) in 2.75s  
make[1]: 进入目录“/home/senjl/MyLearning/computer_science/operating_system/thu-rcore-2023/rCore-Tutoria  
l-v3/user”  
   Finished release [optimized + debuginfo] target(s) in 0.02s  
make[1]: 离开目录“/home/senjl/MyLearning/computer_science/operating_system/thu-rcore-2023/rCore-Tutoria  
l-v3/user”  
   Finished release [optimized] target(s) in 0.01s  
    Running `target/release/easy-fs-fuse -s ../user/src/bin/ -t ../user/target/riscv64gc-unknown-none-  
elf/release/`  
src_path = ../user/src/bin/  
target_path = ../user/target/riscv64gc-unknown-none-elf/release/  
[rustsbi] RustSBI version 0.3.1, adapting to RISC-V SBI v1.0.0  
.______       __    __      _______.___________.  _______..______   __  
|   _  \     |  |  |  |    /       |           | /       ||   _  \ |  |  
|  |_)  |    |  |  |  |   |   (----`---|  |----`|   (----`|  |_)  ||  |  
|      /     |  |  |  |    \   \       |  |      \   \    |   _  < |  |  
|  |\  \----.|  `--'  |.----)   |      |  |  .----)   |   |  |_)  ||  |  
| _| `._____| \______/ |_______/       |__|  |_______/    |______/ |__|  
[rustsbi] Implementation     : RustSBI-QEMU Version 0.2.0-alpha.2  
[rustsbi] Platform Name      : riscv-virtio,qemu  
[rustsbi] Platform SMP       : 1  
[rustsbi] Platform Memory    : 0x80000000..0x88000000  
[rustsbi] Boot HART          : 0  
[rustsbi] Device Tree Region : 0x87e00000..0x87e010de  
[rustsbi] Firmware Address   : 0x80000000  
[rustsbi] Supervisor Address : 0x80200000  
[rustsbi] pmp01: 0x00000000..0x80000000 (-wr)  
[rustsbi] pmp02: 0x80000000..0x80200000 (---)  
[rustsbi] pmp03: 0x80200000..0x88000000 (xwr)  
[rustsbi] pmp04: 0x88000000..0x00000000 (-wr)  
KERN: init gpu  
KERN: init keyboard  
KERN: init mouse  
KERN: init trap  
/**** APPS ****  
adder  
adder_atomic  
adder_mutex_blocking  
adder_mutex_spin  
adder_peterson_spin  
adder_peterson_yield  
adder_simple_spin  
adder_simple_yield  
barrier_condvar  
barrier_fail  
cat  
cmdline_args  
condsync_condvar  
condsync_sem  
count_lines  
eisenberg  
exit  
fantastic_text  
filetest_simple  
forktest  
forktest2  
forktest_simple  
forktree  
gui_rect  
gui_simple  
gui_snake  
gui_uart  
hello_world  
huge_write  
huge_write_mt  
infloop  
initproc  
inputdev_event  
matrix  
mpsc_sem  
peterson  
phil_din_mutex  
pipe_large_test  
pipetest  
priv_csr  
priv_inst  
race_adder_arg  
random_num  
run_pipe_test  
sleep  
sleep_simple  
stack_overflow  
stackful_coroutine  
stackless_coroutine  
store_fault  
sync_sem  
tcp_simplehttp  
threads  
threads_arg  
udp  
until_timeout  
user_shell  
usertests  
yield  
**************/  
Rust user shell  
>>
```

## 其他细节
请查看 [[50-rCore-env-config]] 及 [GitHub - rcore-os/rCore-Tutorial-v3: Let's write an OS which can run on RISC-V in Rust from scratch!](https://github.com/rcore-os/rCore-Tutorial-v3)

