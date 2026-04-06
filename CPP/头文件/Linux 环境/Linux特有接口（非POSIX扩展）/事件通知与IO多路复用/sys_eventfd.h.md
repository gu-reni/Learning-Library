## `<sys/eventfd.h>` 头文件详解（Linux 特有接口）

`<sys/eventfd.h>` 是 Linux 系统提供的头文件，用于创建和管理 **eventfd** 对象。eventfd 是 Linux 内核提供的一种轻量级进程/线程间事件通知机制[reference:0]，通过一个内核维护的 64 位计数器来实现等待/通知功能，常与 `epoll`、`select`、`poll` 等 I/O 多路复用机制结合使用[reference:1]。该机制由 Linux 2.6.22 版本引入内核[reference:2]。

---

## 一、函数详解

### 1. eventfd

**函数原型：**
```c
int eventfd(unsigned int initval, int flags);
```

**作用：** 创建一个 eventfd 对象，返回一个指向该对象的文件描述符。内核为每个 eventfd 对象维护一个 64 位无符号整数计数器，该计数器初始值由 `initval` 指定[reference:3]。

**参数：**
- `initval`：计数器的初始值，通常设为 0[reference:4]。
- `flags`：位掩码，用于改变 eventfd 的行为。可为以下值的按位或组合：
  - `EFD_CLOEXEC`：设置 close-on-exec 标志，`exec` 时自动关闭该文件描述符（Linux 2.6.27+）[reference:5]。
  - `EFD_NONBLOCK`：设置非阻塞模式，读写操作不会阻塞（Linux 2.6.27+）[reference:6]。
  - `EFD_SEMAPHORE`：提供信号量语义，`read` 时计数器减 1 而非清零（Linux 2.6.30+）[reference:7]。

**返回值：** 成功返回非负文件描述符，失败返回 -1 并设置 `errno`。

**示例用法：**
```c
#include <sys/eventfd.h>
#include <unistd.h>
#include <stdio.h>
#include <errno.h>

int main() {
    // 创建 eventfd，初始计数器为 0，非阻塞模式
    int efd = eventfd(0, EFD_NONBLOCK);
    if (efd == -1) {
        perror("eventfd");
        return 1;
    }
    close(efd);
    return 0;
}
```

**实现原理：**
系统调用进入内核，执行 `do_eventfd()` 函数，分配 `eventfd_ctx` 结构体（包含计数器、等待队列和标志位），通过匿名 inode 文件系统创建文件对象，关联 `eventfd_fops` 操作集，返回文件描述符[reference:8][reference:9]。

**线程安全提示：**
线程安全。多个线程可以同时在不同 eventfd 上操作，或对同一 eventfd 并发读写时内核保证原子性，但建议使用锁或其他同步机制避免竞争。

---

### 2. 通过 `read()` / `write()` 读写 eventfd

eventfd 返回的文件描述符支持标准的 `read()` 和 `write()` 系统调用，每次操作需要 8 字节（64 位）缓冲区[reference:10]。

**读操作 (`read`)：**

**作用：** 读取并消费 eventfd 计数器中的值，根据是否设置 `EFD_SEMAPHORE` 标志行为不同[reference:11]：
- **未设置 `EFD_SEMAPHORE`**：若计数器非零，返回计数器的当前值，并将计数器重置为 0。
- **设置了 `EFD_SEMAPHORE`**：若计数器非零，返回 1，并将计数器减 1（信号量语义）。
- **计数器为 0**：阻塞（默认）直到计数器变为非零；若设置了 `EFD_NONBLOCK` 则立即返回 `EAGAIN`。

**写操作 (`write`)：**

**作用：** 向 eventfd 计数器添加指定的 64 位值。计数器的最大值不能超过 `0xfffffffffffffffe`（即 64 位无符号整数最大值减 1），否则写操作会阻塞（默认）或返回 `EAGAIN`（非阻塞模式）[reference:12]。

**示例用法（线程间通知）：**
```c
#include <sys/eventfd.h>
#include <pthread.h>
#include <unistd.h>
#include <stdint.h>
#include <stdio.h>

int efd;

void *reader(void *arg) {
    uint64_t val;
    // 阻塞等待事件，计数器被清零
    ssize_t ret = read(efd, &val, sizeof(val));
    if (ret == sizeof(val)) {
        printf("reader: got %lu\n", val);
    }
    return NULL;
}

int main() {
    efd = eventfd(0, 0);
    if (efd == -1) return 1;

    pthread_t tid;
    pthread_create(&tid, NULL, reader, NULL);

    // 通知 reader 线程
    uint64_t val = 1;
    write(efd, &val, sizeof(val));

    pthread_join(tid, NULL);
    close(efd);
    return 0;
}
```

**实现原理：**
写操作将值累加到计数器并唤醒等待队列中的读进程；读操作原子地读取并修改计数器值，计数器为 0 时进程加入等待队列[reference:13]。

**线程安全提示：**
线程安全，内核保证原子性。但多个线程对同一 eventfd 并发读写时需注意数据一致性，建议使用锁。

---

### 3. eventfd_read / eventfd_write（GNU 扩展）

**函数原型：**
```c
int eventfd_read(int fd, eventfd_t *value);
int eventfd_write(int fd, eventfd_t value);
```

**作用：** 对 eventfd 文件描述符进行读取和写入的封装函数，简化 `read()` / `write()` 调用[reference:14]。

**参数：**
- `fd`：eventfd 文件描述符。
- `value`：指向 `eventfd_t` 的指针（读取）或要写入的值（写入）。

**返回值：**
- `eventfd_read`：成功返回 0，失败返回 -1。
- `eventfd_write`：成功返回 0，失败返回 -1。

**示例用法：**
```c
#include <sys/eventfd.h>

eventfd_t val;
// 读取事件
if (eventfd_read(efd, &val) == -1) {
    perror("eventfd_read");
}
// 写入事件
val = 1;
if (eventfd_write(efd, val) == -1) {
    perror("eventfd_write");
}
```

**线程安全提示：**
与 `read()` / `write()` 相同，线程安全。

---

## 二、结构体详解

### 1. eventfd_t

**定义：**
```c
typedef uint64_t eventfd_t;
```

**作用：** 表示 eventfd 计数器的数据类型，是一个 64 位无符号整数。该类型在大多数系统中被定义为无符号 64 位整型数[reference:15]。

### 2. eventfd_ctx（内核结构体）

**定义：**（内核内部使用，用户空间不可见）
```c
struct eventfd_ctx {
    uint64_t count;        // 64 位计数器
    wait_queue_head_t wqh; // 等待队列头
    int flags;             // 标志位（EFD_SEMAPHORE 等）
};
```

**作用：** 内核中维护 eventfd 状态的核心结构体，管理计数器和等待队列[reference:16]。

---

## 三、宏定义详解

| 宏名 | 作用 |
|------|------|
| `EFD_CLOEXEC` | 设置 close-on-exec 标志，`exec` 时自动关闭文件描述符[reference:17] |
| `EFD_NONBLOCK` | 设置非阻塞模式，读写操作不会阻塞[reference:18] |
| `EFD_SEMAPHORE` | 提供信号量语义，`read` 时计数器减 1 而非清零[reference:19] |

---

## 四、类型定义

| 类型 | 说明 |
|------|------|
| `eventfd_t` | `uint64_t` 的别名，表示 eventfd 计数器值 |
| `pid_t` | 进程 ID 类型（来自 `<sys/types.h>`） |

---

## 五、模板声明

`<sys/eventfd.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---
