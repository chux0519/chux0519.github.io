---
title: "Raspi3 Os 02"
date: 2019-11-24T15:52:14+08:00
showDate: true
draft: false
tags: ["blog","story"]
---

这是基于树莓派学习操作系统的第二部分，之前打通了工具链，能够使用 clang 进行交叉编译，使用 qemu 进行运行。

这一篇文章，期望完成

1. C 语言的控制
2. Rust 的控制

使用 C 语言控制主要为了了解整个流程下面的细节，再换到抽象层次更高的 Rust，打通流程，了解原理后，会一直使用 Rust 进行开发了。

## C 语言介入

我们编写普通 C 程序时，其实 C 是有 runtime 的([crt](https://en.wikipedia.org/wiki/Crt0))，它帮我们做了许多事情，准备好 `.bss` 段就是其中一件。

在 bare metal 的世界里面，我们需要手动完成这件事。我们可以通过 linker script，定义好 bss 的内存布局，在 bss 的开始、结束处定义两个标记，然后通过代码进行初始化。

## Rust 

### 工具链

- 安装 nightly 版本的工 rust 编译器

```bash
curl https://sh.rustup.rs -sSf  \
    |                           \
    sh -s --                    \
    --default-toolchain nightly \
    --component rust-src llvm-tools-preview clippy rustfmt rls rust-analysis

cargo install cargo-xbuild cargo-binutils
```

这里使用 nightly，是因为我们用到的一些语法、工具链等特性都还在 nightly 下，包括以下（不完全）

- `aarch64-unknown-none-softfloat` target
- `asm!`，见 [issue#29722](https://github.com/rust-lang/rust/issues/29722)
- `global_asm!`， 见 [issue#35119](https://github.com/rust-lang/rust/issues/35119)

我们继续使用汇编进行 bss 段的初始化，但有一个 crate 叫做 `r0`，我也可以借助这个 crate 进行 bss 的初始化。

## 初始化 bss 汇编

```asm
    // clear bss
    ldr     x1, =__bss_start
    ldr     w2, =__bss_size
3:  cbz     w2, 4f
    str     xzr, [x1], #8
    sub     w2, w2, #1
    cbnz    w2, 3b

4:  bl      main
    // for failsafe, halt this core too
    b       1b
```

注意，`__bss_start` 和 `__bss_size` 都是在 link.ld 中定义的，这里可以直接使用。

### asm 笔记

#### [cbz](http://www.keil.com/support/man/docs/armasm/armasm_dom1361289867296.htm): Compare and Branch on Zero

用法类似：`CBZ Rn, label`，即表示，w2 等于零时，终止循环跳到 label 是 4 的位置。

label 后面带有的后缀含义如下：

- f: foward，即向下搜索
- b: backward，即向前搜索

另外还有 a, t，详见： [Syntax of numeric local labels](http://www.keil.com/support/man/docs/armasm/armasm_dom1359731175956.htm)

#### [str](http://www.keil.com/support/man/docs/armasm/armasm_dom1361289906890.htm)

这里的用法属于 post-indexed，即将 xzr 寄存器中的值存到 x1 为基址，8 为偏移的位置，完成后，x1 的值再加 8

xzr 寄存器是 0，即相当于逐字节进行填充。

另外两个指令比较容易看懂，这里不再赘述。

在 bss 初始化后，实际上通过 `bl main`，跳到了 main 函数中了。

## main

我们需要在 C/Rust 中暴露出这个符号就可以了，即定义 main 函数即可，需要注意的是，Rust 中，需要使用 `#[no_mangle]` 注解，防止函数名被改变。

## Rust 后记：panic handler

我们想要使用 Rust 编写没有 runtime 的程序，那么我们需要自己实现 panic handler，这里我们使用 `asm!`，用汇编实现无限循环，使得 panic 时，挂起 CPU。


## 源码

- C 语言版本：[step1](https://github.com/chux0519/raspi-os/tree/step1) 分支
- Rust 版本： [step1-rs](https://github.com/chux0519/raspi-os/tree/step1-rust) 分支

