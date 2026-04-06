## `<poll.h>` 头文件详解（Linux / POSIX）

`<poll.h>` 是 POSIX 标准定义的头文件，提供了 `poll()` 和 `ppoll()` 函数，用于 I/O 多路复用。它允许进程监视多个文件描述符，等待其中任何一个就绪（可读、可写或异常）。与 `select()` 相比，`poll()` 没有文件描述符数量上限（受系统资源限制），且使用更灵活的事件掩码。

---

## 一、函数详解

### 1. poll 函数

**函数原型：**
```c
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

**作用：** 监视一组文件描述符，等待其中一个或多个就绪（可读、可写或异常）。

**参数：**
- `fds`：指向 `struct pollfd` 数组的指针，每个元素描述一个待监视的文件描述符及感兴趣的事件。
- `nfds`：数组中的元素个数（即监视的描述符数量）。
- `timeout`：超时时间（毫秒）。
  - `-1`：无限等待。
  - `0`：立即返回（轮询）。
  - `>0`：等待指定毫秒数。

**返回值：**
- 成功：返回就绪的文件描述符个数（即 `revents` 非零的条目数）。
- 超时：返回 0。
- 失败：返回 -1，并设置 `errno`（如 `EINTR` 被信号中断）。

**示例用法：**
```c
#include <poll.h>
#include <stdio.h>
#include <unistd.h>

int main() {
    struct pollfd fds[2];
    fds[0].fd = 0;          // 标准输入
    fds[0].events = POLLIN;
    fds[1].fd = 1;          // 标准输出
    fds[1].events = POLLOUT;

    int ret = poll(fds, 2, 5000);  // 等待 5 秒
    if (ret > 0) {
        if (fds[0].revents & POLLIN) printf("stdin readable\n");
        if (fds[1].revents & POLLOUT) printf("stdout writable\n");
    } else if (ret == 0) {
        printf("Timeout\n");
    } else {
        perror("poll");
    }
    return 0;
}
```

**实现原理：**
1. 系统调用进入内核，将用户空间的 `pollfd` 数组拷贝到内核。
2. 内核遍历所有被监视的文件描述符，调用对应文件系统或协议栈的 `poll` 方法，检查是否有感兴趣的事件发生。
3. 若没有任何描述符就绪且 `timeout` 未到，则进程进入可中断睡眠状态（`TASK_INTERRUPTIBLE`）。
4. 当事件发生或超时到达时，内核唤醒进程，重新检查描述符状态，将实际发生的事件写入 `revents`。
5. 将结果拷贝回用户空间并返回就绪数量。

**线程安全提示：**
`poll()` 是线程安全的。多个线程同时调用 `poll` 监视同一组描述符时，可能导致结果互相干扰（如一个线程修改 `fds` 影响另一个）。通常每个线程应使用独立的 `pollfd` 数组，或确保同步。

---

### 2. ppoll 函数

**函数原型：**
```c
int ppoll(struct pollfd *fds, nfds_t nfds,
          const struct timespec *timeout_ts,
          const sigset_t *sigmask);
```

**作用：** `poll()` 的增强版本，允许指定信号掩码，以避免信号处理与 `poll` 之间的竞态条件。

**参数：**
- `fds`、`nfds`：同 `poll`。
- `timeout_ts`：指向 `struct timespec` 的指针（纳秒精度），`NULL` 表示无限等待。
- `sigmask`：指向信号掩码的指针，在 `ppoll` 执行期间会临时替换当前线程的信号掩码，调用结束后恢复。若为 `NULL`，则等同于 `poll`（但时间精度为纳秒）。

**返回值：** 同 `poll`。

**示例用法：**
```c
#include <poll.h>
#include <signal.h>
#include <time.h>

int main() {
    struct pollfd fds;
    fds.fd = 0;
    fds.events = POLLIN;
    sigset_t newmask;
    sigemptyset(&newmask);
    sigaddset(&newmask, SIGINT);
    struct timespec ts = {5, 0};  // 5 秒
    int ret = ppoll(&fds, 1, &ts, &newmask);
    // ...
}
```

**实现原理：**
`ppoll()` 是 `poll()` 的变体，系统调用实现中会原子性地设置信号掩码、执行监视、再恢复掩码，避免了信号在 `poll` 等待期间递送而无法及时处理的问题。

**线程安全提示：**
线程安全，但 `sigmask` 仅影响调用线程的信号掩码。

---

## 二、结构体详解

### struct pollfd

**定义：**
```c
struct pollfd {
    int   fd;       // 文件描述符
    short events;   // 感兴趣的事件（输入）
    short revents;  // 实际发生的事件（输出）
};
```

**作用：** 描述一个待监视的文件描述符及其事件。

**成员详解：**
- `fd`：要监视的文件描述符。若为负值，则忽略该条目，`revents` 返回 0。
- `events`：输入位掩码，指定调用者感兴趣的事件（如 `POLLIN`、`POLLOUT`）。
- `revents`：输出位掩码，由内核填充，表示实际发生的事件（可能是 `events` 中的子集，也可能是错误或挂起事件如 `POLLERR`、`POLLHUP`）。

**使用注意事项：**
- 每次调用 `poll` 前需重新设置 `events`（因为 `revents` 会被覆盖）。
- 可以监视的 `fd` 数量受系统资源限制（如 `RLIMIT_NOFILE`），无固定上限（与 `select` 不同）。

---

## 三、宏定义详解

### 1. 事件掩码（用于 events 和 revents）

| 宏名称 | 作用 |
|--------|------|
| `POLLIN` | 有数据可读（普通或优先级数据） |
| `POLLRDNORM` | 有普通数据可读 |
| `POLLRDBAND` | 有优先级数据可读（某些实现中与 `POLLIN` 同义） |
| `POLLPRI` | 有高优先级数据可读（如带外数据） |
| `POLLOUT` | 可写（普通数据可写） |
| `POLLWRNORM` | 同 `POLLOUT`（普通数据可写） |
| `POLLWRBAND` | 可写优先级数据 |
| `POLLERR` | 发生错误（输出事件，无需在 `events` 中设置） |
| `POLLHUP` | 挂起（对端关闭连接，输出事件） |
| `POLLNVAL` | 无效文件描述符（未打开，输出事件） |

**注意：**
- `POLLIN` 通常等价于 `POLLRDNORM | POLLRDBAND`。
- 对于普通 TCP 套接字，连接关闭时 `POLLHUP` 会被设置；若同时有数据等待，`POLLIN` 可能先触发，然后下一次 `poll` 再触发 `POLLHUP`。
- 对于 UDP 套接字，`POLLIN` 表示有数据报可读。
- 错误和挂起事件总是会返回，即使没有在 `events` 中显式请求。

---

### 2. 超时相关常量

| 宏名称 | 作用 |
|--------|------|
| `INFTIM` | 过时宏，值为 -1，表示无限等待（POSIX 未定义，某些旧系统使用）。现代代码应使用 `-1`。 |

---

## 四、类型定义

- `nfds_t`：无符号整数类型，用于表示 `pollfd` 数组的元素个数（通常为 `unsigned int`）。

---

## 五、模板声明

`<poll.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---
