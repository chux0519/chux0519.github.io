---
title: "Raspi3 Os 01"
date: 2019-11-16T16:24:14+08:00
showDate: true
draft: false
tags: ["blog", "story"]
---

基于树莓派 3 的操作系统学习，第一部分。

<!--more-->

## 工具链

### qemu

这是个模拟器，我们开发时，可以先用 qemu 模拟树莓派，不用每次真的用真机测试。
这里进行手动编译。

```sh
./configure --target-list=aarch64-softmmu --enable-modules \
  --enable-tcg-interpreter --enable-debug-tcg \
  --python=/usr/bin/python2.7; \
```

### clang & llvm

clang 被设计时，定位就是 cross-compiler

安装 llvm 主要是要使用到 llvm-objcopy

另外还需要 lld

编译：

1. 编译 asm

> clang --target=aarch64-elf -Wall -O2 -ffreestanding -nostdinc -nostdlib -mcpu=cortex-a53+nosimd -c start.S -o start.o

参数说明：

- `--target=aarch64-elf`，指编译的目标平台是 64 位的 arm 架构，elf 是一种可执行文件格式
- `-Wall`，指打开诊断信息
- `-O2`，指优化程度, O2 是中等程度的优化，可实现大多数优化
- `-ffreestanding`，指代编译结果独立运行(bare metal)，没有 host
- `-nostdinc` 和 `-nostdlib` manual 没有讲清楚，个人理解是不包含标准库必须要包含的 flag
- `-mcpu=cortex-a53+nosimd`，树莓派 3b 的芯片是 BCM2837，属于 cortex-a53 的架构，nosimd 指不需要支持 simd。这里个人对 cortex-a53 和 BCM2837 的区别可能存在误解，个人理解是，BCM2837 是具体的一块芯片，而 cortex-a53 指代的是一种 arm 的架构或者说标准，如果我的理解有误，请通过邮件纠正我。
- `-c` 指代生成 `.o` 文件，包含了 assembler 的处理，这里输入是一个 asm，因此需要 -c，主要区别于 `-S`

2. 链接到 elf 格式

> ld.lld -m aarch64elf -nostdlib start.o -o kernel8.elf

object 文件是不能运行的，它里面包含了一些符号信息，需要 linker 把它链接成对应平台的 efl 文件才能狗被执行。

通过 llvm-objdump 可以查看反汇编信息

> llvm-objdump -d start.o

3. 变成二进制镜像

> llvm-objcopy --input-target=aarch64-elf -O binary kernel8.elf kernel8.img

4. 使用 qemu 运行查看效果

> qemu-system-aarch64 -M raspi3 -kernel kernel8.img -d in_asm

至此，我们可以将 start.S 编译、链接成 qemu 可以调试的镜像了。

github repo: 在分支 [step0](https://github.com/chux0519/raspi-os/tree/step0) 查看

