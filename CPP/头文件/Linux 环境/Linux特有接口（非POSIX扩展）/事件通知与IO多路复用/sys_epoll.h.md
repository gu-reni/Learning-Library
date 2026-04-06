## `<sys/epoll.h>` 头文件详解（Linux 特有接口）

`<sys/epoll.h>` 是 Linux 系统特有的头文件，提供了 **epoll** I/O 多路复用机制的函数和数据结构。epoll 能够高效地监控大量文件描述符，适用于高并发网络服务器。相比于 `select()` 和 `poll()`，epoll 具有以下优势：
- 无文件描述符数量上限（仅受系统资源限制）；
- 边缘触发（ET）和水平触发（LT）两种模式；
- 通过事件驱动，避免每次调用都重新扫描所有描述符。

---

## 一、函数详解

### 1. epoll_create / epoll_create1

**函数原型：**
```c
int epoll_create(int size);
int epoll_create1(int flags);
```

**作用：** 创建一个 epoll 实例，返回一个文件描述符（epoll fd），后续操作通过该描述符进行。

**参数：**
- `size`：旧版参数，用于提示内核内部数据结构大小（Linux 2.6.8 后忽略，但必须 > 0）。
- `flags`：可设为 `0` 或 `EPOLL_CLOEXEC`（在 exec 时自动关闭）。

**返回值：** 成功返回 epoll 文件描述符，失败返回 -1。

**示例用法：**
```c
int epfd = epoll_create1(0);
if (epfd == -1) perror("epoll_create1");
```

**实现原理：** 内核创建一个匿名 inode 和 eventpoll 结构体，并分配文件描述符。epoll 实例通过红黑树管理注册的描述符，通过就绪队列存储活跃事件。

**线程安全提示：** 是线程安全的。多个线程可同时创建不同的 epoll 实例。

---

### 2. epoll_ctl

**函数原型：**
```c
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
```

**作用：** 控制 epoll 实例上的文件描述符，进行添加、修改或删除操作。

**参数：**
- `epfd`：epoll 实例的文件描述符。
- `op`：操作类型，可取 `EPOLL_CTL_ADD`（添加）、`EPOLL_CTL_MOD`（修改）、`EPOLL_CTL_DEL`（删除）。
- `fd`：要操作的目标文件描述符。
- `event`：指向 `struct epoll_event` 的指针，描述感兴趣的事件和数据（删除时可为 NULL）。

**返回值：** 成功返回 0，失败返回 -1。

**示例用法：**
```c
struct epoll_event ev;
ev.events = EPOLLIN | EPOLLET;   // 边缘触发模式，监听可读
ev.data.fd = sockfd;
if (epoll_ctl(epfd, EPOLL_CTL_ADD, sockfd, &ev) == -1) perror("epoll_ctl");
```

**实现原理：** 内核根据 `op` 在 epoll 实例的红黑树上插入、修改或删除节点，并关联相应的回调函数（当 `fd` 上事件发生时，回调将事件添加到就绪队列）。

**线程安全提示：** 是线程安全的。对同一个 epoll 实例的并发 `epoll_ctl` 操作由内核同步保护。

---

### 3. epoll_wait / epoll_pwait / epoll_pwait2

**函数原型：**
```c
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
int epoll_pwait(int epfd, struct epoll_event *events, int maxevents, int timeout, const sigset_t *sigmask);
int epoll_pwait2(int epfd, struct epoll_event *events, int maxevents, const struct timespec *timeout, const sigset_t *sigmask);
```

**作用：** 等待 epoll 实例上的事件，将就绪的事件填充到 `events` 数组中。

**参数：**
- `epfd`：epoll 实例的文件描述符。
- `events`：输出数组，用于接收发生的事件。
- `maxevents`：`events` 数组的大小（最多返回的事件数）。
- `timeout`：超时时间（毫秒）。`-1` 表示无限等待，`0` 表示立即返回。
- `sigmask`：在 `epoll_pwait` 系列中，指定临时信号掩码（原子操作）。
- `timeout`（pwait2）：`struct timespec`，纳秒精度。

**返回值：** 成功返回就绪事件个数，0 表示超时，-1 表示错误。

**示例用法：**
```c
#define MAX_EVENTS 10
struct epoll_event events[MAX_EVENTS];
int nfds = epoll_wait(epfd, events, MAX_EVENTS, -1);
if (nfds == -1) perror("epoll_wait");
for (int i = 0; i < nfds; i++) {
    if (events[i].events & EPOLLIN) {
        // 处理可读事件
    }
}
```

**实现原理：** 内核检查就绪队列。若无事件且 `timeout != 0`，当前进程进入可中断睡眠，直到有事件到达或超时。事件发生时，回调将事件放入就绪队列并唤醒等待进程。

**线程安全提示：** 是线程安全的。多个线程可以同时等待同一个 epoll 实例，但每个事件只会被一个线程返回（内核保证）。

---

## 二、结构体详解

### 1. struct epoll_event

**定义：**
```c
struct epoll_event {
    uint32_t     events;   // 感兴趣的事件掩码
    epoll_data_t data;     // 用户数据（联合体）
};

typedef union epoll_data {
    void    *ptr;
    int      fd;
    uint32_t u32;
    uint64_t u64;
} epoll_data_t;
```

**作用：** 描述 epoll 事件，用于 `epoll_ctl` 和 `epoll_wait`。

**成员详解：**
- `events`：位掩码，可取以下值：
  - `EPOLLIN`：可读事件（包括普通数据和优先级数据）。
  - `EPOLLOUT`：可写事件。
  - `EPOLLRDHUP`：对端关闭连接或半关闭（Linux 2.6.17+）。
  - `EPOLLPRI`：带外数据可读。
  - `EPOLLERR`：发生错误（无需显式注册，总是监听）。
  - `EPOLLHUP`：挂起（无需显式注册）。
  - `EPOLLET`：设置为边缘触发（Edge Triggered）模式，默认为水平触发（Level Triggered）。
  - `EPOLLONESHOT`：一次性触发，事件发生后自动删除，需再次注册。
  - `EPOLLWAKEUP`：确保系统在事件处理期间不进入休眠（用于唤醒锁定）。
  - `EPOLLEXCLUSIVE`：避免惊群效应，常用于多线程 accept。
- `data`：联合体，用户可存储自定义数据（如文件描述符、指针等）。

---

## 三、宏定义详解

| 宏名 | 作用 |
|------|------|
| `EPOLL_CLOEXEC` | `epoll_create1` 的标志，设置 `close-on-exec`。 |
| `EPOLL_CTL_ADD` | `epoll_ctl` 操作：添加文件描述符到 epoll 实例。 |
| `EPOLL_CTL_MOD` | 修改已注册文件描述符的事件。 |
| `EPOLL_CTL_DEL` | 从 epoll 实例中删除文件描述符。 |
| `EPOLLIN` | 可读事件。 |
| `EPOLLOUT` | 可写事件。 |
| `EPOLLRDHUP` | 对端关闭连接（TCP 的 FIN）。 |
| `EPOLLPRI` | 带外数据可读。 |
| `EPOLLERR` | 错误事件。 |
| `EPOLLHUP` | 挂起事件。 |
| `EPOLLET` | 边缘触发模式。 |
| `EPOLLONESHOT` | 一次性事件。 |
| `EPOLLWAKEUP` | 防止系统睡眠。 |
| `EPOLLEXCLUSIVE` | 避免惊群效应。 |

---

## 四、类型定义

- `epoll_data_t`：联合体，用于存储用户数据。
- `struct epoll_event`：事件结构体。

---

## 五、模板声明

`<sys/epoll.h>` 是纯 C 头文件，不包含 C++ 模板。在 C++ 环境中使用时，函数接口保持不变。

---
