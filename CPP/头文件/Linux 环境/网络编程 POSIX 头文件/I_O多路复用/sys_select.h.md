## `<sys/select.h>` 头文件详解（Linux / POSIX）

`<sys/select.h>` 是 POSIX 标准定义的 I/O 多路复用头文件，提供了 `select()` 和 `pselect()` 函数，用于同时监视多个文件描述符的可读、可写或异常事件。与 `poll()` 相比，`select()` 使用 `fd_set` 位图来管理文件描述符，具有更好的跨平台兼容性，但受限于 `FD_SETSIZE` 上限（通常为 1024）。

---

## 一、函数详解

### 1. select 函数

**函数原型：**
```c
int select(int nfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *timeout);
```

**作用：** 监视一组文件描述符，等待其中一个或多个就绪（可读、可写或异常）。

**参数：**
- `nfds`：需要检查的最大文件描述符值 + 1（通常为最大 fd + 1）。
- `readfds`：指向可读事件集合的指针（输入输出参数），传入要监视的可读描述符，返回时保留已就绪的描述符。
- `writefds`：指向可写事件集合的指针，类似。
- `exceptfds`：指向异常事件集合的指针（通常用于带外数据）。
- `timeout`：等待时间上限，若为 `NULL` 则无限等待；若为 `0` 则立即返回（轮询）。

**返回值：**
- 成功：返回就绪的文件描述符总数（即三个集合中被置位的总位数）。
- 超时：返回 0。
- 失败：返回 -1，并设置 `errno`（如 `EINTR` 被信号中断）。

**示例用法：**
```c
#include <sys/select.h>
#include <stdio.h>
#include <unistd.h>

int main() {
    fd_set readfds;
    FD_ZERO(&readfds);
    FD_SET(STDIN_FILENO, &readfds);
    struct timeval tv = {5, 0};  // 等待 5 秒
    int ret = select(STDIN_FILENO + 1, &readfds, NULL, NULL, &tv);
    if (ret > 0 && FD_ISSET(STDIN_FILENO, &readfds)) {
        printf("stdin readable\n");
    } else if (ret == 0) {
        printf("Timeout\n");
    } else {
        perror("select");
    }
    return 0;
}
```

**实现原理：**
1. 系统调用进入内核，将用户空间的 fd_set 拷贝到内核。
2. 内核遍历所有被监视的文件描述符，检查其对应的事件是否发生。
3. 若没有任何描述符就绪且 `timeout` 未到，则进程进入可中断睡眠状态（`TASK_INTERRUPTIBLE`），直到有事件发生或超时。
4. 当事件发生时，内核唤醒进程，重新检查描述符状态，将就绪的描述符在 fd_set 中置位，未就绪的清除。
5. 将结果拷贝回用户空间并返回就绪总数。

**线程安全提示：**
`select()` 是线程安全的。但多个线程同时调用 `select` 监视同一组描述符时，可能导致结果互相干扰（如一个线程修改 fd_set 影响另一个）。通常每个线程应使用独立的 fd_set 副本，或确保同步。

---

### 2. pselect 函数

**函数原型：**
```c
int pselect(int nfds, fd_set *readfds, fd_set *writefds,
            fd_set *exceptfds, const struct timespec *timeout,
            const sigset_t *sigmask);
```

**作用：** `select()` 的增强版本，允许指定信号掩码，以避免信号处理与 `select` 之间的竞态条件。

**参数：**
- 前四个参数与 `select` 相同。
- `timeout`：指向 `struct timespec` 的指针（纳秒精度）。
- `sigmask`：指向信号掩码的指针，在 `pselect` 执行期间会临时替换当前线程的信号掩码，调用结束后恢复。若为 `NULL`，则等同于 `select`（但时间精度更高）。

**返回值：** 同 `select`。

**示例用法：**
```c
#include <sys/select.h>
#include <signal.h>
#include <time.h>

int main() {
    fd_set readfds;
    FD_ZERO(&readfds);
    FD_SET(STDIN_FILENO, &readfds);
    sigset_t newmask;
    sigemptyset(&newmask);
    sigaddset(&newmask, SIGINT);
    struct timespec ts = {5, 0};  // 5 秒
    int ret = pselect(STDIN_FILENO + 1, &readfds, NULL, NULL, &ts, &newmask);
    // ...
}
```

**实现原理：**
`pselect()` 是 `select()` 的变体，系统调用实现中会原子性地设置信号掩码、执行监视、再恢复掩码，避免了信号在 `select` 等待期间递送而无法及时处理的问题。

**线程安全提示：**
线程安全，但 `sigmask` 仅影响调用线程的信号掩码。

---

## 二、结构体详解

### 1. fd_set

**定义：**
```c
typedef struct {
    unsigned long fds_bits[FD_SETSIZE / (8 * sizeof(unsigned long))];
} fd_set;
```

**作用：** 表示一组文件描述符的位图，用于 `select()` 和 `pselect()` 中指定待监视的描述符。

**成员详解：**
- `fds_bits`：位图数组，每个位对应一个文件描述符（位索引即 fd 值）。
- 通常不直接操作该结构体，而是使用以下宏来操作：`FD_ZERO`、`FD_SET`、`FD_CLR`、`FD_ISSET`。

**限制：** `FD_SETSIZE` 通常定义为 1024，即最多监视 1024 个文件描述符。如需监视更多，应使用 `poll` 或 `epoll`。

**使用注意事项：**
- 在每次调用 `select` 前，必须重新初始化 `fd_set` 并设置需要监视的描述符，因为 `select` 会修改集合。
- 不能直接复制 `fd_set` 结构体（因为内部是位数组），但可以通过逐位拷贝或使用 `memcpy` 来实现，不过通常推荐重新设置。

---

### 2. struct timeval

**定义：**
```c
struct timeval {
    time_t      tv_sec;   // 秒
    suseconds_t tv_usec;  // 微秒
};
```

**作用：** 用于 `select()` 的超时参数，表示等待时间（秒 + 微秒）。

**成员详解：**
- `tv_sec`：秒数。
- `tv_usec`：微秒数（0 到 999999）。

**使用注意事项：**
- 如果 `tv_sec == 0` 且 `tv_usec == 0`，`select` 立即返回（轮询）。
- 如果 `timeout` 为 `NULL`，则无限等待。
- `select` 返回后，`timeout` 的值未定义（某些实现会修改为剩余时间），因此每次调用前必须重新设置。

---

### 3. struct timespec

**定义：**
```c
struct timespec {
    time_t tv_sec;   // 秒
    long   tv_nsec;  // 纳秒
};
```

**作用：** 用于 `pselect()` 的超时参数，提供更高精度。

**成员详解：**
- `tv_sec`：秒数。
- `tv_nsec`：纳秒数（0 到 999999999）。

---

## 三、宏定义详解

### 1. fd_set 操作宏

| 宏名称 | 作用 |
|--------|------|
| `FD_ZERO(fd_set *set)` | 清空集合，将所有位清零 |
| `FD_SET(int fd, fd_set *set)` | 将文件描述符 `fd` 加入集合（对应位置位） |
| `FD_CLR(int fd, fd_set *set)` | 将文件描述符 `fd` 从集合中移除（对应位清零） |
| `FD_ISSET(int fd, fd_set *set)` | 测试 `fd` 是否在集合中（返回非零表示在集合中） |

**示例用法：**
```c
fd_set readfds;
FD_ZERO(&readfds);
FD_SET(sockfd, &readfds);
if (FD_ISSET(sockfd, &readfds)) { /* 就绪 */ }
FD_CLR(sockfd, &readfds);
```

**实现原理：**
这些宏直接操作 `fd_set` 内部的位数组。例如：
```c
#define FD_SET(fd, set)  ((set)->fds_bits[(fd) / NFDBITS] |= (1UL << ((fd) % NFDBITS)))
```
其中 `NFDBITS` 为 `8 * sizeof(unsigned long)`。

---

### 2. 相关常量

| 宏名称 | 作用 |
|--------|------|
| `FD_SETSIZE` | `fd_set` 能支持的最大文件描述符数（通常为 1024） |
| `NFDBITS` | 每个位图元素包含的位数（通常为 `8 * sizeof(unsigned long)`） |

---

### 3. 错误码（与 select 相关，定义在 `<errno.h>`）

| 宏名称 | 作用 |
|--------|------|
| `EINTR` | 信号中断 |
| `EBADF` | 集合中包含无效文件描述符 |
| `EINVAL` | `nfds` 为负或超出 `FD_SETSIZE` |
| `ENOMEM` | 内存不足 |

---

## 四、类型定义

- `fd_set`：文件描述符集类型。
- `time_t`：时间类型（秒数）。
- `suseconds_t`：有符号微秒类型。
- `struct timeval`、`struct timespec` 结构体。

---

## 五、模板声明

`<sys/select.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---

