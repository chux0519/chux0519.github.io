+++
title = "io_uring 接口"
date = 2021-10-10T14:30:15+08:00
+++

接[上回](/blog/io-uring-00/)

目前来讲，用户应该使用 liburing，不太需要去直接调用底层的 io_uring API，但是去了解 io_uring 本身提供哪些接口是非常有必要的。

接下来会通过一个类似 cat 的程序，来讲解 io_uring 的一些操作。

<!-- more -->

## readv(2) 系统调用

Linux 提供了许多读写操作的系统调用，比如 [read(2)](https://man7.org/linux/man-pages/man2/read.2.html), [write(2)](https://man7.org/linux/man-pages/man2/write.2.html)

io_uring 提供的是 [readv(2)](https://man7.org/linux/man-pages/man2/readv.2.html), [writev(2)](https://man7.org/linux/man-pages/man2/writev.2.html)

以读操作为例，readv 被认为优于 read，因为

1. readv 可以一次读取出多个成员，要用 read 的话，往往需要拷贝数据，然后多次调用 read 完成。
2. readv 操作是原子的，而实现同样功能的多次 read 则不是

## 不用 io_uring

首先作为对比版本，先实现一个没有使用 io_uring 的版本，核心代码如下


```C
int read_and_print_file(char *file_name) {
    struct iovec *iovecs;
    int file_fd = open(file_name, O_RDONLY);
    if (file_fd < 0) {
        perror("open");
        return 1;
    }
    off_t file_sz = get_file_size(file_fd);
    off_t bytes_remaining = file_sz;
    int blocks = (int) file_sz / BLOCK_SZ;
    if (file_sz % BLOCK_SZ) blocks++;
    iovecs = malloc(sizeof(struct iovec) * blocks);
    int current_block = 0;
    /*
     * For the file we're reading, allocate enough blocks to be able to hold
     * the file data. Each block is described in an iovec structure, which is
     * passed to readv as part of the array of iovecs.
     * */
    while (bytes_remaining) {
        off_t bytes_to_read = bytes_remaining;
        if (bytes_to_read > BLOCK_SZ)
            bytes_to_read = BLOCK_SZ;
        void *buf;
        if( posix_memalign(&buf, BLOCK_SZ, BLOCK_SZ)) {
            perror("posix_memalign");
            return 1;
        }
        iovecs[current_block].iov_base = buf;
        iovecs[current_block].iov_len = bytes_to_read;
        current_block++;
        bytes_remaining -= bytes_to_read;
    }
    /*
     * The readv() call will block until all iovec buffers are filled with
     * file data. Once it returns, we should be able to access the file data
     * from the iovecs and print them on the console.
     * */
    int ret = readv(file_fd, iovecs, blocks);
    if (ret < 0) {
        perror("readv");
        return 1;
    }
    for (int i = 0; i < blocks; i++)
        output_to_console(iovecs[i].iov_base, iovecs[i].iov_len);
    return 0;
}
```

核心思路大致是计算所需要的 iovecs 的大小，然后一次性初始化所需的空间，然后进行一次 readv 调用，最后把 iovecs 中的数据输出出来

## io_uring 版本

核心思路是，提交多个 SQE，告诉 io_uring 我们想要调用 readv 读文件，然后 kernel 做完之后，我们去读取 CQEs

### CQE(completion queue entry)

我们只了解了一些 mental 的概念，现在看实际 CQE 的结构

```C
struct io_uring_cqe {
  __u64  user_data;  /* sqe->user_data submission passed back */
  __s32  res;    /* result code for this event */
  __u32  flags;
};
```

user_data 是在 SQE 传入，CQE 传出的字段。假设这样的场景，我们提交了一堆请求到 SQ，完成时，CQ 里面的顺序是不保证和请求顺序一致的，比如一些操作先完成，另一些后完成，这时调用者需要一些标识之类的来识别 CQE 对应的具体是哪个 SQE。

res 字段则是我们在 SQE 中告诉 kernel 我们想要的系统调用的返回值。

PS: 上面提到 CQE 的顺序不保证和 SQE 一致，实际上是可以设置的，见 [io_uring.pdf](https://kernel.dk/io_uring.pdf)

### SQE(submission queue entry)

它的定义更加复杂一些，因为用户是通过它告诉 io_uring 我们想要做什么

```C
struct io_uring_sqe {
  __u8  opcode;    /* type of operation for this sqe */
  __u8  flags;    /* IOSQE_ flags */
  __u16  ioprio;    /* ioprio for the request */
  __s32  fd;    /* file descriptor to do IO on */
  __u64  off;    /* offset into file */
  __u64  addr;    /* pointer to buffer or iovecs */
  __u32  len;    /* buffer size or number of iovecs */
  union {
    __kernel_rwf_t  rw_flags;
    __u32    fsync_flags;
    __u16    poll_events;
    __u32    sync_range_flags;
    __u32    msg_flags;
  };
  __u64  user_data;  /* data to be passed back at completion time */
  union {
    __u16  buf_index;  /* index into fixed buffers, if used */
    __u64  __pad2[3];
  };
};
```

这里以 cat 为例，我们想要对文件进行 readv 系统调用，那么

1. opcode 设置为 `IORING_OP_READV` 代表 readv
2. fd 设置为我们想操作的文件描述符
3. addr 我们需要传入的 iovec
4. len 是 iovecs 的长度

### 初始化

`io_uring_setup` 需要 2 个参数

- entry 大小，这里的例子使用 1 即可
- param `io_uring_params`，见下面

```C
struct io_uring_params {
  __u32 sq_entries;
  __u32 cq_entries;
  __u32 flags;
  __u32 sq_thread_cpu;
  __u32 sq_thread_idle;
  __u32 resv[5];
  struct io_sqring_offsets sq_off;
  struct io_cqring_offsets cq_off;
};
```

cat 这个例子暂时用不上，因此全部初始化为 0 即可

初始化后会返回一个文件描述符，后续的一些 mmap 操作需要用到

### mmap

初始化得到文件描述符后，我们需要 mmap 操作，使 kernel-user 共享 CQ 和 SQ 两个 ring buffer，关键代码如下

```C
  int sring_sz = p.sq_off.array + p.sq_entries * sizeof(unsigned);
  int cring_sz = p.cq_off.cqes + p.cq_entries * sizeof(struct io_uring_cqe);

  if (p.features & IORING_FEAT_SINGLE_MMAP) {
    if (cring_sz > sring_sz) {
      sring_sz = cring_sz;
    }
    cring_sz = sring_sz;
  }

  sq_ptr = mmap(0, sring_sz, PROT_READ | PROT_WRITE, MAP_SHARED | MAP_POPULATE,
                s->ring_fd, IORING_OFF_SQ_RING);
  if (sq_ptr == MAP_FAILED) {
    perror("mmap");
    return 1;
  }

  if (p.features & IORING_FEAT_SINGLE_MMAP) {
    cq_ptr = sq_ptr;
  } else {
    cq_ptr = mmap(0, cring_sz, PROT_READ | PROT_WRITE,
                  MAP_SHARED | MAP_POPULATE, s->ring_fd, IORING_OFF_CQ_RING);
    if (cq_ptr == MAP_FAILED) {
      perror("mmap");
      return 1;
    }
  }

  /* map in the SQE array */
  s->sqes = mmap(0, p.sq_entries * sizeof(struct io_uring_sqe),
                 PROT_READ | PROT_WRITE, MAP_SHARED | MAP_POPULATE, s->ring_fd,
                 IORING_OFF_SQES);
  if (s->sqes == MAP_FAILED) {
    perror("mmap");
    return 1;
  }
```

Q: 为什么有三次 mmap，CQ, SQ 两个就够了

A: 简单讲，SQ ring buffer 里面是间接地管理 SQE array，SQ ring buffer 存储的是 index，因此还需要 mmap 下面的 array。
而 CQ 的部分，内核是直接映射的 CQE。因此用户还需要为 SQE 的 array 进行一次映射(While the completion queue ring directly indexes the shared array of CQEs, the submission ring has an indirection array in between. The submission side ring buffer is an index into this array, which in turn contains the index into the SQEs.)

### 使用 ring buffer

因为 SQ 和 CQ 的 ring buffer 是内核以及用户共享的内存，在多核的 CPU 上，读写操作的顺序就需要额外注意，我们可能需要一些内存屏障保证读写的正确性。

比如内核更新 CQ 后，用户要保证看到最新的修改。反之，用户更新 SQ，消费 CQ 后，要保证内核看到最新的修改。

### 消费 CQ

对于 CQE 而言，它由 kernel 进行添加，并且更新到 CQ 的 tail，而我们是在用户态读取，这里就需要讨论 memory ordering。

要谨记每一行代码都可能发生 context switch，因此在比较 head 和 tail 前，要用 `read_barrier()`，确保我们能读到内核的更新。

而消费完 CQE 后，我们更新了 CQ 的 head，要确保内核清楚我们的更新操作，调用了一次 `write_barrier()`

关键代码如下

```C
  head = *cring->head;

  do {
    read_barrier(); /* ensure previous writes are visible */

    if (head == *cring->tail)
      break;

    cqe = &cring->cqes[head & *s->cq_ring.ring_mask];
    fi = (struct file_info *)cqe->user_data;
    if (cqe->res < 0)
      fprintf(stderr, "Error: %s\n", strerror(abs(cqe->res)));

    int blocks = (int)fi->file_sz / BLOCK_SZ;
    if (fi->file_sz % BLOCK_SZ)
      blocks++;

    for (int i = 0; i < blocks; i++)
      output_to_console(fi->iovecs[i].iov_base, fi->iovecs[i].iov_len);

    head++;
  } while (1);

  *cring->head = head;
  write_barrier();
```

###  提交 SQE

流程大致是，读取当前 tail，获取下一个 tail 的位置，然后在获取 index 前，使用 `read_barrier()` 保证 tail 和 next_tail 的值是正确写入的。

然后对 sqe 进行了初始化，告诉 kernel 我们需要的操作。

接着更新当前的 tail，使用 `write_barrier()` 确保 kernel 读到变更。

最后系统调用 `io_uring_enter()` 把一切交给 kernel

关键代码

```C
  next_tail = tail = *sring->tail;
  next_tail++;
  read_barrier();
  index = tail & *s->sq_ring.ring_mask;
  struct io_uring_sqe *sqe = &s->sqes[index];
  sqe->fd = file_fd;
  sqe->flags = 0;
  sqe->opcode = IORING_OP_READV;
  sqe->addr = (unsigned long)fi->iovecs;
  sqe->len = blocks;
  sqe->off = 0;
  sqe->user_data = (unsigned long long)fi;
  sring->array[index] = index;
  tail = next_tail;

  /* update the tail so the kernel can see it. */
  if (*sring->tail != tail) {
    *sring->tail = tail;
    write_barrier();
  }

  int ret = io_uring_enter(s->ring_fd, 1, 1, IORING_ENTER_GETEVENTS);
  if (ret < 0) {
    perror("io_uring_enter");
    return 1;
  }
```

## 资源

- [loti/low_level.html](https://unixism.net/loti/low_level.html)
- [io-uring-by-example-part-1-introduction](https://unixism.net/2020/04/io-uring-by-example-part-1-introduction/)
- [代码](https://github.com/chux0519/io_uring_example)

