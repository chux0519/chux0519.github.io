---
title: "Leetcode Rust Easy"
date: 2019-12-17T10:37:14+08:00
showDate: true
draft: false
tags: ["blog", "story"]
---

## 算法系列

简单题

这里记录刷简单题过程中感到有意思的题目

## Maximum subarray problem

这类型的题目描述在：[wiki](https://en.wikipedia.org/wiki/Maximum_subarray_problem#Kadane's_algorithm)

算法还有一个名称，叫做 Kadane's algorithm。在刷到 leetcode 的 [121](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) 和 [122](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/) 题目时，卡住了，不像其他类型的题目，看完题目后就会有比较清晰的想法，直接实现即可。这里学习的思想主要是累加，即给出一个数组，要算法求出获利最大，即 A[n] - A[m] 最大(n > m)

核心是等式

> A[n] - A[m] = A[n] - A[n-1] + A[n-1] - A[n-2] + .. + A[m+1] - A[m]

## 异或操作

异或操作符实际上也很有意思，某些题目可能会在题目提示，时间复杂度和空闲复杂度，和能够直接想到的解法通常相差较大。

比如 [#136-single-number](https://leetcode.com/problems/single-number/)

核心就是异或操作的特殊性，一个数在对自己异或的时候，实际会清零，这在汇编里面常用来清空寄存器。

## 链表找环

[141](https://leetcode.com/problems/linked-list-cycle/) 题，找链表是否存在环，使用快慢指针即可

## 链表找交叉点

[160](https://leetcode.com/problems/intersection-of-two-linked-lists/)题，找链表交叉点。
需要解决的问题是，两个链表的长度不一定相同，要想办法使它们能够进行比较。

这里的解法参考：
[讨论区](https://leetcode.com/problems/intersection-of-two-linked-lists/discuss/49785/Java-solution-without-knowing-the-difference-in-len!/165648)
