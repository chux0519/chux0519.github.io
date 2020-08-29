+++
title = "CS144 Lab2 学习笔记"
date = 2020-08-29T10:08:15+08:00
+++

CS144 是一门关于计算机网络的斯坦福大学的公开课。课程的实验是动手写一个用户态的 TCP 协议，本文是第三篇。

- 课程链接: https://cs144.github.io/

- 本文讲义: https://cs144.github.io/assignments/lab2.pdf

- 本文相关代码: https://github.com/hexyoungs/tcplab/tree/lab2

<!-- more -->

## 讲义

Lab2 的内容是实现 TCP 的接收端，讲义从接收端的指责说起，引出我们需要实现的两个重要点，然后给出样板代码和测试用例留作实验。

简单摘要一下，TCP 的接收端有两个职责：

1. 告诉发送端，目前已经成功重组了多少数据（acknowledgment）
2. 告诉发送端，什么范围的数据是目前允许接收的（flow control window）

为了实现这两个点，TCP 协议有一系列相关的字段和约定，我们要做的就是实现这些约定。

另外，讲义里面为我们做了三个序列号的辨析，见下表

|区别|TCP 协议中的序列号|绝对序列号|字节流的 index|
| --- | --- | --- | --- |
| 起始值 | 从 ISN 开始 | 从 0 开始 | 从 0 开始|
| SYN 和 FIN | 包含 SYN 和 FIN | 包含 SYN 和 FIN | 不包含|
| 值范围 | 32 位，溢出后变为 0 | 64位 | 64位 |
| 术语 | seqno | absolute seqno | stream index |


## 要点

- 要点一：实现一个 WrappingInt32，负责 <u>TCP 协议中的序列号</u>、<u>绝对序列号</u>以及 Lab1 中需要的<u>字节流的 index</u> 这三者的互转。
- 要点二：实现 TCP 协议接收端的窗口控制逻辑。


## 参考资料

- [rfc793](https://tools.ietf.org/html/rfc793)
