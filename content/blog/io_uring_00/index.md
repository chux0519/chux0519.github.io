+++
title = "io_uring 简介"
date = 2021-10-10T13:30:15+08:00
+++

## 什么是 io_uring

它是 Linux 下的，新的异步 IO 的 API，由 Facebook 工程师 Jens Axboe created 出来。

### 已有解决方案的局限

aio 是 Linux 中已有的异步 IO 系统调用，支持文件和 socket，但是有一些局限

1. 只支持 O_DIRECT 模式的读写，这是最大的局限性，而现实世界中，几乎没有应用有这种常规性质需求
2. 即使在 O_DIRECT 下，aio 也可能因为 metadata 缺失导致 block
3. 某些存储设备有固定的请求队列大小( slots for request，个人理解类似 socket 的 backlog)，队列满时，aio 的提交也会 block
4. 提交和完成过程，会有 104 字节的额外拷贝开销，提交和完成也会产生两个不同的系统调用

<!-- more -->

### 心智模型 (mental)

io_uring 的名字就暗示了，kernel-user 之间主要通过 ring buffer 来进行交流，里面包含了一些系统调用，设计时尽可能地保持精简，并且提供一种可用的 polling 模式，来尽量减少这些系统调用。

1. 存在 2 个 ring buffer，一个为提交队列 (submission queue，后简称 SQ) 服务，一个为完成队列 (completion queue，后简称 CQ) 服务
2. ring buffer 同时被 user 和 kernel space 共享，通过调用 `io_uring_setup()` 然后通过 `mmap(2)` 映射共享
3. 用户告诉 `io_uring` 要做些什么 (比如读写文件，accept 连接等)，被描述的结构叫做 submission queue entry (后面简称 SQE)，它将会被添加到 SQ ring buffer 的尾部
4. 然后用户进行 `io_uring_enter()` 系统调用，让 io_uring 工作
5. `io_uring_enter()` 提供可选功能，可以在返回前，等待其他的用户请求被处理，意味着用户可以一次性读取所有被处理后的结果
6. kernel 处理用户提交的这些请求，然后添加到 CQ 的 ring buffer 的尾部，被处理完成的请求叫做 completion queue events (CQEs)
7. 用户从 CQ 的 ring buffer 中读取 CQEs，每个 CQE 都对应了一个 SQE，并且包含了对应的状态
8. 用户根据需求继续这个 `添加 SQE` - `读取 CQE` 的流程
9. 用户还可以使用 polling 模式，让 kernel 自己去轮询新的 SQE，可以避免多次手动调用 `io_uring_enter()` 的开销

### 性能

性能提升主要来自于 ring buffer，它不会产生 kernel-user space 之间的数据拷贝。而这些数据拷贝往往需要很多系统调用，系统调用的花销比较大，因此减少系统调用，也就削减了花销，也就提升了性能

另一方面，polling 模式可以减少 `io_uring_enter()` 的开销，这是之前 aio 没有的。

性能的 benchmark 可以参考 [io_uring.pdf](https://kernel.dk/io_uring.pdf)，也可以去网上寻找其他真实世界的案例

### 接口

请直接使用 liburing，它做了底层 API 的封装，有丰富的测试，并且也被 QEMU 用上了

不过了解 io_uring 本身的一些系统调用是非常有帮助也是有必要的，这一节将放在下一部分完成。


## 资源

- [Lord of the io_uring](https://unixism.net/loti/index.html)

