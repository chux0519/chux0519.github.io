+++
title = "liburing - cat"
date = 2021-10-11T11:15:15+08:00
+++

结合[上回](/blog/io-uring-01/)，大部分代码是样板代码，liburing 是为了简化我们手工调用 io_uring 实现的一个包装库

接下来用 liburing 实现 cat

<!-- more -->

## CQE

先看代码

```C
int get_completion_and_print(struct io_uring *ring) {
    struct io_uring_cqe *cqe;
    int ret = io_uring_wait_cqe(ring, &cqe);
    if (ret < 0) {
        perror("io_uring_wait_cqe");
        return 1;
    }
    if (cqe->res < 0) {
        fprintf(stderr, "Async readv failed.\n");
        return 1;
    }
    struct file_info *fi = io_uring_cqe_get_data(cqe);
    int blocks = (int) fi->file_sz / BLOCK_SZ;
    if (fi->file_sz % BLOCK_SZ) blocks++;
    for (int i = 0; i < blocks; i ++)
        output_to_console(fi->iovecs[i].iov_base, fi->iovecs[i].iov_len);

    io_uring_cqe_seen(ring, cqe);
    return 0;
}
```

这里我们不用再自己操心各种内存屏障的问题，对 cqe 的操作通过 liburing 完成。

## 提交 SQE

同样地，提交部分，也全部通过 liburing 操作 SQE

```C
    /* Get an SQE */
    struct io_uring_sqe *sqe = io_uring_get_sqe(ring);
    /* Setup a readv operation */
    io_uring_prep_readv(sqe, file_fd, fi->iovecs, blocks, 0);
    /* Set user data */
    io_uring_sqe_set_data(sqe, fi);
    /* Finally, submit the request */
    io_uring_submit(ring);
```

## 初始化和清理

初始化 liburing 使用

> io_uring_queue_init(QUEUE_DEPTH, &ring, 0);

程序退出前，清理

> io_uring_queue_exit(&ring);

## 总结

总体代码量减少很多，并且不用手工 mmap 和考虑内存屏障问题。 

## 参考

- [cat_liburing](https://unixism.net/loti/tutorial/cat_liburing.html)
- [代码](https://github.com/chux0519/io_uring_example/)

