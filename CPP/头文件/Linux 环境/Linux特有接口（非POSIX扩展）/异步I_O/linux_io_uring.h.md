## `<linux/io_uring.h>` 头文件详解（Linux 特有接口）

`<linux/io_uring.h>` 是 Linux 内核提供的高性能异步 I/O 框架的核心头文件，定义了 io_uring 接口的数据结构、操作码和标志位。io_uring 允许用户态程序和内核通过共享内存队列进行高效通信，避免了传统 I/O 模型的大量系统调用和数据拷贝开销[reference:0]。该头文件由 Jens Axboe 于 2019 年引入内核，是现代 Linux 高性能异步 I/O 的基础[reference:1]。

---

## 一、系统调用详解

**说明：** `io_uring_setup`、`io_uring_enter` 和 `io_uring_register` 是 io_uring 的三个核心系统调用，它们**并不直接**在 `<linux/io_uring.h>` 中声明，而是由内核直接提供[reference:2]。该头文件主要定义这些系统调用使用的结构体和常量。

### 1. io_uring_setup

**函数原型：**
```c
int io_uring_setup(unsigned int entries, struct io_uring_params *params);
```

**作用：** 创建一个 io_uring 实例，返回一个文件描述符。内核会创建提交队列（SQ）和完成队列（CQ）所需的内存结构，用户态后续通过 `mmap` 将这部分内存映射到自己的地址空间进行访问[reference:3][reference:4]。

**参数：**
- `entries`：提交队列（SQ）中的条目数，必须是 1 到 4096 之间的 2 的幂次方。
- `params`：指向 `struct io_uring_params` 的指针，用于配置 io_uring 的行为和接收内核反馈。

**返回值：** 成功返回 io_uring 文件描述符，失败返回 -1 并设置 `errno`。

**示例用法：**
```c
#include <linux/io_uring.h>
#include <sys/syscall.h>
#include <unistd.h>
#include <stdio.h>

int main() {
    struct io_uring_params params = {0};
    int fd = syscall(SYS_io_uring_setup, 4096, &params);
    if (fd < 0) {
        perror("io_uring_setup");
        return 1;
    }
    close(fd);
    return 0;
}
```

**实现原理：** 系统调用进入内核，分配一个 `io_ring_ctx` 上下文结构体，创建提交队列和完成队列的环形缓冲区，并返回一个匿名文件描述符，该文件描述符关联到这个内核上下文[reference:5]。

**线程安全提示：** 是线程安全的。但返回的文件描述符在多线程间共享时，需要用户态自行同步。

---

### 2. io_uring_enter

**函数原型：**
```c
int io_uring_enter(unsigned int fd, unsigned int to_submit,
                   unsigned int min_complete, unsigned int flags,
                   sigset_t *sig);
```

**作用：** 启动或等待 I/O 操作。一次调用可以同时提交新的 I/O 请求和等待已提交请求的完成[reference:6]。

**参数：**
- `fd`：由 `io_uring_setup` 返回的文件描述符。
- `to_submit`：从提交队列中取出的 SQE 数量。
- `min_complete`：在返回之前等待完成的 CQE 数量。
- `flags`：位掩码，可取 `IORING_ENTER_GETEVENTS`、`IORING_ENTER_SQ_WAKEUP`、`IORING_ENTER_SQ_WAIT`、`IORING_ENTER_EXT_ARG` 等。
- `sig`：指向信号掩码的指针。

**返回值：** 成功返回提交的 I/O 数量，失败返回 -1 并设置 `errno`。

**实现原理：** 内核从提交队列中取出 SQE，分发给对应的文件系统或设备驱动处理，完成后将结果放入完成队列。`min_complete` 用于控制阻塞行为，`flags` 控制与 SQ 内核线程的交互[reference:7]。

**线程安全提示：** 线程安全。但对同一个 `fd` 的并发调用需要用户态同步，内核提供 `IORING_SETUP_SINGLE_ISSUER` 标志优化单提交者场景[reference:8]。

---

### 3. io_uring_register

**函数原型：**
```c
int io_uring_register(unsigned int fd, unsigned int opcode, void *arg, unsigned int nr_args);
```

**作用：** 预注册资源（如用户缓冲区、文件描述符等），减少每次 I/O 操作的开销[reference:9]。

**参数：**
- `fd`：io_uring 文件描述符。
- `opcode`：注册操作类型，如 `IORING_REGISTER_BUFFERS`、`IORING_REGISTER_FILES` 等。
- `arg`：操作参数，取决于 `opcode`。
- `nr_args`：参数数量（如 iovec 数组的长度）。

**返回值：** 成功返回 0，失败返回 -1 并设置 `errno`。

**实现原理：** 内核根据 `opcode` 将用户提供的资源（如文件描述符或内存缓冲区）与 io_uring 实例绑定，建立长期映射，避免后续 I/O 操作中的重复验证和映射开销[reference:10]。

**线程安全提示：** 线程安全。但对同一个 `fd` 的并发调用需要用户态同步，内核提供 `IORING_SETUP_SINGLE_ISSUER` 标志优化单提交者场景[reference:11]。

---

## 二、结构体详解

### 1. struct io_uring_sqe

**定义：**
```c
struct io_uring_sqe {
    __u8 opcode;        // 操作类型，如 IORING_OP_READ
    __u8 flags;         // IOSQE_* 标志位
    __u16 ioprio;       // I/O 优先级
    __s32 fd;           // 目标文件描述符
    __u64 off;          // 文件内偏移量
    __u64 addr;         // 数据缓冲区地址或 iovec 数组地址
    __u32 len;          // 缓冲区大小或 iovec 数量
    union {
        __kernel_rwf_t rw_flags;   // 读写标志
        __u32 fsync_flags;         // fsync 标志
    };
    __u64 user_data;    // 用户自定义数据，完成时原样返回
    union {
        __u16 buf_index;           // 固定缓冲区索引
        __u64 __pad2[3];           // 填充字节
    };
};
```

**作用：** 提交队列条目，描述一个 I/O 请求。用户态填充 SQE 后放入提交队列，内核消费并执行[reference:12][reference:13]。

**成员详解：**
- `opcode`：I/O 操作类型，决定请求的行为（见宏定义中的 `IORING_OP_*`）。
- `flags`：控制请求行为的标志位，如 `IOSQE_FIXED_FILE`（使用固定文件集）、`IOSQE_IO_LINK`（链接请求）等[reference:14]。
- `ioprio`：I/O 优先级，影响请求在磁盘调度队列中的顺序。
- `fd`：目标文件描述符，若设置了 `IOSQE_FIXED_FILE` 则为固定文件索引[reference:15]。
- `off`：文件内的起始偏移量。
- `addr`：数据缓冲区的用户空间地址，或 `readv`/`writev` 时的 `iovec` 数组地址。
- `len`：缓冲区大小或 `iovec` 数组的长度。
- `rw_flags` / `fsync_flags`：联合体，根据 `opcode` 不同含义不同。
- `user_data`：用户自定义数据，完成时原样复制到 CQE，通常用于关联请求和上下文。
- `buf_index`：当使用 `IORING_OP_READ_FIXED` 或 `IORING_OP_WRITE_FIXED` 时，指定已注册缓冲区的索引。

---

### 2. struct io_uring_cqe

**定义：**
```c
struct io_uring_cqe {
    __u64 user_data;    // 来自对应 SQE 的 user_data
    __s32 res;          // 操作结果，成功时为字节数，失败时为 -errno
    __u32 flags;        // 完成标志（当前未使用）
};
```

**作用：** 完成队列条目，内核将 I/O 操作的结果填充到 CQE 中，用户态通过读取完成队列获取操作结果[reference:16][reference:17]。

**成员详解：**
- `user_data`：原样返回 SQE 中设置的 `user_data`。
- `res`：操作结果。成功时，对于读操作返回读取的字节数，对于写操作返回写入的字节数；失败时返回负的错误码（如 `-ENOMEM`）[reference:18]。
- `flags`：当前未使用，保留供未来扩展。

---

### 3. struct io_uring_params

**定义：**
```c
struct io_uring_params {
    __u32 sq_entries;           // SQ 中的条目数（内核输出）
    __u32 cq_entries;           // CQ 中的条目数（内核输出）
    __u32 flags;                // 配置标志（用户输入）
    __u32 sq_thread_cpu;        // SQ 内核线程绑定的 CPU
    __u32 sq_thread_idle;       // SQ 内核线程空闲超时（毫秒）
    __u32 resv[5];              // 保留字段
    struct io_sqring_offsets sq_off;   // SQ ring 偏移量
    struct io_cqring_offsets cq_off;   // CQ ring 偏移量
};
```

**作用：** 传递给 `io_uring_setup` 的参数结构体，用于配置 io_uring 实例的行为，并接收内核返回的队列信息[reference:19]。

**成员详解：**
- `sq_entries` / `cq_entries`：内核实际分配的 SQ 和 CQ 条目数，通常等于或大于用户请求的 `entries`。
- `flags`：配置标志（见宏定义中的 `IORING_SETUP_*`）。
- `sq_thread_cpu` / `sq_thread_idle`：当使用 `IORING_SETUP_SQPOLL` 时，指定 SQ 内核线程的 CPU 亲和性和空闲超时。
- `sq_off` / `cq_off`：SQ ring 和 CQ ring 中各个字段相对于 mmap 起始地址的偏移量，用户态通过这些偏移量访问共享内存。

---

### 4. struct io_sqring_offsets / struct io_cqring_offsets

**定义：**
```c
struct io_sqring_offsets {
    __u32 head;         // head 字段偏移
    __u32 tail;         // tail 字段偏移
    __u32 ring_mask;    // ring_mask 字段偏移
    __u32 ring_entries; // ring_entries 字段偏移
    __u32 flags;        // flags 字段偏移
    __u32 dropped;      // dropped 字段偏移
    __u32 array;        // array 字段偏移
    __u32 resv1;
    __u64 resv2;
};

struct io_cqring_offsets {
    __u32 head;
    __u32 tail;
    __u32 ring_mask;
    __u32 ring_entries;
    __u32 overflow;     // 溢出计数偏移
    __u32 cqes;         // CQE 数组偏移
    __u32 resv[2];
};
```

**作用：** 定义 SQ ring 和 CQ ring 共享内存区域中各字段的相对偏移量，用户态通过这些偏移量访问共享内存[reference:20]。

**成员详解：**
- `head` / `tail`：队列的头指针和尾指针，用户态和内核通过这两个指针无锁地协调队列访问。
- `ring_mask`：用于索引计算的掩码，等于 `ring_entries - 1`。
- `ring_entries`：队列的容量。
- `array`（仅 SQ）：SQ 中 SQE 索引数组的偏移，用于支持非连续的 SQE 提交。
- `cqes`（仅 CQ）：CQE 数组的偏移。

---

## 三、宏定义详解

### 1. 操作码（opcode）

| 宏名称 | 值 | 作用 |
|--------|-----|------|
| `IORING_OP_NOP` | 0 | 空操作，用于测试或链接链的占位符 |
| `IORING_OP_READV` | 1 | 向量化读操作（`readv`） |
| `IORING_OP_WRITEV` | 2 | 向量化写操作（`writev`） |
| `IORING_OP_FSYNC` | 3 | 文件同步（`fsync`） |
| `IORING_OP_READ_FIXED` | 4 | 从固定缓冲区读取（需预先注册缓冲区） |
| `IORING_OP_WRITE_FIXED` | 5 | 向固定缓冲区写入 |
| `IORING_OP_POLL_ADD` | 6 | 添加 poll 事件 |
| `IORING_OP_POLL_REMOVE` | 7 | 移除 poll 事件 |
| `IORING_OP_SYNC_FILE_RANGE` | 8 | 同步文件范围（`sync_file_range`） |
| `IORING_OP_SENDMSG` | 9 | 发送消息（`sendmsg`） |
| `IORING_OP_RECVMSG` | 10 | 接收消息（`recvmsg`） |
| `IORING_OP_TIMEOUT` | 11 | 添加超时事件 |
| `IORING_OP_TIMEOUT_REMOVE` | 12 | 移除超时事件 |
| `IORING_OP_ACCEPT` | 13 | 接受连接（`accept`） |
| `IORING_OP_ASYNC_CANCEL` | 14 | 异步取消请求 |
| `IORING_OP_LINK_TIMEOUT` | 15 | 链接链超时 |
| `IORING_OP_CONNECT` | 16 | 连接套接字（`connect`） |
| `IORING_OP_FALLOCATE` | 17 | 文件空间预分配（`fallocate`） |
| `IORING_OP_OPENAT` | 18 | 打开文件（`openat`） |
| `IORING_OP_CLOSE` | 19 | 关闭文件描述符（`close`） |
| `IORING_OP_FILES_UPDATE` | 20 | 更新固定文件集 |
| `IORING_OP_STATX` | 21 | 获取文件状态（`statx`） |
| `IORING_OP_READ` | 22 | 读操作（`read`） |
| `IORING_OP_WRITE` | 23 | 写操作（`write`） |
| `IORING_OP_FADVISE` | 24 | 文件预读建议（`fadvise`） |
| `IORING_OP_MADVISE` | 25 | 内存建议（`madvise`） |
| `IORING_OP_SEND` | 26 | 发送数据（`send`） |
| `IORING_OP_RECV` | 27 | 接收数据（`recv`） |
| `IORING_OP_OPENAT2` | 28 | 增强版 `openat` |
| `IORING_OP_EPOLL_CTL` | 29 | 控制 epoll 文件描述符 |
| `IORING_OP_SPLICE` | 30 | 数据拼接（`splice`） |
| `IORING_OP_TEE` | 31 | 数据复制（`tee`） |
| `IORING_OP_SHUTDOWN` | 32 | 关闭套接字连接（`shutdown`） |
| `IORING_OP_RENAMEAT` | 33 | 重命名文件（`renameat`） |
| `IORING_OP_UNLINKAT` | 34 | 删除文件（`unlinkat`） |
| `IORING_OP_MKDIRAT` | 35 | 创建目录（`mkdirat`） |
| `IORING_OP_SYMLINKAT` | 36 | 创建符号链接（`symlinkat`） |
| `IORING_OP_LINKAT` | 37 | 创建硬链接（`linkat`） |
| `IORING_OP_MSG_RING` | 38 | 跨 io_uring 实例消息传递 |
| `IORING_OP_FSETXATTR` | 39 | 设置扩展属性（`fsetxattr`） |
| `IORING_OP_SETXATTR` | 40 | 设置扩展属性（`setxattr`） |
| `IORING_OP_FGETXATTR` | 41 | 获取扩展属性（`fgetxattr`） |
| `IORING_OP_GETXATTR` | 42 | 获取扩展属性（`getxattr`） |
| `IORING_OP_SOCKET` | 43 | 创建套接字（`socket`） |
| `IORING_OP_URING_CMD` | 44 | 特定于 io_uring 的命令 |
| `IORING_OP_SEND_ZC` | 45 | 零拷贝发送 |
| `IORING_OP_SENDMSG_ZC` | 46 | 零拷贝 `sendmsg` |

**常用操作码说明：**
- `IORING_OP_READ` / `IORING_OP_WRITE`：读写操作，使用 `addr` 指向缓冲区，`len` 为长度，`off` 为偏移量。
- `IORING_OP_READV` / `IORING_OP_WRITEV`：向量化读写，`addr` 指向 `iovec` 数组，`len` 为数组长度。
- `IORING_OP_READ_FIXED` / `IORING_OP_WRITE_FIXED`：使用预注册固定缓冲区，`buf_index` 指定缓冲区索引。
- `IORING_OP_SEND` / `IORING_OP_RECV`：套接字发送/接收，`addr` 指向缓冲区，`len` 为长度，`rw_flags` 为 `MSG_*` 标志。
- `IORING_OP_ACCEPT`：接受连接，`fd` 为监听套接字，`addr` 指向 `sockaddr` 缓冲区，`len` 为地址长度。
- `IORING_OP_ASYNC_CANCEL`：取消之前提交的请求，`addr` 指向要取消的请求的 `user_data`。
- `IORING_OP_LINK_TIMEOUT`：为链接链设置超时，链接中的请求必须在超时前完成。

---

### 2. SQE 标志位（IOSQE_*）

| 宏名称 | 作用 |
|--------|------|
| `IOSQE_FIXED_FILE` | `fd` 字段是固定文件索引，而非真实文件描述符[reference:21] |
| `IOSQE_IO_DRAIN` | 请求前的所有请求完成后再执行[reference:22] |
| `IOSQE_IO_LINK` | 链接当前请求与下一个请求，前一个完成后才启动下一个[reference:23] |
| `IOSQE_IO_HARDLINK` | 类似 `IOSQE_IO_LINK`，但链不会因错误而中断[reference:24] |
| `IOSQE_ASYNC` | 强制异步执行，绕过非阻塞尝试[reference:25] |
| `IOSQE_BUFFER_SELECT` | 为 `IORING_OP_RECV` 或 `IORING_OP_READ` 选择提供的缓冲区 |
| `IOSQE_CQE_SKIP_SUCCESS` | 跳过成功的 CQE（内核 6.0+） |

---

### 3. io_uring_setup 标志位（IORING_SETUP_*）

| 宏名称 | 作用 |
|--------|------|
| `IORING_SETUP_IOPOLL` | 使用轮询完成 I/O，而非中断驱动[reference:26] |
| `IORING_SETUP_SQPOLL` | 创建内核线程轮询提交队列，应用程序无需调用 `io_uring_enter` |
| `IORING_SETUP_SQ_AFF` | 将 SQ 内核线程绑定到 `sq_thread_cpu` 指定的 CPU |
| `IORING_SETUP_CQSIZE` | 指定完成队列大小，`cq_entries` 参数有效 |
| `IORING_SETUP_CLAMP` | 将 `entries` 限制在支持的范围内 |
| `IORING_SETUP_ATTACH_WQ` | 附加到现有 io_uring 的工作队列 |
| `IORING_SETUP_R_DISABLED` | 创建时禁用，需通过 `io_uring_register` 启用 |
| `IORING_SETUP_SUBMIT_ALL` | 提交所有可用的 SQE，直到出错或队列空 |
| `IORING_SETUP_COOP_TASKRUN` | 协同任务运行（内核 5.20+） |
| `IORING_SETUP_TASKRUN_FLAG` | 任务运行标志（内核 5.20+） |
| `IORING_SETUP_SQE128` | 每个 SQE 占 128 字节（而非默认 64 字节） |
| `IORING_SETUP_CQE32` | 每个 CQE 占 32 字节（而非默认 16 字节） |
| `IORING_SETUP_SINGLE_ISSUER` | 声明只有一个线程将提交请求，内核可优化锁定[reference:27] |
| `IORING_SETUP_DEFER_TASKRUN` | 延迟任务运行（内核 6.0+） |

---

### 4. io_uring_enter 标志位

| 宏名称 | 作用 |
|--------|------|
| `IORING_ENTER_GETEVENTS` | 等待 `min_complete` 个事件完成后再返回[reference:28] |
| `IORING_ENTER_SQ_WAKEUP` | 唤醒 SQ 内核线程（当使用 `IORING_SETUP_SQPOLL` 时）[reference:29] |
| `IORING_ENTER_SQ_WAIT` | 等待 SQ 中有可用条目（当使用 `IORING_SETUP_SQPOLL` 时）[reference:30] |
| `IORING_ENTER_EXT_ARG` | 使用 `io_uring_getevents_arg` 扩展参数[reference:31] |

---

### 5. io_uring_register 操作类型（IORING_REGISTER_*）

| 宏名称 | 作用 |
|--------|------|
| `IORING_REGISTER_BUFFERS` | 注册用户缓冲区（固定内存）[reference:32] |
| `IORING_UNREGISTER_BUFFERS` | 注销已注册的缓冲区 |
| `IORING_REGISTER_FILES` | 注册文件描述符集（固定文件） |
| `IORING_UNREGISTER_FILES` | 注销已注册的文件描述符集 |
| `IORING_REGISTER_EVENTFD` | 注册 eventfd 用于事件通知 |
| `IORING_UNREGISTER_EVENTFD` | 注销 eventfd |
| `IORING_REGISTER_FILES_UPDATE` | 更新已注册的文件描述符集 |
| `IORING_REGISTER_EVENTFD_ASYNC` | 注册异步 eventfd |
| `IORING_REGISTER_PROBE` | 探测内核支持的操作码 |
| `IORING_REGISTER_PERSONALITY` | 注册个性（凭据集） |
| `IORING_UNREGISTER_PERSONALITY` | 注销个性 |
| `IORING_REGISTER_RESTRICTIONS` | 设置限制（允许的操作码） |
| `IORING_REGISTER_ENABLE_RINGS` | 启用之前禁用（`IORING_SETUP_R_DISABLED`）的 ring |
| `IORING_REGISTER_FILES2` | 注册文件描述符集（增强版） |
| `IORING_REGISTER_FILES_UPDATE2` | 更新文件集（增强版） |
| `IORING_REGISTER_BUFFERS2` | 注册缓冲区（增强版） |
| `IORING_REGISTER_BUFFERS_UPDATE` | 更新缓冲区 |
| `IORING_REGISTER_IOWQ_AFF` | 设置 I/O 工作队列的 CPU 亲和性 |
| `IORING_UNREGISTER_IOWQ_AFF` | 取消 I/O 工作队列的 CPU 亲和性设置 |
| `IORING_REGISTER_IOWQ_MAX_WORKERS` | 设置 I/O 工作队列的最大工作线程数 |
| `IORING_REGISTER_RING_FDS` | 注册 ring 文件描述符（跨进程共享） |
| `IORING_UNREGISTER_RING_FDS` | 注销 ring 文件描述符 |
| `IORING_REGISTER_PBUF_RING` | 提供缓冲环（提供环形缓冲区） |
| `IORING_UNREGISTER_PBUF_RING` | 注销缓冲环 |
| `IORING_REGISTER_SYNC_CANCEL` | 同步取消请求 |
| `IORING_REGISTER_FILE_ALLOC_RANGE` | 分配固定文件范围 |
| `IORING_REGISTER_USE_REGISTERED_RING` | 指示 fd 是已注册 ring 的索引[reference:33] |

---

### 6. 其他常量

| 宏名称 | 值 | 作用 |
|--------|-----|------|
| `IORING_OFF_SQ_RING` | 0ULL | SQ ring 的 mmap 偏移[reference:34] |
| `IORING_OFF_CQ_RING` | 0x8000000ULL | CQ ring 的 mmap 偏移[reference:35] |
| `IORING_OFF_SQES` | 0x10000000ULL | SQE 数组的 mmap 偏移[reference:36] |
| `IORING_FSYNC_DATASYNC` | (1U << 0) | `fsync` 时仅同步数据（`fdatasync` 语义）[reference:37] |

---

## 四、类型定义

| 类型 | 说明 |
|------|------|
| `struct io_uring_sqe` | 提交队列条目 |
| `struct io_uring_cqe` | 完成队列条目 |
| `struct io_uring_params` | io_uring 实例参数 |
| `struct io_sqring_offsets` | SQ ring 偏移量 |
| `struct io_cqring_offsets` | CQ ring 偏移量 |
| `struct io_uring_probe` | 探测操作码支持 |
| `struct io_uring_restriction` | 操作码限制 |
| `struct io_uring_buf` | 固定缓冲区描述 |
| `struct io_uring_buf_ring` | 缓冲区环 |
| `struct io_uring_getevents_arg` | `io_uring_enter` 扩展参数 |

---

## 五、模板声明

`<linux/io_uring.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。该头文件是 Linux 内核 UAPI（用户态 API）的一部分，用户空间程序可以包含此头文件来使用 io_uring 相关结构体和常量[reference:38]。

---
